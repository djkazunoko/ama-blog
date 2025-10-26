---
date: '2022-06-05T14:16:21+09:00'
draft: false
title: '【Debian】Nginxで自分で作ったhtmlファイルを表示させる方法'
tags: ["未分類"]
slug: '16'
---

この記事では、Nginx上に自分で作ったhtmlファイルを配置し、ブラウザから表示させる方法を紹介します。

## 1. 環境
- Debian 11 bullseye
- nginx 1.21.6

## 2. 手順

### 2-1. Nginxのインストール

Nginxをインストール
```
$ sudo apt update
$ sudo apt install nginx
```

Nginxのインストールを確認
```
$ /usr/sbin/nginx -v
nginx version: nginx/1.21.6
```

### 2-2. Nginxの起動

Nignxを起動
```
$ sudo systemctl start nginx
```

Nginxの起動を確認
```
$ systemctl status nginx
● nginx.service - nginx - high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-06-05 06:38:04 JST; 1h 45min ago
       Docs: https://nginx.org/en/docs/
    Process: 8247 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
   Main PID: 8248 (nginx)
      Tasks: 2 (limit: 529)
     Memory: 1.7M
        CPU: 10ms
     CGroup: /system.slice/nginx.service
             ├─8248 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
             └─8250 nginx: worker process
```

### 2-3. 設定ファイルの確認

Nginxの設定ファイルを確認

`/etc/nginx/nginx.conf`

```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

`/etc/nginx/nginx.conf`内の以下の記述により、`/etc/nginx/conf.d`ディレクトリの拡張子が`.conf`であるファイルを読み込んでいることがわかる。

```
include /etc/nginx/conf.d/*.conf;
```

`/etc/nginx/conf.d`ディレクトリを確認
```
$ ls /etc/nginx/conf.d
default.conf
```

`/etc/ngin/nginx.conf`は`/etc/nginx/conf.d/default.conf`を読み込んでいることがわかる。


`/etc/nginx/conf.d/default.conf`
```
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

`/etc/nginx/conf.d/default.conf`内の以下の記述により、ドキュメントルートに`/usr/share/nginx/html`が指定されていることがわかる。
```
location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```

以上により、htmlファイルを`/usr/share/nginx/html`ディレクトリに配置すればいいことがわかった。

### 2-4. htmlファイルの作成

オリジナルのhtmlファイルを作成

```
$ sudo vi /usr/share/nginx/html/test.html
```
ブラウザから`ホスト名/test.html`にアクセスすると、自分で作ったhtmlファイルの内容が表示される。

---

【参考】

- [nginx連載3回目: nginxの設定、その1 - インフラエンジニアway - Powered by HEARTBEATS](https://heartbeats.jp/hbblog/2012/02/nginx03.html)
- [nginx連載4回目: nginxの設定、その2 - バーチャルサーバの設定 - インフラエンジニアway - Powered by HEARTBEATS](https://heartbeats.jp/hbblog/2012/04/nginx04.html)
- [nginx連載5回目: nginxの設定、その3 - locationディレクティブ - インフラエンジニアway - Powered by HEARTBEATS](https://heartbeats.jp/hbblog/2012/04/nginx05.html)
- [Ubuntu 20.04にNginxをインストールする方法 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04-ja)
- [Nginxで自分のHTMLを表示させる方法：htmlディレクトリはどこにある？ - Just do IT](https://k-koh.hatenablog.com/entry/2019/11/08/100158)
