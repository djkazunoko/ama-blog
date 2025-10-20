---
date: '2022-05-05'
draft: false
title: '【Rails/MySQL】dotenv-railsを使ってデータベースの認証情報を環境変数で管理する'
tags: ["未分類"]
slug: '8'
---

RailsにMySQLを導入する際に、パスワードを`database.yml`に直接書くのはいかがなものかと思い、他の方法を調べた結果、`dotenv-rails`を使って認証情報を環境変数で管理できることがわかりました。

今回は作成済みのRailsアプリケーションにdotenv-railsを導入する手順を紹介します。

## 1. 開発環境
- macOS Monterey：12.3.1
- Ruby：3.1.0
- Ruby on Rails：6.1.5
- dotenv-rails：2.7.6
- MySQL：8.0.28

## 2. dotenv-railsとは
- 認証情報などを環境変数で管理するためのgem
- 起動時にプロジェクトのルートディレクトリにある`.env`を読み込み、環境変数`ENV`に設定する
- [dotenv-rails公式レポジトリ](https://github.com/bkeepers/dotenv)

## 3. 手順

### 3-1. dotenv-railsのインストール
`Gemfile`に以下を追記します。
```
gem 'dotenv-rails'
```
`dotenv-rails`をインストールします。
```
$ bundle install
```

### 3-2. database.ymlの修正
`database.yml`を以下のように修正します。
```yaml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <%= ENV['DATABASE_DEFAULT_USER'] %>
  password: <%= ENV['DATABASE_DEFAULT_PASSWORD'] %>
  host: <%= ENV['DATABASE_DEFAULT_HOST'] %>
  socket: /tmp/mysql.sock

development:
  <<: *default
  database: <%= ENV['DATABASE_DEV_NAME'] %>

test:
  <<: *default
  database: <%= ENV['DATABASE_TEST_NAME'] %>
```

`<<: *default`で一番上のdefaultの設定が呼び出されています。

こうやって書いたのと同じ↓
```yaml
development:
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <%= ENV['DATABASE_DEFAULT_USER'] %>
  password: <%= ENV['DATABASE_DEFAULT_PASSWORD'] %>
  host: <%= ENV['DATABASE_DEFAULT_HOST'] %>
  socket: /tmp/mysql.sock
  database: <%= ENV['DATABASE_DEV_NAME'] %>
```

### 3-3. .envの作成
アプリケーションのルートディレクトリ配下に、`.env`を作成します。
```
$ touch .env
```

`.env`に以下を追記します。
```
DATABASE_DEFAULT_USER = '<ユーザー名>'
DATABASE_DEFAULT_PASSWORD = '<パスワード>'
DATABASE_DEFAULT_HOST = '<ホスト名>'
DATABASE_DEV_NAME = '<開発用のデータベース名>'
DATABASE_TEST_NAME = '<テスト用のデータベース名>'
```

### 3-4. .gitignoreの編集
`.env`をGitの管理対象から除外するために、`.gitignore`に以下を追記します。

```
# /.envの追加
/.env
```

以上でdotenv-railsの導入は完了です。

___

【参考】

- [Railsガイド_3.19 データベースを設定する](https://railsguides.jp/configuring.html#%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)
- [Quora_rails db:createの挙動](https://jp.quora.com/ruby-on-rails%E3%81%A7rake-db-create%E3%82%92%E3%81%99%E3%82%8B%E6%99%82%E3%81%AE%E6%8C%99%E5%8B%95%E3%81%AE%E8%A9%B3%E7%B4%B0%E3%82%92%E6%95%99%E3%81%88%E3%81%A6%E4%B8%8B%E3%81%95%E3%81%84-%E3%81%AA%E3%81%9C)
- [【YAML】Railsのdatabase.ymlについてなんとなく分かった気になっていた記法・意味まとめ](https://qiita.com/terufumi1122/items/b5678bae891ba9cf1e57)
- [RailsのDatabaseをMySQLにして、パスワードをdotenvで定数化する](http://www.kaasan.info/archives/4251)
- [【Rails】 dotenv-railsを使って環境変数を管理しよう](https://pikawaka.com/rails/dotenv-rails)
