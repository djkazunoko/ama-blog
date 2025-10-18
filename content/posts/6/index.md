---
date: '2022-05-03T23:38:43+09:00'
draft: false
title: 'MySQLをHomebrewでインストールしてセキュリティ設定を行う'
tags: ["未分類"]
slug: '6'
---

本稿ではMySQLのインストールからセキュリティ設定までの手順を解説します。

## 1. 開発環境
- macOS Monterey バージョン12.3.1
- Homebrew 3.4.10
- MySQL 8.0.28

## 2. 手順

### 2-1. MySQLのインストール
[Homebrew](https://brew.sh/index_ja)でMySQLをインストールします。
```
$ brew install mysql
```

MySQLがインストールされたことを確認します。
```
$ brew info mysql
```

[brew info - Homebrew Documentation](https://docs.brew.sh/Manpage#info-abv-options-formulacask-)

### 2-2. MySQLのセキュリティ設定
サーバーを起動します。
```
$ mysql.server start
```
セキュリティ設定を開始します。
```
$ mysql_secure_installation
```

以下の項目の設定を行います。

- VALIDATE PASSWORDプラグインの利用確認
- rootユーザーのパスワード設定
- 匿名ユーザーの削除
- リモートからrootユーザでログインできないようにする
- testデータベースの削除
- ユーザーの権限に関するテーブルの再読み込み

```zsh
# VALIDATE PASSWORDプラグインの利用確認
VALIDATE PASSWORD COMPONENT can be used to test passwords
省略
Press y|Y for Yes, any other key for No:<Enterを押してスキップ>
```
```zsh
# rootユーザーのパスワード設定
Please set the password for root here.

New password:<パスワードを入力>
```
```zsh
# 匿名ユーザーの削除
By default, a MySQL installation has an anonymous user,
省略
Remove anonymous users? (Press y|Y for Yes, any other key for No) : <yを入力>
```
```zsh
# リモートからrootユーザでログインできないようにする
Normally, root should only be allowed to connect from
省略
Disallow root login remotely? (Press y|Y for Yes, any other key for No) :<yを入力>
```
```zsh
# testデータベースの削除
Remove test database and access to it? (Press y|Y for Yes, any other key for No) :<yを入力> 
```
```zsh
# ユーザーの権限に関するテーブルの再読み込み
Reload privilege tables now? (Press y|Y for Yes, any other key for No) :<yを入力>
```
[MySQL 8.0 リファレンスマニュアル 「mysql_secure_installation」](https://dev.mysql.com/doc/refman/8.0/ja/mysql-secure-installation.html)

最後に、設定したパスワードでログインできるか確認します。
```
$ mysql -u root -p
```

以上で終了です。

___

【参考】

- [Mac へ MySQL を Homebrew でインストールする手順](https://qiita.com/hkusu/items/cda3e8461e7a46ecf25d)
- [Progate_MySQLの開発環境を用意しよう（macOS）](https://prog-8.com/docs/mysql-env)
