---
layout: post
title: "コマンド構文の表記規則についてメモ"
date: 2015-03-18 16:57:24 +0900
comments: true
categories: 
---

ちょっとしたメモ

<!-- more -->

コマンドの説明の例．

```bash
# これとか
$ git remote add [shortname] [url]

# これとか
$ git add <file>...
```

括弧が「[]」と「<>」で異なっている．
今までイマイチ気にしていなかったので，少し調べてみる．

調べればごろごろ出てくる．

>[コマンド構文規則 - IBM](http://publib.boulder.ibm.com/tividd/td/TRM/GC32-1320-00/ja_JA/HTML/cmdref18.htm)
>[4. コマンド構文](https://docs.oracle.com/cd/E37520_01/b69426/vmutl-preface-syntax.html)
>[コマンドの記載形式](http://itdoc.hitachi.co.jp/manuals/3020/30203R3331/PCRE0058.HTM)
>[コマンド ライン構文の文字 - CA ARCserve® Backup for ...](http://support.ca.com/cadocs/0/CA%20ARCserve%20%20Backup%20r16-JPN/Bookshelf_Files/HTML/cmndline/index.htm?toc.htm?cl_cmd_line_syntax_char.htm)
>[表記規則 - 日本オラクル](http://otndnld.oracle.co.jp/document/products/oracleVM/220/generic/B57076-01/preface.htm)

ここら辺を見れば意味はわかるんだけど，何が元になってるのか気になってもうちょっとうろうろしてみた．すると以下のような記載を発見．

>ってかこれって統一文法みたいなのがあるのか？UNIXのmanコマンド表記がベースになってるのかな？
>[コマンド表記法 - 俺の基地](http://yakinikunotare.boo.jp/orebase/index.php?%A5%B3%A5%DE%A5%F3%A5%C9%C9%BD%B5%AD%CB%A1)

UNIXの`man`コマンドを知らなかったので調べてみる．

>[基本的なUNIXコマンド](http://www.cc.kyoto-su.ac.jp/~hirai/text/unixcommand.html)
>[UNIXコマンド - man (Linux/FreeBSD/Solaris) - k-tanaka.net](http://www.k-tanaka.net/unix/man.php)
>[manページ - Wikipedia](http://ja.wikipedia.org/wiki/Manページ)

`man`コマンドは，UNIXコマンドのマニュアルを表示するコマンドのようだ．その前身ともいえる UNIX Programmer's Manual は1971年に出版され，その後オンラインのmanページが1971年に執筆された．そのレイアウトは以下のような構成になっている．

* NAME(名前)
* SYNOPSIS(書式)
* DESCRIPTION(説明)
* EXAMPLES(使用例)
* SEE ALSO(関連項目)

このうち，**SYNOPSIS**に構文規則を用いた表記法が用いられているので，これが元なのではないかということ．実際にこれが元かはわからないけど，とりあえず~~無駄~~知識を増やすことができたので満足した．暇があったらまた調べてみよう．

とりあえず，代表的なものをまとめておく．

|表記法|概要|例|
|---|---|---|
|[]|なくてもよい|hoge [-h]：`-h`はなくてもよい|
|{}, <>|必須|hoge <*filename*>：`filename`は必ず記載|
|斜体|変数|hoge <*filename*>：`filename`は変数．適切な値を記述する|
|&#124;|エレメントの選択|hote [-a&#124;-b]：`-a`か`-b`を選択|
|...|繰り返し|hoge <*file*>... ：`hoge file1 file2 file3`と使用できる|
