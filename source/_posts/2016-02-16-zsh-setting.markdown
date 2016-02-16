---
layout: post
title: "OS X でターミナルをきれいにする"
date: 2016-02-16 11:20:22 +0900
comments: true
categories:
- terminal
- zsh
---

ターミナルをきれいにする手順をメモってなかったなぁと思ったのでメモっておく．
本当に，単純に見た目を変えるだけ．

<!-- more -->

ターミナルをきれいにします．やることは，

* コマンドラインの表示形式をきれいにする
* 色をきれいにする

テーマを手軽に導入するために Antigen を導入したいので，それに伴って zsh を導入します．

##zsh

Homebrew で導入する．

```bash
$ brew install zsh
```

bash で設定していたパスを zsh でも有効になるようにお引越し．

``` bash
$ cp ~/.bash_profile ~/.zprofile
```

ターミナルの環境設定から、`一般` > `開くシェル` > `コマンド` に `/usr/local/bin/zsh` を設定すればおk．

##Antigen

プラグイン管理ツールの [Antigen](https://github.com/zsh-users/antigen) を導入する．`antigen theme テーマ名` で，oh-my-zsh で提供されているテーマなら手軽に導入できるので便利．

```bash
$ cd
$ git clone https://github.com/zsh-users/antigen.git
```

`.zshrc` に以下を記述

```
source ~/antigen/antigen.zsh

# Load the oh-my-zsh's library.
antigen use oh-my-zsh

# Syntax highlighting bundle.
antigen bundle zsh-users/zsh-syntax-highlighting

# Load the theme.
antigen theme https://github.com/denysdovhan/spaceship-zsh-theme spaceship

# Tell antigen that you're done.
antigen apply
```

テーマは [denysdovhan/spaceship-zsh-theme](https://github.com/denysdovhan/spaceship-zsh-theme) にした．


##Color theme

tommorow theme に幾つかカラーテーマがまとめられている．

[chriskempson/tomorrow-theme: Tomorrow Theme the precursor to Base16 Theme](https://github.com/chriskempson/tomorrow-theme)

適当なフォルダに clone する．

``` bash
$ git clone https://github.com/chriskempson/tomorrow-theme.git
```

ターミナルの設定 > 一般 を選択．カラーテーマ群が表示されているリストの下部に歯車マークがあるので、そこから「読み込む...」を選択する．その後，clone した tomorrow-theme/OS X Terminal/ 以下から好きなテーマを読み込む．すると，リストにテーマが追加されるので，テーマを選択した状態でリスト下部の「デフォルト」を選択すれば良い．

おしまい．
