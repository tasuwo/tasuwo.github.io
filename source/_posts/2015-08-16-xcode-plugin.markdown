---
layout: post
title: "Xcode のプラグイン導入についてメモ"
date: 2015-08-16 16:08:37 +0900
comments: true
categories: 
---

メモしておこう．

<!-- more -->

##Alcatraz
パッケージ管理ツールをいれる．
パッケージのインストールや管理がラクになる．

>[supermarin/Alcatraz · GitHub](https://github.com/supermarin/Alcatraz)

以下を実行．

```bash
curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh
```

Xcode を起動すると以下のような画面が表示されるので，`Load Bundle`する．

![xcode_alcatraz](/images/xcode_alcatraz.png)

`Window > Package Manager` からパッケージをインストールできるようになる．

##いれたもの

とりあえず一通り見てみていれてみたもの．

### プラグイン
* BlockJump
* FuzzyAutocomplete
* VVDocumenter-Xcode
* XAlign

###テーマ
* Tomorrow Night


まだ全然使っていないので，使用感だとかどんなものだとかは気が向いたら書く．


##その他

他にも Xcode 使う上で知っておくといいことがいろいろあるらしいけど，気が向いたら読む．

>[Xcodeを便利に使って爆速開発という発表をしました - Think Big Act Local](http://himaratsu.hatenablog.com/entry/xcode)
