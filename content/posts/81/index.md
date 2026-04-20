---
date: '2022-11-29T22:22:31+09:00'
draft: false
title: '【Git】git stashでコミットしていない変更を一時的に退避させる'
tags: ["未分類"]
slug: '81'
---

## 1. 変更を退避する

- `git stash` = `git stash push`

```zsh
# 作業ディレクトリとインデックスの変更を退避
$ git stash push
```

```zsh
# 作業ディレクトリの変更を退避
$ git stash push -k
```

```zsh
# 作業ディレクトリとインデックスの変更 + 新規作成ファイルを退避
$ git stash push -u
```

```zsh
# 作業ディレクトリとインデックスの変更 + 新規作成ファイル + ignoredファイルを退避
$ git stash push -a
```

```zsh
# メッセージ付きでstashする
$ git stash push -m "message"
```

## 2. 退避した変更を確認する

```zsh
# stashを一覧表示する
$ git stash list
```

```zsh
# 特定のstashを表示する
$ git stash show # stash@{0}を指定したのと同じ
$ git stash show stash@{1}
```

```zsh
# 特定のstashの差分を表示する
$ git stash show -p stash@{1}
```
## 3. 退避した変更を再適用する

```zsh
# 特定のstashを再適用する(stashは削除されない)
$ git stash apply stash@{1}
```

```zsh
# 特定のstashを再適用して削除する(apply + drop)
$ git stash pop stash@{1}
```

- `apply`と`pop`は共にデフォルトで、stashを作業ディレクトリに再適用する
  - addしていた変更も、addされていない状態で戻る
  - インデックスにも再適用したい場合は、後述の`--index`オプションを使う
- `pop`によってコンフリクトが発生した場合、そのstashは削除されない

```zsh
# 特定のstashを作業ディレクトリとインデックスに再適用する
$ git stash apply --index stash@{1} 
$ git stash pop --index stash@{1}
```

## 4. 退避した変更を削除する

```zsh
# 特定のstashを削除する
$ git stash drop stash@{1}
```

```zsh
# 全てのstashを削除する
$ git stash clear
```

---

【参考】

- [Git - git-stash Documentation](https://git-scm.com/docs/git-stash)
- [Git stash - 変更の保存 | アトラシアン Git チュートリアル](https://www.atlassian.com/ja/git/tutorials/saving-changes/git-stash)
- [【git stash】コミットはせずに変更を退避したいとき #Git - Qiita](https://qiita.com/chihiro/items/f373873d5c2dfbd03250)
