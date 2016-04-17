---
layout: post
title: "PythonでHTMLのテーブル情報を取得する"
date: 2016-04-17 10:37:30 +0900
comments: true
categories:
- Python
- crowling
---

研究室内で今までHTMLのテーブルでデータを管理していたものを，ちゃんとDBつくって管理したくなったので，Pythonを使ってcrawlingしてみようという試み．

***!!注意!! : 筆者はPython初心者です．コードの表現等は適切でない場合があります...おかしな部分がありましたらぜひコメントで指摘をお願いします...***

<!-- more -->

## やりたいことと問題点

以下のようなテーブルが存在した時，その内容を抽出してDBに格納したい．

``` html
<table width="95%" border="1" cellspacing="0" cellpadding="2">
  <tr align="center" bgcolor="yellow">
    <td width="80">日付</td>
    <td width="80">記録</td>
    <td width="80">担当</td>
    <td>題目</td>
  </tr>
  <tr align="center">
    <td rowspan="2">2XX5/12/24</td>
    <td rowspan="2">記録者名1</td>
    <td nowrap>担当者名1</td>
    <td align="left">論文情報1</td>
  </tr>
  <tr align="center">
    <td>担当者名2</td>
    <td align="left">論文情報2</td>
  </tr>
  <tr align="center">
    <td rowspan="2">2XX5/12/25</td>
    <td rowspan="2">記録者名2</td>
    <td nowrap>担当者名3</td>
    <td align="left">論文情報3</td>
  </tr>
  <tr align="center">
    <td>担当者名4</td>
    <td align="left">論文情報4</td>
  </tr>
</table>
```

上記のHTML記述は以下のように描画される．

<table>
  <tr align="center" bgcolor="yellow">
    <td width="80">日付</td>
    <td width="80">記録</td>
    <td width="80">担当</td>
    <td>題目</td>
  </tr>
  <tr align="center">
    <td rowspan="2">2XX5/12/24</td>
    <td rowspan="2">記録者名</td>
    <td nowrap>担当者名1</td>
    <td align="left">論文情報1</td>
  </tr>
  <tr align="center">
    <td>担当者名2</td>
    <td align="left">論文情報2</td>
  </tr>
  <tr align="center">
    <td rowspan="2">2XX5/12/25</td>
    <td rowspan="2">記録者名2</td>
    <td nowrap>担当者名3</td>
    <td align="left">論文情報3</td>
  </tr>
  <tr align="center">
    <td>担当者名4</td>
    <td align="left">論文情報4</td>
  </tr>
</table>

問題は，**`rowspan`オプションを使用しているため，日付&記録とtrタグが1:1になっていない**という部分．調べてみると，[各trタグがもつtdタグと，rowspanオプションを持つtdタグをあらかじめ保存しておき，後者の情報から前者を更新する方法](http://stackoverflow.com/questions/28763891/what-should-i-do-when-tr-has-rowspan)があるらしい．しかし，**今回はとりあえず論文情報だけ取得できればよかったので，`align`オプションを持つtdタグのみを抽出する．**

また，論文情報は以下の形式をとる．ここから各種情報を抽出してDBに格納したい．

```
著者名(所属), ... : 原文題目[翻訳題目], 論文誌名, Vol.番号, No.番号, pp.開始頁-終了頁 (発行年/月)
```

ここでの問題は，**`,`や`:`が全角の場合と半角の場合が入り混じっている(!?)**という部分(各論文情報間だけでなく，なんと一つの論文情報内に全角/半角が混ざっている場合もある...)．みんなが適当に更新したから仕方ないね...さらに，論文誌名やVol.，No.等はあったりなかったりするので，**今回はとりあえず著者名と題目のみ取得する．**

## やるべきこと

手順としては以下のようになる．対象ページはBasic認証が必要なので対応する．

- Basic認証して対象ページのHTMLを取得する
- HTMLにおけるテーブルから特定のオプションが負荷されたtdタグを抽出する
- 得られた論文情報から著者名と題目の情報を抽出する

## 実装する

### HTMLの取得

ライブラリは簡単そうだったので`requests`を使用した．Basic認証を行うためには引数で`auth`を指定する．

``` python
import requests
if __name__ == "__main__":
	URL = 'http://...'
	USER_NAME = "your_user_name"
	PASS_WORD = "your_pass_word"
	PAGES_DATA = requests.get(URL, auth=(USER_NAME, PASS_WORD))
	CORRECT_ENCODING = PAGES_DATA.apparent_encoding
	PAGES_DATA.encoding = CORRECT_ENCODING
	HTML = PAGES_DATA.text.encode(CORRECT_ENCODING)
	print HTML
```

`apparent_encoding`を取得したデータのencodingに設定し直しているのは，`requests`が正しくエンコード情報を取得してくれない場合があるかららしい．さらに，データに含まれるテキストを正しくエンコードすると，日本語を文字化けさせずにデータを取得できた．

>[[Python]requestsが正しくエンコード情報を返してくれない場合は apparent_encoding を使うとよいかもしれない | aoshiman.org](http://blog.aoshiman.org/entry/118/)

### HTMLのパース

ライブラリは`BeautifulSoup`を使用した．

``` python
from bs4 import BeautifulSoup

SOUP = BeautifulSoup(HTML, "lxml")
TABLES = SOUP.find_all('table')
for table in TABLES:
	tmp = table.find_all('tr')
	ALL_ROWS = tmp[1:]
	for _, tr_tag in enumerate(ALL_ROWS):
		scholar_strings = []
		for _, data in enumerate(tr_tag.find_all('td')):
			if data.has_attr("align"):
				scholar_strings.append(data.get_text())
```

基本的には，`find_all`でタグを検索し，`has_attr`で指定オプションを持つかどうかを判定する，という流れ．これで論文情報の文字列群は取得できる．

### 文字列の分割

論文情報は`執筆者名:論文情報`の形式で区切られており，さらに論文情報は`論文題目,その他の情報,...`のような形式で区切られている．さらに，`:`及び`,`は半角と全角が混在している．また，論文題目に`:`が含まれる場合も存在するし，論文題目以外の論文情報には抜けや漏れが存在する可能性がある．
そこで，以下のように分割処理を行う．

1. 一番左端の`:`(半角/全角)で分割し，執筆者情報と論文情報に分割する
2. 執筆者情報を`,`(半角/全角)で分割し，各執筆者の除法を取得する
3. 論文情報を`,`(半角/全角)で分割し，論文題目のみ取得する

文字列を複数のデリミタで区切る，もしくは指定回数のみ分割するには，`re`モジュールを使用して以下のようにする．

``` python
import re
re.split("delimter1|delimiter2|...", "分割対象の文字列", 分割回数)
```

デリミタ群をリストで渡したかったので，以下のような関数を作成した．

``` python
def split_by_delimiters(delimiter_list, target_string, split_num=0):
	u"""
	Return a list of the words, using words in delimiter_list as
	the delimiter strings
	"""
	# Prepare delimiter string
	delimiter_str = ''
	for pattern in delimiter_list:
		delimiter_str = delimiter_str + pattern + '|'
	delimiter_str.rstrip('|')
	# Split target_string
	if split_num == 0:
		splited_str_list = re.split(delimiter_str, target_string)
	else:
		splited_str_list = re.split(delimiter_str, target_string, split_num)
	if len(splited_str_list) >= 2:
		return splited_str_list
	return target_string
```

これを使用して，論文情報の文字列から著者情報と題目情報を抽出する関数を作った．

``` python
COLON_DLIMITERS = ["\xef\xbc\x9a", ":"]
COMMA_DELIMITERS = ["\xef\xbc\x8c", ","]
def get_scholar_json_model(scholr_info_str):
	u"""Return json model for scholar information"""
	scholr_info_str = scholr_info_str.encode('utf-8')
	if len(scholr_info_str) < 15:
		return -1
	# Split the string into authors and other informations
	author_others_info = split_by_delimiters(COLON_DLIMITERS, scholr_info_str, 1)
	if len(author_others_info) == 1:
		return -1
	# Split the string into scholar's title and other informations
	title_others_info = split_by_delimiters(COMMA_DELIMITERS, author_others_info[1])
	# Create JSON model
	authors = split_by_delimiters(COMMA_DELIMITERS, author_others_info[0])
	authors_model = []
	for author in authors:
		authors_model.append({"name": author.strip()})
	title = title_others_info[0]
	scholar_model = {"authors": authors_model, "title": title.strip()}
	return scholar_model
```

`scholar_info_str`が15より小さいか比較している部分は，nbspとか[未登録]みたいな文字列を取得してしまった際に無視するため(よくない)．

最終的なコードは以下．

``` python
# -*- coding: utf-8 -*-

import re
from collections import namedtuple
import requests
from bs4 import BeautifulSoup


AuthInfo = namedtuple('AuthInfo', 'user_name, pass_word')
def get_html_from(url, auth_info):
	u"""Get html in url"""
	user_name = auth_info.user_name
	pass_word = auth_info.pass_word
	pages_data = requests.get(url, auth=(user_name, pass_word))
	pages_data.encoding = pages_data.apparent_encoding
	return pages_data.text.encode('utf-8')


COLON_DLIMITERS = ["\xef\xbc\x9a", ":"]
COMMA_DELIMITERS = ["\xef\xbc\x8c", ","]
def get_scholar_json_model(scholar_info_str):
	u"""Return json model for scholar information"""
	scholar_info_str = scholar_info_str.encode('utf-8')
	if len(scholar_info_str) < 15:
		return -1
	# Split the string into authors and other informations
	author_others_info = split_by_delimiters(COLON_DLIMITERS, scholar_info_str, 1)
	if len(author_others_info) == 1:
		return -1
	# Split the string into scholar's title and other informations
	title_others_info = split_by_delimiters(COMMA_DELIMITERS, author_others_info[1])
	# Create JSON model
	authors = split_by_delimiters(COMMA_DELIMITERS, author_others_info[0])
	authors_model = []
	for author in authors:
		authors_model.append({"name": author.strip()})
	title = title_others_info[0]
	scholar_model = {"authors": authors_model, "title": title.strip()}
	return scholar_model


def split_by_delimiters(delimiter_list, target_string, split_num=0):
	u"""
	Return a list of the words, using words in delimiter_list as
	the delimiter strings
	"""
	# Prepare delimiter string
	delimiter_str = ''
	for pattern in delimiter_list:
		delimiter_str = delimiter_str + pattern + '|'
	delimiter_str.rstrip('|')
	# Split target_string
	if split_num == 0:
		splited_str_list = re.split(delimiter_str, target_string)
	else:
		splited_str_list = re.split(delimiter_str, target_string, split_num)
	if len(splited_str_list) >= 2:
		return splited_str_list
	return target_string


def get_scholars_json_model(html):
	u"""Return JSON model for scholars information"""
	soup = BeautifulSoup(html, "lxml")
	table = soup.find_all('table')[0]
	results = []

	for _, tr_tag in enumerate(table.find_all('tr')):
		scholar_strings = []
		for _, data in enumerate(tr_tag.find_all('td')):
			if data.has_attr("align"):
				scholar_strings.append(data.get_text())
		for scholar_string in scholar_strings:
			info = get_scholar_json_model(scholar_string)
			if info != -1:
				results.append(info)
	return results


if __name__ == "__main__":
	URL = 'http://...''
	AUTH_INFO = AuthInfo(user_name='your user name', pass_word='your password')
	HTML = get_html_from(URL, AUTH_INFO)

	SCHOLARS = get_scholars_json_model(HTML)
	for scholar in SCHOLARS:
		print scholar["title"]
		for author in scholar["authors"]:
			print author["name"]
```

本当はDBに格納するところまでやりたかったけど，今回はここまで．
