---
date: '2022-06-21T09:19:54+09:00'
draft: false
title: '【VSCode】ファイル末尾に改行を自動で挿入する設定'
tags: ["未分類"]
slug: '31'
---

この記事では、Visual Studio Codeのファイル保存時に、自動で末尾に改行が挿入されるように設定する方法を紹介します。

## 1. 環境
- VS Code バージョン 1.68.1

## 2. 手順

1. `⌘,(command + ,)`で設定を開く
2. `insertFinalNewline`で検索
3. `Files: Insert Final Newline`にチェックを入れて設定を有効にする

設定すると、`settings.json`に以下が追記される。
```
"files.insertFinalNewline": true
```

※ 上記を settings.json に直接追記しても設定可能

- settings.jsonの開き方
  - `⇧⌘P(shift + command + P)`でコマンドパレットを開く
  - `open settings`で検索
  - `Preferences: Open Settings (JSON)`を選択

---

【参考】

- [User and workspace settings](https://code.visualstudio.com/docs/configure/settings)
