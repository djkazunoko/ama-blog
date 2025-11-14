---
date: '2022-07-10T23:35:51+09:00'
draft: false
title: '【VSCode】オススメ拡張機能まとめ'
tags: ["未分類"]
slug: '40'
---

この記事では、Visual Studio Codeのオススメの拡張機能を紹介していきます。(随時更新中)

## 1. 一般

- [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker)
- [Japanese Language Pack for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-ja)
- [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)
- [Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager)

## 2. HTML

- [HTML CSS Support](https://marketplace.visualstudio.com/items?itemName=ecmel.vscode-html-css)
- [HTMLHint](https://marketplace.visualstudio.com/items?itemName=mkaufman.HTMLHint)
  - こちらの[HTMLHint](https://marketplace.visualstudio.com/items?itemName=HTMLHint.vscode-htmlhint)に移行推奨
- [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)
- [Auto Complete Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-complete-tag)
  - VSCodeの基本設定だけで代替可能([詳細](https://ama-tech.hatenablog.com/vscode-auto-closing-and-renaming-html-tags-without-extensions))
- [open in browser](https://marketplace.visualstudio.com/items?itemName=techer.open-in-browser)
  - Live Serverがあればほぼいらない、`<script type="module">`で書いたJSファイルを読み込んでるhtmlファイルは`file://` プロトコル経由で開いても動作しない

## 3. Ruby

- [Ruby LSP](https://marketplace.visualstudio.com/items?itemName=Shopify.ruby-lsp)
- [VSCode rdbg Ruby Debugger](https://marketplace.visualstudio.com/items?itemName=KoichiSasada.vscode-rdbg)

### 3-1. Rails

- [Slim](https://marketplace.visualstudio.com/items?itemName=sianglim.slim)
  - slim使う場合はあったほうがいい

## 4. JavaScript
- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [JavaScript (ES6) code snippets](https://marketplace.visualstudio.com/items?itemName=xabikos.JavaScriptSnippets)
- [IntelliCode](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode)
- [Formatting Toggle](https://marketplace.visualstudio.com/items?itemName=tombonnike.vscode-status-bar-format-toggle)

### 4-1. Vue.js

- [Volar](https://marketplace.visualstudio.com/items?itemName=vue.volar)
  - Vue3からはVolar、Vue2なら[Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur)

## 5. PostgreSQL
- [PostgreSQL](https://marketplace.visualstudio.com/items?itemName=ckolkman.vscode-postgres)

## 6. Git
- [Git History](https://marketplace.visualstudio.com/items?itemName=donjayamanne.githistory)
- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)

## 7. Docker
- [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)

---

【参考】

- [HTML Extensions in Visual Studio Code](https://code.visualstudio.com/docs/languages/html#_html-extensions)
- [JavaScript extensions for VS Code](https://code.visualstudio.com/docs/nodejs/extensions)
