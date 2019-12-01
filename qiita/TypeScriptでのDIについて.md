## 背景

マイクロサービスの簡単な勉強として、 RESTful API を Node.js で作成しようと開発を始めました。
また、静的型付にしたかったため、流行りの TypeScript を採用。あと、フレームワークでは王道の Express を使用します。
せっかくなので、クリーンアーキテクチャにも挑戦したいなというのもあり、そのアーキテクチャで設計や実装を始めました。
実装を進めていく上で、いちいち `constructor` で `new` するのが嫌だったのと、 interface の実態がなんなのかを１つのファイルで完結させたかった（まさに DI Container）というのがあり、なんかいいのがないのか調べたところ、 `InversifyJS` となるものを発見。
これはなかなかいいなと感じたため、記事にしてみようかと思った次第どす。

## InversifyJS とは

TypeScript での強力で軽量の DI コンテナです。

* [git InversifyJS](https://github.com/inversify/InversifyJS)

### セットアップ

Node.js でのプロジェクトが作成されている前提で話を進めていきます。
TypeScript の開発環境構築に関しては、以下のコマンドを実行するだけで出来上がるかと思われます。
ただ、Node や npm などのインストール方法に関しては省略します。あと、npx も使える前提で進めます。

```sh
$ mkdir <project name>
$ cd <project name>
$ npm init -y
$ npm i -D typescript ts-node
$ npx tsc --init
```

ここまでで、TypeScript の環境はできました。（他にもやることは色々ありますが、、）
注意点としては、InversifyJS は TypeScript のバージョン **2.0 以上** をサポートしているため、インストールの際はバージョンに気をつけてください。
それでは、InversifyJS の環境構築していきます。
っとは言っても、やり方は README に記載している通りにすれば完了です。笑
まずはプロジェクトのルートで以下コマンドを実行

```sh
$ npm i -S inversify reflect-metadata
```

次に tsconfig.json のコンパイルに関する設定を編集します。

```json
{
    "compilerOptions": {
        "target": "es5",
        "lib": ["es6"],
        "types": ["reflect-metadata"],
        "module": "commonjs",
        "moduleResolution": "node",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}
```

個人的には、 `target` と `lib` に関しては、 `esnext` でいい気もしますが、そこはお好みで。
あと、相対パスの設定も含めたいので、以下の設定を追加します。（ここもお好みで）

```json
{
    "compilerOptions": {
        ...
        "emitDecoratorMetadata": true,
        "paths": {
            "@/*": [
                "src/*"
            ]
        },
    }
}
```

以上で、セットアップ完了です。

## 実装

それでは実際に実装に入ってみます。
実装に関してはクリーンアーキテクチャを自分なりに組んでおり、そのソースを記載してく感じになりますが、InversifyJS 雰囲気だけ感じてもらえればなと思います。
今回は１リソースをサンプルに記述していこうかと思います。
内容はポケモン情報一覧取得です。（ポケモンに関する薄っぺらい情報一覧を返すエンドポイントを作成します）

### 各パーツの作成

まず、データベースからデータ取得する repository を定義してきます。
こちらは、マルチ DB 対応・モックデータ取得といったように、汎用性を効かせるため、interface で定義してきます。
今回使用している ORM では [TypeORM](https://github.com/typeorm/typeorm#readme) ってのを用いています。（そちらの説明は主旨とは異なるため省きます）

```typescript:src/domain/repositories/IPokemonRepository.ts
import Pokemons from '../entities/Pokemons';

export default interface IPokemonRepository {
    findAll(): Promise<Pokemons[]>;
}
```

こちらの interface に対して、どの実態のインスタンスを格納するかを InversifyJS で設定していきます。
その設定の話は後にするとして、まずは実態を実装していきましょう。

```typescript:src/infrastructure/repositories/PokemonRepository.ts
import { injectable } from 'inversify';

import IPokemonRepository from '@/domain/repositories/IPokemonRepository';
import Pokemons from '@/domain/entities/Pokemons';

@injectable()
export default class PokemonRepository implements IPokemonRepository {
    public async findAll(): Promise<Pokemons[]> {
        return Pokemons.find().catch(err => {
            throw err;
        });
    }
}
```

DI に関わるクラスなどには `@injectable()` を付与します。

次にポケモン一覧取得の Usecase を定義します。

```typescript:src/usecases/pokemons/ISearchPokemonUsecase.ts
import PokemonSearchResponse from '@/usecases/dto/models/PokemonSearchResponse';

export default interface ISearchPokemonUsecase {
    search(): Promise<PokemonSearchResponse[]>;
}
```

こちらも、実態を実装していきます。

```typescript:src/interactores/pokemons/SearchPokemonInteractor.ts
import { injectable, inject } from 'inversify';
import 'reflect-metadata';

import ISearchPokemonUsecase from '@/usecases/pokemons/ISearchPokemonUsecase';
import TYPES from '@/registories/inversify.types';
import IPokemonRepository from '@/domain/repositories/IPokemonRepository';
import Pokemons from '@/domain/entities/Pokemons';
import PokemonSearchResponse from '@/usecases/dto/models/PokemonSearchResponse';

@injectable()
export default class SearchPokemonInteractor implements ISearchPokemonUsecase {
    @inject(TYPES.IPokemonRepository)
    private repository: IPokemonRepository;

    public async search(): Promise<PokemonSearchResponse[]> {
        const pokemons: Readonly<Pokemons>[] = await this.repository.findAll();
        return pokemons.map(
            (p): PokemonSearchResponse =>
                new PokemonSearchResponse(p.id, p.code, p.name, p.generationNo)
        );
    }
}
```

こちらも同様に `@injectable()` を先頭に付与します。また、先ほどの repository の interface をメンバ変数として定義しています。
こちらには `@inject` を付与します。その引数に関しては後に定義していきます。
最後に Controller を定義します。

```typescript
import { Request, Response } from 'express';
import { injectable, inject } from 'inversify';
import 'reflect-metadata';

import TYPES from '@/registories/inversify.types';
import ISearchPokemonUsecase from '@/usecases/pokemons/ISearchPokemonUsecase';
import PokemonSearchResponse from '@/usecases/dto/models/PokemonSearchResponse';
import PokemonSearchResponseViewModel from '@/usecases/dto/viewModels/PokemonSearchResponseViewModel';

@injectable()
export default class PokemonController {
    @inject(TYPES.ISearchPokemonUsecase)
    private usecase: ISearchPokemonUsecase;

    async search(_: Request, res: Response): Promise<void> {
        const response: PokemonSearchResponse[] = await this.usecase.search();

        const result: PokemonSearchResponseViewModel[] = response.map(
            (r): PokemonSearchResponseViewModel =>
                new PokemonSearchResponseViewModel(
                    r.id,
                    r.code,
                    r.name,
                    r.generationNo
                )
        );

        res.status(201).json(result);
    }
}
```

DTO の中身に関しては、Entity とほぼほぼ変わらないクラスとなっています。
まだまだ、ハードコードを直すこと（HTTP statusとか）やトランザクション周りの設定など、やることは多いですがざっとこんな感じで完成です。

### DI Container の定義

ソース中に出てきた `TYPES` や実態をどのように設定しているかについて、記載していきます。
まずは `TYPES` の定義をしてきます。
ここでは、どのクラスが実態となるのかの識別子を定義しています。定義方法は自由で、クラスでも文字列リテラルでもいいそう。
今回は README にあるような `Symbol` で定義しています。

```typescript:src/registories/inversify.types.ts
const TYPES = {
    PokemonController: Symbol.for('PokemonController'),
    IPokemonRepository: Symbol.for('IPokemonRepository'),
    ISearchPokemonUsecase: Symbol.for('ISearchPokemonUsecase')
} as const;

export default TYPES;
```

最後に DI Container の定義です。

```typescript
import { Container } from 'inversify';

import IPokemonRepository from '@/domain/repositories/IPokemonRepository';
import PokemonRepository from '@/infrastructure/repositories/PokemonRepository';
import ISearchPokemonUsecase from '@/usecases/pokemons/ISearchPokemonUsecase';
import SearchPokemonInteractor from '@/interactores/pokemons/SearchPokemonInteractor';
import PokemonController from '@/controllers/pokemons/PokemonController';

import TYPES from '@/registories/inversify.types';

const container = new Container();
container
    .bind<IPokemonRepository>(TYPES.IPokemonRepository)
    .to(PokemonRepository);
container
    .bind<ISearchPokemonUsecase>(TYPES.ISearchPokemonUsecase)
    .to(SearchPokemonInteractor)
    .inSingletonScope();
container
    .bind<PokemonController>(TYPES.PokemonController)
    .to(PokemonController);

export default container;
```

まず、今まで定義した interface とその実態クラス、クラス中に `@inject` しているクラスをインポートします。
その後に、先ほど定義した `TYPES` の識別子を用いて、どの interface にはどの実態クラスが格納されるといった設定をしていきます。
今回は普通に設定しましたが、環境変数を参照してこの実態クラスを格納する、といったような設定を記述していくと思われます。

```typescript:src/registories/inversify.config.ts
const { NODE_ENV } = process.env;
if (NODE_ENV === 'development') {
    container
        .bind<IPokemonRepository>(TYPES.IPokemonRepository)
        .to(PokemonRepository)
        .inSingletonScope();
    container
        .bind<ISearchPokemonUsecase>(TYPES.ISearchPokemonUsecase)
        .to(SearchPokemonInteractor)
        .inSingletonScope();
    container
        .bind<PokemonController>(TYPES.PokemonController)
        .to(PokemonController)
        .inSingletonScope();
} else if (NODE_ENV === 'test') {
    container
        .bind<IPokemonRepository>(TYPES.IPokemonRepository)
        .to(PokemonMock)
        .inSingletonScope();
    container
        .bind<ISearchPokemonUsecase>(TYPES.ISearchPokemonUsecase)
        .to(SearchPokemonTestInteractor)
        .inSingletonScope();
    container
        .bind<PokemonController>(TYPES.PokemonController)
        .to(PokemonController)
        .inSingletonScope();
}
```

すいません。すっごい適当に書いてます。笑
理想は、環境ごとに config ファイルを用意して（ `inversify.dev.ts` や `inversify.test.ts` など） どれを読み込むかを `inversify.config.ts` でいい感じにするがいいのかもしれません。

### 定義した Controller を Express でコールしてみる

ここまで定義したのを Express Router に読み込ませます。

```typescript:src/app.ts
import * as express from 'express';
import { Request, Response } from 'express';
import 'reflect-metadata';

import container from '@/registories/inversify.config';
import PokemonController from '@/controllers/pokemons/PokemonController';
import TYPES from '@/registories/inversify.types';

const pokemonControllerContainer = container.get<PokemonController>(
    TYPES.PokemonController
);

const app = express();

app.get('/', (req: Request, res: Response) =>
    pokemonControllerContainer.search(req, res)
);

const port = 3000;
app.listen(port, () => console.log(`Example app listening on port ${port}!`));
```

これで config に基づいて DI された実態クラスのメソッドがコールされ、処理が実行されます。
また、エンドポイントでの処理を以下のように記述するとなぜかうまくいきませんでした。。。（ここ、ちょっとハマりました）

```typescript
app.get('/', pokemonControllerContainer.search);
```

## まとめ

とりあえず、記事まとめるのはとても疲れますね。（後半、結構雑になってるかもしれません。。。）
結構シンプルな実装になるので、キャッチアップも早くできるかと思われます。
個人的に、DI される実態クラスを設定できるのはとてもありがたいので、サーバサイド開発では今後とも使っていこうと思っています。
結構スター数も多いので安心して使えるパッケージです。気になる方は是非使用してみてください！
