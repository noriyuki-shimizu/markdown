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

簡潔にいうと、`this`というのは

* <strong>`this` は `function` を呼んだ時の `.` の前についているオブジェクトを指している</strong>  
※@takkyunさんの記事引用

らしい。
`.`の前に何もなければ、<strong>グローバルオブジェクト</strong>を`this`は参照するらしい。
グローバルオブジェクトについては、知っている前提で話を進めます。
もし、わからない方がいれば

[Global object (グローバルオブジェクト)
](https://developer.mozilla.org/ja/docs/Glossary/Global_object)

を参照してください。

以下のソースは、簡単にグローバルオブジェクトと`this`についての挙動をまとめたソースとなります。

```js
// グローバルオブジェクト(Window Object)
console.log(this);

// グローバルな領域で変数を定義。
// すると、以下のargはグローバルオブジェクトのメンバ変数として追加される。
var arg = "argument0";

// 以下は全て同じものを参照している。
console.log(arg);        // -> argument0
console.log(this.arg);   // -> argument0
console.log(window.arg); // -> argument0

function func() {
    console.log(this.arg);
}

// 上記で宣言したメソッドをメンバ変数として追加。
var obj = {
    arg: "argument1",
    func: func
}

// 直接メソッドを呼ぶと . の前が存在しないため、グローバル領域のargが出力される。
func();      // -> argument0
// . の前についているオブジェクト(obj)を参照する。
obj.func();  // -> argument1
```

なるほど、だんだんわかってきましたぞ。

### あ。アロー関数は？
そもそもアロー関数を使用したら、うまくいかなかったというのに、少し`this`の話が長くなってしまいました。。。
それでは、アロー関数での`this`の挙動はどうなるのでしょうか？
こちらも簡潔にいうと

* <strong>アロー関数式で宣言された関数は、宣言された時点で、`this`を確定（＝束縛）させてしまうのです。</strong>  
※@mejilebenさんの記事引用

こちらも実際にソースをみてみましょう。

```js
var arg = "argument0";

// アロー関数でメソッドを定義
var func = () => {
    console.log(this.arg);
}

// 上記で宣言したメソッドをメンバ変数として追加。
var obj = {
    arg: "argument1",
    func: func
}

// こちらは、先ほどと変わらず、グローバル領域の変数を参照します。
func();      // -> argument0
// アロー関数で定義した時点でthisが決定するため
// objを参照せず、その外側の領域(グローバル領域)を参照します。
obj.func();  // -> argument0
```

### いやー、結構わかってきたぞー
`this`での挙動についてある程度理解しました。
そこで、自分なりに色々意地悪なソースを考えてみたりして。。。
皆さんも、以下のソースで実際にどのような出力結果となるか考えてみてください！

```js
arg = "arg1";

function consoleLog() {
    console.log(this.arg);
}

var obj1 = {
    arg: "argument2",
    func: consoleLog
}

var obj2 = {
    arg: "argument3",
    func: () => {
        console.log(this.arg);
    }
}

var obj3 = {
    arg: "argument4",
    func: function() {
        var arg = "argument5";
        var subFunc = () => {
            console.log(this.arg);
        }

        subFunc();
    }
}

var obj4 = {
    arg: "argument6",
    func: function() {
        var arg = "argument7";
        var subFunc = function() {
            console.log(this.arg);
        }

        subFunc();
    }
}

consoleLog(); // -> ???
obj1.func();  // -> ???
obj2.func();  // -> ???
obj3.func();  // -> ???
obj4.func();  // -> ???
```

ここまでをしっかり把握していれば、そんなに難しくないと思われます！
答えに関しては、Chromeブラウザなどの開発者ツールから上記ソースをコピペではっつけて出力結果をみてみてください！！

### まとめ
ここまで、説明が足りない箇所もあったかと思われますが、ざっとまとめてみました。
JavaScriptでの`this`は少し複雑だなと感じました。
あとは、何も考えずにアロー関数を用いるのは危険であることもわかりました。（気をつけます）

`this`の操作に慣れると、ソースも安全に組むことができますし、正しい`this`の使い方でコーディングできると思われます。

### 補足
