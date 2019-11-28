# TypeScript での DI 周り

hash: Node.js TypeScript InversifyJS DI

## 背景

マイクロサービスの簡単な勉強として、 RESTful API を Node.js で作成しようと開発を始めました。（なぜ Node.js かというと、一番慣れている言語だからです）
また、静的型付にしたかったため、流行りの TypeScript を採用。
せっかくなので、クリーンアーキテクチャにも挑戦したいなというのもあり、そのアーキテクチャで設計や実装を始めました。
実装を進めていく上で、いちいち `constructor` で `new` するのが嫌だったのと、 interface の実態がなんなのかを１つのファイルで完結させたかった（まさに DI Container）というのがあり、なんかいいのがないのか調べたところ、 `InversifyJS` となるものを発見。
これはなかなかいいなと感じたため、記事にしてみようかと思った次第どす。

## InversifyJS とは

TypeScript での強力で軽量の DI コンテナです。

* [git InversifyJS](https://github.com/inversify/InversifyJS)

### セットアップ

Node.js でのプロジェクトが作成されている前提で話を進めていきますが、まだ作成していないって場合は以下のコマンドを実行し、作成してみてください。
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



