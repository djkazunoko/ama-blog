---
date: '2022-06-25T22:55:04+09:00'
draft: false
title: '【VSCode】HTMLタグの自動閉じタグ補完・自動タグ名変更を拡張機能なしで設定する方法'
tags: ["未分類"]
slug: '37'
---

## 1. はじめに

Visual Studio Codeの拡張機能である[Auto Complete Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-complete-tag)が提供する機能は、Visual Studio Codeの基本設定だけで代替可能です。

この記事では、拡張機能をインストールすることなく、HTMLタグの自動閉じタグ補完機能と自動タグ名変更機能を有効にする方法を紹介します。



## 2. Auto Complete Tag
Visual Studio Codeの拡張機能に[Auto Complete Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-complete-tag)というものがあります。

Auto Complete Tagとは[Auto Close Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag)と[Auto Rename Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag)の機能を合わせたもので、この拡張機能を有効にするとHTMLタグの自動閉じタグ補完(Auto Close Tag)と自動タグ名変更(Auto Rename Tag)ができるようになります。

### 2-1. Auto Close Tag

- 開始タグの閉じ括弧を入力すると、終了タグが自動的に挿入されます。

![Auto Close Tag](images/1.gif)

### 2-2. Auto Rename Tag

- 一方のHTMLタグの名前を変更すると、対になるHTMLタグの名前も自動的に変更されます。

![Auto Rename Tag](images/2.gif)

## 3. 設定方法

### 3-1. 自動閉じタグ補完(Auto Close Tag)

1. `command + ,`で設定を開く
1. `Auto Closing Tags`で検索
1. `HTML: Auto Closing Tags`にチェックを入れて設定を有効にする

`settings.json`を直接編集する場合は以下を記入。
```
{
  "html.autoClosingTags": true
}
```

### 3-2. 自動タグ名変更(Auto Rename Tag)

1. `command + ,`で設定を開く
1. `Editor: Linked Editing`で検索
1. `Editor: Linked Editing`にチェックを入れて設定を有効にする

`settings.json`を直接編集する場合は以下を記入。
```
{
  "editor.linkedEditing": true
}
```

※`editor.renameOnType`は廃止され`editor.linkedEditing`に変更([November 2020 (version 1.52)](https://code.visualstudio.com/updates/v1_52#_html))

---

【参考】

- [VS Code - You don't need that extension](https://www.roboleary.net/vscode/2020/08/05/dont-need-extensions)
- [コードを書くのが楽になる！知っておくと便利なVS Codeの機能・設定のまとめ | コリス](https://coliss.com/articles/build-websites/operation/work/vs-code-dont-need-extensions.html)
- [User and workspace settings](https://code.visualstudio.com/docs/configure/settings)
- `settings.json`の開き方
  1. `shift + command + P`でコマンドパレットを開く
  1. `open settings`で検索
  1. `Preferences: Open User Settings (JSON)`を選択
