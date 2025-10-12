---
date: '2022-04-08'
draft: false
title: '【MySQL】ERROR 1064 (42000)→構文ミスが見つからない→予約語が原因かも'
tags: ["未分類"]
slug: '3'
---

MySQLで構文は絶対正しいはずなのに、ERROR 1064 (42000)が出た時の話です。

- 解決法
  - 予約語をバッククォートで囲む

## 1. ERROR 1064 (42000)について
ERROR 1064 (42000)は構文エラーです。
以下のようなエラーメッセージが出力されます。
```
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near [入力したSQL文の一部] at line 1
```
解決法としては、基本的にはドキュメントと自分が書いたSQLを見比べて、構文ミスを見つけるだけなんですが...

[MySQL 8.0 リファレンスマニュアル(SQLステートメント)](https://dev.mysql.com/doc/refman/8.0/ja/sql-statements.html)

## 2. 構文ミスが見つからない→予約語が原因かも

### 2-1. エラー発生
以下のSQL文でERROR1064(42000)の構文エラーが発生しました。
```
mysql> CREATE TABLE order (id INT);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'order (id INT)' at line 1
```

### 2-2. 解決法
予約語である「order」をバッククォートで囲むことでエラーが解消されました。
```
mysql> CREATE TABLE `order` (id INT);
Query OK, 0 rows affected (0.01 sec)
```

「orderは予約語です」って言ってくれればいいのに。。

【参考】

- [MySQL 8.0 リファレンスマニュアル(キーワードと予約語)](https://dev.mysql.com/doc/refman/8.0/ja/keywords.html)
