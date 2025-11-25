---
date: '2022-07-30T22:36:55+09:00'
draft: false
title: 'RubyでJSONファイルを扱う方法'
tags: ["未分類"]
slug: '47'
---

この記事ではJSONの基礎と、RubyでJSONファイルを扱う方法を紹介します。

## 1. JSONの基礎

### 1-1. JSON(JavaScript Object Notation)とは

- データ記述言語の1つ
- プログラミング言語を問わず利用することができる(JavaScript, Java, PHP, Ruby, Python など)
- 名称と構文はJavaScriptにおけるオブジェクトの表記法に由来する
- MIEMタイプは`application/json`
- 拡張子は`json`
- 文字コードは`UTF-8`でエンコードすることが必須
- ウェブアプリケーションでデータを転送する場合によく使われる


### 1-2. JSONの表記方法

#### JSONのデータ型

JSONは以下のデータ型で構成されます。

1. 文字列(string)
1. 数値(number)
1. 真偽値(boolean)
1. ヌル値(null)
1. オブジェクト(object)
1. 配列(array)

##### 1. 文字列(string)

```json
{"name":"John"}
```

- ダブルクォーテーションで囲んだ文字列を指定(シングルクォーテーションは使えない)
- バックスラッシュでエスケープしたUnicode文字で構成される

##### 2. 数値(number)

```json
{
  "number_1" : 210,
  "number_2" : 215,
  "number_3" : 21.05,
  "number_4" : 10.05
}
```

- 10進法表記のみ(8進、16進法表記は使えない)
- 浮動小数点数も使用できる

##### 3. 真偽値(boolean)

```json
{
  "active_flag": true,
  "delete_flag": false
}
```

- `true` と`false`はすべて小文字で指定

##### 4. ヌル値(null)

```json
{"middlename":null}
```

- `null`はすべて小文字で指定


##### 5. オブジェクト(object)

```json
{
  "employee":{
    "name":"John",
    "age":30,
    "city":"New York"
  }
}
```

- キーとして使うデータ型は文字列に限る
- JavaScriptにおける連想配列、Rubyにおけるハッシュ


##### 6. 配列(array)

```json
[
  {
    "name":"John",
    "age":30
  },
  {
    "name":"Carol",
    "age":21
  }
]
```

- 配列要素には、文字列、数値、真偽値、ヌル値、オブジェクト、配列すべてを使用できる


## 2. RubyでJSONファイルを扱う方法

### 2-1. JSONファイルを読み込んでRubyオブジェクトに変換する

- [File.open](https://docs.ruby-lang.org/ja/latest/method/File/s/open.html)
- [JSON.#load](https://docs.ruby-lang.org/ja/latest/method/JSON/m/load.html)


**array.json**
```json
["apple","orange"]
```

上記のarray.jsonファイルを読み込んでRubyの配列オブジェクトに変換します。
```ruby
require 'json'

array = File.open('array.json') { |file| JSON.load(file) }

p array #=> ["apple", "orange"]
```

**object.json**
```json
{
  "employee": {
    "name": "Molecule Man",
    "age": 29
  }
}
```

上記のobject.jsonファイルを読み込んでRubyのハッシュオブジェクトに変換します。

```ruby
require 'json'

hash = File.open('object.json') { |file| JSON.load(file) }

p hash #=> {"employee"=>{"name"=>"Molecule Man", "age"=>29}}
```

- [【Ruby】JSON.parseエラー対処【no implicit conversion of File into String (TypeError)】 | あまブログ](https://ama-blog.com/46/)

### 2-2. RubyオブジェクトをJSONファイルへ書き込む

- [File.open](https://docs.ruby-lang.org/ja/latest/method/File/s/open.html)
- [JSON.#dump](https://docs.ruby-lang.org/ja/latest/method/JSON/m/dump.html)


Rubyの配列オブジェクトをsample1.jsonファイルに書き込みます。
```ruby
require 'json'

array = ["apple","orange"]

File.open("sample1.json", 'w') { |file| JSON.dump(array, file) }

```

**sample1.json**
```json
["apple","orange"]
```

Rubyのハッシュオブジェクトをsample2.jsonファイルに書き込みます。
```ruby
require 'json'

hash = { "Ocean" => { "Squid" => 10, "Octopus" =>8 }}

File.open("sample2.json", 'w') { |file| JSON.dump(hash, file) }
```

**sample2.json**
```json
{"Ocean":{"Squid":10,"Octopus":8}}
```

---

【参考】

- JSONの基礎
  - [JSON - Wikipedia](https://ja.wikipedia.org/wiki/JSON)
  - [とほほのJSON入門 - とほほのWWW入門](https://www.tohoho-web.com/ex/json.html)
  - [JSON の操作 - ウェブ開発の学習 | MDN](https://developer.mozilla.org/ja/docs/Learn_web_development/Core/Scripting/JSON)
  - [JavaScript JSON](https://www.w3schools.com/js/js_json.asp)
- RubyでJSONファイルを扱う方法
  - [RubyでJSONを扱うときに便利な関数まとめ - ポップインサイト](https://popinsight.jp/blog/?p=13387)
  - [RubyでJSONファイルを更新する - No Solution for Life](https://masuyama13.hatenablog.com/entry/2020/06/16/193153)
