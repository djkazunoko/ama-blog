---
date: '2022-06-06T12:25:33+09:00'
draft: false
title: '【Debian】Nginx公式リポジトリにある最新バージョンのNginxをインストールする'
tags: ["未分類"]
slug: '18'
---

この記事では、DebianにNginx公式リポジトリにある最新バージョンのNginxをインストールする手順を紹介します。

※Debianのデフォルトの設定で`apt install nginx`した場合、Nginx公式リポジトリからではなく、Debian公式リポジトリから[少し古いバージョンのNginx](https://packages.debian.org/bullseye/nginx)がインストールされます。

## 1. 実行環境

- Debian 11 bullseye

## 2. 手順

### 2-1. 必要なパッケージのインストール

Nginxのインストールに必要な以下のパッケージをインストールします。

- [curl](https://packages.debian.org/bullseye/curl)
- [gnupg2](https://packages.debian.org/bullseye/gnupg2)
- [ca-certificates](https://packages.debian.org/bullseye/ca-certificates)
- [lsb-release](https://packages.debian.org/bullseye/lsb-release)
- [debian-archive-keyring](https://packages.debian.org/bullseye/debian-archive-keyring)

以下のコマンドを実行してパッケージをインストール。
```
$ sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring
```

- パッケージの情報を表示：`apt show <パッケージ名>`
- インストール済みパッケージの確認：`apt list --installed <パッケージ名>`

### 2-2. 署名鍵のインポート

aptがパッケージの信頼性を確認できるようにするために、Nginxが公式に公開している署名鍵をインポートします。

以下のコマンドを実行して署名鍵をインポート。
```
$ curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

- コマンドの意味
  - `curl https://nginx.org/keys/nginx_signing.key`
    - curlコマンドで署名鍵ファイルをダウンロード
  - `gpg --dearmor`
    - 署名鍵ファイルをgpgコマンドで暗号化
  - `sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg`
    - teeコマンドで /usr/share/keyrings 配下に nginx-archive-keyring.gpg というファイル名で保存
  - `>/dev/null`
    - コマンド実行時の出力結果を /dev/null に渡している

以下のコマンドを実行して、ダウンロードした署名鍵が正しいことを確認します。
```
$ gpg --dry-run --quiet --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```
- コマンドの意味
  - [GPG Input and Output (Using the GNU Privacy Guard)](https://www.gnupg.org/documentation/manuals/gnupg/GPG-Input-and-Output.html)
  - [gpg(1): OpenPGP encryption/signing tool - Linux man page](https://linux.die.net/man/1/gpg)

出力結果は以下のようになります。
```
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```
出力されたフィンガープリントの値が`573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62`であることを確認します。

フィンガープリントの値が異なる場合は、署名鍵をインポートし直してください。

### 2-3. リポジトリの追加

aptの設定ファイルにNginxの公式リポジトリを追加します。

以下のコマンドを実行して `/etc/apt/sources.list.d` 配下に設定ファイルを作成します。
```
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" | sudo tee /etc/apt/sources.list.d/nginx.list
```

作成された `/etc/apt/sources.list.d/nginx.list` の内容は以下のようになります。
```
deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/mainline/debian bullseye nginx
```

上記の設定ではmainlineのリポジトリを追加しています。

- Nginxの`Mainline version`と`Stable version`のどちらを使うべきか
  - https://www.nginx.com/blog/nginx-1-6-1-7-released
    - `Mainline version`→特に理由がなければこっち
    - `Stable version`→新機能の追加によるバグなどを懸念している場合はこっち

### 2-4. Nginxのインストール

最後に、以下のコマンドを実行してNginxをインストールします。
```
$ sudo apt update
$ sudo apt install nginx
```

以上で終了です。

---

## 3. 【番外編】apt-keyコマンドを使う

番外編として、署名鍵のインポートに[apt-keyコマンド](https://manpages.debian.org/unstable/apt/apt-key.8.en.html)を使った場合のインストール手順を紹介します。

※apt-keyコマンドは[2022年ごろの廃止](https://salsa.debian.org/apt-team/apt/-/commit/ee284d5917d09649b68ff1632d44e892f290c52f)が予定されています。

### 3-1. 署名鍵のインポート

以下のコマンドを実行して、署名鍵をインポートします。
```
$ sudo wget https://nginx.org/keys/nginx_signing.key
$ sudo apt-key add nginx_signing.key
```

- apt-key addで追加した署名鍵の確認：`apt-key list`
- apt-key addで追加した署名鍵の削除：`apt-key del <keyid>`
  - keyidはフィンガープリントの下8桁
    - [debian - How to identify gpg key IDs so they may be deleted - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/420961/how-to-identify-gpg-key-ids-so-they-may-be-deleted)

### 3-2. リポジトリの追加

`/etc/apt/sources.list` を編集します。
```
$ sudo vi /etc/apt/sources.list
```

以下を追記。
```
deb https://nginx.org/packages/mainline/debian/ <CODENAME> nginx
deb-src https://nginx.org/packages/mainline/debian/ <CODENAME> nginx
```

コードネームの確認：`lsb_release -cs`

Debian 11 bullseyeの場合
```
deb https://nginx.org/packages/mainline/debian/ bullseye nginx
deb-src https://nginx.org/packages/mainline/debian/ bullseye nginx
```

### 3-3. Nginxのインストール

以下のコマンドを実行して、Nginxをインストールします。

```
$ sudo apt update
$ sudo apt install nginx
```

以上で終了です。

---

【参考】

- 公式インストールマニュアル
  - 【nginx.org】[nginx: Linux packages](https://nginx.org/en/linux_packages.html#Debian)←本稿では主にこちらを参考
  - 【nginx.com】https://www.nginx.com/resources/wiki/start/topics/tutorials/install
  - 【docs.nginx.com】[Installing NGINX Open Source | NGINX Documentation](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/)
- [nginx: download](https://nginx.org/en/download.html)
- [apt(8) — apt — Debian unstable — Debian Manpages](https://manpages.debian.org/unstable/apt/apt.8.ja.html)
- [第675回　apt-keyはなぜ廃止予定となったのか | gihyo.jp](https://gihyo.jp/admin/serial/01/ubuntu-recipe/0675)
- [What is GPG / PGP and how do I use it?](https://www.privex.io/articles/what-is-gpg)
- [Linux(Debian) に最新版 Nginx をインストールする方法メモ - Just do IT](https://k-koh.hatenablog.com/entry/2019/11/08/100041)
- [Ubuntu 20.04 LTSに最新版Nginxをインストールする | ジコログ](https://self-development.info/ubuntu-20-04-lts%E3%81%AB%E6%9C%80%E6%96%B0%E7%89%88nginx%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B/)
- [【Ubuntu】nginx公式リポジトリを使用してインストール | server-memo.net](https://www.server-memo.net/ubuntu/ubuntu_nginx_install.html)
