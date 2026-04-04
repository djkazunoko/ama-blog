---
date: '2022-11-25T12:42:40+09:00'
draft: false
title: '【Git】git add -Aとgit add .とgit add -uの違い'
tags: ["未分類"]
slug: '76'
---

|  Command  |  New Files  |  Modified Files  |  Deleted Files  |  Scope
| ---- | ---- | ---- | ---- | ---- |
|  `git add -A`  |  ⚪︎  |  ⚪︎  |  ⚪︎  |  全てのディレクトリ
|  `git add .`  |  ⚪︎  |  ⚪︎  |  ⚪︎  |  カレントディレクトリ
|  `git add -u`  |  × |  ⚪︎  |  ⚪︎  |  全てのディレクトリ

## `git add -A`
- 変更された全てのファイル(新規作成・更新・削除されたファイル)が`add`される
- `git add -A dir1`→`dir1`以下の変更された全てのファイルが`add`される

## `git add .`
- カレントディレクトリ以下の、変更された全てのファイルが`add`される
- `git add dir1`→`dir1`以下の変更された全てのファイルが`add`される
- Git Version 1.xまでは削除されたファイルは`add`されなかったが、2.xから上記の仕様になった([https://github.com/git/git/blob/master/Documentation/RelNotes/2.0.0.txt:title])

## `git add -u`
- 更新・削除された追跡対象ファイルが`add`される(新規作成ファイルは`add`されない)
- `git add -u dir1`→`dir1`以下の変更・削除された追跡対象ファイルが`add`される
- `git commit -a` = `git add -u` + `git commit`

---

【参考】

- [Git - git-add Documentation](https://git-scm.com/docs/git-add)
- [git add - Difference between "git add -A" and "git add ." - Stack Overflow](https://stackoverflow.com/questions/572549/difference-between-git-add-a-and-git-add/26039014#26039014)
- [git add - What's the difference between "git add -u" and "git add -A"? - Stack Overflow](https://stackoverflow.com/questions/15011311/whats-the-difference-between-git-add-u-and-git-add-a)
- [git add -u と git add -A と git add . の違い | note.nkmk.me](https://note.nkmk.me/git-add-u-a-period/)
- [Git - git-commit Documentation](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt--a)
