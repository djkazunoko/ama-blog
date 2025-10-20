---
date: '2025-05-08'
draft: false
title: '【Rails/MySQL】RailsでMySQLを使う方法'
tags: ["未分類"]
slug: '10'
---

## 1. はじめに
Railsではデフォルトのデータベースにsqlite3を使用しています。

今回はRailsアプリケーションのデータベースでMySQLを使用する方法を紹介します。

## 2. 開発環境
- macOS Monterey：12.3.1
- Ruby：3.1.0
- Bundler：2.3.12
- Ruby on Rails：6.1.5
- MySQL：8.0.28

## 3. 手順

### 3-1. MySQLのインストールとセキュリティ設定
まずは、MySQLのインストールとセキュリティの設定を行います。

[MySQLをHomebrewでインストールしてセキュリティ設定を行う | あまブログ](https://ama-blog.com/6/)

### 3-2. MySQLのユーザー作成
次にMySQLの開発用ユーザーを作成します。

rootユーザーでログインします。
```
$ mysql -u root -p
```
ユーザーの作成します。
```
mysql> CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
```
- [MySQL 8.0 リファレンスマニュアル_CREATE USER ステートメント](https://dev.mysql.com/doc/refman/8.0/ja/create-user.html)

ユーザーが作成されたことを確認します。
```
mysql> SELECT User,Host FROM mysql.user;
 +------------------+-----------+
 | User             | Host      |
 +------------------+-----------+
 | mysql.infoschema | localhost |
 | mysql.session    | localhost |
 | mysql.sys        | localhost |
 | root             | localhost |
 | newuser          | localhost |
 +------------------+-----------+
 5 rows in set (0.00 sec)
```

権限の付与します。
```
mysql> GRANT ALL ON * . * TO 'newuser'@'localhost';
```
今回の場合は全てのデータベースへのフルアクセス権を付与していますが、本来はユースケースに合わせた適切な権限を付与します。

- [MySQL 8.0 リファレンスマニュアル_GRANT ステートメント](https://dev.mysql.com/doc/refman/8.0/ja/grant.html)

権限が付与されたことを確認します。
```
mysql> SHOW GRANTS FOR 'newuser'@'localhost';
```
- [MySQL 8.0 リファレンスマニュアル_SHOW GRANTS ステートメント](https://dev.mysql.com/doc/refman/8.0/ja/show-grants.html)

### 3-3. Railsアプリケーションの作成
次に、使用するデータベースにMySQLを指定してRailsアプリケーションを作成します。
```
$ rails new <アプリ名> -d mysql
```
- [【Rails/MySQL】bundle installでgem mysql2がインストールできない時の解決法 - あまブログ](https://ama-tech.hatenablog.com/mysql2-cannot-install-with-bundle)

アプリの作成後に`config/database.yml`を以下のように修正します。

```yaml
省略

default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <ユーザー名>
  password: <ユーザーのパスワード>
  socket: /tmp/mysql.sock

development:
  <<: *default
  database: アプリ名_development

省略

test:
  <<: *default
  database: アプリ名_test

省略

production:
  <<: *default
  database: アプリ名_production
  username: アプリ名
  password: <%= ENV['アプリ名_DATABASE_PASSWORD'] %>
```

### 3-4. dotenv-railsの導入
次に、データベースの認証情報を安全に管理するために`dotenv-rails`を導入します。

[【Rails/MySQL】dotenv-railsを使ってデータベースの認証情報を環境変数で管理する | あまブログ](https://ama-blog.com/8/)

なお、作成したアプリをローカル環境でしか使わないなどの場合は特に気にする必要はないので、この手順はとばしてokです。


### 3-5. データベースの作成
以下のコマンドを実行してデータベースを作成します。
```
$ rails db:create
```

`rails db:create`コマンドはデフォルトで`development`と`test`の2つのデータベースを作成します。

最後にMySQLにログインしてデータベースが作成されたことを確認します。
```
mysql> SHOW DATABASES;
+-----------------------+
| Database              |
+-----------------------+
| information_schema    |
| ***_development       |
| ***_test              |
| mysql                 |
| performance_schema    |
| sys                   |
+-----------------------+
6 rows in set (0.06 sec)
```

以上で終了です。

___

【参考】

- [【Rails/MySQL】RailsにMySQLを導入する方法【プログラミング学習149日目】](https://qiita.com/fuku_tech/items/a380ebb1fd156c14c25b)
- [Ruby on RailsでMySQLを使用・接続する際の手順と注意点](https://arrown-blog.com/rails-mysql/)
- [【初心者向け】RailsでMySQLを使うための手順をコマンド付きで解説！](https://himakuro.com/rails-mysql-setup)
- [MySQL用のデータベース設定ファイル(database.yml)](https://www.javadrive.jp/rails/model/index2.html)
- [MySQLで新しいユーザーを作成して権限を付与する方法](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql-ja)
- [[MySQL]権限の確認と付与](https://qiita.com/shuntaro_tamura/items/2fb114b8c5d1384648aa)
