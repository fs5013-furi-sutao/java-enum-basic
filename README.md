# 列挙子 Enum の使い方（基本）

## Enum とは

Enum は enumeration（エナメレーション）の略で、意味的には「列挙」「数え上げること」「一覧表」となる。

### 読み方

なので読み方は、英語的には「エナム」となるが、日本では「イーナム」で通ることが多い。

### 列挙子

列挙子と言われても、なかなか一般的に使わない言葉だが、「問題点を列挙する」というと「問題点を 1 つ 1 つ挙げていく」という意味合いになる。

Enum の役割的にも、Enum は「要素を列挙したもの」ということになる。

## Enumの宣言

Enum は次のような構文で宣言する。

ここでは「果物」を例としたい。ファイル名は Fruit.java となる。

``` java title="Fruits.java
enum Fruit {

    APPLE,
    ORANGE,
    BANANA,
    PEACH,
    ;

}

``` 

この Enum では「果物（Fruits）は、りんご（APPLE）、みかん（ORANGE）、バナナ（BANANA）、モモ（PEACH）のいずれかである」と宣言している。

見て分かる通り、宣言に利用されているキーワードは enum で、class ではない。つまり、Enum はクラスではない。

ように見えるが、Java の内部的には enum は、抽象クラス Enum を継承（extends）したクラスである。

OpenJDK 15: `Class Enum<E extends Enum<E>>`  
https://cr.openjdk.java.net/~iris/se/15/latestSpec/api/java.base/java/lang/Enum.html

上記で例に出した enum 宣言は、以下のクラス宣言と等価（結果が同じ）となる。

```java
class Fruit {
    public static final Fruit APPLE = new Fruit();
    public static final Fruit ORANGE = new Fruit();
    public static final Fruit BANANA = new Fruit();
    public static final Fruit PEACH = new Fruit();

    // 外部からインスタンス化できないようにする
    private Fruit() {}
}
```

### enum と class の違い

ただし、enum がクラスと違うのは、暗黙的に便利なメソッドが備わることである。暗黙的に備わっているメソッドについては後述する。

### enum の値はなぜ大文字か？

enum の 値について、表記を大文字にしているのは、Java の一般的な慣習で定数は大文字としているからだ。そのため、小文字で表記してもコンパイルエラーにはならない。

しかし、Java の標準 API や多くのフレームワークでも enum の値は「すべて大文字で単語の区切りはアンダースコア」となっている。

そのため、このルールを守った方が、共同開発する際には他のチームメイトが、コードをメンテしやすくなる。

## enum の値の使い方

enum の値は、クラスにおける static フィールドとほぼ同じである。

enum の値、すなわちインスタンスへのアクセスは、static フィールドへのアクセスと同じ「enum名. 値の名称」となる。

上記で作成した Fruit の列挙子を使ってみる。

``` java
System.out.println(Fruit.BANANA);
```

実行結果:

``` output
BANANA
```

もちろん、Fruit で定義したもの以外を呼び出すことはできない（コンパイルエラー）。

``` java
// コンパイルエラー
Fruit.GORILLA;
```

enum で定義した値は、Enum 自身のインスタンスである。String や Integer ではない。つまり、Fruit. BANANA は、「果物の 1 つとしてのバナナ」を表現していることになる。

``` java
boolean isBananaFruit = Fruit.BANANA instanceof Fruit;
System.out.println(isBananaFruit);
```

実行結果:

``` output
true
```

enum はクラスと同じように型として扱える。変数として宣言したり、enum のインスタンスを参照したりすることができる。

``` java
Fruit myFavoriteFruit = Fruit.PEACH;
Fruit myHateFruit = Fruit.APPLE;

showFruit(myFavoriteFruit);

public void showFruit(Fruit fruit) {
    System.out.println(fruit);
}
```

つまり、enum はクラスのインスタンスと同じように扱えるものである。

### Enum はインスタンスを生成できない

Enum が通常のクラスと違う最大のポイントは「インスタンスの生成を enum の外部からできない」ことである。enum として存在するインスタンスは、enum 内であらかじめ定義した値に限られる。

これが Enum の存在意義である。

だからこそ、enum の各インスタンスは、実行されるプログラムの中で「ただ 1 つしかない」ことが保証される「定数」として扱える。

``` java
// コンパイルエラー
Fruit fruit = new Fruit();
```

### Enum は、アプリの中で設定値として利用される

以上の性質から、Enum はアプリの中で設定値として利用されることが多い。

今回の Fruit の例では、プログラム上では、果物には「りんご」「みかん」「バナナ」「モモ」の 4 種類しか存在しないことが保証される。

アプリで言えば、例えば会員のランクを表す列挙子として MemberRank という enum を作ったとしよう。

SILVER と GOLD という値を定義すれば、会員ランクには 2 種類しか存在しないことが保証される。

アプリを改修したい時でも、「一般会員」を増やすなら、ここに GENERAL を増やすだけで、改修が可能となる。

## Enum を使ってみる

### Enum と Enum の比較

以下は、if や switch の条件判定に Enum を使うケース。

``` java
Fruit fruit = Fruit.BANANA;

if (fruit == Fruit.BANANA) {
    System.out.println("バナナ");
}
```

``` java
Fruit fruit = Fruit.BANANA;
switch (fruit) {
    case APPLE:
        System.out.println("りんご");
        break;
    case ORANGE:
        System.out.println("みかん");
        break;
    case BANANA:
        System.out.println("バナナ");
        break;
    case PEACH:
        System.out.println("モモ");
        break;
}
```

この通り、Enum 同士の比較は == で可能となる。equals() メソッドでも比較は可能だが、可読性が悪くなるので、オススメできない。

Java の equals() メソッドの挙動について、しっかり理解している人は、文字列を比較する事例から「== だと、保持する値は同じだけれども、インスタンスが違ったら false になるのでマズいのでは？」と心配するかもしれない。

しかし、enum ではその心配は不要となる。

なぜなら、enum の値、つまりインスタンスは、実行されるプログラム上で必ず 1 つだけになることが保証されているからである。Enum インスタンスの比較は == で OK である。

## Enum が暗黙的に持つメソッド

Enum には標準で備わっているメソッドがある。以下で説明するものは、enum であれば、定義、実装をしないでも共通的に使えるものである。

### name() メソッド

name は enum の値の名称を String として返すメソッドである。変数が参照しているのがどの値なのかを確認する時などに利用する。

name() メソッドは、enum 値（インスタンス）から呼び出す、インスタンスメソッドである。

``` java
Fruit fruit = Fruit.BANANA;

System.out.println(fruit.name());
```

### ordinal() メソッド

ordinal() メソッドは Enum が宣言された順番（インデックス）を数値で返すメソッドである。インデックスは、ゼロ始まりとなる。

ordinal() メソッドも、enum 値（インスタンス）から呼び出す、インスタンスメソッドである。

``` java
System.out.println(Fruit.APPLE.ordinal());
System.out.println(Fruit.ORANGE.ordinal());
System.out.println(Fruit.BANANA.ordinal());
System.out.println(Fruit.PEACH.ordinal());
```

実行結果:

``` output
0
1
2
3
```

### valueOf() メソッド

valueOf() メソッドは文字列に対応する enum 値を返すメソッドである。valueOf() メソッドは、Enum の static メソッドなので、enum 値（インスタンス）から呼び出せるメソッドではない。

``` java
String fruitName = "BANANA";

Fruit fruit = Fruit.valueOf(fruitName);
System.out.println(fruit);
```

実行結果:

``` output
BANANA
```

enum で定義された値に該当しない文字列を指定すると、実行時例外が発生する。

``` java
Fruit fruit = Fruit.valueOf("DORAGON");
```

実行結果:

``` output
java.lang.IllegalArgumentException: No enum constant Fruit.DORAGON
```

### values() メソッド

values() メソッドは、すべての enum 値の配列を返す static メソッドである。全ての値に対して処理を実行したい場合に利用する。

``` java
Fruit[] fruits = Fruit.values();

for (Fruit fruit : fruits) {
    System.out.println(fruit);
}
```

実行結果:

``` output
APPLE
ORANGE
BANANA
PEACH
```

## まとめ

Java における Enum とは何か、から基本的な使い方までを見てきた。

Enum はクラスと同じように使うことができるが、インスタンスを生成できない。よって、各 enum 値は実行されるプログラム中にただ 1 つ存在することが保証されたインスタンスである。

Enum の利用価値は、定数として、さらにアプリの中では設定値して有効となる。

また、Enum には、暗黙的に name()、ordinal()、valueOf()、values() といったメソッドが用意されている。
