---
date: '2022-06-10T09:29:29+09:00'
draft: false
title: 'DebianにPostgreSQLをインストールして外部から接続する方法'
tags: ["未分類"]
slug: '25'
---

この記事では、さくらのVPS上のDebianにPostgreSQLをインストールして、Macから外部接続する方法を紹介します。

## 1. 環境
- Debian GNU/Linux 11 bullseye (さくらのVPS)
- macOS Monterey 12.4
- PostgreSQL 14.3

## 2. 手順

以下の流れで進めていきます。

1. PostgreSQLのインストール
1. 接続確認
1. ユーザの作成
1. 外部接続の設定
1. 外部接続の確認


### 2-1. PostgreSQLのインストール

#### 2-1-1. 必要なパッケージのインストール
まずは、PostgreSQLのインストールに必要な以下のパッケージをインストールします。

- [curl](https://packages.debian.org/bullseye/curl)
- [gnupg2](https://packages.debian.org/bullseye/gnupg2)
- [ca-certificates](https://packages.debian.org/bullseye/ca-certificates)
- [lsb-release](https://packages.debian.org/bullseye/lsb-release)

以下のコマンドを実行してパッケージをインストール。
```
$ sudo apt install curl gnupg2 ca-certificates lsb-release
```

- パッケージの情報を表示：`apt show <パッケージ名>`
- インストール済みパッケージの確認：`apt list --installed <パッケージ名>`

#### 2-1-2. 署名鍵のインポート

次に、aptがパッケージの信頼性を確認できるようにするために、PostgreSQLが公式に公開している署名鍵をインポートします。

以下のコマンドを実行して署名鍵をインポート。
```
$ curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/apt.postgresql.org.gpg >/dev/null
```

#### 2-1-3. リポジトリの追加

aptの設定ファイルにPostgreSQLの公式リポジトリを追加します。

以下のコマンドを実行して `/etc/apt/sources.list.d` 配下に設定ファイルを作成します。

```
$ echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
```

作成された `/etc/apt/sources.list.d/pgdg.list` の内容は以下のようになります。

```
deb http://apt.postgresql.org/pub/repos/apt bullseye-pgdg main
```

#### 2-1-4. PostgreSQLのインストール

以下のコマンドを実行してPostgreSQLをインストールします。
```
$ sudo apt update
$ sudo apt install postgresql
```

上記の一連の流れについて、以下でより詳細に解説しています。

[【Debian】Nginx公式リポジトリにある最新バージョンのNginxをインストールする | あまブログ](https://ama-blog.com/18/)

### 2-2. 接続確認
次に、データベースへの接続確認を行います。

PostgreSQLをインストールするとpostgresユーザが自動で作成されます。

以下では、postgresユーザでデータベースへの接続確認を行います。

まずは、postgresユーザが作成されていることを確認。
```
$ cat /etc/passwd | grep postgres
```

次に、postgresユーザにパスワードを設定します。
```
$ sudo passwd postgres
New password:
Retype new password:
passwd: password updated successfully
```

postgresユーザに切り替え。(先程設定したパスワードを入力。)
```
$ su - postgres
```

postgresユーザに切り替わっていることを確認し、以下のコマンドを実行してデータベースに接続します。
```
postgres@ ~$ psql
```

データベースに接続できました。
```
postgres=# SELECT current_database();
 current_database
------------------
 postgres
(1 row)
```

上記ではPostgreSQLインストール時に自動生成されるpostgresデータベースに接続しています。

PostgreSQLの基本的な操作方法は以下を参照してください。

[【macOS】PostgreSQLの基本操作 | あまブログ](https://ama-blog.com/22/)

### 2-3. ユーザの作成

次に、外部接続で使用する、DB操作用のユーザを作成します。

以下のコマンドを実行して、ユーザを作成します。
ユーザ名には、OS管理ユーザ(現在ログインしているユーザ)の名前を入力します。
```
postgres=# CREATE USER <ユーザ名> SUPERUSER PASSWORD '<パスワード>';
CREATE ROLE
```

- [CREATE USER](https://www.postgresql.org/docs/current/sql-createuser.html)

### 2-4. 外部接続の設定

次に、外部接続の設定を行います。

#### 2-4-1. postgresql.confの編集

以下のコマンドを実行して、`/etc/postgresql/14/main/postgresql.conf` を編集します。

```
$ sudo vi /etc/postgresql/14/main/postgresql.conf
```

編集前(60行目あたり)
```zsh
#listen_addresses = 'localhost'		# what IP address(es) to listen on;
```

編集後(60行目あたり)
```zsh
#listen_addresses = 'localhost'		# what IP address(es) to listen on;
listen_addresses = '*'
```

#### 2-4-2. pg_hba.confの編集

以下のコマンドを実行して、`/etc/postgresql/14/main/pg_hba.conf` を編集します。(hba：host-based authentication)
```
$ sudo vi /etc/postgresql/14/main/pg_hba.conf
```

編集前(96行目あたり)
```zsh
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
```

編集後(96行目あたり)
```zsh
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             <グローバルIPアドレス>/32            scram-sha-256
```

<グローバルIPアドレス>にはMacのグローバルIPアドレスをIPv4形式で入力します。

グローバルIPアドレスの確認
```
$ curl https://ifconfig.me
XXX.XXX.XXX.XXX
```

`postgresql.conf` と `pg_hba.conf` の編集後、PostgreSQLを再起動して設定の変更を反映させます。

```
$ sudo systemctl restart postgresql
```

### 2-5. 外部接続の確認

最後に、外部接続の確認を行います。

Macで以下のコマンドを実行します。
```
$ psql -U <ユーザ名> -d <データベース名> -h <ホスト名>
```
<ユーザ名>に先程作成したDB操作用のユーザ名、<データベース名>にpostgres、<ホスト名>にさくらのVPSのホスト名(またはIPアドレス)を入力します。

以上で終了です。

---

【参考】

- 手順の参考
  - [Debian 8.7(Jessie)にPostgreSQL 9.6をインストールし、外部接続を許可する - Symfoware](https://symfoware.blog.fc2.com/blog-entry-1948.html)
  - [DebianにPostgreSQLをインストールして外部から接続する](https://zenn.dev/ryo18/articles/d5d051455c83e7)
  - [環境構築：Debian 10(buster) にPostgreSQL 11をインストールし、新規DBを作成する方法](https://debimate.jp/2020/03/20/%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89%EF%BC%9Adebian-10buster-%E3%81%ABpostgresql-11%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%97%E3%80%81%E6%96%B0%E8%A6%8Fdb%E3%82%92/)

- インストール
  - [PostgreSQL: Linux downloads (Debian)](https://www.postgresql.org/download/linux/debian/)
  - [Apt - PostgreSQL wiki](https://wiki.postgresql.org/wiki/Apt)

- postgresql.confの設定
  - [PostgreSQL: Documentation: 18: 19.3. Connections and Authentication](https://www.postgresql.org/docs/current/runtime-config-connection.html)

- pg_hba.confの設定
  - [PostgreSQL: Documentation: 18: 20.1. The pg_hba.conf File](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)
  - [PostgreSQL | pg_hba.confファイルの設定方法](https://www.javadrive.jp/postgresql/ini/index2.html)
  - Password Authentication
    - [PostgreSQL: Documentation: 18: 20.5. Password Authentication](https://www.postgresql.org/docs/current/auth-password.html)
    - [To scram-sha-256 from MD5 in PostgreSQL| CYBERTEC PostgreSQL | Services & Support](https://www.cybertec-postgresql.com/en/from-md5-to-scram-sha-256-in-postgresql/)

- peer認証
  - [PostgreSQL: Documentation: 18: 20.9. Peer Authentication](https://www.postgresql.org/docs/current/auth-peer.html#:~:text=The%20peer%20authentication%20method%20works,only%20supported%20on%20local%20connections.)
  - https://command-f.tech/postgresql/1

- IPアドレスについて
  - [用語集「IPアドレスとは？」](https://www.cman.jp/network/term/ip/)

---

【メモ】

- Linuxユーザ確認
  - 一覧確認：`cat /etc/passwd`
  - 現在のユーザ確認：`whoami`
  - ユーザ切り替え：`su - <ユーザ名>`

- SSH接続時のグローバルIPアドレスの確認方法
  - [自分がどのIPアドレスでSSH接続しているのかを確認する | DevelopersIO](https://dev.classmethod.jp/articles/amazon-linux-2-ssh-ipaddress/)

- プライベートIPアドレスの確認方法
  - 方法1：[Macでネットワークインターフェイスに割り当てられたIPアドレスを調べる方法 #ifconfig - Qiita](https://qiita.com/suin/items/e65e5477a954d7b81e1b)
  - 方法2：「システム環境設定」>「ネットワーク」>「詳細」>「TCP/IP」>「IPv4アドレス」
