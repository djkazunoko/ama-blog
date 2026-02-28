---
date: '2022-10-29T23:42:30+09:00'
draft: false
title: '【Bundler】Gemfileのバージョン指定の書き方'
tags: ["未分類"]
slug: '66'
---

GemfileはBundlerでgemの依存関係を管理するために必要な設定ファイルです。

Gemfileにはgem名とバージョンを`gem 'rails', '~> 6.1.5'`のようなフォーマットで記述します。

**`~> 6.1.5`の正確な意味わかりますか？**

**`~>`の呼び方知ってますか？**(<span style="color: red;">**トゥウィドルワッカ**</span>と呼びます)

この記事ではGemfileのバージョン指定の書き方を解説していきます。

## 1. Gemfileのバージョン指定子の意味

| バージョン指定 | 意味 |
|-----------|------------|
| `1.0` | 1.0のみ |
| `>= 1.0` | 1.0以上 |
| `~> 1.0` | 1.0以上2.0未満 |
| `~> 1.0.0` | 1.0.0以上1.1未満 |
| 指定なし | 最新版 |

- `~> 1.0`はマイナーバージョン(0.x.0)は上がってもいいがメジャーバージョン(x.0.0)は上げたくないことを意味する
- `~> 1.0.0`はパッチバージョン(0.0.x)は上がってもいいがマイナーバージョン(0.x.0)は上げたくないことを意味する


```ruby
# 以下二つは同じ意味
gem 'rails', '~> 6.1'
gem 'rails', '>= 6.1', '< 7.0'
```

```ruby
# 以下二つは同じ意味
gem 'rails', '~> 6.1.5'
gem 'rails', '>= 6.1.5', '< 6.2'
```

## 2. twiddle wakka(トゥウィドルワッカ)とは

`~>`がtwiddle wakkaです。

Gemfileのバージョン指定にはoptimistic version constraint(楽観的なバージョン指定)とpessimistic version constraint(悲観的なバージョン指定)があります。[^1]

- optimistic version constraint
  - `gem 'library', '>= 2.2.0'`
- pessimistic version constraint
  - `gem 'library', '>= 2.2.0', '< 3.0'`

このpessimistic version constraintの省略記法としてできたのが`~>`(twiddle wakka)です。

`gem 'library', '>= 2.2.0', '< 3.0'`をtwiddle wakkaを使って書くと`gem 'library', '~> 2.2'`になります。

Jim Weirich氏により当初は`>* `という書式で追加されたのちに、現在の`~> `に変更されたみたいです。[^2]

`~> `をなんでtwiddle wakkaと呼ぶようになったかについては、プログラマーたちの間で`~`(チルダ)をtwiddleと呼び、`<>`(山括弧)をwakkaと呼ぶようになったことからきてるらしいです。[^3]

---

【参考】

- [Gemfile - Bundler](https://bundler.io/guides/gemfile.html)
- [Semantic Versioning](https://semver.org/)
- [【Rails】Gemfileのバージョン指定の書き方 - Yohei Isokawa](https://blog.yuhiisk.com/archive/2017/04/24/specify-the-version-of-gemfile.html)
- [Get to know your twiddle wakka - Depfu Blog](https://depfu.com/blog/2016/12/14/get-to-know-your-twiddle-wakka)
- [Ruby の gem のバージョン指定に利用する ~> の名前に関するまとめ #Ruby - Qiita](https://qiita.com/tbpgr/items/e892fd9536d187f7c220)

[^1]: [Pessimistic version constraint - RubyGems Guides](https://guides.rubygems.org/patterns/#pessimistic-version-constraint)
[^2]: [ruby - Why is twiddle wakka designed like this? - Stack Overflow](https://stackoverflow.com/questions/28332750/why-is-twiddle-wakka-designed-like-this)
[^3]: [The Strange Case of the Twiddle Wakka | by Alyssa Lerner First | Medium](https://alerner1st.medium.com/the-strange-case-of-the-twiddle-wakka-5a70a0f5a509)
