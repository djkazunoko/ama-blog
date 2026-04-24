---
date: '2022-12-06T11:07:35+09:00'
draft: false
title: 'VSCodeでNode.jsをデバッグする'
tags: ["未分類"]
slug: '85'
---

VSCodeでNode.jsをデバッグする方法は主に以下の３つがあります。

1. [Auto Attach](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_auto-attach)
1. [JavaScriptデバッグターミナル](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_javascript-debug-terminal)
1. `launch.json`を使ったデバッグ

この記事では、Auto AttachとJavaScriptデバッグターミナルでのデバッグ方法を紹介します。

## 1. Auto Attach

- 1.コマンドパレット(`shift + ⌘ + P`)で`Toggle Auto Attach`を選択
- 2.`Always`を選択
  - `Always`：VSCodeの統合ターミナルで起動されたすべての Node.jsプロセスがデバッグされる
  - 以降はステータスバーの`Auto Attach`から設定変更可能
- 3.ブレイクポイントを設定して、VSCodeの統合ターミナルからNode.jsを実行
  - 統合ターミナルを開く ： `` control + ` ``


## 2. JavaScriptデバッグターミナル

JavaScriptデバッグターミナルで実行したNode.jsプロセスは自動的にデバッグされます。

- 1.ターミナルを開く(`` control + ` ``)
- 2.ターミナルスイッチャーのドロップダウンメニューから「JavaScriptデバッグターミナル」を選択
  - ターミナルスイッチャーのドロップダウンメニュー：ターミナルパネルの右側の`v`のようなマーク
  - またはコマンドパレットで`JavaScript Debug Terminal`を選択
- 3.ブレイクポイントを設定して、JavaScriptデバッグターミナルからNode.jsを実行



---

【参考】

- [Node.js debugging in VS Code](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)
