---
layout: post
title: "クラスにおける Failable Initializerについて"
date: 2016-02-02 13:58:13 +0900
comments: true
categories: 
- swift
---



Swift で，構造体やenum，クラスにおいて，初期化が失敗しうるコンストラクタを利用したい時には，*Failable Initializer* を利用すれば良いらしいのだが，これをクラスで利用しようとしたらハマったのでメモ．

<!-- more -->

##Failable Initializer とは

初期化に失敗したことを，`nil` を返すことで伝えることのできるイニシャライザ．

``` swift
struct Animal {
    let species: String

    // failable initializer
    // init の末尾に ? を付加する
    init?(species: String) {
        if species.isEmpty {
            // 初期化失敗の場合には nil を返す
            return nil
        }
        self.species = species
    }
}
```

これで，初期化がうまくいかなかった場合には，`nil` が返される．

##ハマったこと

やろうとしたことは，コンストラクタ内で例外を扱うことで，具体的には以下のような感じ．

``` swift
class Hoge {
    private var foo1: Int!
    private var foo2: Int!

    init?() {
        do {
            foo1 = try /* 例外を投げうる処理 */
            foo2 = 0   /* 単純な代入 */
        } catch {
            return nil
        }
    }
}
```

ところが，このように記述すると，`return nil` の位置で以下のようなエラーが発生する．

```
All stored properties of a class instance must be initialized before returning nil from an initializer
```

同じことでハマった人がいるらしい．

> [All stored properties of a class instance must be initialized before returning nil from an initializer](http://stackoverflow.com/questions/26495586/best-practice-to-implement-a-failable-initializer-in-swift)

結論から言うと，値型である構造体や列挙型ではいかなるタイミングでも初期化を失敗(`return nil`)できるが，クラスについては，すべての stored property が明示的に初期化された後でなければ初期化を失敗させることができないらしい．

>  A failable initializer for a value type (that is, a structure or enumeration) can trigger an initialization failure at any point within its initializer implementation. In the Animal structure example above, the initializer triggers an initialization failure at the very start of its implementation, before the species property has been set.
> For classes, however, a failable initializer can trigger an initialization failure only after all stored properties introduced by that class have been set to an initial value and any initializer delegation has taken place.
> [The Swift Programming Language (Swift 2.1): Initialization](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID224)

##動作を確認してみる

Playground でサンプルコードを動かして動作を確かめてみる．
まずは，構造体の場合．

``` swift
// 構造体
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty { return nil }
        self.species = species
    }
}

let hoge: String = /* ここに任意の値 */
let anonymousCreature = Animal(species: hoge)
if anonymousCreature == nil {
    print("NG") /* hoge="" の時 */
} else {
    print("OK") /* hoge="任意の文字列" の時 */
}
```

次に，クラスの場合．

``` swift
class Product {
    let name: String!
    init?(name: String) {
        self.name = name
        if name.isEmpty { return nil }
    }
}

let hoge: String = /* ここに任意の値 */
if let bowTie = Product(name: hoge) {
    print("The product's name is \(bowTie.name)") /* hoge="任意の文字列" の時．The product's name is <任意の文字列> */
} else {
    print("NG") /* hoge="" の時 */
}
```

公式ドキュメント曰く，stored property である name は `String!` として宣言する．すると，デフォルト値として `nil` が格納されるが，初期化成功時には，stored property が nil かどうかを気にせずにアクセスしたい．(stored property に対する nil チェックが必要ないようにしたい)ので，クラスでは全ての stored property を初期化してから return nil する必要があるそうだ．
でも，コンストラクタの返り値が nil であったならどちらにしろプロパティにアクセスはしないわけで，なぜ全プロパティを初期化してから nil を返す必要があるのか，いまいちわからなかった．


##参考
[Failable Initializers - Swift Blog - Apple Developer](https://developer.apple.com/swift/blog/?id=17)
