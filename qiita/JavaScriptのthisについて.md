### 本記事を投稿しとうとしたきっかけ
JavaScriptの勉強をしていて、`prototype`でのオブジェクト指向について勉強をやっていた時
`this`の挙動がよくわからなくなったため、色々調べました。
その際に調べた`this`について、本記事で色々まとめてみたいと思います。
何か不足している情報などあれば、コメントください。。。

### prototypeオブジェクト
よくJavaScriptで、機能ごとにオブジェクトにまとめたいなどあると思います。
その際に必ず、`prototype`に出会うことができると思います。

```js
var UserInfo = function(name, age, sex) {
    this.name = name;
    this.age = age;
    this.sex = sex;
}

UserInfo.prototype = {
    toString() {
        console.log('名称：' + this.name);
        console.log('年齢：' + this.age);
        console.log('性別：' + this.sex);
    }
}

var userInfo = new UserInfo('太郎', 20, '男');
userInfo.toString();
// -> 名称：太郎
// -> 年齢：20
// -> 性別：男
```

上記ソースを簡単に説明すると、「名称」「年齢」「性別」の3つのメンバ変数を持ったユーザ情報オブジェクトがあり、そのオブジェクトの`prototype`には`toString`でユーザ情報について表示する簡単なオブジェクトです。
メンバ変数に注目すると、`this`が出現します。
その`this`で定義された変数は、`prototype`で定義したメソッド内（上記ソースだと`toString`メソッド）などで用いることができます。

しかし、以下のソースを実行してみると、予想とは違う挙動になります。

```js
var UserInfo = function(name, age, sex) {
    this.name = name;
    this.age = age;
    this.sex = sex;
}

UserInfo.prototype = {
	toString: function() {
		console.log('名称：' + this.name);
		console.log('年齢：' + this.age);
		console.log('性別：' + this.sex);
	},
	// 追記
	set: (name, age, sex) => {
		this.name = name;
		this.age = age;
		this.sex = sex;
	}
}

var userInfo = new UserInfo('太郎', 20, '男');
// 追記
userInfo.set('花子', 30, '女');

userInfo.toString();
// -> 名称：太郎
// -> 年齢：20
// -> 性別：男
```

### あれ？変わらない？
先ほどの追記したソースをみてみると、`set`メソッドは[アロー関数(Arrow_functions)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Arrow_functions)で記述されています。
勉強していく上で、いちいち`function`って書くのがめんどくさくなり、アロー関数をたくさん書くようにしたら、上記のようなソースでハマりました。。。

よくわからないまま直そうとした際、とりあえず、アロー関数を普通の`function`にしてみようと思い、戻してみました。
すると表示結果は自分の思っていたような正常動作になりました。

```js
	// ...省略
	set: function(name, age, sex) {
		this.name = name;
		this.age = age;
		this.sex = sex;
	}
}

var userInfo = new UserInfo('太郎', 20, '男');
userInfo.set('花子', 30, '女');
userInfo.toString();
// -> 名称：花子
// -> 年齢：30
// -> 性別：女
```

### つまり、アロー関数が悪い？
アロー関数と`this`の相性が悪いのか？っと考え、色々調べてみました。
すると

>2 つの理由から、アロー関数が導入されました。1 つ目の理由は関数を短く書きたいということで、2 つ目の理由は this を束縛したくない、ということです。

あぁなるほど〜束縛したくないからね〜
わかりません。。。

さらに色々調べてみました。戦いですね。
するといい記事が発見できました。

* [【JavaScript】アロー関数式を学ぶついでにthisも復習する話](https://qiita.com/mejileben/items/69e5facdb60781927929)
* [JavaScript の this を理解する多分一番分かりやすい説明](https://qiita.com/takkyun/items/c6e2f2cf25327299cf03)

@mejileben, @takkyunさん、ありがとうございます！
とってもわかりやすかったです！

簡潔にいうと、`this`というのは、
