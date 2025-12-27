---
date: '2022-09-14T23:09:12+09:00'
draft: false
title: '【Git】コミット済みファイルを管理対象から除外する方法'
tags: ["未分類"]
slug: '57'
---

Gitは一度ファイルを追跡すると、`.gitignore`に追加しても追跡は継続されます。

- `.gitignore`に追加する前にコミットしてしまった。
- リモートリポジトリにすでに追跡されているファイルの追跡をやめたい。

このような既にGitの管理対象になっているファイルの追跡を止めるには以下のコマンドを実行します。

**ファイルの場合**
```
git rm --cached <ファイル>
```

**ディレクトリの場合**
```
git rm -r --cached <ディレクトリ>
```

---

【参考】

- [Git - git-rm Documentation](https://git-scm.com/docs/git-rm)
- [How do I make Git forget about a file that was tracked, but is now in .gitignore? - Stack Overflow](https://stackoverflow.com/questions/1274057/how-do-i-make-git-forget-about-a-file-that-was-tracked-but-is-now-in-gitignore)
- [Git: Stop Tracking File After Adding to .gitignore](https://stackabuse.com/git-stop-tracking-file-after-adding-to-gitignore/)
- [How to Stop Git Tracking a File | Alpha Efficiency.™](https://alphaefficiency.com/git-stop-tracking-file)
- [gitの管理対象から特定のファイル、ディレクトリを削除する #Git - Qiita](https://qiita.com/ytkt/items/a2afd6be8e4f06c1ea25)
