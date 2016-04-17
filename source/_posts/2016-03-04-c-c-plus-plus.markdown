---
layout: post
title: "EmacsにおけるC,C++の環境を整える"
date: 2016-03-04 21:47:12 +0900
comments: true
categories: 
- Emacs
- C言語
- C++
---

大昔に取り組んでいた自作OS、せっかくなので再挑戦してみようと考えた．CLionとかそこらへんのIDEを使用しても良かったけれど，せっかくだからEmacs で C, C++ の環境を整えるたいので，メモしとく．
基本的に[C/C++ Development Environment for Emacs](https://tuhdo.github.io/c-ide.html)に全部書いてあったのでつまんでみる．

<!-- more -->

helm+helm-gtags もしくは ggtags を使う．
自分は helm-gtags を使うことになるだろう．

# 1.GTAGS
プロジェクトのルートディレクトリで gtags コマンドを実行すると，以下のファイル群が生成されるはず．

``` bash
$ cd /path/to/project/root
$ gtags
$ ls G*
GPATH   GRTAGS  GTAGS
```

それぞれ以下の情報を保持している．

* **GTAGS**  : 定義
* **GRTAGS** : 参照
* **GPATH**  : パス名

# 2.基本操作
作業を快適にするために把握しておくべきEmacsにおける基本操作は以下

|コマンド|コマンド名|概要|
|---|---|---|
|C-M-f|forward-sexp|閉じカッコの前に行く|
|C-M-b|backward-sexp|閉じカッコの後ろに行く|
|C-M-k|kill-sexp|閉じカッコ内を削除する|
|C-M-<SPC>,C-M-@|mark-sexp|閉じカッコ内を選択する|
|C-M-a|beginning-of-defun|関数の前に行く|
|C-M-e|end-of-defun|関数の後ろに行く|
|C-M-h|mark-defun|関数を選択する|

# 3.定義参照
## 3.1.バッファ内参照
### ggtags-mode
Imenuを使用する
`(setq-local imenu-create-index-function #'ggtags-build-imenu-index)`
### helm
moo-jump-localを使用する

## 3.2.プロジェクト内参照
### ggtags-mode
|コマンド|コマンド名|概要|
|---|---|---|
|M-.|ggtags-finde-tag-dwim|・定義にポイントしていれば参照先を表示する<br>・参照にポイントしてれば定義を表示する<br>・include ヘッダーをポイントしていればそのヘッダーを表示する<br>・その他の場所であれば定義，参照一覧が表示され，絞り込みができる|
|M-,|pop-tag-mark|ジャンプ元へ戻る|
|M-n, M-p||候補内移動|
|M-g s||候補内検索|

### helm-mode
|コマンド|コマンド名|概要|
|---|---|---|
|M-.|helm-gtags-dwim|ggtags-find-tag-dwim と一緒|
|M-,|tags-loop-continue|pop-tag-mark と一緒|
|C-j|helm-gtags-select|空白部分で M-. するのと一緒．定義や山椒を一覧から絞り込み&ジャンプできる|

# 4.参照元ジャンプ
### gtags-mode
ggtags-find-reference, ggtags-find-tag-dwimを使う
### helm-gtags
|コマンド|コマンド名|概要|
|---|---|---|
|C-c g r|helm-gtags-find-rtag|・関数内で呼び出したら，その関数についての参照先を検索する<br>・関数名上で呼び出したら，参照先のリストを表示する<br>・変数名にポイントしていたら，なにもしない
|C-c g s|htlm-gtags-find-symbol|変数名ポイント時に参照元を検索する|
|C-c g a|htlm-gtags-tagas-in-this-function|現在の関数が参照する関数一覧|

# 5.ファイル検索
### ggtags-mode
ggtags-find-file
### helm-gtags
helm-gtags-find-files

正直Projectile使ったほうが良いとのこと．

# 6.過去に訪れたタグへジャンプ
### ggatgs-mode
ggtags-view-tag-history(C-c g h)
### helm-gtags
helm-gtags-show-stack

# 7.Speedbar
ソースツリーを見れるパッケージ．ただのソースツリーではなくて，戻り値や関数なども一覧できるのが便利っぽい．

|コマンド|操作|
|---|---|
|SPC|子ノードを開く|
|RET|ノードを別ウインドウで開く|
|U|親ノードへ移動|
|n,p|ノードを上下移動|
|M-n,M-p|現在の階層内でノードを上下移動|
|b|Speedbarのバッファリストに戻る|
|f|ファイルリストに戻る|

## 7.1.sr-speedbar
Speedbarを便利にするパッケージ．

* 起動/終了
  * sr-speedbar-open, sr-speedbar-toggle : 開く
  * sr-speedbar-cloe, sr-speedbar-toggle : 閉じる
* 改善点
  * フレームの代わりにEmacs windowを使用する
  * `C-x 1`でSpeedbarを除くすべてのウインドウを削除する
  * `C-x o`でSpeedbarに移動するのを防ぐ(sr-speedbar-skip-other-window-pをtにする)

# 8.Company-mode (補完)
company-mode を使う．company-mode はEmacsのための補完フレームワーク．
```
(require 'company)
(add-hook 'after-init-hook 'global-company-mode)
```

## 8.2.使い方
|コマンド|操作|
|---|---|
|M-n,M-p|候補移動|
|RET,TAB|候補決定|
|C-s,C-r,C-o|候補検索|
|M-(数値)|候補簡易選択|
|<f1>|選択中候補のドキュメントを表示|
|C-w|選択中候補のソースコード表示|

`company-backends`で候補に使用するリソースを指定する．

## 8.3.C言語の補完
C言語でcompanyの補完を利用するためには，以下を記述する．
``` bash
(delete 'company-semantic company-backends)
(define-key c-mode-map  [(tab)] 'company-complete)
(define-key c++-mode-map  [(tab)] 'company-complete)
```
上記の設定では，`company-semantic`を削除している．理由は後述．`company-semantic`についてはCEDITの項で詳しく説明する．

companyの補完として，以下の二つが働く．

### 8.3.1.company-clang
補完候補の取得のために`clang`を使用する．プロジェクトではなく，ヘッダファイルによって補完を行う．デフォルトでは`company-clang`は`company-semantic`のサブセットであるため，上記設定を行っていれば他に特別な設定はいらない．
上記せて血で`company-semantic`を削除したのは，そうしないと`company-complete`が`company-clang`ではなく`company-semantic`を使用してしまうため．これは，`company-backends`内の優先度がそうなっているため生じる．
補完候補をプロジェクト内から取得するためには，`.dir-locals.el`をプロジェクトルートに配置する必要がある．

``` emacs-lisp
((nil . ((company-clang-arguments . ("-I/home/<user>/project_root/include1/"
                                     "-I/home/<user>/project_root/include2/")))))
```
helmを使っているなら，`C-x C-f`によるファイル検索中に，対象ファイル選択状態から`C-c i`によって絶対パスを挿入できる．
`nil`を指定すると設定をすべてのサブディレクトリ，ファイルに適用し，`non-nil`であれば設定を適用するメジャーモードを指定できる．`company-clang-arguments`はインクルードパスを指定するリストである．

### 8.3.2comapny-gtags
`GNU Global`の`GTAGS`から補完候補を取得する．プロジェクトによる補完を行うことができる．

## 8.4.ヘッダーの補完
プロジェクト内のヘッダーを補完したいなら，`company-c-headers`を使用する．以下のように`company-backends`に追加すれば良い．

``` emacs-lisp
(add-to-list 'company-backends 'company-c-headers)
```

C++でヘッダーの補完を行いたいならば，パスを追加する必要がある．`company-c-header`はシステムのインクルードパスとして`/usr/include/`と`/usr/local/include/`しか含んでいない．例としては以下のように追加する．

``` emacs-lisp
(add-to-list 'company-c-headers-path-system "/usr/include/c++/4.8/")
```

# 9.CEDET
CEDETはCollection of Emacs Development Environment Toolsの略称．CEDETのデメリットは，Emacs Lispで書かれているため，Emacsのパフォーマンスに影響すること．23.2以降のEmacsにはマージされているので，インストールの必要はない．
最新版は以下のようにダウンロードすれば良い．
``` bash
$ git clone http://git.code.sf.net/p/cedet/git cedet
$ cd cedet
$ make # wait for it to complete
$ cd contrib
$ make
```
Emacs からロードする．
``` emacs-lisp
(load-file (concat user-emacs-directory "/cedet/cedet-devel-load.el"))
(load-file (concat user-emacs-directory "cedet/contrib/cedet-contrib-load.el"))
```
## 9.1.Semanticマイナーモード
`Semantic`は，ソースコードパーサを利用して構文を考慮した補完を行ってくれるパッケージ．
### 9.1.1セットアップ
``` emacs-lisp
(require 'cc-mode)
(require 'semantic)

(global-semanticdb-minor-mode 1)
(global-semantic-idle-scheduler-mode 1)

(semantic-mode 1)
```
### 9.1.2.`semantic-mode`
Semantic-modeでは，Emacsは現在のバッファをパースする．シンボルにカーソルを合わせるとsemanticはすべてのincludeファイルを読みに行くので，たまに時間がかかる．しかし一回パースすれば終わりなので，問題はない．

### 9.1.3.パスの追加
Semantic のデフォルトのインクルードパスは`semantic-dependency-system-include-path`に格納されており，追加したい場合は以下のようにする．

``` emacs-lisp
(semantic-add-system-include "/usr/include/boost" 'c++-mode)
(semantic-add-system-include "~/linux/kernel")
(semantic-add-system-include "~/linux/include")
```

### 9.1.4.`company-mode`におけるSemantic-mode

`company-mode`には`company-semantic`コマンドがあり，これがSemanticDBを補完候補の取得に利用する．`company-semantic`の良いところは，`semantic-ia-complete-symbol`が改善されているところ．元は1文字以上タイプしていなければ補完を検索してくれなかったが，`company-semantic`ではプレフィクスなしで補完してくれる．

* `global-sematicdb-minor-mode`
  * パース結果をキャッシュする．キャッシュ結果は`semanticdb-default-save-directory`変数内のパスに保存されるが，デフォルトでは`~/.emacs.d/semanticdb`いかに保持される
* `global-semantic-idle-scheduler-mode`
  * このモードが有効になっていると，バッファーが期限切れになっていた時，ユーザがタイプしていない間にパーサをし直す．これがオフだと，バッファはコマンドによって手動でパースし直さなければならない


## 9.2.CEDETのその他の機能
Semanticがソースコードをパースし作成したデータベースは，コードの補完の他にも様々な使い道がある．コードナビゲーションや定義元・参照元ジャンプなど．
### 9.2.1.Senator
CEDETの一部で，SEmainticNAvigaTORの略称．

### 9.2.2.デバッグ
GDBとかGUDとかがあるらしい．



あとは気が向いたら．
