---
date: '2022-11-11T22:24:57+09:00'
draft: false
title: '【Git】複数のコミットを一つにまとめる(rebase -i で squash)'
tags: ["未分類"]
slug: '72'
---

この記事では、`git rebase -i`を使って複数のコミットを一つにまとめる方法を紹介します。

## 1. 連続したコミットの場合

以下の`edit2`, `edit3`, `edit4`を一つにまとめて、`edit 2 & 3 & 4`にします。
```
$ git log --oneline
96a8b1e (HEAD -> main) edit4
30600dd edit3
452c39e edit2
ac56720 edit1
```

`ac56720`以降のコミットを表示させたいので、以下のコマンドを実行
```zsh
$ git rebase -i ac56720
# git rebase -i HEAD~3でも可
```

テキストエディタに以下が表示される(古いコミットが上)
```zsh
pick 452c39e edit2
pick 30600dd edit3
pick 96a8b1e edit4
```

`edit3`と`edit4`の`pick`を`s`(または`squash`)に変更して、ファイルを保存して閉じる。
```zsh
pick 452c39e edit2
s 30600dd edit3
s 96a8b1e edit4
```
- `f`(`fixup`)にするとコミットメッセージの編集画面は表示されず、まとめられたコミットのコミットメッセージが`edit2`になる(`squash`と`fixup`の違いはこれだけ)

以下のコミットメッセージの編集画面が表示される
```zsh
# This is a combination of 3 commits.
# This is the 1st commit message:

edit 2

# This is the commit message #2:

edit 3

# This is the commit message #3:

edit 4
```

コミットメッセージを`edit 2 & 3 & 4`に変更してファイルを保存して閉じる。(他のコミットメッセージは消しても消さなくてもいい)
```zsh
# This is a combination of 3 commits.
# This is the 1st commit message:

edit 2 & 3 & 4

# This is the commit message #2:

edit 3

# This is the commit message #3:

edit 4
```

コミットを一つにまとめることができた
```
$ git log --oneline
2897eb6 (HEAD -> main) edit2 & 3 & 4
ac56720 edit1
```


## 2. 離れたコミットの場合

以下の`add 1 to a.txt`と`add 2 to a.txt`を一つにまとめて、`add 1 & 2 to a.txt`にします。
```
$ git log --oneline
bec04cd (HEAD -> main) add 2 to a.txt
58c00dd add 222 to b.txt
2407669 add 1 to a.txt
c90dfb0 add 111 to b.txt
```

`c90dfb0`以降のコミットを表示させたいので、以下のコマンドを実行
```zsh
$ git rebase -i c90dfb0
# git rebase -i HEAD~3でも可
```

テキストエディタに以下が表示される(古いコミットが上)
```
pick 2407669 add 1 to a.txt
pick 58c00dd add 222 to b.txt
pick bec04cd add 2 to a.txt
```

`add 2 to a.txt`の`pick`を`s`(または`squash`)に変更
```
pick 2407669 add 1 to a.txt
pick 58c00dd add 222 to b.txt
s bec04cd add 2 to a.txt
```

`s`にした`add 2 to a.txt`を合体させたいコミット(`add 1 to a.txt`)の下に移動して、ファイルを保存して閉じる。
```
pick 2407669 add 1 to a.txt
s bec04cd add 2 to a.txt
pick 58c00dd add 222 to b.txt
```

以下のコミットメッセージの編集画面が表示される
```zsh
# This is a combination of 2 commits.
# This is the 1st commit message:

add 1 to a.txt

# This is the commit message #2:

add 2 to a.txt
```

コミットメッセージを`add 1 & 2 to a.txt`に変更してファイルを保存して閉じる。
```zsh
# This is a combination of 2 commits.
# This is the 1st commit message:

add 1 & 2 to a.txt

# This is the commit message #2:

add 2 to a.txt
```

コミットを一つにまとめることができた
```
$ git log --oneline
fa57a41 (HEAD -> main) add 222 to b.txt
4eb72b8 add 1 & 2 to a.txt
c90dfb0 add 111 to b.txt
```

---

【参考】

- [Git - git-rebase Documentation](https://git-scm.com/docs/git-rebase#_interactive_mode)
- [複数のコミットを1つにまとめる方法 | DevelopersIO](https://dev.classmethod.jp/articles/git-commit-matomeru/)
- [Gitの複数コミットをrebaseとsquashでまとめる方法 | iwb.jp](https://iwb.jp/git-commit-rebase-squash/)
- [git コミットログを綺麗にしたい。fixupとsquash - かもメモ](https://chaika.hatenablog.com/entry/2019/02/25/170000)
- [【Git】離れた複数のcommit（コミット）をまとめる方法｜Webエンジニア研究室](https://www.engilaboo.com/summarize-multiple-commits/)
- [git - How do I squash my last N commits together? - Stack Overflow](https://stackoverflow.com/questions/5189560/how-do-i-squash-my-last-n-commits-together)
