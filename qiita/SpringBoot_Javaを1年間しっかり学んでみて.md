この記事は[ぷりぷりあぷりけーしょんず Advent Calendar 2020](https://qiita.com/advent-calendar/2020/ppap)の6日目の記事です。

## 背景
研修時代でも Spring Boot や Java は触ってきましたが、業務でしっかり触ったことはなく、業務を通して学んだことや参考書・勉強会で学んだことを本記事でアウトプットしていこうと思います。
自分のローカル環境における Spring Boot / Java のバージョンは以下となります。

* Spring Boot ... 2.3.1
* Java ... 14

そのバージョンならではの技術にも触れていこうと思っています。

## Spring Boot
### v2.3
個人開発でもよく使用していた Spring Boot フレームワークですが、今年の 2 月に 2.3 がリリースされたことをうけ、[JJUG](https://www.java-users.jp/page/about/) でオンライン勉強会が開かれました。
その際の勉強会で魅力的な機能が豊富にあり、個人開発でも 1.5.9 -> 2.3.1 に大規模アップデートを実施しました。（普通ならあり得ないと思います笑）
リリースされたバージョンで追加された機能は以下となります。

* Kubernetes Support
  * Actuator Healthchek
  * Graceful Shutdown
  * Support of wildcard locations for configuration files
* Docker Image Support
  * Cloud Native Buildpacks
  * Layered Jar Format
* Other Changes
  * Java 14 support

色々魅力のある機能がありましたが、自分は `Actuator Healthchek` / `Graceful Shutdown` / `Cloud Native Buildpacks` を導入してみました。また Java 14 をサポートしたと言うのもあり、Java も 14 にアップデートしちゃいました。
特によかったと思う機能は `Cloud Native Buildpacks` で、Heroku や Pivotal で利用されていた Buildpack をサポートするようになりました。
それにより、OCI 標準な Docker Image を作成することができるようになり、また jar の起動で面倒だった `-Xmx` などの JVM のヒープメモリ割り当てなどを考慮せず最適な状態のイメージを作成することができるというなんとも素晴らしい、、、
以下は Docker Image を作成するコマンドです。（ちなみに自分は gradle を使用しています）

```
Maven
$ mvn spring-boot:build-image

Gradle
./gradlew bootBuildImage
```

Intellij を使用している場合は GUI でぽちぽちする感じかと思われます。
せっかくいい感じのイメージを作成したので、その Docker Image を Docker hub にあげて、EC2 上でも使用できるように build.gradle にコマンドを追加しました。

```build.gradle
def dockerImageName = '{docker_hub_account}/{image_name}'

bootBuildImage {
	imageName = dockerImageName
}

task imagePush(type: Exec) {
	commandLine 'docker', 'push', dockerImageName + ':latest'
}
```

最近 AWS Lambda でコンテナイメージをサポートしたらしいので、EC2 から Lambda に移行しようかなぁとか考えてます。

### Wavefront
こちらも当時のオンライン勉強会でおまけとして発表されていたモニタリングサービスです。
導入が非常に簡単で、ライブラリを追加しアプリケーションを起動すると API トークンが表示されるので、そちらを `application.properties` or `application.yml` に追加するだけでモニタリングをすることができます。
詳細なやり方については、ググればたくさん情報が出てくると思うので省略させてください、、、🙇‍♂️
実際に導入した際のモニタリングがこちらとなります。

![スクリーンショット 2020-12-02 22.59.02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/15e18c38-1de8-2489-df63-95ae3e50104c.png)

## Java
今まではなんとなくで実装していた java でしたが、Effective Java を読んだり（まだ読み終えていません）コードのデザインを考えたりした際に、とにかく書くことが多い・気をつけないと行けない点が多いなどがあった反面、しっかり書けると面白いなと感じることが多かったので、そのコードについてアウトプットしていきたいと思いまする。

### static factory method
通常、クラスのインスタンスを生成する方法として、public constructor がありますが、そこを static factory method でインスタンスを生成するような作りにすることもできます。
デザインパターンをやっている方だと、よく Flyweight や Singleton とかでよく見かけた書き方だったと思われます。

```java:AClass.java
public class AClass {
    private final String arg1;
    private final String arg2;

    private AClass(String arg1, String arg2) {
        this.arg1 = arg1;
        this.arg2 = arg2;
    }

    public static AClass of(String arg1, String arg2) {
        return new AClass(arg1, arg2);
    }
}
```

lombok を使用している場合は、もっと省略的に記述できます。

```java:AClass.java
@Data(staticConstructor = "of")
public class AClass {
    private final String arg1;
    private final String arg2;
}
```

ソースが大分減りました。 `@Data(staticConstructor = "of")` を付与すると constructor を private にしてくれて、static factory の of メソッドを自動で作成してくれます。他にも equals や canEqual などのメソッドも自動生成してくれます。便利ですね。
static factory method の長所としては、やはり method なので名前を持つことが大きいなと感じました。
method の名前として共通する命名規則は下記のような感じとなります（Effective Java から引用）

|  Method name  |  説明  |
| :---- | :---- |
|  from  |  単一のパラメータを受け取り、当の型を持つ対応するインスタンスを返す型変換メソッド  |
|  of  |  複数のパラメータを受け取り、それらを含んだ当の型のインスタンスを返す集約メソッド  |
|  valueOf  |  from や of の代わりとなる、冗長なメソッド  |
|  instance / getInstance  |  パラメータがあればそのパラメータで表されているインスタンスを返すが、必ずしも同じ値を持つとは限らない  |
|  create / newInstance  |  呼び出しごとにメソッドは新たなインスタンスを返す  |
|  getType  |  ファクトリメソッドが対象のクラスとは異なるクラスにある場合に使われる  |
|  newType  |  ファクトリメソッドが対象のクラスとは異なるクラスにある場合に使われる  |
|  type  |  getType や newType の代わりとなる簡潔な名前のメソッド  |

### 継承を避ける
Food と言うクラスがあって、Junk クラスは Food クラスを継承したいと言うようなケースがあった場合に、自分はよく何も考えずに継承をしていました。
しかし継承は、正しく文書化して設計されていないと危険なことが多いみたいです。例えば、スーパークラスで定義されているメソッドの本来の意味を履き違えて間違った Override をしてしまうなど。
Override する場合はそのメソッドにどのような意味が込められていて、どのような実装をすることが正しいのかをしっかり認識しないと間違ったロジックとなり、いずれバグにつながることがあるようです。
それを防ぐためにも文書化をする（設計をしっかりする）ようにし、Javadoc に明記しておくような対応をとるような心掛けが必要のようです。既存のライブラリなどで Override をする場合も、そのメソッドの Javadoc を見て、正しくドキュメント化されているか（ `@implSpec` があるか）を確認し、設計して実装する、ドキュメントが整っていない場合は Override しない（なんなら継承しない）方がいいようですね。

### enum
enum を定義し、その enum のパラメータを使用したロジックを定義しようとした場合、下記のような実装を今までしていました。

```java
public enum Permission {
    READ(1),   // 1 << 0
    WRITE(2),  // 1 << 1
    EXECUTE(4) // 1 << 2
    ;

    private final int value;

    Permission(final int value) {
        this.value = value;
    }

    public static boolean isRead(int value) {
        return (READ.value & value) != 0;
    }

    public static boolean isWrite(int value) {
        return (WRITE.value & value) != 0;
    }

    public static boolean isExecute(int value) {
        return (EXECUTE.value & value) != 0;
    }
}
```

わりかしスマートにかけて満足しましたが、少し欠点があります。
それは、↑の例だと、新たに権限の種類が増えた場合、`isXXX` のようなメソッドを増やさないといけないですが、それを忘れてしまった場合、不具合につながるケースがあったりします。新たなパラメータが増えたときに何かしらのアクションをさせるような仕組みが欲しかったりするわけです。
そのような仕組みを作るにあたって、抽象メソッド（abstract）を持たせるようにすることで上記問題を解決できます。

```java
public enum Permission {
    READ(1) {
        @Override
        public boolean is(int value) {
            return (READ.value & value) != 0;
        }
    },
    WRITE(2) {
        @Override
        public boolean is(int value) {
            return (WRITE.value & value) != 0;
        }
    },
    EXECUTE(4) {
        @Override
        public boolean is(int value) {
            return (EXECUTE.value & value) != 0;
        }
    }
    ;

    private final int value;

    Permission(final int value) {
        this.value = value;
    }
    
    public abstract boolean is(final int value);
}
```

このような書き方をすることで、新たにパラメータが増えても、コンパイルエラーとなり、処理を追加させることを強制することができます。
さらに Javs8 以降を使用している場合は Function パッケージと組み合わせることで以下の書き方もできます。

```java
import java.util.function.IntPredicate;

public enum Permission {
    READ(1, value -> (Permission.READ.value & value) != 0),   // 1 << 0
    WRITE(2, value -> (Permission.WRITE.value & value) != 0),  // 1 << 1
    EXECUTE(4, value -> (Permission.EXECUTE.value & value) != 0) // 1 << 2
    ;

    private final int value;
    private final IntPredicate ip;

    Permission(final int value, final IntPredicate ip) {
        this.value = value;
        this.ip = ip;
    }

    public boolean is(final int value) {
        return ip.test(value);
    } 
}
```

### アノテーション
自作のアノテーションとなると、結構作るの面倒そうってのと、アノテーションを作成する時ってどんな時なのかがイメージできていなく、業務でも個人開発でも作成することはなかったのですが、目印という意味で使えるなと思いました。
例えば権限系のアノテーションを作成して、そのアノテーションが付与されているメソッドでは権限チェックをするような感じで使えるなど。

```java:ReadPermission.java
/**
 * Read permission.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ReadPermission {
}
```

```java:SampleController.java
@RestController
@RequestMapping(value = "/sample")
public class SampleController {
    @GetMapping
    @ReadPermission
    public List<Sample> handleGetAll() {
        return List.of(Sample.of("sample"));
    }
}
```

```java:Interceptor.java
public class Interceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(@NonNull HttpServletRequest request, @NonNull HttpServletResponse response, @NonNull Object handler) throws Exception {
        final HandlerMethod hm = (HandlerMethod) handler;
        final Method method = hm.getMethod();

        final Optional<ReadPermission> annotation = Optional.ofNullable(
                AnnotationUtils.findAnnotation(method, ReadPermission.class)
        );
        if (annotation.isPresent()) {
            // Check read permission
        }
        return true;
    }
}
```

↑の例では、アノテーションを作成して、コントローラにアノテーションを付与し、interceptor でアノテーションチェックを実施する感じですね。
このように実装することで、共通処理などにできる箇所もあると思うのでスマートに実装できそうだなと思いました。
今後もマーキング的な意味でアノテーションは使っていきたいなって思っています。

### ジェネリクス
ジェネリクスはそんなに難しいイメージを持っていなかったんですが、参考書を読んでみて全然理解していないことを理解しました笑（今でも全然理解ができていないところが多いです）
ジェネリクスとは List や Set, Map などによくみられる `<T>` です。この T に入る方は１つの型しか持たせることしかできません。例えば全てのスーパークラスである `Object` クラスですが、 `List<Object>` と定義した場合、 `List<Integer>` を代入することはできないです。
そこの T を柔軟にしたい場合は、ワイルドカード `?` を使うとか、 `extends` を使用して抽象的にする必要があります。（うまく説明できている気が全くしません。申し訳ないです。）

```java
List<? extends Object> list = List.of(1, 2);
```

最近コマンドラインアプリを復習がてらで作っていたんですが、ジェネリクスを使って抽象クラスを作成してみました。細かい内容は書くの疲れたので気が向いたら書きますがジェネリクスの部分だけ一旦下記に残します。

```java:AbstractRepository.java
public abstract class AbstractRepository<E extends Entity<ID>, ID> implements Repository<E, ID> {
    protected final DataStore<E, ID> STORE = DataStore.of();

    @Override
    public List<E> findAll() {
        final List<E> all = new ArrayList<>();
        for (E topping : STORE.get()) {
            try {
                @SuppressWarnings("unchecked") E clone = (E) topping.clone();
                all.add(clone);
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
                return List.of();
            }
        }
        return all;
    }

    // もっと処理はありますが、省略します
}
```

こちらのソースを見てわかる通り、なんのこっちゃって感じです笑
ジェネリクスを使ってみて、使うタイミングなどはしっかり設計した上で実装するのが重要だなと思いました。（しっかり設計するのはどの言語でも共通ですが）
個人的には、よくできたなぁうんうんと思っていますが、また勉強したらこのソースを修正したりしたい感じですね。

## まとめ
もっと書きたい内容はありましが、書くの疲れたんで許してください。。。
ただ、復習という意味でアウトプットしたいのが目的なので、記事は更新していこうかなと思っています（多分やらない気もしますが）
Spring Boot や Java を本格的にやってみて、やはり書くことが多いなと改めて実感しました笑
ただ、Spring Boot のバージョンアップに対しては、書くことの多さが軽減されていると思っています。
来年は Golang を本格的にやっていきたいなって思っています。

