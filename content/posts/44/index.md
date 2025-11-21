---
date: '2022-07-25T21:02:39+09:00'
draft: false
title: '【Git】特定のコミットを修正する方法【rebase -i】'
tags: ["未分類"]
slug: '44'
---

## 1. はじめに

本稿では、`git rebase -i`を使った特定のコミットの修正方法を解説します。

- 直前のコミットだけではなく、2つ以上前のコミットを修正できる
- コミットメッセージの修正だけではなく、ファイルの編集内容の修正も可能

**チーム開発等で既にpushしているコミットに対しての使用には注意が必要です。**

## 2. 手順

以下の手順で進めていきます。

1. コミットログの確認
1. git rebase -i HEAD~n
1. 修正したいコミットの「pick」を「edit」に変更
1. ファイルの修正
1. git add
1. git commit --amend
1. git rebase --continue

### 2-1. コミットログの確認
以下のコマンドを実行して、修正したいコミットを確認します。

```zsh
git log --oneline
a00d4fa (HEAD -> main) file2.txtを修正
091fed5 file2.txtを作成
d4a404b file1.txtを修正 # これを修正したい
6f86174 file1.txtを作成
```

今回は以下のコミットを修正したいと思います。

```
d4a404b file1.txtを修正
```

### 2-2. git rebase -i HEAD~n
`git rebase -i HEAD~n` コマンドで、デフォルトのテキストエディタに直近のn個のコミットを表示できます。

今回修正したいコミットはHEADを含めて3つ目のコミットなので、以下のコマンドを実行します。

```
git rebase -i HEAD~3
```

すると、テキストエディタで以下のように表示されます。

```zsh
# コミット一覧のファイル
pick d4a404b file1.txtを修正
pick 091fed5 file2.txtを作成
pick a00d4fa file2.txtを修正

# Rebase 6f86174..a00d4fa onto 6f86174 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified); use -c <commit> to reword the commit message
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
```

### 2-3. 修正したいコミットの「pick」を「edit」に変更
`d4a404b`のコミットを`pick`から`edit`に変更します。

(同時に複数のコミットを修正することも可能)

```zsh
# コミット一覧のファイル
edit d4a404b file1.txtを修正
pick 091fed5 file2.txtを作成
pick a00d4fa file2.txtを修正
```

**※上から古い順に表示されているので注意**

その後、コミット一覧のファイルを保存して閉じます。

### 2-4. ファイルの修正
ファイルの修正を行います。

(ファイルの修正がない場合はそのまま次に進みます。)

### 2-5. git add
修正したファイルをgit addしてステージングエリアに追加します。

```
git add <修正したファイル名>
```

### 2-6. git commit --amend
以下のコマンドを実行して、コミットを修正します。

```zsh
# コミットメッセージを修正する場合
git commit --amend -m "修正後のコミットメッセージ"
```

コミットメッセージの修正が必要ない場合は以下のコマンドを実行します。

```zsh
# コミットメッセージを修正しない場合
git commit --amend --no-edit
```

### 2-7. git rebase --continue
最後に以下のコマンドを実行して、rebaseを完了させます。

```
git rebase --continue
```

- 複数のコミットを修正している場合
  - `git rebase --continue`で次のコミットに進み、手順4~6を行う
  - 上記を最後のコミットまで繰り返す

以上で終了です。

## 3. まとめ

修正前と修正後のコミットログを比べてみると、`git rebase -i HEAD~3`で変更対象に含まれた3つのコミットの履歴が変わっていることがわかります。

```zsh
# 修正前
git log --oneline
a00d4fa (HEAD -> main) file2.txtを修正
091fed5 file2.txtを作成
d4a404b file1.txtを修正
6f86174 file1.txtを作成
```

```zsh
# 修正後
git log --oneline
75a9b32 (HEAD -> main) file2.txtを修正
2e94ab7 file2.txtを作成
b0860b3 file1.txtを修正
6f86174 file1.txtを作成
```

このように履歴が変わってしまうため、**チーム開発等で既にpushしているコミットに対しての使用には注意が必要です。**

[【Git】プルリクエストを使った開発の流れ | あまブログ](https://ama-blog.com/33/)

---

【参考】

- [サル先生のGit入門_rebase -i でコミットを修正する](https://backlog.com/ja/git-tutorial/stepup/33/)
- [[git]特定のコミットの内容を修正する](https://dackdive.hateblo.jp/entry/2014/09/21/122200)
- [git rebase についてまとめてみた](https://qiita.com/KTakata/items/d33185fc0457c08654a5)
- [Git - git-rebase](https://git-scm.com/docs/git-rebase)
