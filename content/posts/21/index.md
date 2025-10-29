---
date: '2022-06-07T08:57:52+09:00'
draft: false
title: 'SQLの基本を理解する'
tags: ["未分類"]
slug: '21'
---

## 1. SQLの概要

### 1-1. SQLとは

- リレーショナルデータベースを操作するための言語
- 標準規格に準拠したSQLを標準SQLと言う

### 1-2. SQLの基本的な記述ルール
- SQL文の最後に`;`(セミコロン)をつける
- キーワードに大文字/小文字の区別はない
- 定数は`''`(シングルクォーテーション)で囲む

## 2. SQLの分類

### 2-1. DDL(Data Definition Language)

- CREATE
- DROP
- ALTER

### 2-2. DML(Data Manipulation Language)

- SELECT
- INSERT
- UPDATE
- DELETE

### 2-3. DCL(Data Control Language)

- COMMIT
- ROLLBACK
- GRANT
- REVOKE

## 3. SQL基礎

### 3-1. CREATE TABLE

テーブルの作成

```sql
CREATE TABLE Staff
(id    CHAR(4)    NOT NULL,
name   TEXT       NOT NULL,
age    INTEGER    ,
PRIMARY KEY (id));
```

- 主な制約
  - PRIMARY KEY
  - UNIQUE
  - REFERENCES 表名(列名)
  - CHECK(条件)
  - NOT NULL
  - DEFAULT 値

### 3-2. INSERT

行の挿入

```sql
INSERT INTO Staff (id, name, age) VALUES ('0001', '山田太郎', 26);
INSERT INTO Staff VALUES ('0004', '渡辺さつき', 28);
```

### 3-3. SELECT

列の取得

```sql
SELECT name FROM Staff;
SELECT * FROM Staff;
```

### 3-4. UPDATE

フィールド(セル)の更新

```sql
UPDATE Staff SET name = '桜井さつき' WHERE id='0004';
UPDATE Staff SET age = 46;
```

### 3-5. DELETE

行の削除

```sql
DELETE FROM Staff WHERE id='0002';
DELETE FROM Staff;
```

### 3-6. DROP TABLE

テーブルの削除

```sql
DROP TABLE staff;
```

### 3-7. トランザクション

- トランザクションとは複数の更新処理の集まり
- データの整合性を保ちながらデータの更新処理を行うことができる
  - トランザクションを利用することで、一部の処理が正しく実行されなかったときには、すべての更新処理をキャンセルし、作業全体を取り消すことが可能になる。

#### 3-7-1. 構文
- トランザクション開始の宣言
  - `BEGIN;`
- 処理の確定
  - `COMMIT;`
- 処理の取消
  - `ROLLBACK;`

※COMMIT文が実行されると、ROLLBACK文による処理の取消を行うことができなくなる。

---

【参考】

- [SQL入門 | PostgreSQLではじめるDB入門](https://db-study.com/archives/category/sql%E5%85%A5%E9%96%80)
- [トランザクション処理をさらっとマスターしよう：さらっと覚えるSQL＆T-SQL入門（12）（1/3 ページ） - ＠IT](https://atmarkit.itmedia.co.jp/ait/articles/0803/24/news138.html)
- [PostgreSQL: Documentation: 14: BEGIN](https://www.postgresql.org/docs/14/sql-begin.html)
