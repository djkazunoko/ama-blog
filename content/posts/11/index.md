---
date: '2022-05-19T15:42:32+09:00'
draft: false
title: '【Linux】SSH公開鍵認証方式の設定手順'
tags: ["未分類"]
slug: '11'
---

## 1. はじめに

本稿では、クライアントからSSHでLinuxサーバーに接続する手順を解説します。

リモートのサーバーにローカルPCのSSHクライアントで公開鍵認証方式を使ってログインできるようにします。

## 2. 実行環境

- 【SSHクライアント】macOS Monterey：12.3.1
- 【SSHサーバー】Debian GNU/Linux 11 (bullseye)

## 3. 手順

以下の手順で進めていきます。

1. SSHのインストール
1. パスワード認証方式での設定
1. 公開鍵認証方式での設定
1. セキュリティ設定
1. 追加設定

### 3-1. SSHのインストール

サーバー側で以下のコマンドを実行して、OpenSSHをインストールします。

```
# apt install -y ssh
```

### 3-2. パスワード認証方式での設定

公開鍵認証方式の設定を行うために、一時的にパスワード認証方式でログインできるように設定します。

サーバー側で以下のコマンドを実行して、設定ファイル(`/etc/ssh/sshd_config`)のバックアップを取ります。

```
# cp /etc/ssh/sshd_config  /etc/ssh/sshd_config.bk
```

`/etc/ssh/sshd_config`のパスワード認証の項目を以下に変更します。

```
PasswordAuthentication yes
```

**※その他の項目は後で設定します。**

SSHサーバーを再起動して設定を反映させます。

```
# systemctl restart ssh
```

クライアント側で以下のコマンドを実行して、パスワード認証方式でSSHサーバーにログインできることを確認します。

```
$ ssh <ユーザ名>@<SSHサーバーのIPアドレス>
```

### 3-3. 公開鍵認証方式での設定

次に公開鍵認証方式の設定を行います。

#### 3-3-1. キーペアの生成

クライアント側で以下のコマンドを実行して、公開鍵と秘密鍵のキーペアを作成します。

```
$ ssh-keygen -t ed25519
```

公開鍵：`~/.ssh/id_ed25519.pub`

秘密鍵：`~/.ssh/id_ed25519`

**※秘密鍵は絶対に流出させてはいけません。**

#### 3-3-2. 公開鍵の登録

次に、サーバー側に公開鍵を登録します。

`ssh-copy-id`を使って公開鍵の登録を行います。

クライアント側で以下のコマンドを実行し、`ssh-copy-id`をインストールします。

```
$ brew install ssh-copy-id
```

クライアント側で以下のコマンドを実行し、サーバー側に公開鍵を登録します。

```
$ ssh-copy-id -i ~/.ssh/id_ed25519.pub <ユーザ名>@<SSHサーバーのIPアドレス>
```

ssh-copy-idコマンドがやってくれたことを手動で行う場合は、以下のようになります。

```zsh
# サーバー側で
$ mkdir ~/.ssh
$ chmod 700 ~/.ssh

# クライアント側で
$ scp ~/.ssh/id_ed25519.pub <ユーザ名>@<SSHサーバーのIPアドレス>:~/.ssh/authorized_keys

# サーバー側で
$ chmod 600 ~/.ssh/authorized_keys
```

#### 3-3-3. 公開鍵認証方式でログイン

クライアント側で以下のコマンドを実行して、公開鍵認証方式でSSHサーバーにログインできることを確認します。

```
$ ssh <ユーザ名>@<SSHサーバーのIPアドレス>
```

### 3-4. セキュリティ設定

サーバー側で追加のセキュリティ設定を行います。

`/etc/ssh/sshd_config`を以下のように編集します。

#### 3-4-1. パスワード認証を無効化
`/etc/ssh/sshd_config`
```
PasswordAuthentication no

ChallengeResponseAuthentication no

PermitEmptyPasswords no
```

#### 3-4-2. rootでのログインを禁止にする

`/etc/ssh/sshd_config`
```
PermitRootLogin no
```

#### 3-4-3. デフォルトの22番ポートでログインできないようにする
`/etc/ssh/sshd_config`
```
Port 10022
```

※ウェルノウンポート以外の任意の番号

最後に、SSHサーバーを再起動して設定を反映させます。

```
# systemctl restart ssh
```
以上で終了です。

### 3-5. 追加設定

クライアント側の`~/.ssh/config`を設定して、SSH接続時のコマンドを省略できます。

|項目名|説明|
|---|---|
|Host|任意の接続名（接続するsshのエイリアス名）|
|Hostname|接続するサーバーのIPアドレスまたはドメイン|
|User|接続するユーザー名|
|Port|接続するポート番号|
|IdentityFile|秘密鍵のファイルパス|

例えば`~/.ssh/config`を以下のように設定すると

```
Host hoge
	Hostname 192.168.88.211
	User foo
	Port 10022
	IdentityFile ~/.ssh/id_ed25519
```

SSH接続時のコマンドを以下のように省略できます。

```zsh
# これが
$ ssh foo@192.168.88.211 -p 10022
# こうなる
$ ssh hoge
```

---

【公式ドキュメント】

- [OpenSSH](https://www.openssh.com/)
- [sshd_config](https://www.ssh.com/academy/ssh/sshd_config)
- [ssh_config(~/.ssh/config)](https://www.ssh.com/academy/ssh/config)
    - [man ssh_config](https://man.openbsd.org/OpenBSD-current/man5/ssh_config.5)
- [ssh-keygen](https://www.ssh.com/academy/ssh/keygen)
- [ssh-copy-id](https://www.ssh.com/academy/ssh/copy-id)

---

【参考】

- [SSH(セキュアシェル)とは](https://e-words.jp/w/SSH.html)
- [Debian GNU/Linux 4.0(etch)にOpenSSHをインストール](http://www.bnote.net/kuro_box/ssh_inst.html)
- [Debian 11 (bullseye) - SSH サーバ構築！](https://www.mk-mode.com/blog/2021/09/15/debian-11-ssh-installation/)
- [ssh接続を鍵認証で行う](http://www.tooyama.org/ssh-key.html)
- [SSHのポート番号を変更 - SSHサーバーの設定](https://webkaru.net/linux/change-ssh-port/)
- [Mac OS X から Linuxサーバ へ、RSA 鍵を用いて SSH 接続する](https://joker.hatenablog.com/entry/2013/06/05/192336)
- [新しい SSH キーを生成して ssh-agent に追加する](https://docs.github.com/ja/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [2017年版 SSH公開鍵認証で使用する秘密鍵ペアの作り方](https://qiita.com/wnoguchi/items/a72a042bb8159c35d056)
- [セキュアなSSHサーバの設定](https://qiita.com/comefigo/items/092137ac40f319cb14fa)
- [ssh公開鍵認証設定まとめ](https://qiita.com/ir-yk/items/af8550fea92b5c5f7fca)
- [SSH公開鍵認証で接続するまで](https://qiita.com/kazokmr/items/754169cfa996b24fcbf5)
- [【ssh接続】ホスト名を設定してIPアドレスを省略する設定方法（入力補完）](https://tofusystem.work/programming-hack/easy-ssh-login-setting/)
