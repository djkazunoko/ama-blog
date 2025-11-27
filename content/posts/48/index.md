---
date: '2022-08-03T22:57:02+09:00'
draft: false
title: '【ls -l】ファイルタイプとファイルモードの記号の意味'
tags: ["未分類"]
slug: '48'
---

この記事ではLinuxのlsコマンドの-lオプションで表示されるファイルタイプとファイルモードの記号の意味を解説します。

`ls -l`コマンドを実行すると、以下のようなファイルの詳細情報が表示されます。

```
$ ls -l
-rw-r--r--   1 uname  staff    0 11 28 12:31 default.txt
drwxr-xr-x   2 uname  staff   64 11 28 11:55 default_dir
```

各行の先頭に表示される`-rw-r--r--`や`drwxr-xr-x`のような10桁の記号は、そのファイルのファイルタイプとファイルモードを表します。

- 1桁目：ファイルタイプ
- 2桁目~10桁目：ファイルモード

以降では、ファイルタイプとファイルモードの記号の意味を解説していきます。

## 1. ファイルタイプ

`ls -l`で表示される10桁の記号のうち1桁目はファイルタイプを表します。

ファイルタイプを表す記号の種類は以下の7つです。

|記号  |ファイルタイプ|
|---|---|
|`b`|ブロックデバイス(Block special file)|
|`c`|キャラクタデバイス(Character special file)|
|`d`|ディレクトリ(Directory)|
|`l`|シンボリックリンク(Symbolic link)|
|`s`|ソケット(Socket link)|
|`p`|名前付きパイプ(FIFO)|
|`-`|通常ファイル(Regular file)|


## 2. ファイルモード

`ls -l`で表示される10桁の記号のうち2桁目~10桁目はファイルモードを表します。

このファイルモードは以下の3文字×3組で表されます。

* 3文字(権限なし：`-`)
  * `r`：読み込み
  * `w`：書き込み
  * `x`：実行
* 3組
  * 所有者(owner)
  * グループ(group)
  * その他のユーザ(other)

例えばファイルモードが`rwxr-xr-x`の場合、所有者は読み込み・書き込み・実行の全てを許可(`rwx`)、グループは読み込み・実行を許可(`r-x`)、その他のユーザも読み込み・実行を許可(`r-x`)という意味になります。

### 2-1. 特殊権限(スティッキービット、SGID、SUID)

ファイルモードを表す記号には前述の`r`,`w`,`x`,`-`以外に、特殊権限を表す記号があります。

特殊権限の種類と記号の対応は以下のようになります。

|特殊権限 |記号|位置|
|---|---|---|
|スティッキービット|`t`または`T`|3組目の3文字目(その他のユーザの実行権限)|
|SGID(Set Group ID)|`s`または`S`|2組目の3文字目(グループの実行権限)|
|SUID(Set User ID)|`s`または`S`|1組目の3文字目(所有者の実行権限)|

一つのファイルに同時に複数の特殊権限を付与することはできません。

特殊権限の記号の小文字と大文字の区別は以下のようになります。

- スティッキービット
  - スティッキービットが設定されており、且つその他のユーザの実行権限を持つ場合：`t`
  - スティッキービットが設定されており、且つその他のユーザの実行権限を持たない場合：`T`
- SGID(Set Group ID)
  - SGIDが設定されており、且つグループの実行権限を持つ場合：`s`
  - SGIDが設定されており、且つグループの実行権限を持たない場合：`S`
- SUID(Set User ID)
  - SUIDが設定されており、且つ所有者の実行権限を持つ場合：`s`
  - SUIDが設定されており、且つ所有者の実行権限を持たない場合：`S`

```
$ ls -l
-rwxrwsr--  1 uname  staff          0  7 22 11:31 setgid_executable
-rwxrwSr--  1 uname  staff          0  7 22 11:32 setgid_not_executable
-rwsr--r--  1 uname  staff          0  7 22 11:25 setuid_executable
-rwSr-xr--  1 uname  staff          0  7 22 11:29 setuid_not_executable
drwxr-xr-t  2 uname  staff         64  7 22 11:23 sticky_executable
drwxr-xr-T  2 uname  staff         64  7 22 11:20 sticky_not_executable
```

---

【参考】

- [「ls -l」コマンドの表示からファイルの属性を理解しよう：“応用力”をつけるためのLinux再入門（9） - ＠IT](https://atmarkit.itmedia.co.jp/ait/articles/1605/18/news015.html)
- [Linuxにおける「ファイルモード」と「パーミッション」とは？【初心者向け】 | LFI](https://linuxfan.info/filemode-and-permission)
- [lsコマンドで表示されるファイルのモード(drwxr-xr-x) 〜RubyのFile::Stat#modeとは〜](https://zenn.dev/universato/articles/20201202-z-mode)
- [ls Man Page - macOS - SS64.com](https://ss64.com/mac/ls.html)
- [File-system permissions - Wikipedia](https://en.wikipedia.org/wiki/File-system_permissions)
- [Changing File Permissions - Oracle](https://docs.oracle.com/cd/E19683-01/816-4883/6mb2joat4/index.html)
- [Using Special File Permissions (setuid, setgid and Sticky Bit) - Oracle](https://docs.oracle.com/cd/E19504-01/802-5750/6i9g464q0/index.html)
- [Setting and Searching for Special Permissions - Oracle](https://docs.oracle.com/cd/E19504-01/802-5750/6i9g464q3/index.html)
