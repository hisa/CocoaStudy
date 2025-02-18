footer: 藤本尚邦 / Cocoa勉強会(関東) #74 2015年9月5日
slidenumbers: true

# iOSプログラミングでAutoLayoutを数式のように記述する方法

![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)

---
<!-- # iOS/Macアプリ開発の<br>AutoLayoutプログラミング<br>について考えてみた

![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)

--- -->
# 自己紹介

* 藤本尚邦 (@fhisa)
* [https://github.com/fhisa](https://github.com/fhisa)
* フリーランスプログラマー
* RubyCocoaフレームワーク原作者
* Mac開発歴、薄く長く約25年
* iOS開発歴、1年弱

![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)

---
# AutoLayoutの基本

あるビューの位置やサイズを、**別の何か**との関係式として定義すること:

```
       ビューXの縦幅 ＝ ビューYの縦幅 × A ＋ B
       ビューXの縦幅 ＝ ビューXの横幅
       ビューXの上端 ＝ 上部マージン + A
       ビューXの横幅 ＝ A
```
_(※ AとBは定数)_

---
# AutoLayoutの基本

別の何かは、ビューの位置やサイズとは限りません:<br>

* 自身または他のビューの位置や高さ
* マージン(UILayoutSupportプロトコル)
* なし(nil)

---
# AutoLayoutの基本

iOS/Mac OS Xプログラミングでは、この関係式をNSLayoutConstraintクラスのオブジェクトとして扱います。
つまり、NSLayoutConstraint(レイアウト制約)を定義することが、AutoLayoutプログラミングの基本となります。

---
# レイアウト例

![inline](layout_portrait_ok.pdf)

---
# AutoLayoutを定義する方法

1. storyboard/xibファイルをXcodeのGUIで編集
2. プログラムでNSLayoutConstraintを生成

---
# 質問です

* AutoLayoutを、Xcode GUIではなくプログラムで定義することはありますか？
* どんなときにプログラムで定義しますか？

---
# この発表ではプログラムでのNSLayoutConstraint定義に的を絞ります

---
# NSLayoutConstraintの生成方法

1. ビジュアル言語で制約を記述し、文字列引数として与えて生成
2. 制約に関係する全ての値を引数に与えて生成

---
# ビジュアル言語

* プログラム実行時に構文がチェックされる
 * つまり走らせてみなければわからない
* 覚えるのも面倒
* 実用に難ありなので、ここでは触れません

---
# ならば全ての値を引数渡し

```swift
class NSLayoutConstraint: NSObject {
    convenience init(item view1: AnyObject,
           attribute attr1: NSLayoutAttribute,
           relatedBy relation: NSLayoutRelation,
              toItem view2: AnyObject?,
           attribute attr2: NSLayoutAttribute,
          multiplier multiplier: CGFloat,
            constant c: CGFloat)
}
```

---
# [fit] **引数多すぎワロた**www

---
# [fit] 実際に使うとこんな感じ…

---
# NSLayoutConstraint

```swift
baseView.addConstraints([
    NSLayoutConstraint(
        item: mainView, attribute: .Height, relatedBy: .Equal,
        toItem: baseView, attribute: .Height, multiplier: 3.0 / 4.0, constant: 0),
    NSLayoutConstraint(
        item: mainView, attribute: .Leading, relatedBy: .Equal,
        toItem: baseView, attribute: .Leading, multiplier: 1, constant: 8),
    NSLayoutConstraint(
        item: mainView, attribute: .Top, relatedBy: .Equal,
        toItem: baseView, attribute: .Top, multiplier: 1, constant: 8),
    NSLayoutConstraint(
        item: mainView, attribute: .Trailing, relatedBy: .Equal,
        toItem: baseView, attribute: .Trailing, multiplier: 1, constant: -8),
    // 以下、長いので略
    ])
```

---
# [fit] **文字多すぎワロた**<br>＼(^o^)／
---
# ワロえない<br>\(´・ω・｀)

---
# **NSLayoutConstraint なんて<br>大っ嫌いなんだからねっ！**

# [fit] ٩(๑`^´๑)۶

---
# ？
![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)

---
# ！
![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)

---
# [fit] **！**
![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)

---
# こんな風に書けたら読み書きしやすいなぁ

```swift
baseView.addConstraints([
    mainView[.Height] * 4 == baseView[.Height] * 3,
    mainView[.Leading] == baseView[.Leading] + 8,
    mainView[.Top] == baseView[.Top] + 8,
    mainView[.Trailing] == baseView[.Trailing] - 8,
    // 以下も略さないよ
    infoView[.Top] == mainView[.Bottom] + 8,
    infoView[.Leading] == baseView[.Leading] + 8,
    infoView[.Trailing] == baseView[.Trailing] - 8,
    infoView[.Bottom] == baseView[.Bottom] - 8,
    ])
```

---
そこで奥さん！<br>
# [fit] **FormulaStyleConstraint**
<br>ですよ！

---
# FormulaStyleConstraint

NSLayoutConstraintを等式・不等式などの数式で定義できるようにするSwift用のフレームワーク

[https://github.com/fhisa/FormulaStyleConstraint](https://github.com/fhisa/FormulaStyleConstraint)

---
# FormulaStyleConstraintの使用例

```swift
baseView.addConstraints([
    mainView[.Height] * 4 == baseView[.Height] * 3,
    mainView[.Leading] == baseView[.Leading] + 8,
    mainView[.Top] == baseView[.Top] + 8,
    mainView[.Trailing] == baseView[.Trailing] - 8,
    infoView[.Top] == mainView[.Bottom] + 8,
    infoView[.Leading] == baseView[.Leading] + 8,
    infoView[.Trailing] == baseView[.Trailing] - 8,
    infoView[.Bottom] == baseView[.Bottom] - 8,
    ])
```

---
# FormulaStyleConstraintの特徴

* NSLayoutConstraintの数式による定義に目的を絞ったシンプルな構成
* ソースコードはたったの154行、実装の理解が容易です (バージョン1.2)

---
# FormulaStyleConstraintの課題

* UILayoutSupportプロトコルのサポート
 * マージン(topLayoutMargin, bottomLayoutMargin)を扱えない
 * Swift 2.0 の protocol extension でおそらく対応可能
* Mac OS Xのサポート

---
# FormulaStyleConstraintの競合品

**制約を数式で定義するというアイディア**は、誰かがすでに作ってる可能性大。しかし、自作する楽しみを味わいたかったので調べずに作りました。<br>
ひとまず完成してから調べたところ、やっぱりありました(´・ω・｀)

---
# Cartography

>Using Cartography, you can set up your Auto Layout constraints in declarative code and without any stringly typing!

[https://github.com/robb/Cartography](https://github.com/robb/Cartography)

---
# Cartographyの使用例

```swift
layout(baseView, mainView) {
    $1.height == $0.height * (3.0 / 4.0)
    $1.Leading == $0.leading + 8
    $1.top == $0.top + 8
    $1.trailing = $0.trailing - 8
    // 以下略
}
```

---
# Cartographyの特徴

* 制約の定義をブロック内に記述するDSLタイプ
* 整列(align)などの拡張機能あり
* ソースコード約1400行(バージョン0.5)
* 名前が短くてかっこいい
* GitHubのスター数約3000 (FormulaStyleConstraintは0、ゼロ、Nothing)

---
# 開発にあたっての感想など

* 初めてTravis CIを使ってみた(![inline](https://travis-ci.org/fhisa/FormulaStyleConstraint.png?branch=master)バッジを付けてみたかっただけ)
* 初めてCarthageに対応してみた(これは便利)
* 初めてプルリクをもらった (ただしボットにw)
* ひとり焼き肉、ひとりディズニーランド、ひとり美ら海水族館、ひとりGitHub

---
# まとめ

FormulaStyleConstraintにせよ、Cartographyにせよ、素でNSLayoutConstraintのコードを書くよりはるかに楽ちんなので、AutoLayoutをプログラムで書いている人にはたいへんオススメです！

* [https://github.com/fhisa/FormulaStyleConstraint](https://github.com/fhisa/FormulaStyleConstraint)
* [https://github.com/robb/Cartography](https://github.com/robb/Cartography)

---
# [fit] **Thank you!**
![](https://farm3.staticflickr.com/2949/15476722862_40e6f37f4f_z_d.jpg)
