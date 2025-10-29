---
date: '2022-06-08T09:14:49+09:00'
draft: false
title: '【Ruby】コンソールに出力結果を色付きで表示する方法【ANSIエスケープシーケンス】'
tags: ["未分類"]
slug: '23'
---

この記事ではRubyの出力結果を色付きで表示する方法を紹介します。

ターミナルの出力結果で色を使うためにはエスケープシーケンスというものを利用します。

エスケープシーケンスはターミナル上で色を含む特定の制御を実現するための特殊な文字列です。

- [ANSI escape code - Wikipedia](https://en.wikipedia.org/wiki/ANSI_escape_code#SGR_(Select_Graphic_Rendition)_parameters)
- [ANSI Escape Codes](https://gist.github.com/fnky/458719343aabd01cfb17a3a4f7296797)

この記事では色の変更に関するエスケープシーケンスのみを紹介します。

## 1. 書き方

`print "<エスケープシーケンス>文字列"`

```ruby
# 赤い文字を表示したい場合
print "\e[31m文字"
```

１つの文字列に複数のシーケンスを適用することもできる。
```ruby
# 文字色：赤、背景色：緑色
print "\e[31m\e[42m文字"
```

末尾に`\e[0m`でリセットできる。
```ruby
print "\e[31m赤文字\e[0m"
print "普通の色の文字"
```

末尾に`\e[0m`がないと、次の出力も変更した色のまま。
```ruby
print "\e[31m赤文字"
print "これも赤文字"
```

## 2. カラーコード

例：`\e[31`

|色の名前|文字色|背景色|
|---|---|---|
|黒|30|40|
|赤|31|41|
|緑|32|42|
|黄色|33|43|
|青|34|44|
|マゼンタ|35|45|
|シアン|36|46|
|白|37|47|
|デフォルト|39|49|
|リセット|0|0|

デフォルト：色のみをリセット
```ruby
print "\e[3m\e[31mイタリック赤文字\e[39m"
print "イタリック普通の色の文字"
```

リセット：色やその他の効果を全てリセット
```ruby
print "\e[3m\e[31mイタリック赤文字\e[0m"
print "普通の書式で普通の色の文字"
```

---

【参考】

- [RubyでANSIカラーシーケンスを学ぼう！](https://melborne.github.io/2010/11/07/Ruby-ANSI/)
- [ANSIエスケープシーケンス チートシート #Bash - Qiita](https://qiita.com/PruneMazui/items/8a023347772620025ad6)
- [Ruby: コンソールで色付き文字をエスケープシーケンスを使って表示 | Jura-Zakki 樹羅雑記](https://makandat.wordpress.com/2016/09/18/ruby-%E3%82%B3%E3%83%B3%E3%82%BD%E3%83%BC%E3%83%AB%E3%81%A7%E8%89%B2%E4%BB%98%E3%81%8D%E6%96%87%E5%AD%97%E3%82%92%E3%82%A8%E3%82%B9%E3%82%B1%E3%83%BC%E3%83%97%E3%82%B7%E3%83%BC%E3%82%B1%E3%83%B3/)
- [Rubyでコンソールに色つきで出力 #Ruby - Qiita](https://qiita.com/kymmt@github/items/4dc72bcb04da7b90a3a1)
- [Rubyで出力に色を付ける方法 #rubygems - Qiita](https://qiita.com/khotta/items/9233a9ffeae68b58d84f)
