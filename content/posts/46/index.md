---
date: '2022-07-30T22:29:25+09:00'
draft: false
title: '【Ruby】JSON.parseエラー対処【no implicit conversion of File into String (TypeError)】'
tags: ["未分類"]
slug: '46'
---

## 1. バージョン情報
- Ruby：3.1.0
- robocop：1.32.0

## 2. 経緯

以下のような、JSONファイルを読み込むコードを書いていた。
```ruby
require 'json'

file_path = "test.json"

p File.open(file_path) { |f| JSON.load(f) } #=> ファイルの内容
```

これをrubocopでチェックすると以下の警告文が表示された。
```
Security/JSONLoad: Prefer JSON.parse over JSON.load.
```
[JSON.load()](https://docs.ruby-lang.org/ja/latest/method/JSON/m/load.html)ではなく[JSON.parse()](https://docs.ruby-lang.org/ja/latest/method/JSON/m/parse.html)を使えと言っている。

以下のように`JSON.parse()`を使うように修正したところエラーが発生した。
```ruby
require 'json'

file_path = "test.json"

p File.open(file_path) { |f| JSON.parse(f) } #=> no implicit conversion of File into String (TypeError)
```

## 3. エラー対処法

`JSON.parse()`は引数に文字列を指定する必要があり、引数にファイルは指定できない。

なので、先ほどのコードを以下のように修正すると上手くいく。

```ruby
require 'json'

file_path = "test.json"

p File.open(file_path) { |f| JSON.parse(f.read) } #=> ファイルの内容
```
- [Fileクラス](https://docs.ruby-lang.org/ja/latest/class/File.html)
  - [IO#read](https://docs.ruby-lang.org/ja/latest/method/IO/i/read.html)

---

【参考】

- [RubyDoc.info: Class: RuboCop::Cop::Security::JSONLoad – Documentation for rubocop (0.46.0) – RubyDoc.info](https://www.rubydoc.info/gems/rubocop/0.46.0/RuboCop/Cop/Security/JSONLoad)
- [ruby - No implicit conversion of file into string - Stack Overflow](https://stackoverflow.com/questions/28685897/no-implicit-conversion-of-file-into-string)
