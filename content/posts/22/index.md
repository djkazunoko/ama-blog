---
date: '2022-06-08T09:08:47+09:00'
draft: false
title: '【macOS】PostgreSQLの基本操作'
tags: ["未分類"]
slug: '22'
---

この記事では、PostgreSQLの基本操作を解説します。

- バージョン情報
  - macOS Monterey 12.4
  - PostgreSQL 14.3

## 1. インストール

**インストール**

```
$ brew install postgresql
```

**バージョン確認**

```
$ psql --version
psql (PostgreSQL) 14.3
```

## 2. データベースサーバの起動・停止

### 2-1. brew servicesコマンド

[brew services](https://github.com/Homebrew/homebrew-services)

**起動**

```
$ brew services start postgresql
```

**停止**

```
$ brew services stop postgresql
```

### 2-2. pg_ctlコマンド

[pg_ctlコマンド](https://www.postgresql.org/docs/current/app-pg-ctl.html)

**起動**

```
$ pg_ctl -D /usr/local/var/postgres start
```

- `pg_ctl -D <データディレクトリのパス> start`

**停止**

```
$ pg_ctl -D /usr/local/var/postgres stop
```

- `pg_ctl -D <データディレクトリのパス> stop`


**データディレクトリのパスの確認**

```
postgres=# SHOW data_directory;
     data_directory
-------------------------
 /usr/local/var/postgres
(1 row)
```

**`-D`オプションの省略**

環境変数`PGDATA`にデータディレクトリのパスを設定することで、`-D`オプションを省略して起動・停止ができる。

`~/.zshrc`に以下を追加

```
export PGDATA=/usr/local/var/postgres
```

`.zshrc`を再読み込み

```
$ source ~/.zshrc
```

以下のように起動・停止が可能

```
$ pg_ctl start
$ pg_ctl stop
```

## 3. データベースへの接続

[psql](https://www.postgresql.org/docs/current/app-psql.html)

**データベースへの接続**

```
$ psql postgres
```

- PostgreSQLインストール時にデフォルトで作成済みの`postgres`データベースに接続
- `psql -U <ユーザ名> -d <データベース名>`
  - `-U <ユーザ名>`を省略するとシステムのユーザ(`$USER`)でデータベースに接続
  - 1つ目の引数にデータベース名を指定した場合、`-d`オプションは省略可

**接続中のデータベースの確認**

```
postgres=> SELECT current_database();
 current_database
------------------
 postgres
(1 row)
```

**現在の接続情報の確認**

```
postgres=# \conninfo
You are connected to database "postgres" as user "testuser" via socket in "/tmp" at port "5432".
```

## 4. ユーザの作成

### 4-1. コマンドライン

[createuser](https://www.postgresql.org/docs/current/app-createuser.html)

```
$ createuser -s testuser
```

- `createuser [オプション] <ユーザ名>`
  - `-s`オプションでスーパーユーザを作成

### 4-2. SQLコマンド

[CREATE USER](https://www.postgresql.org/docs/current/sql-createuser.html)

```
postgres=# CREATE USER testuser WITH SUPERUSER;
CREATE ROLE
```

- `CREATE USER <ユーザ名> [オプション];`
  - `WITH SUPERUSER`オプションでスーパーユーザを作成

**ユーザ一覧の確認**

```
postgres=# SELECT * FROM pg_user;
 usename  | usesysid | usecreatedb | usesuper | userepl | usebypassrls |  passwd  | valuntil | useconfig
----------+----------+-------------+----------+---------+--------------+----------+----------+-----------
 user     |       10 | t           | t        | t       | t            | ******** |          |
 testuser |    16387 | f           | t        | f       | f            | ******** |          |
(2 rows)
```

**ロール一覧の確認**

```
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 user      | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 testuser  | Superuser
```

**現在のユーザの確認**

```
postgres=# SELECT current_user;
 current_user
--------------
 testuser
(1 row)
```

## 5. ユーザの削除

### 5-1. コマンドライン

[dropuser](https://www.postgresql.org/docs/current/app-dropuser.html)

```
$ dropuser testuser
```

- `dropuser [オプション] <ユーザ名>`

### 5-2. SQLコマンド

[DROP USER](https://www.postgresql.org/docs/current/sql-dropuser.html)( = [DROP ROLE](https://www.postgresql.org/docs/current/sql-droprole.html))

```
postgres=# DROP USER testuser;
DROP ROLE
```

- `DROP USER <ユーザ名>;`

## 6. データベースの作成

### 6-1. コマンドライン

[createdb](https://www.postgresql.org/docs/current/app-createdb.html)

```
$ createdb testdb -O testuser
```

- `createdb [データベース名] [オプション]`
  - データベース名を省略した場合、データベース名は`$USER`になる
  - `-O <ユーザ名>`でデータベースの所有者となるユーザを指定(省略した場合の所有者は`$USER`になる)

### 6-2. SQLコマンド

[CREATE DATABASE](https://www.postgresql.org/docs/current/sql-createdatabase.html)

```
postgres=# CREATE DATABASE testdb OWNER testuser;
CREATE DATABASE
```

- `CREATE DATABASE <データベース名> [オプション];`
  - `OWNER <ユーザ名>`でデータベースの所有者となるユーザを指定(省略した場合の所有者は現在データベースサーバーにログイン中のユーザになる)

**データベース一覧の確認**

```
postgres=# \l
                          List of databases
   Name    | Owner  | Encoding | Collate | Ctype | Access privileges
-----------+--------+----------+---------+-------+-------------------
 postgres  | user   | UTF8     | C       | C     |
 template0 | user   | UTF8     | C       | C     | =c/user          +
           |        |          |         |       | user  =CTc/user  
 template1 | user   | UTF8     | C       | C     | =c/user          +
           |        |          |         |       | user  =CTc/user  
(3 rows)
```

## 7. データベースの削除

### 7-1. コマンドライン

[dropdb](https://www.postgresql.org/docs/current/app-dropdb.html)

```
$ dropdb testdb
```

- `dropdb <データベース名> [オプション]`

### 7-2. SQLコマンド

[DROP DATABASE](https://www.postgresql.org/docs/current/sql-dropdatabase.html)

```
postgres=# DROP DATABASE testdb;
DROP DATABASE
```

- `DROP DATABASE <データベース名> [オプション];`

---

【参考】

- [PostgreSQL サーバの起動と停止 #PostgreSQL - Qiita](https://qiita.com/domodomodomo/items/12fe7555513de6b078db)
- [[macOS High Sierra][Homebrew] PostgreSQL のインストールからDB作成まで #homebrew - Qiita](https://qiita.com/ksh-fthr/items/b86ba53f8f0bccfd7753)
