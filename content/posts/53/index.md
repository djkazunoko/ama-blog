---
date: '2022-08-16T21:39:57+09:00'
draft: false
title: '【Ruby】rbenvのよく使うコマンド一覧'
tags: ["未分類"]
slug: '53'
---

この記事ではrbenvのよく使うコマンドを紹介していきます。

## 1. バージョン確認

### 1-1. 現在使用中のRubyのバージョンを表示
```
$ rbenv version
```

### 1-2. インストール済みのRubyのバージョンを表示
```
$ rbenv versions
```
- `~/.rbenv/versions`のRubyのバージョンを表示

## 2. インストール

### 2-1. インストール可能なRubyの最新安定版のバージョンを表示
```
$ rbenv install -l/--list
```

### 2-2. インストール可能なRubyの全バージョンを表示
```
$ rbenv install -L/--list-all
```

### 2-3. Rubyのインストール
```
$ rbenv install <version>
```
- `~/.rbenv/versions`にインストールされる

### 2-4. Rubyのアンインストール
```
$ rbenv uninstall <version>
```

## 3. バージョン指定

**バージョンの優先順位**

1. 環境変数： ``RBENV_VERSION``
1. ローカル ：``rbenv local <version>``
1. グローバル：``rbenv global <version>``

### 3-1. デフォルトで使用するRubyのバージョンを指定
```
$ rbenv global <version>
```
- `~/.rbenv/version`が作成され、グローバルに設定したRubyのバージョンが書き込まれる

### 3-2. カレントディレクトリで使用するRubyのバージョンを指定
```
$ rbenv local <version>
```
- カレントディレクトリに`.ruby-version`が作成され、ローカルに設定したRubyのバージョンが書き込まれる
- `rbenv local --unset`で解除

### 3-3. シェル単位で使用するRubyのバージョンを指定
```
$ rbenv shell <version>
```
- 環境変数``$RBENV_VERSION``に指定したRubyのバージョンがセットされる
- `rbenv shell --unset`で解除

## 4. その他

### 4-1. rbenvのバージョンを表示
```
$ rbenv -v
```

### 4-2. rbenvとruby-buildのアップグレード
```
$ brew upgrade rbenv ruby-build
```
- 上記はHomebrewのコマンド
- `rbenv install -l`でRubyの最新安定版のバージョンが表示されない時は`rbenv`と`ruby-build`の更新が必要かも

### 4-3. rbenvのヘルプを表示
```
$ rbenv -h
```

### 4-4. rbenvのコマンド詳細情報を表示
```
$ rbenv help <command>
```

---

【参考】

- [rbenv](https://github.com/rbenv/rbenv)
- [ruby-build](https://github.com/rbenv/ruby-build)
- [ちゃんと理解するrbenv](https://mogulla3.tech/articles/2021-07-10-understanding-rbenv-1)
- [[rbenv]コマンド備忘録](https://qiita.com/a_ishidaaa/items/8cc14453289dba1413dd)
- [rbenv cheatsheet](https://devhints.io/rbenv)
- [rbenvのよく使うコマンドまとめ - TASK NOTES](https://www.task-notes.com/entry/20141204/1417662000)



