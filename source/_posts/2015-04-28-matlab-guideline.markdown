---
layout: post
title: "MATLAB のコーディングガイドラインについてメモ"
date: 2015-04-28 12:13:30 +0900
comments: true
categories: matlab
---

研究で MATLAB を利用しているので，コードを綺麗にかくためのメモ．

<!-- more -->

##コーディング作法

MATLAB のコードを書く際のコーディング作法を記述しているドキュメントを発見した．

>[MATLAB Programming Style Guidelines 2.0 | MATLAB CENTRAL](http://www.mathworks.com/matlabcentral/fileexchange/46056-matlab-style-guidelines-2-0)

日本語訳をしていらっしゃる方もいて素晴らしい．

>[MATLAB Programming Style Guidelines 1 - はてなダイアリー](http://myenigma.hatenablog.com/entry/20120103/1325575787#fn-a7d7d54e)

自分はプログラミング経験が浅いので，とても参考になる．コードがぐっと見やすくなった．


##単体テスト

作成しているツールの規模が大きくなってきたので，しっかりテストしたいと思うようになってきた．MATLAB には単体テスト用のフレームワークが用意されているようだ．

>[単体テストの作成 - MATLAB & Simulink - MathWorks 日本](http://jp.mathworks.com/help/matlab/write-unit-tests-1.html)
>[スムーズワークス日想 » Blog Archive » m-fileに品質を！（2）](http://blog.smooth-works.net/archives/2183)
>[Jenkins 導入と MATLAB の自動テスト (教員のための Mac Tips:9)](http://d.hatena.ne.jp/hkob/20131226)

Jenkins による継続的インテグレーションにも興味があるので，気が向けば触れてみたいと思う．具体的なやり方については，そのうちまとめる．たぶん．


##その他の参考文献

>[MATLABプログラミングスタイル | スムーズワークス日想 シミュレーション業界関連情報](http://blog.smooth-works.net/archives/2901)
