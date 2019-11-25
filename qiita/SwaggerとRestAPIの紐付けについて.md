## 背景

近年のアプリケーション開発において、マイクロサービスがとても流行ってますね。
現場によっては、社内サービスまでマイクロサービスにしちゃってる企業もあったりします。（モノリスなアプリでいいと思うのですが）
マイクロサービスが流行るにいたって、[RESTful API (REST API)](http://e-words.jp/w/RESTful_API.html) というのもキーワードとして多く出てきます。
その RESTful API はどのようなリソースがあり、どんな値をリクエストに含め、どのような値が返ってくるのか、認証は何を使用している？といったドキュメント（API 定義）が必要になります。（というか必須だと思ってます）
そのようなドキュメントがあることにより、フロントエンド開発チームとの連携が用意になり、実装パフォーマンスが上がります。
そこで現在、「世界標準となるかもしれない」というほどトレンドであるモック API サーバ Swagger に実際に触れて、自作の RESTful API との紐付けを docker-compose でやってみましたって内容を記載してこうと思っています。

## Swagger とは

Swagger の説明に関しては、ググればたくさんの情報が出てくるので省略しますが、ざっくり言えば、REST API を定義するためのオープンソースフレームワークです。（←）
yaml や json を用いて定義します。（今回、json ではやっていません）
個人的に curl で実際のエンドポイントを叩けるのは素敵と感じました。
yaml や json の記述方法などについては以下の記事にとてもわかりやすくまとまっています。

* [SwaggerでRESTful APIの管理を楽にする - @disc99](https://qiita.com/disc99/items/37228f5d687ad2969aa2)
* [Swaggerの記法まとめ - @rllllho](https://qiita.com/rllllho/items/53a0023b32f4c0f8eabb)

今回は docker hub に上がっている Swagger のイメージを使用し、実装していきます。
イメージは [swagger-editor](https://hub.docker.com/r/swaggerapi/swagger-editor) と [swagger-ui](https://hub.docker.com/r/swaggerapi/swagger-ui) を使用します。
swagger-editor に関しては、ウェブからでも閲覧できますね。

* [Web Swagger Editor](https://editor.swagger.io/)

## Docker Compose

Swagger や REST API などを立ち上げるため、docker-compose を使用します。
Docker の説明やインストール方法などに関しては今回は記載しません。
では、実際に先ほどの docker hub のイメージを使用し、 `docker-compose.yaml` を作成してきます。

### yaml の定義

yaml を定義する前に、実際のフォルダ構成を先に記載しておきます。
今回の記事に必要なファイルやフォルダを tree として表示しています。

```
root
├── api
│   ├── Dockerfile
│   ├── swagger
│   │   └── openapi.yaml
│   └── その他 REST API のアプリケーションソース
└── compose
    └── docker-compose.yaml
```

REST API に関しては Node + TypeScript + Express で簡単な API を作成しました。
上記のフォルダ構成を元に `docker-compose.yaml` を以下に記載します。

```yaml:docker-compose.yaml
version: '3'

services:
  # Swagger API
  swagger-editor:
    image: swaggerapi/swagger-editor
    container_name: "swagger-editor"
    ports:
      - "8001:8080"
  swagger-ui:
    image: swaggerapi/swagger-ui
    container_name: "swagger-ui"
    ports:
      - "8002:8080"
    volumes:
      - ../api/swagger/openapi.yaml:/openapi.yaml
    environment:
      SWAGGER_JSON: /openapi.yaml
  # REST API
  api:
    tty: true
    build:
      context: ../api
      dockerfile: Dockerfile
    container_name: 'api'
    volumes:
      - ../api/:/usr/app/
      - /usr/app/node_modules
    environment:
      NODE_ENV: development
    ports:
      - '3001:3001'
    command: 'npm run dev'
```

こんな感じで定義しましたが、上記以外にも Database などの定義もあります。しかし、今回の記事とは関係ないというのもあり、またその設定も含めちゃうと、逆にわかりにくくなるため、必要な箇所だけ記述しています。

### 各々の説明

* swagger-editor  

docker hub からイメージを引っ張ってきたくらいで、特別な設定は記載していません。
こちらを立ち上げると Swagger Editor がポート 8001 で立ち上がります。
![スクリーンショット 2019-11-22 22.43.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/83a74e3d-2b26-e0e9-9ed1-8d90e3edd1f1.png)

* swagger-ui

こちらも docker hub からイメージを引っ張ってきて、使用しています。
`volumes` に今回の REST API の定義ファイルを相対パスで指定します。（ `../api/swagger/openapi.yaml` ）
こちらを立ち上げると、REST API の定義を閲覧することができます。（Swagger Editor の左半分がなくなっただけ）
ポートは 8002 を使用。
![スクリーンショット 2019-11-22 22.44.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/30d262e7-4664-c970-0013-51d94532bdca.png)


* api

自作した REST API になります。
急成長中の TypeScript や Node の王道フレームワーク Express などを用いて REST API を作成しました。（まだ完成はしていませんが、１リソースだけ作成しました笑）
[npm trends](https://www.npmtrends.com/) を利用してイケイケなパッケージを導入したり、完璧ではありませんがクリーンアーキテクチャに挑戦してみたりしています。
マルチパートな言語でもないので、色々苦戦はしていますが、こちらのアプリケーションの記事を別で投稿したいとも考えています。
こちらでは、ポート 3001 を用いてアプリケーションを立ち上げています。また、こちらのアプリの定義を Swagger で記載していく感じとなります。

### 立ち上げてみよう

では実際に、docker-compose コマンドで立ち上げてみます。
compose ディレクトリ内で以下コマンドを実行。

```sh
$ docker-compose up --build --remove-orphans
```

`docker-compose ps` でプロセスを確認しましょう。

```sh
$ docker-compose ps
              Name                            Command               State               Ports
----------------------------------------------------------------------------------------------------------
api                                ./scripts/dev.sh                 Up      0.0.0.0:3001->3001/tcp
swagger-editor                     sh /usr/share/nginx/docker ...   Up      80/tcp, 0.0.0.0:8001->8080/tcp
swagger-ui                         sh /usr/share/nginx/run.sh       Up      80/tcp, 0.0.0.0:8002->8080/tcp
```

これで３つのプロジェクトが立ち上がったと思われます。



## 落とし穴

REST API 定義のために必要な API は立ち上がりました。
あとは Swagger Editor から REST API 定義を記述していくのですが、API のホストやポートを指定して Swagger から実際のエンドポイントを叩く（ `curl` ）と以下のエラーとなりました。

![スクリーンショット 2019-11-22 22.56.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/d2ef076e-4e2c-6381-489c-5b011c273ffa.png)

要は Swagger のコンテナから REST API のコンテナへアクセスできていないようです。（localhost は自分自身を指し示しているので、当然な気もします）
ここからは戦いの時間です。色々調査しました。。。
結果的に、以下の記事にすぐたどり着くことができました。流石に Swagger に関する記事はたくさん存在しますね。助かります。

* [Swagger EditorとSwagger UIとSwaggerのモックAPIサーバーをdocker-compose化してみた - @matsuda_chikara](https://qiita.com/matsuda_chikara/items/a4119a972535a4b69201)

こちらは、実際にサンプルも git 管理し、公開してくれているのでとてもわかりやすかったです。

## 解決策

**nginx** をかませばいいだけでした。
Swagger と REST API の仲介役として nginx を用意するだけで、上記の問題はすぐ解決するようです。（インフラの目線からしたら当然なのかもしれませんが）
上記の記事を参考に `docker-compose.yaml` を以下のように書き換えます。
今回は、ポート 8003 で nginx 経由して Swagger API から REST API にアクセスできるように設定しました。

```yaml:docker-compose.yaml
version: '3'

services:
  # Swagger API
  swagger-editor:
    image: swaggerapi/swagger-editor
    container_name: "swagger-editor"
    ports:
      - "8001:8080"
  swagger-ui:
    image: swaggerapi/swagger-ui
    container_name: "swagger-ui"
    ports:
      - "8002:8080"
    volumes:
      - ../api/swagger/openapi.yaml:/openapi.yaml
    environment:
      SWAGGER_JSON: /openapi.yaml
  # ===== 追加 =====
  swagger-nginx:
    image: nginx:mainline-alpine
    container_name: "swagger-nginx"
    depends_on:
      - api
    ports:
      - "8003:8081"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      swagger_link:
        aliases:
          - local.swagger.api
  # ===============
  # REST API
  api:
    tty: true
    # ===== 追加 =====
    networks:
      swagger_link:
        aliases:
          - local.swagger.apisprout
    # ===============
    build:
      context: ../api
      dockerfile: Dockerfile
    container_name: 'api'
    volumes:
      - ../api/:/usr/app/
      - /usr/app/node_modules
    environment:
      NODE_ENV: development
    ports:
      - '3001:3001'
    command: 'npm run dev'
# ===== 追加 =====
networks:
  swagger_link:
    external: true
# ===============
```

また、compose ディレクトリ以下に `nginx/default.conf` を作成します。

```conf:default.conf
server {
  listen 8081;
  server_name localhost.example.com;
  location / {
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods "POST, GET, PATCH, DELETE, PUT, OPTIONS";
    add_header Access-Control-Allow-Headers "Origin, Authorization, Accept";
    add_header Access-Control-Allow-Credentials true;
    proxy_pass "http://local.swagger.apisprout:3001";
  }
}
```

## 再度、立ち上げてみよう

ここまでの修正が完了したら、docker のイメージを削除し、また立ち上げてみます。

```sh
$ docker-compose down && docker system prune -a

$ docker-compose up --build --remove-orphans
```

ワクワクしながら立ち上がりを待っていると、速攻怒られました。

```
Creating network "compose_default" with the default driver
ERROR: Network swagger_link declared as external, but could not be found. Please create the network manually using `docker network create swagger_link` and try again.
```

`networks` 設定において、[external](http://docs.docker.jp/compose/compose-file.html#external) を `true` で設定すると宣言されたネットワーク（今回で言うと `swagger_link`）を外部から探しに行ってくれるようです。あれば、それをコンテナにマウントするようですね。
なければ、エラー文にも書いてあるように `docker network create swagger_link` を実行してくださいとのこと。

```sh
$ docker network create swagger_link
```

完了したら、再度 `docker-compose up` してみましょう。
自分はこれで正常に docker が立ち上がりました。

## 確認

ドキドキしながら、再度 Swagger からエンドポイントを叩いてみました。
正常に行きました。ふぅ。
![スクリーンショット 2019-11-22 23.38.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/aa8b88fc-ad28-4c61-f777-15c4d27c06bd.png)

最後に、最終的なフォルダ構成を以下に記載します。

```
root
├── api
│   ├── Dockerfile
│   ├── swagger
│   │   └── openapi.yaml
│   └── その他 REST API のアプリケーションソース
└── compose
    ├── docker-compose.yaml
    └── nginx
        └── default.conf
```

## まとめ

Swagger と REST API の環境を Docker comopse で作成するところまでできました。
この環境構築がベストプラクティスかは、実際の業務などで触れてはいないのでわかりませんが、個人的には結構嬉しい第一歩となりました。
あとは中身をしっかりしたものにすれば問題ないですかね。
記事に色々な方のリンクを貼らせていただきました。ほんと助かりました。ありがとうございます。

個人開発でマイクロサービスっぽいことをやりたいという方は、是非参考にしてみてください！（参考にできたら）