---
layout: post
title: "OpenCV を Homebrew から導入し直して Xcode に設定し直した"
date: 2016-02-16 10:19:02 +0900
comments: true
categories:
- homebrew
- opencv
- xcode
---

既に OpenCV を導入し Xcode から利用していたが、Homebrew で設定しなおす機会があったので、やったことをメモしておく。こうしたらできた！というメモなので、コマンド実行等は自己責任でお願いします。

<!-- more -->

環境は以下です．

* Max OS X 10.10.1 Yosemite
* Xcode 6.1.1
* OpenCV 2.4.12

##導入

事前に `brew doctor` を解決しておくと良いかも。問題がなければ、opencv を導入する。

```bash
$ brew tap homebrew/science
$ brew install opencv
```

既に OpenCV を導入している場合、過去のファイル群は邪魔なので撤去しなくてはならない。`brew doctor` 時点で影響を及ぼしそうなファイルはリストアップされるので、それらを削除するなりバックアップをとるなりしてから、シンボリックリンクを貼ると良いかも。

```bash
$ brew link opencv
```

`--force` オプションをつけてね〜的な warning が出るかもしれない。その場合は付加すれば実行できる。ただし、既存の OpenCV のシンボリックリンクを上書きすることになるので、注意。

##Xcode で使う

Homebrew で導入した OpenCV の場所は `/usr/local/Cellar/opencv` となる。Xcode から利用する場合は、適切な場所を参照するように設定を行う必要がある。以下、パス内のバージョンに関しては適宜置き換えること(今回の場合は2.4.12_2)。

###ヘッダー及びライブラリの Search Paths

- `Build Settings` > `Header Search Paths`
	- `/usr/local/Cellar/opencv/2.4.12_2/include`
	- `/usr/local/Cellar/opencv` にして、recursive に設定すると説明しているサイトがあったが、こちらの方が綺麗に見える
- `Build Settings` > `Library Search Paths`
	- `/usr/local/Cellar/opencv/2.4.12_2/lib`

OpenCV 導入済み & Xcode 設定済みだった場合は、既存の設定を削除するか、上記で設定したパスの優先度をあげる(リスト内で上方に移動する)と良いかもしれない。

###ライブラリとのリンク

OpenCV をすでに導入し、Xcode で設定済みであった場合は、`Build Settings` > `Linking` > `Other Linker Flags` にいろいろ設定されているかもしれないが、全部消す(と、自分の環境ではエラーがなおった)。

Homebrew で導入後にライブラリとのリンクを張るためには `Build Phases` > `Link Binary With Libraries` に、`/usr/local/Cellar/opencv/2.4.12_2/lib` から必要なライブラリを追加する。`Shitf + Command + G` から `/usr/local/Cellar/opencv/2.4.12\_2/lib` と入力すれば楽。

##トラブルシューティング

###Undefined symbols
`Undefined symbols for architecture x86_64:` みたいなエラーがドバッと出ることがある。

[c++ - OpenCV Undefined symbols for architecture x86_64: error - Stack Overflow](http://stackoverflow.com/questions/24985713/opencv-undefined-symbols-for-architecture-x86-64-error)

OpenCV に必要なライブラリがリンクされていないことによるエラーのため、Link Binary With Libraries で必要なライブラリがリンクされているか確認する。

###dyld: Library not loaded

Xcode における Build & Run 後に、以下のようなエラーが吐かれた。

```bash
dyld: Library not loaded: /usr/local/opt/libpng16.16.dylib
  Referenced from: /usr/local/opt/opencv/lib/libopencv_highgui.2.4.dylib
  Reason: Incompatible library version: libopencv_highgui.2.4.dylib requires version 33.0.0 or later, but libpng16.16.dylib provides version 32.0.0
Trace/BPT trap: 5
```

`/usr/local/opt` の libpng を読みに行ってるのが悪いのかな？そもそも `/usr/local/opt` の OpenCV 見に行ってるけどなんでかな？と思っていたが、どうやら試した環境では、 `/usr/local/opt` 内の libpng、 opencv は Cellar 以下の各ソフトウェアへのシンボリックリンクになってるらしかった。Homebrew がやってくれてるんだっけ？わからん...

バージョンが低いと言われているので、`brew update & brew upgrade` したら、とりあえず治ったけど、実際何が原因だったのかはわからなかった...

[c++ - Error with homebrew + opencv + libpng - Stack Overflow](http://stackoverflow.com/questions/28124359/error-with-homebrew-opencv-libpng)


##その他参考

[HomebrewとXcode5でつくるOpenCVの環境 – なんてこったいブログ](http://nantekottai.com/2014/04/16/opencv-xcode5-homebrew/)
