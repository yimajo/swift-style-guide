Swiftコーディングスタイルガイド

# はじめに

これは株式会社キュリオシティソフトウェアiOSアプリ開発者のためのSwiftコーディング用のスタイルガイドです。

それぞれの項目には[理由]が書かれています。[理由]に対して正当性がない場合においてはスタイルに沿った例のようなコーディングをする必要はありません。

また、これらすべて必須の項目ではなくプロジェクトごとに最適な選択肢があれば、メンバー内で議論しそれを加味する事が前提です。

コーディングスタイルガイドはコーディング規約ではなくSwiftをよりよく書くため/リファクタリングするための推奨する書き方でありmustな項目は存在しません。

# コーディングスタイル

## インデントは4スペースに

* インデントは4スペースにしよう。ハードタブ(\t)は使わず、ソフトタブによる4スペースが好ましいです（Xcodeのデフォルトで問題ない）


***理由***

* ハードタブを使うとエディタによって見た目が変わったり、GitHubなどウェブサービス上での表示で思いもよらない形になります

## varとletの使い分け

基本的にはすべてletにより定義し、varは本当に必要がある場面に直面した際に利用します。

***理由***

letはその値が変更されないことを保証すると同時にそれを明示します。結果的にvarが使われているコードを読んだ際に、その値は状態により変化があるということを推測できます。

## コロンの後ろにはスペースを置く

```swift
let hoge: Int = 1
```

```swift
func removeItems(identifiers: [String]) {}
```


***理由***

明確な理由があるわけではないですが、Appleのサンプルで同じように使われていること、Objective-CやRubyなど":"が使われる際に同じようにスペースをおいていることが理由です。

## カンマの後ろにはスペースを置く

```swift
var dictionary: Dictionary<String, String> = [
    "title": "value"
]
```

***理由***

明確な理由があるわけではないですが、Appleのサンプルで同じように使われていること、Objective-CやRubyなど","が使われる際に同じようにスペースをおいていることが理由です。


## Optionalの変数は基本的に"?"を使い、初期化が遅れる場合に"!"を使う

nilを許容する変数や戻り値はOptionalとして"?"を宣言しましょう。

```swift
var hoge: Int? = piyo() // piyoの戻り値がnilの可能性がある 
```

しかし、すぐに初期化することができない変数の場合はImplicitly Unwrapped Optionalとして"!"を利用しましょう。

```swift
var fuga: Int!       // !を利用する
// ...この間色々あって
var foo = ...
// ...色々終わり
fuga = hogehoge()    // hogehogeの戻り値はnilの可能性はない
```

この例は極端ですが、頻繁にIBOutletで利用されています。

ただし、これには例外があり、変数の初期化をその場ですぐに出来ないことが明白である場合には"!"を使う必要はありません。

```swift
var fuga: Int
switch veggetable {
    case "celery":
       fuga = 1
    case "cucumber", "watercress":
       fuga = 2
    default:
       fuga = 3
}
```

具体的にはCellを条件によって変える場合に利用することが出来ます。

***理由***

Optionalな変数に対して安易に"!"を使って宣言してしまうと、変数を使う際、nilの場合にクラッシュします。"!"を使うメリットはunwrapして変数を取り出すことで都度都度チェックをする必要なく利用しやすくすることですが、"!"を使う場合を初期値がnilで必ず初期化される場合に限定することでそのメリットを活かすことができます。

## Forced UnwrappingよりOptional CHainingを使う

Optionalなオブジェクトに対して、そのメソッドを使いたい場合、"?"を使ってオブジェクトのメソッドを実行することが推奨されます。

```swift
let mySimpleObject: SimpleObject? = simpleObject()
mySimpleObject?.myMember?.exec()
```


***理由***

オブジェクトに"!"を付けることでForced Unwrappingでき同じように見えますが、nilの場合はクラッシュします。

## Optionalな変数をチェックして使う場合はOptional Bindingを使う


```swift
if let title = dictionary["title"] {
    // titleがnilでない場合の色々な処理...
}
```

## 変数をチェックしてすぐに処理を抜ける場合はguardを使う(Swift2.0から)

```swift
guard arg != 0 else {
    // 条件に当てはまらず0の場合はここでreturnして以降の処理をガードする
    return 
}

// ガードしたかった処理
let hoge = fuga / arg
// このあと色々続く
```

```swift
guard let title = dictionary["title"] else {
    // 条件に当てはまらずnilの場合はここでreturnして以降の処理をガードする
    return 
}

// ガードしたかった処理
let hoge = "[" + title + "]"
// このあと色々続く
```


## 引数がnil指定できるようにするならばOptionalにする

クロージャもOptionalにできる

```swift
class func removeAllItems(completionHandler: (() -> Void)?) {
    competionHandler?()
}
```

***理由***

クロージャの実行についても"?"をつけることで、nilの場合でもクラッシュしない効果を持つ。これはOptional Chainingの仕組みと同じ。


## 最後の引数がクロージャならTrailing Closureを使う

関数やメソッドの最後の引数がクロージャの場合、トレイリングクロージャで副作用なくコード量を減らすことが出来ます。

```swift
presentViewController(composeViewController, animated: true) {
    println("画面を表示した")
}
```

トレイリングクロージャを使わない例

```swift
presentViewController(composeViewController, animated: true, completion: {
    println("画面を表示した")
})
```

ただし複数のクロージャを引数に取る場合、Trailing Closureを使わない

***理由***

クロージャが複数ある場合にTrailing Closureを使うと引数の最後にあるクロージャが特別な形になってしまうのは統一性を崩してしまう。

## メソッド呼び出しやプロパティアクセスにselfを使うのは最低限に留める

***理由***

Objective-Cとは違い、selfを使わなくてもメンバやメソッドにアクセスできるため、必要になる場合に限りselfを使う。



# 参考

https://github.com/jarinosuke/swift-style-guide/blob/master/README_JP.md

https://github.com/dekatotoro/swift-style-guide/blob/master/ja_style_guide.md
