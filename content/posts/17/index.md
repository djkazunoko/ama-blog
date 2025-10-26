---
date: '2022-06-05T14:21:28+09:00'
draft: false
title: 'telnetでGETとPOSTでHTTPリクエストを送る方法'
tags: ["未分類"]
slug: '17'
---

この記事では、クライアントからtelnetを使ってGETメソッドとPOSTメソッドでWebサーバにHTTPリクエストを送信する方法を紹介します。

HTTPは、Webサーバ(サーバ)⇄Webブラウザ(クライアント)間のWebページの送受信に使われるプロトコルです。

## 1. 環境
- macOS Monterey：12.4

## 2. 手順

### 2-1. telnetのインストール

```
$ brew install telnet
```

### 2-2. GETメソッド

以下のコマンドで、`example.com`の80番ポート(HTTP)に接続
```
$ telnet example.com 80
```

書式：`telnet <ホスト名> <ポート番号>`


続けて、以下のHTTPリクエストを実行
```zsh
GET / HTTP/1.1 # リクエスト行
Host:  example.com # ヘッダ
<改行>
```

- リクエスト行
  - メソッド：GETメソッド、リクエストURI：/、プロトコルバージョン：HTTP/1.1
- ヘッダ
  - フィールド名：Host、フィールド値：example.com
    - Host→HTTP/1.1 で唯一の必須ヘッダ。フィールド値にURLのホスト部分を指定。
- <改行>
  - 1行空けることでヘッダの終わりを伝える

### 2-3. POSTメソッド

`xxx.com/articles`に以下のようなフォームがある場合を想定

```
<form action="/articles" method="post">
  <div>
    <label for="article_title">Title</label>
    <input type="text" name="article[title]" id="article_title">
  </div>
  <div>
    <label for="article_body">Body</label>
    <textarea type="text" name="article[body]" id="article_body"></textarea>
  </div>
  <div>
    <input type="submit" name="commit" value="Submit" data-disable-with="Submit">
  </div>
</form>
```

以下のコマンドで、`xxx.com`の80番ポート(HTTP)に接続

```
$ telnet xxx.com 80
```

続けて、以下のHTTPリクエストを実行

```zsh
POST /articles HTTP/1.1 # リクエスト行
Host: xxx.com # ヘッダ
Content-Length: 35 # ヘッダ
<改行>
article[title]=Hi&article[body]=Mom # メッセージボディ
```

- リクエスト行
  - POSTメソッド、パスに/articles、HTTPバージョン1.1を指定
- ヘッダ
  - フィールド名→Host、フィールド値→example.com
  - フィールド名→Content-Length、フィールド値→35
    - Content-Length：コンテンツ（メッセージボディ）の長さをバイト単位で示す
- <改行>
  - 1行空けることでヘッダの終わりを伝える
- メッセージボディ
  - `<inputタグのname属性の値>=<テキスト>&<inputタグのname属性の値>=<テキスト>&...`

---

【参考】

- [とほほのHTTP入門 - とほほのWWW入門](https://www.tohoho-web.com/ex/http.html)
- [もいちどイチから！ HTTP基礎訓練中 連載インデックス - ＠IT -](https://atmarkit.itmedia.co.jp/fsecurity/index/index_httpbasic.html)
- [HTTPの基本1 #2 telnetでGETとPOSTを試してみる - いづいづブログ](https://izumii19.hatenablog.com/entry/2018/12/23/010234)
- [フォームデータの送信 - ウェブ開発の学習 | MDN](https://developer.mozilla.org/ja/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data)
- [拡張子とMIMEタイプ - とほほのWWW入門](https://www.tohoho-web.com/wwwxx015.htm)
- [イラスト図解式 この一冊で全部わかるWeb技術の基本](https://www.amazon.co.jp/dp/B06XNMMC9S/)
