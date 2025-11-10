---
date: '2022-06-25T10:24:31+09:00'
draft: false
title: '【Git】作業ツリー上の変更を取り消す方法'
tags: ["未分類"]
slug: '36'
---

この記事では`git add`も`git commit`もしていない、作業ツリー上のファイル(インデックスに登録されていないワークツリー上のファイル)の変更を取り消して元に戻す方法を紹介します。

## 1. 方法

```
$ git checkout -- <ファイル名>
```
- `git checkout -- .`で全てのファイルを指定

---

【参考】

- [git ローカルの変更を元に戻す方法 #Git - Qiita](https://qiita.com/macer_fkm/items/ea2337a9b504b982c295)
- [Git で変更を取り消して、元に戻す方法 (事例別まとめ) | WWWクリエイターズ](https://www-creators.com/archives/1290)
- https://docs.gitlab.com/ee/topics/git/numerous_undo_possibilities_in_git/
- [Git - git-checkout Documentation](https://git-scm.com/docs/git-checkout#_description)
