---
date: '2022-09-16T21:12:44+09:00'
draft: false
title: '【Git】グローバルなgitignoreの設定方法'
tags: ["未分類"]
slug: '60'
---

この記事では、グローバルなgitignoreの設定方法を紹介します。

グローバルなgitignoreファイル(`~/.config/git/ignore`)を作成し、全てのリポジトリでGitの追跡対象外とするファイルを定義します。

## 1. gitignoreとは
- 意図的に未追跡のファイルを指定して、それらをGitが追跡しないようにするためのもの
- すでに Git に追跡されているファイルは影響を受けない
  - 現在追跡しているファイルの追跡を止めるには、`git rm --cached` を使う

## 2. gitignoreの使い分け

- `.gitignore`：特定のリポジトリで全ての人が無視したいファイル
- `$GIT_DIR/info/exclude`：特定のリポジトリで自分だけが無視したいファイル
- `$XDG_CONFIG_HOME/git/ignore`：全てのリポジトリで自分だけが無視したいファイル

## 3. グローバルなgitignoreの設定

グローバルなgitignoreの設定には`$XDG_CONFIG_HOME/git/ignore`を使用します。

### 3-1. $XDG_CONFIG_HOME/git/ignoreとは
- `core.excludesfile`のデフォルト値
- `$XDG_CONFIG_HOME`が未設定の場合、代わりに`$HOME/.config/git/ignore`が使用される

つまり

`$ git config --global core.excludesfile ~/.gitignore_global`して`~/.gitignore_global`を作成する必要はなく、代わりに`~/.config/git/ignore`を作成すればok。

※`~/.config/git/ignore`を使用する場合、`core.excludesfile`の設定の削除が必要

### 3-2. 設定方法

`~/.config/git/ignore`を作成し、無視したいファイルを追加するだけです。

- gitignoreテンプレート：[github/gitignore](https://github.com/github/gitignore)
- gitignore作成：[gitignore.io](https://www.toptal.com/developers/gitignore)



---

【参考】

- [Git - gitignore Documentation](https://git-scm.com/docs/gitignore)
- [ファイルを無視する - GitHub ドキュメント](https://docs.github.com/ja/get-started/git-basics/ignoring-files)
- [gitignore を正しく理解したい - うさぎ小屋](https://kmyk.github.io/blog/blog/2020/11/08/man-gitignore/)
- [~/.gitignore_global を指定するのをやめ、デフォルトの置き場に置こう](https://zenn.dev/qnighy/articles/1a756f2857dc20)
- [「$HOME/git/ignore」と 「$GIT_DIR/info/exclude」と「.gitignore」の使い分け - セットプチフォッカ](https://ikmbear.hatenablog.com/entry/2021/01/02/104829)
- [Git - git-rm Documentation](https://git-scm.com/docs/git-rm)
- [XDG Base Directory - ArchWiki](https://wiki.archlinux.jp/index.php/XDG_Base_Directory)
- [最近目にする$HOME/.configというディレクトリ - 理系学生日記](https://kiririmode.hatenablog.jp/entry/20180404/1522843531)
- [SourceTree が Git のグローバルな無視リストを書き換えて困った話 - てっく煮ブログ](https://tech.nitoyon.com/ja/blog/2013/04/05/sourcetree/)
- [.gitignorがないのにファイルがコミットできないときの対処方法](https://alaki.co.jp/blog/?p=2582)
