---
date: '2022-09-17T11:24:09+09:00'
draft: false
title: 'VSCodeでRailsをデバッグする【Rails + vscode-rdbg(debug.gem)】'
tags: ["未分類"]
slug: '61'
---

この記事では、Ruby 3.1で標準ライブラリとなった[debug.gem](https://rubygems.org/gems/debug)を使って、VSCode上でRailsアプリのデバッグを行う方法を紹介します。

## 1. 設定方法

Railsアプリのデバッグを行うために必要な設定を行います。

### 1-1. debug.gemのインストール

※ Rails 7以降ではデフォルトでdebug.gemが使われるため以下の設定は不要

Gemfileに以下を記述します。

```ruby
group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
end
```

`bundle install`を実行してdebug.gemをインストールします。

### 1-2. vscode-rdbgのインストール

VSCodeの拡張機能「VSCode rdbg Ruby Debugger」を追加します。

[VSCode rdbg Ruby Debugger - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=KoichiSasada.vscode-rdbg)

### 1-3. launch.jsonファイルの作成

VSCodeの[アクティビティバー](https://code.visualstudio.com/api/ux-guidelines/activity-bar)から「実行とデバッグ(Run and Debug)」を選択し、「launch.jsonファイルを作成します(create a launch.json file)」を選択します。

次に、「デバッガーの選択(Select debugger)」で「Ruby(rdbg)」を選択します。

するとプロジェクトのルートフォルダに`.vscode/launch.json`が作成されます。

作成された`.vscode/launch.json`を以下のように編集してください。

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "rdbg",
      "name": "Debug Rails",
      "request": "launch",
      "cwd": "${workspaceRoot}",
      "script": "bin/rails server",
      "args": [],
      "askParameters": false,
      "useBundler": true,
    },
  ]
}
```

以上で設定は終了です。

## 2. デバッグ方法

1. 任意の箇所にブレークポイントを設定
1. 「実行とデバッグ」の「Debug Rails」(`.vscode/launch.json`の`name`で設定した名前)を選択

※よくわからないエラーが出る場合

→VSCodeの画面左下の`rescue RuntimeError`のチェックを外してデバッガを再起動する

---

【参考】

- [ruby/debug](https://github.com/ruby/debug)
- [ruby/vscode-rdgb](https://github.com/ruby/vscode-rdbg)
- [Railsの練習帳：Visual Studio CodeでRailsアプリを開発](https://zenn.dev/igaiga/books/rails-practice-note/viewer/ruby_rails_vscode)
- [RubyKaigi 2021で発表された新しいdebug.gemを試してみた](https://zenn.dev/leaner_dev/articles/20210915-rubykaigi-2021-debug-gem)
- [Dev Container + Rails + vscode-rdbg (debug.gem)](https://zenn.dev/takeyuwebinc/articles/50793a2313824a)
- [Debugging in Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging)
- [第2回　Visual Studio Codeのデバッグ構成ファイルの基本：Node.js編](https://atmarkit.itmedia.co.jp/ait/articles/1708/04/news027.html)
