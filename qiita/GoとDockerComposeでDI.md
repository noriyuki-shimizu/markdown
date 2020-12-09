# Go と Docker Compose で DI してみる

この記事は[ぷりぷりあぷりけーしょんず Advent Calendar 2020](https://qiita.com/advent-calendar/2020/ppap)の9日目の記事です。

## はじめに
今年学んだことのアウトプット第二弾です。
本記事に書く内容に関しては、結構前に実装した際のお話になります。
また、個人開発ではなくペアプロ（@MSHR-Dec さんとの共同開発）した際のものとなります。とても楽しかった開発でした😊
結構月日が立ってしまった為、忘れないうちに記事に落とし込もうと思います。

## 本記事で話さないこと
* Go のセットアップ
* Go の書き方
* Docker のセットアップ
* 今回作成したプロジェクト詳細
* 個人的ニュース

## DI とは
DI （dependency injection） はよく耳にした方が多いかと思われます。
調べると「 **オブジェクトの注入** 」とのような直訳した意味が多く見つかります。
こちらをもう少し噛み砕いて説明すると、オブジェクト指向プログラミングにおける開発手法の１つであり、インスタンスの生成や管理を行ってくれます。
オブジェクト指向な言語のフレームワークでは結構 DI が採用されています。

## Go の DI ライブラリ wire
こちらの本題に入る前に、そもそもなぜ DI をしたかったのかを簡単に説明します。
本プロジェクトでは Clean architecture を採用しており（完璧な実装ではありませんが）、依存関係逆転の法則を守るために interface を用いて型を定義し、起動のタイミングでオブジェクトの注入を行いたかった。とか、各レイヤーで単体テストを書きたかったためモックと実際のロジックをいい感じに差し込みたかった。など、interface の定義が多く、起動のタイミングやテスト実行のタイミングでインスタンスの生成をいい感じにしたかったため、DI ライブラリを導入しようとなったのが経緯となります。

### wire とは
Google 製の DI ライブラリとなっており、2018 年 12 月に公開されました。
特別なビルドタグをつけた Go のコードをデータソースとして、コンパイルタイムでインジェクタのコードを生成してくれるみたいです。

wire 取得の説明の前に、本実装での Go のバージョンは以下となります。

```
$ go version
go version go1.13 darwin/amd64
```

wire は以下のコマンドで取得できます。

```
$ go get github.com/google/wire/cmd/wire
```

get が完了したら wire コマンドが使用できるかと思われます。

```
$ wire help
Usage: wire <flags> <subcommand> <subcommand args>

Subcommands:
        check            print any Wire errors found
        commands         list all command names
        diff             output a diff between existing wire_gen.go files and what gen would generate
        flags            describe all known top-level flags
        gen              generate the wire_gen.go file for each package
        help             describe subcommands and their syntax
        show             describe all top-level provider sets


Use "wire flags" for a list of top-level flags
```

### 実際に使ってみる
今回は repository 周りを例に見ていきたいと思います。
まずは interface の定義から

```golang
package repository

import (
	"api/src/domain/model"
)

type IArticleRepository interface {
	Create(article *model.Article)
	Update(article *model.Article)
	FindAll() []model.Article
	FindOne(ID uint64) model.Article
}
```

次に、その interface を実装した struct を定義します。
ちなみに、本プロジェクトは ORM を使用しており、[Gorm](https://gorm.io/ja_JP/docs/index.html) ライブラリを使用しています。RDBMS は MySQL を使用しています。

```golang
package datastore

import (
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
	"github.com/thoas/go-funk"

	"api/src/domain/model"
	"api/src/domain/repository"
)

type ArticleDatastore struct {
	db *gorm.DB
}

func NewArticleDatastore(d *gorm.DB) repository.IArticleRepository {
	return &ArticleDatastore{
		db: d,
	}
}

func (a *ArticleDatastore) Create(article *model.Article) {
	a.db.Create(&article)
}

func (a *ArticleDatastore) Update(article *model.Article)  {
	a.db.Save(&article)
}

func (a *ArticleDatastore) FindAll() []model.Article {
	var articles []model.Article
	a.db.Select("id").Find(&articles)

	results := funk.Map(articles, func(article model.Article) model.Article {
		a.db.First(&article).Related(&article.User, "User").Related(&article.Shop, "Shop").Related(&article.Categories, "Categories")
		return article
	}).([]model.Article)

	return results
}

func (a *ArticleDatastore) FindOne(ID uint64) model.Article {
	article := model.Article{ID: ID}
	a.db.First(&article).Related(&article.User, "User").Related(&article.Shop, "Shop").Related(&article.Categories, "Categories")
	return article
}
```

ここまで定義されたオブジェクトを使用する処理が以下となります。

```golang
package interactor

import (
	"time"
	"api/src/domain/model"
	"api/src/domain/repository"
	"api/src/handler/request"
	"api/src/usecase"
)

type ArticleInteractor struct {
	Repository repository.IArticleRepository
}

func NewArticleInteractor(
	repository repository.IArticleRepository,
) usecase.IArticleUsecase {
	return &ArticleInteractor{
		Repository: repository,
	}
}

func (a *ArticleInteractor) Create(ra *request.UpsertArticleRequest) uint64 {
	article := model.Article{
		Title:    ra.Title,
		Body:     ra.Body,
		Status:   ra.Status,
		UserID:   ra.UserID,
		ShopID:   ra.ShopID,
		CreateAt: time.Now(),
	}

	a.Repository.Create(&article)

	return article.ID
}

func (a *ArticleInteractor) Update(id uint64, ra *request.UpsertArticleRequest) {
	article := model.Article{
		ID:       id,
		Title:    ra.Title,
		Body:     ra.Body,
		Status:   ra.Status,
		UserID:   ra.UserID,
		ShopID:   ra.ShopID,
		CreateAt: time.Now(),
	}

	a.Repository.Update(&article)
}

func (a *ArticleInteractor) GetAll() []model.Article {
	return a.Repository.FindAll()
}

func (a *ArticleInteractor) GetOne(ID uint64) model.Article {
	return a.Repository.FindOne(ID)
}
```

本当は↑のオブジェクトも interface が定義されているのですが、今回は１つだけをサンプルとします。
最後に DI を定義していくのですが、ソースの先頭にビルドタグをつけるのを忘れないように気をつけましょう！
また、main パッケージの層にソースを配置しないとうまくインジェクタのコードを生成（wire generate）できませんでした。（ほんとは registory パッケージとかの中に入れたかったのですが）

```golang:wire.go
//+build wireinject

package main

import (
	"github.com/google/wire"
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"

	"api/src/infrastructure/datastore"
	"api/src/interactor"
)

func InitArticleInteractor(d *gorm.DB) *interactor.ArticleInteractor {
	wire.Build(
		datastore.NewArticleDatastore,
		interactor.NewArticleInteractor,
	)
	return nil
}
```

こちらの定義だけだと、正直不安だらけでした笑
このソースでコンストラクタ引数にちゃんと必要なもの入れてくれるの？とか色々疑いまくりましたが、こちらのソースで実際の DI を generate したら以下のソースが吐き出されました。

```
$ wire src/wire.go
```

```golang:wire_gen.go
// Code generated by Wire. DO NOT EDIT.

//go:generate wire
//+build !wireinject

package main

import (
	"github.com/jinzhu/gorm"
	"api/src/infrastructure/datastore"
	"api/src/interactor"
)

import (
	_ "github.com/jinzhu/gorm/dialects/mysql"
)

// Injectors from wire.go:

func InitArticleInteractor(d *gorm.DB) *interactor.ArticleInteractor {
	iArticleRepository := datastore.NewArticleDatastore(d)
	articleInteractor := interactor.NewArticleInteractor(iArticleRepository)
	return articleInteractor
}
```

こちらのソースが吐き出された時は感動しました。
勝手にコンストラクタ引数を詰めてくれて、`return nil` と記述していたのに、しっかり該当のインスタンスを返してくれているではありませんか！
generate してみて感動はしましたが、`wire.go` の書き方には慣れが必要そうだなという印象です。（当初はなかなか慣れることができませんでした笑）

あとはエントリーポイントである main.go で `InitArticleInteractor` を呼べば、`wire_gen.go` で定義した方のメソッドが呼ばれるため、依存関係が解決した状態のインスタンスを使用することができます。

## Docker と wire で DI を自動 Generate
本プロジェクトは Docker を使用しており、コンテナのビルド・立ち上げの際に先程の wire generate ができたらいいなと思い、やってみました。
とは言っても、以下の Dockerfile を定義する感じです。こちらは @MSHR-Dec さんにたくさん助けていただきました。

```docker:Dockerfile
FROM golang:1.13.7-alpine3.11 as build

WORKDIR /api

ENV GO111MODULE=on

COPY . .

RUN go get github.com/google/wire/cmd/wire \
  && wire src/wire.go \
  && cd src \
  && CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ../main \
  && cd ..

FROM alpine

COPY --from=build /api/main .

RUN addgroup go \
  && adduser -D -G go go \
  && chown -R go:go ./main

CMD ["./main"]
```

あとはビルドして、立ち上げれば自動 generate してくれて API が起動します。

## まとめ
アベンドカレンダー２記事目ということもあり、勢いで書き上げたため、ちょっとわかりにくい箇所が多いかと思われます。🙇‍♂️
不備などありましたらコメントどんどんください！
今回 DI ライブラリを使用してみて、慣れない箇所には時間がかかりそうでしたが、実際 generate でソースを吐き出してみると、結構な感動だったため、今後も Golang で開発をする際は、wire を用いて開発をしていこうと思います。（DI するような実装なら）

## おまけ
本記事には載せられませんでしたが、本プロジェクトではホットリロードとして [realize](https://github.com/oxequa/realize) ってのを使用しています。こちらも感動ものでした。
また Dockerfile を見ていただければわかるかもですが、[マルチステージビルド](https://matsuand.github.io/docs.docker.jp.onthefly/develop/develop-images/multistage-build/)ってのも初めて使いました。

