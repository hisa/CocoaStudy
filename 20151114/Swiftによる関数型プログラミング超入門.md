slidenumbers: true

# [fit] Swiftによる<br>関数型プログラミング<br>超入門

<br>
### 2015年11月14日
### 藤本尚邦 / Cocoa勉強会(関東) #75

---
# 自己紹介

- 藤本尚邦 (@fhisa)
- [https://github.com/fhisa](https://github.com/fhisa)
- フリーランスプログラマー
- RubyCocoaフレームワーク原作者
- Mac開発歴、薄く長く約25年
- iOS開発歴、約1年

![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)


---
# アジェンダ

- 始めに
- 総和・総乗
- 総和・総乗の抽象化
- SequenceTypeプロトコル
- 関数型プログラミングとは
- まとめ

^
まず、総和・総乗のプログラムを具体例としてお話します。
次に、Swiftで重要なSequenceTypeプロトコルを取り上げます。
最後に、関数型プログラミングとは何かについて、私が学んだ経験について簡単にお話します。

---
## 始めに

- 関数型プログラミング[^1]ってよく聞くけど何それ？
- C言語には関数があるから関数型プログラミングができるの？
- `map` とか `reduce` って何？どうやって使うの？

このような立ち位置の人を想定して、関数型プログラミング風の考え方の一端を紹介します。

[^1]: 英語で _Fanctional Programming_ といいます。短縮して _FP_ と呼ばれています。

^
以降、省略してFPと呼ばせていただきます。

---
## 始めに

私自身は、語れるほど_FP_を理解しているわけではありません。ただし、以前自分でもそれと気付かずに_FP_を学んだことがあります。

その経験をもとに発表の構成を考えました。

^
なんだそんなことか、という程度の話なので、気軽に聞いていただければと思います。
^
ここにはFPに詳しい方がいらっしゃるかもしれません。
どうぞ生暖かい目で見守りながら、用語・言葉の使い方など、こりゃひどいと思ったら適当にツッコんでください。
^
ひとつ種明かししておくと、僕が学んだ本によるFPの定義では、
FPの「関数」は数学的な意味の関数のことです。
C言語やSwiftの「関数」とは意味がちょっと違います。
その本では、C言語やSwiftの「関数」に相当するものは「手続き」と呼ばれています。

---
# 総和・総乗

^
本当は、iOS や Mac ならではの例で説明したかったのですが、
うまい例を思い付きませんでした。
^
関数型プログラミングの入門にありがちなつまらない題材になってしまいますが、
1からnまでの自然数の和および積を計算するプログラムを具体例として考えていきます。

---
## 総和・総乗

定義１:

$$
f(n) = \sum_{i=1}^{n} i = 1 + 2 + 3 + \cdots + n
$$

$$
f(n) = \prod_{i=1}^{n} i = 1 × 2 × 3 × \cdots × n
$$

^
数学の教科書で、総和・総乗がこんな風に定義されているのを見たことがあると思います。
この関数をSwiftでプログラムしてみましょう。

---
## 総和・総乗

```Swift
func sum(n: Int) -> Int {
    var accumulator = 0
    for i in 1...n { accumulator += i }
    return accumulator
}
```

```Swift
func prod(n: Int) -> Int {
    var accumulator = 1
    for i in 1...n { accumulator *= i }
    return accumulator
}
```

ループと代入を使った手続き的なプログラム

^
さっきの定義から素直に思いつくのは、こんな感じのプログラムではないかと思います。
^
C, Pascal, FORTRAN, BASICなどの手続き型プログラミング言語の範疇にはいる言語を
を主に使っている方が、一番普通にプログラムしてもこれに近い感じになると思います。

---
## 総和・総乗

定義２:

$$
f(n) = \begin{cases}
0 & \text{if } n = 0 \\
n + f(n - 1) & \text{if } n > 0
\end{cases}
$$

$$
f(n) = \begin{cases}
1 & \text{if } n = 0 \\
n × f(n - 1) & \text{if } n > 0
\end{cases}
$$

^
さきほどの定義1はループをイメージさせるものでしたが、
こういう総和・総乗の定義も見たことがあるのではないでしょうか？
これは関数自身を使って関数を定義する再帰的な定義になっています。

---
## 総和・総乗

```Swift
func sum(n: Int) -> Int {
    return n == 0 ? 0 : n + sum(n - 1)
}
```

```Swift
func prod(n: Int) -> Int {
    return n == 0 ? 1 : n * prod(n - 1)
}
```

再帰呼出を使った_FP_の香り漂うプログラム

^
単なる再帰呼出しを使ったプログラムだけど、ここに関数型プログラミングの基本的な要素が含まれています。
^
今回、説明をシンプルにするために単純な再帰呼出しのプログラムにしましたが、
現実世界で使うには、このプログラムには問題があります。
というのは、この単純な再帰呼出はnのサイズに比例してスタックを食い潰すからです。
コンピュータサイエンス用語では、空間計算量がO(N)といいます。
前出のループによるプログラムは、空間計算量がO(1)、つまりnのサイズにかかわらずメモリ消費量は変わらりません。
^
FPに詳しい人は、ここで末尾呼出最適化(Tail Call Optimization、略してTCO)という言葉が思い浮かんだかもしれませんが、
ここではそちらには深入りしません。

---
## 総和 (番外編)

ガウス(小３)による総和のプログラム:

```Swift
func sum(n: Int) -> Int {
    return n * (n + 1) / 2
}
```
<br>
総和についてはガウスの方法が賢いやり方だが、総乗への応用は多分きかない。

^
ガウスが小学３年生のときの授業で、1から10までの和を解く問題を出されて、
他の子が律儀に足し算してる最中に、この方法でささっと解いて
先生を驚かせたという逸話がありますね。

---
# 総和・総乗の抽象化

^
総和・総乗の定義にしてもプログラムにしても、パターンが似ていることにお気づきかと思います。
パターンが似ているときにすることといえば抽象化ですね。
再帰呼出しによる総和・総乗のプログラムを題材に抽象化してみましょう。

---
## 総和・総乗の抽象化

２つの関数の似ている点・違う点を探してみよう:

```Swift
func sum(n: Int) -> Int {
    return n == 0 ? 0 : n + sum(n - 1)
}
```

```Swift
func prod(n: Int) -> Int {
    return n == 0 ? 1 : n * prod(n - 1)
}
```

---
## 総和・総乗の抽象化

２つの関数の違う点:<br>

- `n==0`のとき、前者は0を返し後者は1を返す
- `n!=0`のとき、前者は足し算で後者は掛け算

^
この２点だけです。あとは同じ。

---
## 総和・総乗の抽象化

元の引数nに加えて、総和・総乗で違う以下の２点:

- `n==0`のときの値 (初期値)
- `n!=0`のときの計算 (関数)

<br>
を引数にすれば、同じパターンの計算に使えるより汎用的な関数が出来上がるはずです。

---
## 総和・総乗の抽象化

```swift
typealias IntCombinator = (Int, Int) -> Int

func reduce(n: Int, initial: Int, combine: IntCombinator) -> Int
{
    if n == 0 {
        return initial
    } else {
        return combine(
            n,
            reduce(n - 1, initial: initial, combine: combine))
    }
}
```

総和・総乗のパターンを抽象化した関数 `reduce`

^
その汎用的な関数がこのreduceになります。
引数 initial が n == 0 のときの初期値で、
引数 combine が n != 0 のときの計算です。
一見、複雑になったように見えるかもしれませんが、
再帰呼出し版の sum や prod とよく見比べてもらえば、
違っている点として見つけた部分が引数になっているだけで
同じ形をしていることがわかるはずです。

---
## 総和・総乗の抽象化

関数reduceを使って総和・総乗を計算してみよう:

```swift
// ふたつの整数の和算・乗算をする関数を定義
func add2(n: Int, m: Int) -> Int { return n + m }
func mul2(n: Int, m: Int) -> Int { return n * m }

reduce(10, initial: 0, combine: add2)
reduce(10, initial: 1, combine: mul2)
```

---
## 総和・総乗の抽象化

関数reduceを使って総和・総乗を計算してみよう(2):

```swift
// 無名関数(関数リテラル)を直接渡す
reduce(10, initial: 0, combine: { $0 + $1 })
reduce(10, initial: 1, combine: { $0 * $1 })

// Ruby風シンタックスシュガー
reduce(10, initial: 0) { $0 + $1 }
reduce(10, initial: 1) { $0 * $1 }
```

^
もしかすると、FPに慣れていない人の場合、
前のスライドのように和算・乗算をする関数に名前をつけて定義したくなるかもしれませんが、
FP的には、このようなその場限りの関数にはいちいち名前を付けて定義せず、
無名関数(関数リテラルともいう)として直接引数に渡します。
^
このあたりの感覚は、JavaScriptやRubyなど無名関数やブロック構文のある
プログラミング言語に慣れ親しんでいるプログラマの方なら
自然と身についているかもしれませんね。

---
## 総和・総乗の抽象化

Swiftでは `+` や `-` も関数なので引数に渡せます:

```swift
// モダンな関数型プログラミング風
reduce(10, initial: 0, combine: +)
reduce(10, initial: 1, combine: *)
```

<br>
`add2` や `mul2` のような関数を定義する必要はありませんでした。

^
最初に、add2 や mul2 という和算・乗算をする関数を定義しましたが、
そもそもSwiftでは + や * 自体が
同じ計算をするファーストクラスオブジェクトの関数なので
直接渡せばよかったのです。

---
## 総和・総乗の抽象化

ここまで、関数 `reduce` の定義にいたる道筋を示すことで、関数型プログラミングの考え方の土台を簡単に紹介しました。

今回は、関数を引数にとる関数を題材にしましたが、関数を返す関数を使ったプログラミングなどさらに奥深い世界が存在しています。

^
また、計算対象をIntに限定しましたが、Swiftでは型も引数にできるので
そちら方向にも reduce のさらなる抽象化の余地があります。

---
# SequenceTypeプロトコル

^
ここからは、SwiftのFP的な活用方法の代表例として
SequenceTypeプロトコルの使い方をざっと見ていきます。

---
## SequenceTypeプロトコル

```swift
for item in collection {
    statements
}
```

Swiftの For-In構文では、繰り返しの対象となる `collection` が `SequenceType` プロトコルに適合している必要があります。

---
## SequenceTypeプロトコル

代表的なインスタンスメソッド:

- `filter`
- `map`
- `reduce`
- `sort`

^
さきほど定義したreduceも、実は SequeceType のメソッドとしてすでに存在しています。
^
さっき定義したreduce関数は説明のために定義したもので、
空間計算量がO(N)の問題のある実装になっています。
自分で定義せずに、こちらを使うようにしましょう。

---
## SequenceTypeプロトコル

### `filter`

引数で与えられた関数で各`item`を調べ、`true`を返した`item`のみによる配列を返します。

```swift
(1...10).filter { $0 % 2 == 0 }  // => [2, 4, 6, 8, 10]
(1...10).filter { $0 % 2 != 0 }  // => [1, 3, 5, 7, 9]
```

---
## SequenceTypeプロトコル

### `map`

数学で写像(mapping)と呼ばれているものに相当します。各`item`に関数を適用した結果の配列を返します。

```swift
(1...5).map { $0 * 2 } // => [2, 4, 6, 8, 10]
(1...5).map { String($0) } // => ["1", "2", "3", "4", "5"]
(1...5).map(String.init)  // => ["1", "2", "3", "4", "5"]
```

---
## SequenceTypeプロトコル

### `reduce`

現実的・汎用的な本物の`reduce`です[^2][^3]。

```Swift
(1...10).reduce(0, combine: +) // Swift的な総和の計算
(1...10).reduce(1, combine: *) // Swift的な総乗の計算

["aa", "bb", "cc"].reduce("") { "\($0)/\($1)" }
  // => "/aa/bb/cc"
```

[^2]:`SequenceType`では`item`の型も変数なので、`Int`型に限定せず扱うことができます。

[^3]:言語によっては `inject` (Ruby, Smalltalkなど) や `fold` (Haskellなど) と呼ばれています。

---
## SequenceTypeプロトコル

### sort

`item`を比較関数で並べ変えた配列を返します。`item`が`Comparable`プロトコルに適合している場合は、比較関数を省略できます。

```Swift
[5,4,3,2,1].sort()  // => [1, 2, 3, 4, 5]
(1...5).sort(>)  // => [5, 4, 3, 2, 1]
```

---
## SequenceTypeプロトコル

次のようなPersonと名簿が定義されているとします:

```swift
struct Person {
    let name: String
    let age: Int
    let sex: Sex
    enum Sex { case Male, Female, Others }
}

let BABYMETAL: [Person] = [
    Person(name: "中元すず香", age: 17, sex: .Female),
    Person(name: "菊地最愛", age: 16, sex: .Female),
    Person(name: "水野由結", age: 16, sex: .Female),
    Person(name: "青山英樹", age: 29, sex: .Male),
    Person(name: "BOH", age: 33, sex: .Male),
    Person(name: "大村孝佳", age: 31, sex: .Male),
    Person(name: "藤岡幹大", age: 34, sex: .Male),
]
```

---
## SequenceTypeプロトコル

```swift
//: BABYMETAL女子の名前
BABYMETAL
    .filter { $0.sex == .Female }
    .map { $0.name }
```

```swift
//: BABYMETAL全員の平均年齢
Float(BABYMETAL
    .map { $0.age }
    .reduce(0, combine: +)) / Float(BABYMETAL.count)
```

^
このコードは、ループをそれぞれ2回、回していて効率が悪そうに見えます。
ループが逐次的に実行されることが前提であれば確かにそうです。
しかし、このコードには代入もなく、
Objective-Cメソッド呼び出しのような実行時バインディングもありません。
将来コンパイラが賢くなれば
各ループを並列実行できるよう最適化できるようになる余地がありそうです。

---
# 関数型プログラミングとは

^
ここまで説明しておいてなんですが、実はよくわかりません。
なので、ここからは話がgdgdになります。すみません。

---
## 関数型プログラミングとは

Swiftの特徴[^4]:

- 関数がファーストクラスオブジェクト
  - 関数を引数や返り値として扱える
- 関数と普通の変数の名前空間が一緒

[^4]: JavaScript はこの特徴を持つプログラミング言語の代表例

---
## 関数型プログラミングとは

> 代入を使わない限り、同じ引数による同じ手続きの呼び出しは２回評価しても同じ結果になるので、手続きは数学関数の計算とみなすことができる…まったく代入を使わないプログラミングは、そのため**関数型プログラミング**(*functional programming*)と呼ばれている。
-- Structure and Interpretation of Computer Programs (真鍋宏史日本語訳版) より

^
これは、日本語名「計算機プログラムの構造と解釈」通称SICPと呼ばれている
コンピュータサイエンスの入門書から引用したFPの定義です。
^
最初に話した「自分でもそれと気付かずに関数型プログラミングの基礎を学んだことがある」
というのは10年くらい前の数年間「素人くさいSICP読書会」という会で
SICPを読んでいるうちに気づいたらFPの基本を学んでいたということです。
^
ここで言う「手続き」というのは、C言語やSwiftの関数に相当するものです。
SICPで用いられているSchemeというプログラミング言語では、
CやSwiftの関数に相当するものを「手続き」(procedure)と呼んでいます。
^
正直なところ、この定義は、自分にはいまひとつピンときてないところもあります。
^
MITの教養過程で使われていたコンピュータサイエンスの入門書。
入門書なのにとてもディープ。
このSICPという本はFP抜きにしても、
とてもプログラミングやコンピュータサイエンスについて知見の深まる本なので
簡単に内容を紹介します。

---
## 関数型プログラミングとは

「計算機プログラムの構造と解釈」(通称SICP):

1. 手続きを用いた抽象化の構築 (高階関数)
2. データを用いた抽象化の構築 (データ構造)
3. モジュール性、オブジェクト、状態 (代入初登場)
4. メタ言語抽象化 (プログラミング言語の実装)
5. レジスタマシンによる計算 (CPU・コンパイラ)

^
SICPでは、1章で今回やったような関数を引数や返り値として扱う関数、
つまり高階関数について取り上げています。
おもしろいのは、この本では3章で初めて代入が導入されることです。
^
私を含め、ほとんどのプログラマはプログラミングに入門してすぐに代入を知り、
それをあたり前のように受け入れたと思うのですが、
この本を読み進めていくと、本当は代入がなくても、
1,2章で見たようにかなりのことができてしまうことに、
3章で代入が導入されたところで気づくわけです。
3章では、代入のメリットとともに、代入がいかにプログラムを複雑にするかというデメリットに
ついても語られています。
^
4章では、プログラミング言語処理系(インタープリタ)の実装がテーマになっています。
普通のLISPインタープリタの他に、遅延評価タイプのLISPインタープリタや
非決定性計算、推論・論理プログラミングの言語処理系。
5章は、CPU(レジスタマシン)やコンパイラなど。
私は、5章の途中で病気により読書会から脱落しました。

---
# まとめ

Swiftは手続き型プログラミングと関数型プログラミングの双方のメリットを利用できるバランスの良い言語です。大いに活用しましょう！

^
OOPを含め手続き的プログラミングはプログラミングパラダイムのひとつに過ぎません。
慣れ親しんだパラダイムだけにとらわれず、頭をまっさらにして
関数型プログラミングの世界を覗けば見えてくるものがあるかもしれません。
私はそのことをSICPを読むことで学びました。

---
# 参考文献

- [Structure and Interpretation of Computer Programs (SICP) 真鍋宏史日本語訳版](https://github.com/hiroshi-manabe/sicp-pdf)
- [The Swift Programming Language (Swift 2.1)](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/#//apple_ref/doc/uid/TP40014097-CH3-ID0)
- [Swift Standard Library Reference](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/SwiftStandardLibraryReference/index.html#//apple_ref/doc/uid/TP40014608)

---
# [fit] **Thank you!**
![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)

### 2015年11月14日
### 藤本尚邦 / Cocoa勉強会(関東) #75
