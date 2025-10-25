---
date: '2022-06-01T23:38:19+09:00'
draft: false
title: '【Debian 11 bullseye】sudoコマンドをインストールする方法'
tags: ["未分類"]
slug: '15'
---

本稿では、さくらVPSにインストールしたDebian 11にsudoコマンドをインストールする方法を紹介します。

sudoは一般ユーザにスーパーユーザ(root)の権限を与えるコマンドです。

なお、さくらVPSにDebian 11をインストール方法は以下をご参照ください。

[さくらのVPSにDebian11 (bullseye)をインストールする方法 | あまブログ](https://ama-blog.com/14/)

## 1. sudoのインストール

sudoコマンドをインストールします。

```
# apt install -y sudo
```

## 2. /etc/sudoersの編集

sudoコマンドの設定ファイルである`/etc/sudoers`を編集して、権限の設定を行います。

`/etc/sudoers`は**直接テキストエディタで開いて編集してはいけません。**

書き方を誤るとsudoが動作せずに、どのユーザもsudoが使えなくなってしまうことがあるためです。

`/etc/sudoers`の編集はvisudoコマンドを使用します。

```
# visudo
```

visudoコマンドを実行したら、`/etc/sudoers`に以下を追記します。

```
foo ALL=(ALL) ALL
```

書式：`<ユーザ> <マシン名>=(<権限>) <コマンド>`

`(<権限>)`のところを`(<ALL:ALL>)`と書いた場合、左側がユーザ、右側がグループを表す。

ファイルを保存し、設定したユーザでsudoコマンドが実行できれば設定は完了です。

---
【参考】

- [【Debian 8 Jessie】sudoコマンドをインストールする #Linux - Qiita](https://qiita.com/osktak/items/f1746d64797a6a4dd6ae)
- [sudoers覚え書き #sudo - Qiita](https://qiita.com/progrhyme/items/6f936033b9d23efb1741)
- [sudoersを変更する、よく使う設定例 - それマグで！](https://takuya-1st.hatenablog.jp/entry/20090806/1249554458)
- [新しいLinuxの教科書 9章 03 スーパーユーザ](https://www.amazon.co.jp/dp/B072K1NH76/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1)

