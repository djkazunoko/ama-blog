---
date: '2022-11-25T09:39:29+09:00'
draft: false
title: '【Git】git commit --amendで直前のコミットを修正する'
tags: ["未分類"]
slug: '75'
---

この記事では、`git commit --amend`の使い方を紹介します。

- `git commit --amend`：直前のコミットの修正
  - 現在のステージングエリアの状態を元に、直前のコミットを作り直す
  - 修正されたコミットは実際は新しいコミットのため、リモートにpush済みのコミットへの使用には注意が必要


## 1. コミットメッセージの修正

ステージングエリアの変更がない状態で`git commit --amend`を実行すると、直前のコミットのコミットメッセージだけを修正できます(コミットの内容は変更されない)。

```sh
$ git add hello.rb
$ git commit -m "コミットメッセージ"
$ git commit --amend -m "修正後のコミットメッセージ"
```
- `git commit --amend`する前(上記2行目と3行目の間)に作業ディレクトリ(ステージングエリアに追加していない)のファイルの修正があっても問題ない

## 2. コミットの修正

### 2-1. `git add`し忘れた場合
```sh
# hello.rbとmain.rbを修正
$ git add hello.rb
$ git commit -m "hello.rbとmain.rbを修正"
# main.rbをaddし忘れたことに気付く
$ git add main.rb
$ git commit --amend --no-edit
```
- `--no-edit`でコミットメッセージを変更せずにコミットを修正

### 2-2. ファイルの修正
```sh
# hello.txtを修正
$ echo "hello" > hello.txt
$ git add hello.txt
$ git commit -m "helloに修正"
# hello worldに修正すべきだったことに気付く
$ echo "hello world" > hello.txt
$ git add hello.txt
$ git commit --amend -m "hello worldに修正"
```
- 不要なコミットを追加しないで済む

---

【参考】

- [Git - git-commit Documentation](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---amend)
- [Git amend | Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/rewriting-history)
- [コミットの修正には git commit --amend が便利 - RAKUS Developers Blog | ラクス エンジニアブログ](https://tech-blog.rakus.co.jp/entry/20191113/git)
