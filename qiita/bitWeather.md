## 作成したツール
今回、BitBarというツールとNode.jsを使って、天気予報を表示させるツールを作成してみた。
ツール名は「<strong>BitWeather</strong>」
<img src="https://raw.githubusercontent.com/noriyuki-shimizu/images/master/BitWeather_v0.0.1.gif" width="400px">

一応gitにも公開したので、是非とも閲覧且つ、インストール且つ、スターをつけるをお願いします!!

* [Git Hub - BitWeather](https://github.com/noriyuki-shimizu/BitWeather)

そんでもって、今回の投稿が初投稿になります。皆さんよろしくです。

### BitBarとは
任意のスクリプトまたはプログラムからの出力をMac OS Xのメニューバーに直接表示できる無料ツール。
また、cronのように任意の実行間隔でスクリプトなど、処理を実行できるそう。

### なるほどってことで、使ってみました。
結論から言うと、導入がクッソ簡単ですww
[公式ページ](https://getbitbar.com/)、またはターミナル上から`BitBar`をインストールします。ターミナルだと、以下の感じで

```
$ brew cask install bitbar
```

インストールされたら、それを実行するだけ( ・∇・)
すると、あら不思議!!メニューバーに`BitBar`が表示されたじゃないのよ( ・∇・)!!
<img src="https://user-images.githubusercontent.com/31028330/52277096-2fc8cc80-2997-11e9-8915-aa1d03272573.png" width="500px">
あとは、BitBar専用のプラグインフォルダを作成して、そのディレクトリに`shell`やら`Ruby`やら`node`やらのスクリプトを記述すれば、メニューバーに色々な機能が追加できるって感じっす( ・∇・)

BitBarについては以下のサイトがとてもわかりやすくまとめているため、詳細などは以下を参照してください!!

* [BitBarでMacのメニューバーにオリジナリティを](https://qiita.com/ta-chibana/items/169cb7690b6a0174ea36)

### 天気予報の取得
今回作成したツールでは、<strong>OpenWeatherMap</strong>っていう無料天気予報APIを使用して天気予報を取得しました。
「天気予報　取得」で検索すると、結構ヒットするのがOpenWeatherMapだったので、自分も使ってみようという感じですね。

<img width="1440" alt="スクリーンショット 2019-02-11 16.13.59.png" src="https://qiita-image-store.s3.amazonaws.com/0/341734/6f397462-a6a2-1da9-0316-80ede06bfdad.png">

あまり、お金かけるの嫌なんで、有料枠のプランではなく無料枠のプランで使用してみることに。
無料のFreeプランだと5日後までの3時間毎の天気予報を取得できて、1分間に60回までAPIコールできるそう。(所感としては、全然無料枠でええやろって感じっす)
あとは、[公式サイト](https://openweathermap.org/api)からAPI keyを発行し、APIにリクエストを投げれば、天気予報が簡単に取得できます。(説明大分省略してます。。。)

詳しく知りたい方は、「[OpenWeatherMap 天気予報 取得](https://www.google.com/search?q=OpenWeatherMap+%E5%A4%A9%E6%B0%97%E4%BA%88%E5%A0%B1+%E5%8F%96%E5%BE%97&rlz=1C5CHFA_enJP829JP829&oq=OpenWeatherMap+%E5%A4%A9%E6%B0%97%E4%BA%88%E5%A0%B1+%E5%8F%96%E5%BE%97&aqs=chrome..69i57j69i61.192j0j7&sourceid=chrome&ie=UTF-8)」で検索!!( ・∇・)

### 現在地の取得
せっかくだし、現在地も取得したいなぁっと思ったので、色々調べてみました。
色々調べてみた結果、「<strong>ipinfo.io</strong>」というAPIと出会うことができました。
こちらのAPIは意外と正確に現在地を取得してくれます。
何より

> 10万人以上の企業や開発者から信頼されています。

っとか言われたら、そりゃ安心して使っちゃうでしょ!!
また、こちらのAPIも使うのが簡単で、[公式ページ](https://ipinfo.io/)から「SIGN UP」を選択し、アカウント登録するとtokenが発行できます。
そのtokenをリクエストパラメータとして使用しAPIをたたくと現在地情報など取得できます。

こちらも詳しく知りたい方は、「[ipinfo.io 現在地 取得](https://www.google.com/search?q=ipinfo.io+%E7%8F%BE%E5%9C%A8%E5%9C%B0+%E5%8F%96%E5%BE%97&rlz=1C5CHFA_enJP829JP829&oq=ipinfo.io+%E7%8F%BE%E5%9C%A8%E5%9C%B0%E3%80%80%E5%8F%96%E5%BE%97&aqs=chrome..69i57.12122j0j7&sourceid=chrome&ie=UTF-8)」で検索!!( ・∇・)

### 実装におけるプチこだわり
今回の実装では、なるべくドメイン駆動っぽく実装したかったため、1つのjsファイルに1つの関心事を詰めること意識し実装しました。
jsでのオブジェクトとして、以下のような雛形からオブジェクトを作成しました。

```js
var Obj = {
    create: (arg) => {
        var obj = Object.create(Obj.prototype);
        obj.arg = arg;
        return obj;
    },
    prototype: {
        method() {
            // Use arg process.
        }
    }
}
```

割と上記の書き方は好きです。わかる人にはわかりやすいオブジェクトの書き方ではないかと(勝手に)思ってます。

上記の雛形より、例えばリクエストURLとかのオブジェクトは以下のように作成しました。

```js:url.js
exports.create = (url, parameter) => {
    return Url.create(url, parameter);
}

var Url = {
    create: (requestUrl, requestParameter) => {
        var url = Object.create(Url.prototype);

        url.requestUrl = requestUrl;

        url.requestParameter = requestParameter;

        return url;
    },
    prototype: {
        createUrl() {

            if(this.requestParameter === undefined) {
                return this.requestUrl;
            }

            var keys = Object.keys(this.requestParameter);

            var mappingKeys = keys.map((key, index) => {
                return key + '=' + this.requestParameter[key];
            });

            return this.requestUrl + '?' + mappingKeys.join('&');
        }
    }
}

// 使用する側
var Url = require('./url');
var url = Url.create('https://request/url', {id: 111, value: 'hoge'});
url.createUrl(); // -> "https://request/url?id=111&value=hoge"
```

まぁ、車輪の再発明感はあるはメソッド名はわかりにくいやら。。。
リファクタリングはまだまだやります。。。

## まとめ
今回、BitWeatherを作成してみましたが、まずは使ってみる価値ありだと思われます!!(笑)
Node.jsでドメイン駆動っぽく実装したのが初めてだったので、開発には時間がかかりました。

まだまだBitWeatherは拡張していくつもりなのでバージョンをどんどん上げていこうと思っていますはい!!
バージョンの付け方やら細かいことは全くよくわかってないので雰囲気でどんどん進めていこうと思っていますですはい!!

以上です。
最後まで閲覧してくれた方、ありがとうございます🙇‍♂️

