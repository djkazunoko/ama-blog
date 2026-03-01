---
date: '2022-10-30T11:18:18+09:00'
draft: false
title: '【Rails】gemのアンインストール方法'
tags: ["未分類"]
slug: '67'
---

この記事では、RailsアプリでBundlerを使ってインストールしたgemを削除する方法を紹介します。

## 手順

```
$ bundle exec gem uninstall <gem名>
↓
Gemfileから該当のgemの行を削除
↓
$ bundle install
```

上記により`Gemfile.lock`からも削除されます。


---

【参考】

- [gem uninstall](https://guides.rubygems.org/command-reference/#gem-uninstall)
- [gem install](https://guides.rubygems.org/command-reference/#gem-install)
- [bundle exec](https://bundler.io/v2.4/man/bundle-exec.1.html)
- [【Rails】gemのアンインストール・バージョン変更 #Rails5 - Qiita](https://qiita.com/manbolila/items/f412a29782e4352d1168)
- [あえて言うほどではないけれどもGemを一括削除する方法 | TECHSCORE BLOG](https://www.techscore.com/blog/2012/10/30/gem%E3%82%92%E4%B8%80%E6%8B%AC%E5%89%8A%E9%99%A4%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95/)
