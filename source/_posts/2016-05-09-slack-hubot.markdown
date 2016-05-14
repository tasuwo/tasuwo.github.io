---
layout: post
title: "Slack bot を作成した"
date: 2016-05-09 21:19:47 +0900
comments: true
categories: 
- heroku
- slack
- hubot
---

Slack の bot を作った．
何を参考にしたのか聞かれたのでメモしておく．

<!-- more -->

研究室のゴミ当番表がアナログだったので，去年から導入した Slack を通して自動的に担当を割り振り&通知する bot を作った．
成果物は↓

[tasuwo/GomiManBot: GomiMan slack bot written in coffee script with hubot.](https://github.com/tasuwo/GomiManBot)

これから実際に使ってみてバグが見つかり次第修正していく予定．
英語力がゴミなのでおかしいところ発見次第修正していく予定．

## メモ書き
hubot + slack bot + heroku に関するサイト量は飽和状態であまり書く気しなかったのだけど，中には「エラーは持ち前のスルースキルで無視します！」みたいな記事もあってげんなりしたので，何を参考にしたのかだけも一応書いておく．
導入については以下の記事を参考にした．

[Slack で Hubot を使えるようにする - Qiita](http://qiita.com/misopeso/items/1f418dd02e89234499b3)
[Slackと連携させたHubotに毎朝今日の予定をお知らせしてもらう - Qiita](http://qiita.com/tk3fftk/items/6ae172abc57f72eabeb2)

公式ドキュメントは以下．世に出回ってる hubot の記事は大抵古くて今では不要な手順が含まれていたりするからこちらを参考にするのが確実．

[HUBOT](https://hubot.github.com/docs/)

また，無料版の heroku だと 1日6時間の間sleepさせる必要があるので，うまくやる．
起き続けさせるためには hubot-heroku-keepalive，寝たのを起こすためには何らかのアドオンを使用する．

[雑記帳@iimuz by iimuz](https://iimuz.github.io/2015/11/11/hubotKeepalive.html)
[Heroku 上で動く Hubot をうまく休ませる - Qiita](http://qiita.com/misopeso/items/8cde2ecbb82e7bfc01b4)

アドオンは shceduler を使用した．

## Hubot とは
Hubot は Github 社が製作した bot 作成用のフレームワーク．
Node.js + CoffeeScript で書かれており，オープンソースである．
アダプターによって異なるチャット(Slack とか Chatworks とか)に対応できる．

[HUBOT](https://hubot.github.com/)

## Heroku とは
AWS という IaaS 上に構築された PaaS．以下用語説明．

* IaaS : Infrastructure as a Service
  * 情報システムの稼働に必要な仮想サーバとしたシュシュのインフラをインターネット上のサービスとして提供する
  * Google Compute Engine, Amazon Elastic Compute Cloud(EC2)
* PaaS : Platform as a Service
  * アプリケーションの稼働に必要なハードウェアやOS等のプラットフォーム一式をインターネット上のサービスとして提供する
  * 例 : Google App Engine, Windows Azure
* SaaS : Software as a Service
  * パッケージ製品として提供されていたソフトウェアを，インターネット経由でサービスとして提供する
  * 例 : Google Apps, Salesforce

Heroku は Web アプリを作成した時に，Git を使用して簡単に公開できる．
最初は Ruby のみだったが今では対応言語も多い．
ビルドパックを使用することで未対応の言語についてもデプロイが可能．

[Heroku](https://dashboard.heroku.com/)

以上です．
