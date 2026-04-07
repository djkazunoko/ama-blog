---
date: '2022-11-25T22:27:19+09:00'
draft: false
title: '【Git】任意の箇所にコミットを挿入する'
tags: ["未分類"]
slug: '77'
---

以下のようなコミット履歴がある
```
A -- B -- C
```

コミットBとコミットCの間に新しいコミットDを挿入したい
```
A -- B -- D -- C
```

## やり方

コミット履歴の確認
```
$ git log --oneline
a065513 (HEAD -> main) コミットC
1e4ebc6 コミットB
b4c8aca コミットA
```

コミットBの後ろに挿入したいので`rebase -i`で`b4c8aca`(コミットA)を指定
```
$ git rebase -i b4c8aca
```

テキストエディタに以下が表示される
```
pick 1e4ebc6 コミットB
pick a065513 コミットC
```

コミットBの`pick`を`edit`に変えてファイルを保存して閉じる
```
edit 1e4ebc6 コミットB
pick a065513 コミットC
```

ファイルの修正を行い、以下を実行(通常のrebaseでは`git commit --amend`だが今回は新たなコミットを挿入するため`git commit`)
```
$ git add .
$ git commit -m "コミットD"
$ git rebase --continue
```

コミットBとコミットCの間にコミットDを挿入できた
```
073d6f0 (HEAD -> main) コミットC
a453e0f コミットD
1e4ebc6 コミットB
b4c8aca コミットA
```

---

【参考】

- [git - How to inject a commit between some two arbitrary commits in the past? - Stack Overflow](https://stackoverflow.com/questions/32315156/how-to-inject-a-commit-between-some-two-arbitrary-commits-in-the-past)
