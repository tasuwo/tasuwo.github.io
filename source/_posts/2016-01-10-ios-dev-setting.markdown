---
layout: post
title: "iOSアプリ開発時の個人的に好みなディレクトリ構成とか xib ファイルの使い方とか"
date: 2016-01-10 20:47:24 +0900
comments: true
categories: 
- iOS
- Swift
- xcode
---

自分が iOS アプリケーションを作るとき使いまわそうと思った，テンプレ的な設定とかをまとめておく．

<!-- more -->

##やりたいこと

やることは以下．

1. ディレクトリ構成をきれいにする
	* MVCアーキテクチャに対応させる
2. Storyboard を削除する
	* segue とかで画面遷移させたり，一つの storyboard に複数の view controller が対応しているのは管理しづらくなりそうかな，と思ったので
3. UI 整形に xib ファイルを使う
	* UIView を各画面ごとに作成し，対応した xib ファイルを UI 整形用に使う

##0. 環境

* Xcode7.2
* Swift2.0
* OS X Yosemite

##1. ディレクトリ構成をきれいにする

iOS アプリケーションは MVC アーキテクチャに則っているので，対応したディレクトリ構成にする．`File > New > Project` から `Single View Application` を作成すると，デフォルトのディレクトリ構成は以下のようなかんじ．

![default.png](/images/20160110_default.png)

これを，以下のようなディレクトリ構成に変更する．

```
┬ /resources
│  ├ info.plist
│  └ Assets.xcassets
└ /src
   ├ AppDelegate.swift
   ├ /model
   ├ /view
   └ /controller
       └ ViewController.swift
```

実際のディレクトリ構成と Xcode のカラム上でのディレクトリ構成は異なるので，その同期を撮りたいときには [venmo/synx](https://github.com/venmo/synx) を使うと良い．

##2. Storyboard を削除する

デフォルトで存在する，`Main.storyboard`, `LaunchScreen.storyboard` は削除する．削除するだけだとエラーとなってしまうので，以下のようにプロジェクトの設定を変更する．

* `Deployment Info > Main Interface` のテキストフィールドを空にする
* `App Icons and Launch Images > Launch Screen File` のテキストフィールドを空にする

これでプロジェクトから storyboard を除くことはできたが，この時点で Run しても画面に何も表示されない．rootViewConstroller を設定できていないためである．

![black.png](/images/20160110_black.png)

そこで，`AppDelegate.swift` に以下を追記することで，ViewController を rootViewController に設定する．

```swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        
        window = UIWindow(frame: UIScreen.mainScreen().bounds)
        if let window = window {
            window.backgroundColor = UIColor.whiteColor()
            window.rootViewController = ViewController()
            window.makeKeyAndVisible()
        }
        
        return true
    }
```

これでOK．

![white.png](/images/20160110_white.png)



##3. UI 整形に xib ファイルを使う

まず，以下のような `MainView.swift` を `src/view` 以下に作成する．

```swift
import UIKit

class MainView : UIView {
}
```

次に，`New file` から `iOS > User Interface` 内の `view` を選択し， xib ファイル作成する．その `File's Owner` の Custom Class を `Main View` に設定

![mainview.png](/images/20160110_mainview.png)

この UI が適用されたことがわかるように，適当にラベルを設置しておく．

![main.png](/images/20160110_main.png)

xib ファイル側の view を `MainView.swift` に対応させる．

![outlet.png](/images/20160110_outlet.png)

MainView.swift を以下のように編集する．

```swift
import UIKit

class MainView : UIView {
    @IBOutlet var MainView: UIView!
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        
        NSBundle.mainBundle().loadNibNamed("MainView", owner: self, options: nil)
        MainView.frame = frame
        
        addSubview(MainView)
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

さらに，`ViewController.swift` に以下を追記

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    let view = MainView(frame: self.view.frame)
    self.view.addSubview(view)
}
```

これで Run すればよい．

![mianvi.png](/images/20160110_mianvi.png)



##捕捉 : Auto layout

中央揃えしたいとかそういうのは auto layout を活用すれば良い．以下のサイトが参考になる．

[iOS - xcode6でAutoLayoutでレスポンシブデザイン - Qiita](http://qiita.com/teradonburi/items/94b89379aa5a0bfdc71d)
[AutoLayout - Swiftサラリーマン](http://swift-salaryman.com/autolayout.php)

ラベルから Ctrl 押しながら 親view へドラッグ

![ auto.png](/images/20160110_auto.png)

制約を適当に付加する．

![constraint.png](/images/20160110_constraint.png)

今回はこんな感じ．

![cons.png](/images/20160110_cons.png)

これで Run すると中央揃えになる．

![mmm.png](/images/20160110_mmm.png)

##おわりに

最終的なディレクトリ構成は以下．

![V.png](/images/20160110_V.png)

##参考
[【iOS】【swift】カスタムViewとxibを紐付ける - tanihiro.log](http://tanihiro.hatenablog.com/entry/2015/10/13/092710)
[[Swift]xibファイルを呼び出す最も簡単な方法 - Qiita](http://qiita.com/iKichiemon/items/3cfa6c2bf2a0acb299a0)
[Swift+xibで簡単レイアウトでカスタムビュー - Qiita](http://qiita.com/noppefoxwolf/items/11401622950768c93fd2)
[XCode7 - Storyboardにxib利用 - Qiita](http://qiita.com/MTattin/items/61beb3b4afcc779f707f)