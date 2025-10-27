---
date: '2022-06-06T12:28:13+09:00'
draft: false
title: '【Debian】NginxでVirtual Hostsを使って複数のドメインを設定する方法【Server Blocks】'
tags: ["未分類"]
slug: '19'
---

NginxのWebサーバではVirtual Hosts(Server Blocks)を使って1台のサーバで複数のドメインを運用することができます。

この記事では、Debian 11上のNginxにVirtual Hostsを設定する方法を紹介します。

なお、本稿の手順を進める前にDebian上にNginxがインストールされている必要があります。

DebianでのNginxのインストール方法は以下の記事を参照してください。

[【Debian】Nginx公式リポジトリにある最新バージョンのNginxをインストールする | あまブログ](https://ama-blog.com/18/)

## 1. 環境
- Debian 11 bullseye
- nginx 1.21.6

## 2. 手順

### 2-1. ドキュメントルートディレクトリの作成

ドキュメントルートは各ドメインのhtmlファイルが保存されるディレクトリで、任意の場所に作ることができます。

以下では `/var/www`ディレクトリの中にサーバでホストしたいドメインごとに html ディレクトリを作成します。

今回はサンプルとして`test1.com`と`test2.com`の2つのドメインを使用します。


以下のコマンドを実行して、ドキュメントルートディレクトリを作成します。
```
$ sudo mkdir -p /var/www/test1.com/html
$ sudo mkdir -p /var/www/test2.com/html
```

### 2-2. サンプルページの作成
次に各サイトのデフォルトページをドキュメントルートディレクトリの中に作成します。

以下のコマンドを実行して、1つ目のサンプルページを作成します。
```
$ sudo vi /var/www/test1.com/html/index.html
```

テスト用として、以下の内容で作成します。

`/var/www/test1.com/html/index.html`
```html
<html>
    <head>
        <title>test1</title>
    </head>
    <body>
        <h1>test1.com</h1>
    </body>
</html>
```

続けて、2つ目のサンプルページも作成します。
```
$ sudo cp /var/www/test1.com/html/index.html /var/www/test2.com/html/
$ sudo vi /var/www/test2.com/html/index.html
```

内容は以下のようになります。

`/var/www/test2.com/html/index.html`
```html
<html>
    <head>
        <title>test2</title>
    </head>
    <body>
        <h1>test2.com</h1>
    </body>
</html>
```

### 2-3. Server Blocksファイルの作成
次にNginxでVirtual Hostsを使うための設定ファイル(Server Blocksファイル)を作成します。

各ドメイン毎にServer Blocksファイルを作成します。


以下のコマンドを実行して、1つ目のServer Blocksファイルを作成します。

```
$ sudo vi /etc/nginx/sites-available/test1.com
```

内容は以下のようになります。

`/etc/nginx/sites-available/test1.com`
```zsh
server {
        listen 80;

        root /var/www/test1.com/html;
        index index.html;

        server_name test1.com www.test1.com;

        access_log /var/log/nginx/test1.com.access.log;
        error_log /var/log/nginx/test1.com.error.log;
}
```

上記の設定では、`test1.com`と`www.test1.com`に対するリクエストに応答し、ドキュメントルートの`/var/www/test1.com/html`にある`index.html`を表示します。

また、アクセスログを`/var/log/nginx/test1.com.access.log`に、エラーログを`/var/log/nginx/test1.com.error.log`に保存します。

続けて、2つ目のServer Blocksファイルを作成します。
```
$ sudo cp /etc/nginx/sites-available/test1.com /etc/nginx/sites-available/test2.com
$ sudo vi /etc/nginx/sites-available/test2.com
```

内容は以下のようになります。

`/etc/nginx/sites-available/test2.com`
```zsh
server {
        listen 80;

        root /var/www/test2.com/html;
        index index.html;

        server_name test2.com www.test2.com;

        access_log /var/log/nginx/test2.com.access.log;
        error_log /var/log/nginx/test2.com.error.log;
}
```

### 2-4. Server Blocksファイルの有効化

最後にServer Blocksファイルの有効化を行います。

Nginxでは設定ファイルは`/etc/nginx/sites-available`ディレクトリに保存され、`/etc/nginx/sites-enabled`ディレクトリに設定ファイルへのシンボリックリンクを作成します。

`/etc/nginx/sites-available`ディレクトリには現在有効になっているかどうかに関わらず、全てのVirtual Hostsの設定を保存します。

一方`/etc/nginx/sites-enabled`ディレクトリには、現在有効になっているドメインのみ、設定ファイルのシンボリックリンクを作成します。

これにより選択的にVirtual Hostsを有効化することができます。

以下のコマンドを実行して、ドメインごとのシボリックリンクを作成します。
```
$ sudo ln -s /etc/nginx/sites-available/test1.com /etc/nginx/sites-enabled/
$ sudo ln -s /etc/nginx/sites-available/test2.com /etc/nginx/sites-enabled/
```

続けて、`/etc/nginx/nginx.conf`を編集します。
```
$ sudo vi /etc/nginx/nginx.conf
```

`/etc/nginx/nginx.conf`に以下を追加して、Nginxが起動時に`/etc/nginx/sites-enabled`ディレクトリを読み込むように設定します。
```
include /etc/nginx/sites-enabled/*;
```

以下を実行して、設定ファイルのテストを行います。
```
$ sudo nginx -t
```

問題がなければ、nginxを再起動して設定の変更を反映させます。
```
$ sudo systemctl restart nginx
```

以上で設定は終了です。

ブラウザでドメインにアクセスして作成したサンプルページが表示されればOKです。

---

【参考】

- https://www.nginx.com/resources/wiki/start/topics/examples/server_blocks/
- [How To Set Up Nginx Server Blocks (Virtual Hosts) on Ubuntu 16.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04)
- [How To Set Up Nginx Server Blocks on Ubuntu 18.04 | Linuxize](https://linuxize.com/post/how-to-set-up-nginx-server-blocks-on-ubuntu-18-04/)
- [nginx連載3回目: nginxの設定、その1 - インフラエンジニアway - Powered by HEARTBEATS](https://heartbeats.jp/hbblog/2012/02/nginx03.html)
- [nginx連載4回目: nginxの設定、その2 - バーチャルサーバの設定 - インフラエンジニアway - Powered by HEARTBEATS](https://heartbeats.jp/hbblog/2012/04/nginx04.html)
- [nginx連載5回目: nginxの設定、その3 - locationディレクティブ - インフラエンジニアway - Powered by HEARTBEATS](https://heartbeats.jp/hbblog/2012/04/nginx05.html)
- [nginx　バーチャルホスト - nginx @ ウィキ - atwiki（アットウィキ）](https://w.atwiki.jp/nginx/pages/13.html)
- [Alphabetical index of directives](https://nginx.org/en/docs/dirindex.html)
- [Module ngx_http_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html)
- [linux - Difference in sites-available vs sites-enabled vs conf.d directories (Nginx)? - Server Fault](https://serverfault.com/questions/527630/difference-in-sites-available-vs-sites-enabled-vs-conf-d-directories-nginx)
