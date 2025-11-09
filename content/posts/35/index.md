---
date: '2022-06-25T10:08:10+09:00'
draft: false
title: '【Git】派生元ブランチの変更'
tags: ["未分類"]
slug: '35'
---

この記事ではGitの派生元ブランチ(親ブランチ)の特定と変更方法を紹介します。

間違った親ブランチからブランチを作成してしまいコミットもしてしまった時などに有効です。

## 1. 派生元ブランチの特定方法

```
$ git show-branch | grep "*" | grep -v "$(git rev-parse --abbrev-ref HEAD)" | head -n 1 | awk -F'[]~^[]' '{print $2}'
```

- `grep "*"`：*がついてる行のみ表示
- `grep -v "$(git rev-parse --abbrev-ref HEAD)"`：カレントブランチを除外
- `head -n 1`：1行目を表示
- `awk -F'[]~^[]' '{print $2}’`：ブランチ名のみ表示

## 2. 派生元ブランチの変更方法

### 2-1. git rebase --onto

```
                            H---I---J topicB
                           /
                  E---F---G  topicA
                 /
    A---B---C---D  main
```
↓
```
$ git rebase --onto main topicA topicB
```
↓
```
                 H'--I'--J'  topicB
                /
                | E---F---G  topicA
                |/
    A---B---C---D  main
```

- `git rebase --onto <変更後の親ブランチ> <現在の親ブランチ> <移動するブランチ>`
- rebaseによりコミットが改変されているため、リモートへのpushには`git push -f`が必要



### 2-2. git cherry-pick

```
$ git checkout -b topicB-new master
$ git cherry-pick <コミットid>
```

- 新しいブランチを作り、欲しいコミットだけを`git cherry-pick`で移動させる
- 適用させたいコミット数が少ない場合に有効

---

【参考】

- 派生元ブランチの特定方法
  - [Gitで今のブランチの派生元ブランチを特定する #Git - Qiita](https://qiita.com/upinetree/items/0b74b08b64442f0a89b9)
  - [How to find the nearest parent of a Git branch - Stack Overflow](https://stackoverflow.com/questions/3161204/how-to-find-the-nearest-parent-of-a-git-branch/)
  - [Git - git log --first-parent Documentation](https://git-scm.com/docs/git-log#Documentation/git-log.txt---first-parent)
- 派生元ブランチの変更方法
  - [Gitでブランチの派生元を間違えたときの解決方法（rebase –onto、cherry-pick） – 株式会社グランフェアズ](https://www.granfairs.com/blog/entry-3219/)
  - [How change parent branch in git?](https://womanonrails.com/replace-parent-branch)
  - [Git - git-rebase Documentation](https://git-scm.com/docs/git-rebase)
  - [Git - git-cherry-pick Documentation](https://git-scm.com/docs/git-cherry-pick)
