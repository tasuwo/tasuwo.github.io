---
layout: post
title: "[Swift] 関数とクロージャについて"
date: 2015-05-29 15:46:10 +0900
comments: true
categories: swift
---

クロージャって何？？？と思ったので．

<!-- more -->

##関数

クロージャについてまとめる前に，まず関数について理解する．

###定義
`func`で宣言し，返り値は`->`を用いる．
返り値は複数指定可能．
```swift
func testFunc() -> (first:Int, second:Int) {
    return (10, 20)
}
```

###特徴

Swiftの関数には以下の特徴がある．

1. 可変数の引数を指定可能
2. ネストして宣言可能
3. 引数に指定可能
4. 返り値に指定可能

```swift
///////////////////////////
// 1. 可変数の引数を指定可能
//    引数を配列で指定できる．
func sumOf(numbers: Int...) -> Int {
    var sum = 0
    for number in numbers {
        sum += number
    }
    return sum
}
sumOf(10, 11)        // 21
sumOf(42, 597, 12)   // 651

///////////////////////////
// 2. ネストして宣言可能
//    ネストした関数は，外の関数の変数にアクセス可能
func returnFifteen() -> Int {
    var y = 10

    func add() {
        y += 5
    }
    add()

    return y
}
returnFifteen()     // 15

///////////////////////////
// 3. 引数に指定可能
// 数値のリストと条件となる関数を与えると，
// 条件にマッチした数値がリスト内に存在するか調べる関数．
func hasAnyMatches(list: [Int], condition: Int -> Bool) -> Bool {
    for item in list {
        if conition(item) {
            return true
        }
    }
    return false
}
// 10より小さい値ならば true
func lessThanTen(number: Int) -> Bool {
    return number < 10
}
var numbers = [20, 19, 7, 12]
hasAnyMathces(numbers, lessThanTen)    // true
numbers = [20, 19, 17, 12]
hasAnyMathces(numbers, lessThanTen)    // false

///////////////////////////
// 4. 返り値に指定可能
func makeIncrementer() -> (Int -> Int) {
    // ネストした関数
    func addOne(number: Int) -> Int{
        return 1 + number
    }
    // ネストして宣言された関数をそのまま返す
    return addOne
}
```

##クロージャとは？

実行可能なコードブロックのこと...?
一言で言い表そうとすると難しい．公式では，`関数は再利用が可能な特別なクロージャである．`と言われている．

###宣言

とりあえず，書き方を見てみる．
名前が省略可能で，`{}`で囲んで記述する．引数と返り値の後に`in`を記述してから本体を記述する．

```swift
// クロージャ
var greetClosure: (String) -> String
paramClosure = {
    name -> String in
    return "Hello " + name + "."
}
// 関数でも同様の振る舞いを定義をしてみる
func greetFunc(name: String) -> String {
    return "Hello " + name + "."
}

// 実行結果はどちらも変わらない．
println(greetClosure("tasuwo"))
println(greetFunc("tasuwo"))
```

これだけだと，クロージャの意味がイマイチわからない．つまり，なぜ関数じゃダメなのか？
自分なりの解釈だが，クロージャの良い点は**複数の処理の記述をコンパクトにまとめられること**なのではと思う．
例えば，`map`を例にとって考える．
`map`はクロージャを引数にとり(つまり，関数を与えても問題はない)，配列の各値にその関数を適用・変換する．

```swift
var numArray = [10, 15, 1]
// 配列の各値を三倍にする
numArray.map({
    (number: Int) -> Int in
    let result = 3 * number
    return result
})
```

クロージャは，引数や返り値の型が自明である時，その指定を省略可能である．また，返り値が1つに減退されている場合には，`return`も記述しなくて良い．
よって，以下のように記述を省略できる

```swift
numArray.map({
    number in 3 * number
})
```

記述がかなりコンパクトになったし，何をしているのかも一目見れば大体わかる．
また，第二引数を以下のように外に出す書き方もできるそうだ．自分はこれを知らなくて，以下のような記述を見るたびに(なんだこれは...?)と頭をひねっていた．

```swift
numArray.map(){
    number in 3 * number
}
```

もう一つの例を見てみる．
`sorted`は，与えられた配列を並び替える．クロージャを引数にとると，その内容に従って並び替える．

```swift
var numArray = [10, 15, 1]
let sortedNum = sorted(numArray){
    (str1: Int, str2: Int) -> Bool in
    return str1 > str2
}
```

引数，返り値の型と，`return`を省略する．

```swift
let sortedNum = sorted(numArray){
    str1, str2 in str1 > str2
}
```

さらに，自分自身が引数となっている時，自身の引数を`$0,$1...`で記述可能．

```swift
let sortedNum = sorted(numArray){$0 > $1}
```

これでかなり省略できる．

#参考文献
>[Swift さくっと確認したい基礎文法 クロージャ(closure)
](http://qiita.com/yuinchirn/items/2ebb6fed6de0c9c1c3c9)
>[swift Sort関数とClosure](http://qiita.com/mst/items/b18e9531ac0cbdf2f3c3)
>[A Swift Tour](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/GuidedTour.html)
>[iOS Swiftのクロージャを使う](http://chicketen.blog.jp/archives/14886216.html)
