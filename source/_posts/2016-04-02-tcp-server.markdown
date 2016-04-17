---
layout: post
title: "TCPサーバ/クライアントを車輪の再発明する"
date: 2016-04-02 12:51:38 +0900
comments: true
categories: 
- c++
- tcp
- socket
- network
---

TCP Server/Client を車輪の再発明することで，ネットワーク通信の下の方を勉強してみようという試み．ついでに用語の整理もしてみます．

<!-- more -->

## Stream
*stream* を使用しない場合のデータのよみ出しは，`read_block()` というシステムコールによって以下のように行える．

``` cpp
char buffer[256];
read_block("hello.c", buffer, 32, 256);
```

上記では，`hello.c` の先頭から32byte目から256bytes分読み込まれる．つまり，固定の範囲(ブロック)でデータにランダムにアクセスできる．

上記方法は，扱う対象がメモリやハードディスク上であれば同様の処理で十分だが，例えばキーボードからの入力等を読み込む場合には適さない．前者のようなデバイスを ***block device*** ．後者のようなデバイスを ***character device*** という．

* ***block device***
  * データにブロック単位でアクセス可能なデバイス．データのバッファリングやランダムアクセスが可能．
  * 例: メモリ，ハードディスク
* ***character device***
  * データにbyte単位でアクセス可能なデバイス．データはバッファリングされず，ランダムアクセスもできない．
  * 例: キーボード，マウス

2つのデバイスを同じシステムコールで扱うためには，使用するシステムコールを *character device* に合わせておけば良い．ここで使われている抽象化が ***stream*** である．

* ***stream***
  * データの供給元(ex: ハードディスク)と受け手(ex: プログラム)の間に入り，データの一時保存を行う抽象データ構造
  * 1byte単位でデータを受け取り，FIFO方式で受け手に渡す

*stream* の良いところは，**供給元と受け手の転送量の違いを吸収できる**部分．供給元は一度に100byteのデータを送信したいが受け手は一度に10byteしか受け取れない場合， *stream* は余った90byteを一時的に保存できる．

*stream* を使用したデータのよみ出しは以下のように行える．open システムコールによって OS によみ出し開始を通知し，同時に *stream* の識別子である file descriptor を得る．それ以降は file descriptor を使用して read，write システムコールによって読み書きを行い，close によって終了する．

``` cpp
char buffer0[32];
char buffer1[256];
int fd;

fd = open("hello.c", O_RDONLY, 0x666);
read(fd, buffer0, 32);
read(fd, buffer1, 256);
close(fd);
```

## Socket
TCP/IPによるネットワーク通信も，結局はデータのやりとりなので， *stream* の概念を流用できるとわかりやすい．しかしそのまま適用はできなかったので，特別な *stream* として ***socket*** を定義した．socket は通常の *stream* と異なり，作成時には open ではなく socket システムコールを使用する．また，作成時の処理が多少複雑になっている．

## TCP socket
TCPによる通信では以下の約束事がある．

* メッセージが届いたかチェックし，届いてなければ再送する
* 到着順序は保障されている
* それが失敗した場合はエラーになることが保障されている

TCP通信用の socket の作成方法は以下の通り．

``` cpp
socket_df = socket(AF_INET, SOCK_STREAM, 0);
```

第一引数は**アドレスファミリ**を表している．アドレスファミリとは，socket が使用するアドレス体系のこと．よく使用されるのは以下．

|アドレスファミリ|内容|
|---|---|
|AF_INET|IPv4用|
|AF_INET6|IPv6用|
|AF_UNIX|ローカルなプロセス間通信用|
|AF_PACKET|デバイスレベルのパケットインタフェース|

第二引数は**ソケットタイプ**を表している．ソケットタイプはソケットの性質を表しており，Linuxで使用可能な代表的なものは以下．

|ソケットタイプ|解説|
|---|---|
|SOCK_STREAM|順序性，信頼性を備え双方向接続された byte stream を提供する|
|SOCK_DGRAM|データグラム(接続，信頼性なし，固定最大長メッセージ)をサポートする|

第三引数は使用するプロトコルを表す．0にしておくとデフォルトのものが使用される．基本的には上記二つの組み合わせによって通信方式が決定される．例えば，以下のように．

|アドレスファミリ|ソケットタイプ|通信方式|
|---|---|---|
|AF_INET|SOCK_STREAM|IPv4 + TCP|
|AF_INET6|SOCK_DGRAM|IPv6 + UDP|

## 通信手順
クライアントとサーバがそれぞれ以下の手順でシステムコールを呼び出し，通信する．クライアントサーバモデルは，同期方法が非対称的であり，サーバ側とクライアント側でプログラムが異なる．これは電話の呼び出し方式と似ている．

|No|Client|Server|
|---|---|---|
|1|socket(ソケット生成)|socket(ソケット生成)|
|2||bind(ポート指定)
|3||listen(待ち受け)|
|4||accept(接続待ち)|
|5|connect(ソケット接続要求)||
|6|send(送信)||
|7||recv(受信)|
|8||send(送信)|
|9|recv(受信)||
|10|shutdown(切断)|shutdown(切断)|
|11|close(ソケット切断)|close(ソケット切断)|

Client 側は socket 作成後，connect で同期を行う．この時，通信相手のサーバを引数(host名，port番号)で指定する．

Server 側は socket 作成後が多少複雑．socket は作成時点ではアドレスが割り当てられていないので，bindによってアドレスを割りあてる．この操作は伝統的に「ソケットに名前をつける」と呼ばれる．

ここでいうアドレスは，接続に必要なネットワーク層レベルの情報を保持する構造体として定義されている．保持する情報と定義は以下の通り．

* 通信プロトコル
* アドレス
* ポート番号

``` c++
struct sockaddr_in {
   sa_family_t    sin_family; /* address family: AF_INET */
   u_int16_t      sin_port;   /* port in network byte order */
   struct in_addr  sin_addr;  /* internet address */
};

/* Internet address. */
struct in_addr {
   u_int32_t      s_addr;     /* address in network byte order */
};
```

bind 後，socket を passive socket(接続待ちソケット)，すなわちサーバ側の socket であることをOSに通知する．ここでいう passive socket とは，accept による接続要求を受け付けるのに使用するソケットのこと．accept では passive socket から接続要求を取り出し，それを元に接続済みソケットを作成し，その socket を参照する新しい file descriptor を返す．

なぜわざわざ passive socket を用意しているのかというと，通常サーバに対してクライアントは複数存在し，その要求をさばくためには接続要求をためておく stream が必要であるため．

通信終了時には close 処理を行うが，多少注意が必要．sockt は送信データが正しく受信したことを確認しないと破棄されないので，close しても即座に破棄はされない．一般には，サーバよりも先にクライアントを close すればよい．サーバ側から強制的に close したい時には，close の直前に以下を呼べば良い．

``` cpp
shutdown(socket, 2);
```

もしくは bind の前に以下を設定する．

``` cpp
int value = 1;
setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &value, sizeof(value));
```

`setsockopt` は，与えられた socket のオプションを設定するもので，以下のように定義されている．

``` cpp
int setsockopt(int sockfd, int level, int optname,
               const void *optval, socklen_t optlen);
```

よく使われるオプションは `SO_REUSEADDR` であり，これを設定することで再度同じポート番号で bind できる．

## 実装

ネット上のいろいろを参考に TCP サーバを C++ で書いた．C++は全然書いたことない初心者なのでひどいコードになっているかもしれない...飽くまで学習用なので，あしからず．

<script src="https://gist.github.com/tasuwo/18fa141e82da1f87901419eb78353f2a.js"></script>

Server，Client の順で起動し，サーバ側から文字列を送信できる．実行結果は以下のような感じ

![tcp](/images/tcp_connect.png)

## 参考
[Lecture Notes](http://www.csg.ci.i.u-tokyo.ac.jp/~chiba/lecture/web/web01.html)
[setsockopt - 車輪のx発明 ~B.G's Blog~](http://bg1.hatenablog.com/entry/2015/08/19/210000)
[Linuxネットワークプログラミング : あきみち : 本 : Amazon.co.jp](http://www.amazon.co.jp/Linux%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0-%E3%81%82%E3%81%8D%E3%81%BF%E3%81%A1/dp/4797354526)
[socket(2), bind(2), listen(2), accept(2) - kotaroito's notes](http://kotaroito.hatenablog.com/entry/2014/01/31/224138)
[LinuxにおけるTCPソケット通信を利用したプロセス間通信(C++) - MyEnigma](http://myenigma.hatenablog.com/entry/20140308/1394252286)

<!-- [TCP接続する時に使ういろいろな構造体の整理 - Qiita](http://qiita.com/0xfffffff7/items/6ffb317df8345070d0b5) -->
<!-- [Structure](http://www-cms.phys.s.u-tokyo.ac.jp/~naoki/CIPINTRO/NETWORK/struct.html) -->
<!-- [7.1 デバイス・ファイルについて](https://docs.oracle.com/cd/E39368_01/e48214/ol_about_devices.html) -->
<!-- [システムコールを理解する | UNIX world](http://curtaincall.weblike.jp/portfolio-unix/api.html) -->
<!-- [知ってトクするシステムコール（2）：システムコールと標準ライブラリ関数の違いを知る (1/2) - ＠IT](http://www.atmarkit.co.jp/ait/articles/1112/13/news117.html) -->
<!-- [ソケット通信メモ(Hishidama's TCP/UDP Socket Memo)](http://www.ne.jp/asahi/hishidama/home/tech/socket/) -->
<!-- [ソケット](http://research.nii.ac.jp/~ichiro/syspro98/socket.html) -->
<!-- [3 システムコール](http://akita-nct.jp/yamamoto/lecture/2007/2E/17th/html/node3.html) -->
<!-- [Linuxシステムコールの勉強(その１６) - Webプログラミングをしてみよう!!](http://d.hatena.ne.jp/web_develop/20071205/1196875088) -->
<!-- [socket - システムコールの説明 - Linux コマンド集 一覧表](http://kazmax.zpp.jp/cmd/s/socket.2.html) -->
<!-- [socket関数とプロトコル](http://www.fireproject.jp/feature/c-language/socket/basic.html) -->
<!-- [C言語:ソケット(Socket)でネットワークプログラム入門](http://www.geocities.jp/sugachan1973/doc/funto45.html) -->
<!-- [Lecture Notes](http://www.csg.ci.i.u-tokyo.ac.jp/~chiba/lecture/web/web03.html) -->
<!-- [絵で見てわかるファイルディスクリプタ・パイプ・リダイレクト - あしのあしあと](http://d.hatena.ne.jp/higher_tomorrow/20110426/1303830417) -->
<!-- [C++と Pthreads でミニマルなHTTPサーバを書く - bkブログ](http://0xcc.net/blog/archives/000178.html) -->
<!-- http://yuuki.hatenablog.com/entry/2015-webserver-architecture -->
<!-- http://mattn.kaoriya.net/software/lang/c/20090729235933.htm -->
<!-- http://myenigma.hatenablog.com/entry/20140308/1394252286 -->
<!-- [socket() で使用するアドレス・ファミリー・プロトコル](https://publib.boulder.ibm.com/html/as400/v4r5/ic2962/info/RZAB6ADDFAM.HTM) -->

