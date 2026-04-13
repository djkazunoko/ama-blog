---
date: '2022-11-28T23:14:32+09:00'
draft: false
title: '【VSCode】ESLintとPrettierのインストールと設定'
tags: ["未分類"]
slug: '79'
---

この記事では[ESLint](https://eslint.org/)(静的検証ツール)と[Prettier](https://prettier.io/)(コードフォーマッター)のインストールと設定方法、またこれらをVSCodeで使用する方法を紹介します。

## 1. ESLint

[Documentation - ESLint - Pluggable JavaScript Linter](https://eslint.org/docs/latest/)

### 1-1. インストール

eslintをグローバルにインストール
```
$ npm install -g eslint
```

インストールを確認
```
$ eslint -v
```

### 1-2. 設定ファイル

[ESLint - Configuring ESLint](https://eslint.org/docs/latest/user-guide/configuring/)

eslintを使うには設定ファイルである`.eslintrc.*`が必要です。

- `.eslintrc.*`で使えるファイル形式
  - JavaScript：`.eslintrc.js`
  - JSON：`.eslintrc.json`
  - YAML：`.eslintrc.yml`

ここでは`.eslintrc.json`を使用します。

**設定ファイルの作成**

- 方法1：直接`.eslintrc.*`を作成
- 方法2：`eslint --init`コマンドで対話形式で作成
  - 事前に`package.json`の作成が必要
  - [eslint --init](https://eslint.org/docs/latest/user-guide/command-line-interface#--init)は`npm init @eslint/config`と同じ

**設定ファイルのプロパティ**

`.eslintrc.json`の例

```json
{
  "env": {
    "browser": true,
    "node": true,
    "es2022": true
  },
  "extends": ["eslint:recommended", "prettier"],
  "overrides": [],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "rules": {}
}
```
- `env`
  - [ESLint - Specifying Environments](https://eslint.org/docs/latest/user-guide/configuring/language-options#specifying-environments)
- `extends`
  - [ESLint - Using eslint:recommended](https://eslint.org/docs/latest/user-guide/configuring/configuration-files#using-eslintrecommended)
  - [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier#installation)
- `parserOptions`
  - [ESLint - Specifying Parser Options](https://eslint.org/docs/latest/user-guide/configuring/language-options#specifying-parser-options)

### 1-3. コマンドラインでESLintを実行

```
$ eslint yourfile.js
```
- 検証した結果、問題がなければ何も表示されない
- [ESLint - Command Line Interface](https://eslint.org/docs/latest/user-guide/command-line-interface)

### 1-4. VSCodeとの統合

VSCodeに[vscode-eslint](https://github.com/microsoft/vscode-eslint)をインストール

## 2. Prettier

[Prettier - Documentation](https://prettier.io/docs/en/index.html)

### 2-1. インストール

[Prettier - Install](https://prettier.io/docs/en/install.html)

Prettierをローカルにインストール

```
$ npm install --save-dev --save-exact prettier
```

### 2-2. Prettierの実行

フォーマットの実行
```
$ npx prettier --write .
```

フォーマットの確認のみ
```
$ npx prettier --check .
```

### 2-3. ESLintとの統合

[Prettier - Integrating with Linters](https://prettier.io/docs/en/integrating-with-linters.html)

[eslint-config-prettier](https://github.com/prettier/eslint-config-prettier)をインストール

```
$ npm install --save-dev eslint-config-prettier
```

`.eslintrc.json`に`"prettier"`を追記
```json
{
  "extends": [
    "some-other-config-you-use",
    "prettier"
  ]
}
```
- `"extends"`の配列の最後に追加する

### 2-4. VSCodeとの統合

[Prettier - Editor Integration](https://prettier.io/docs/en/editors.html#visual-studio-code)

VSCodeに[prettier-vscode](https://github.com/prettier/prettier-vscode)をインストールします。

次に、VSodeのデフォルトのフォーマッターをPrettierに設定し、ファイル保存時にフォーマットが実行されるように設定します。

**設定方法1. `settings.json`を直接編集する**

`settings.json`に以下を追記

```json
{
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
  },
}
```
- `settings.json`の開き方
  1. `shift + command + P`でコマンドパレットを開く
  1. `open settings`で検索
  1. `Preferences: Open User Settings (JSON)`を選択

**設定方法2. VSCodeの「設定」から設定する**

1. `cmd + ,`で「設定」を開く
1. `format on save`で検索
1. `Editor: Format On Save`にチェックを入れる
1. `default formatter`で検索
1. `Editor: Default Formatter`を`Prettier - Code formatter`に選択

この場合、`settings.json`は以下のようになる。

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true, 
}
```

---

【参考】

- [ESLint 最初の一歩 #JavaScript - Qiita](https://qiita.com/mysticatea/items/f523dab04a25f617c87d)
- [ESLint の設定ファイル (.eslintrc) の各プロパティの意味を理解する｜まくろぐ](https://maku.blog/p/j6iu7it/)
