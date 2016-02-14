---
layout: post
title: "El Capitan でブログを更新しようとしたらエラーを吐かれた"
date: 2016-02-14 13:06:20 +0900
comments: true
categories:
- el-capitan
- octopress
---

Mac OS X El Capitan にしてからブログを更新しようとしたらエラーを吐かれたので，解決方法をメモしておく．

<!-- more -->


octpress でブログを書こうとしたらエラーを吐かれた．

```bash
$ rake preview
Starting to watch source with Jekyll and Compass. Starting Rack on port 4000
rake aborted!
Errno::ENOENT: No such file or directory - jekyll
/Users/Tasuku_Tozawa/Documents/STUDY/blog/octopress/Rakefile:84:in `spawn'
/Users/Tasuku_Tozawa/Documents/STUDY/blog/octopress/Rakefile:84:in `block in <top (requir                                                                                          
ed)>'
Tasks: TOP => preview
(See full trace by running task with --trace)
```

El Capitan におけるバグらしい．

[When I upgraded the Mac system, I can't Preview · Issue #1749 · imathis/octopress](https://github.com/imathis/octopress/issues/1749)

ruby のバージョンをあげたいんだけどエラーになる．

```bash
$ rbenv install 2.2.3
The following versions contain `2.2.3' in the name:
  rbx-2.2.3

See all available versions with `rbenv install --list'.

If the version you need is missing, try upgrading ruby-build:

  brew update && brew upgrade ruby-build
```

[rbenvでインストールできるバージョンリストを最新にする - Qiita](http://qiita.com/ngtk/items/cc85c0d916ac4bcc2188)

`brew update && brew upgrade ruby-build` したら，インストールできた．

```bash
$ rbenv install 2.2.3
Downloading ruby-2.2.3.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.3.tar.bz2
Installing ruby-2.2.3...
Installed ruby-2.2.3 to /Users/Tasuku_Tozawa/.rbenv/versions/2.2.3
```

その後，octopress の root ディレクトリで以下を実行．

```bash
$ rbenv local 2.2.3
$ gem install bundler
$ rbenv rehash
$ bundle install
```

無事実行できた．

``` bash
$ rake preview                                                                              56|                                                                                    
Starting to watch source with Jekyll and Compass. Starting Rack on port 4000                57| ``` bash                                                                           
[2016-02-14 12:50:40] INFO  WEBrick 1.3.1                                                   58|                                                                                    
[2016-02-14 12:50:40] INFO  ruby 2.2.3 (2015-08-18) [x86_64-darwin15]                       59| ```                                                                                
[2016-02-14 12:50:40] INFO  WEBrick::HTTPServer#start: pid=80458 port=4000
```
