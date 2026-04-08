---
date: '2022-11-28T14:37:38+09:00'
draft: false
title: '【nvm】Node.jsのインストールとバージョン管理'
tags: ["未分類"]
slug: '78'
---

この記事では、Node.jsのバージョン管理ツールである[nvm](https://github.com/nvm-sh/nvm)のインストール方法・設定・使い方を紹介します。

## 1. nvmのインストール

[GitHub - nvm-sh/nvm：Install & Update Script](https://github.com/nvm-sh/nvm#install--update-script)

以下のコマンドを実行して、nvmのインストールスクリプトを実行
```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
```

上記により`~/.nvm`が作成され、`~/.zshrc`(bashの場合は`~/.bash_profile`)に以下が追記される
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```
- 上記3行目のbash_completionについて→[GitHub - nvm-sh/nvm：Bash Completion](https://github.com/nvm-sh/nvm#bash-completion)

ターミナル再起動後に以下を実行してnvmがインストールされたことを確認
```
$ nvm -v
```

## 2. `.nvmrc`ファイルの自動読み込み設定

[GitHub - nvm-sh/nvm： .nvmrc](https://github.com/nvm-sh/nvm#nvmrc)

この設定をすることで、`.nvmrc`ファイルがあるディレクトリに移動すると、自動で`nvm use`が実行され`.nvmrc`ファイルに記載されているバージョンのnodeに自動で切り替わる

`~/.zshrc`に以下を追記して設定は完了
```sh
# place this after nvm initialization!
autoload -U add-zsh-hook
load-nvmrc() {
  local nvmrc_path="$(nvm_find_nvmrc)"

  if [ -n "$nvmrc_path" ]; then
    local nvmrc_node_version=$(nvm version "$(cat "${nvmrc_path}")")

    if [ "$nvmrc_node_version" = "N/A" ]; then
      nvm install
    elif [ "$nvmrc_node_version" != "$(nvm version)" ]; then
      nvm use
    fi
  elif [ -n "$(PWD=$OLDPWD nvm_find_nvmrc)" ] && [ "$(nvm version)" != "$(nvm version default)" ]; then
    echo "Reverting to nvm default version"
    nvm use default
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```

`.nvmrc`ファイルの中身はバージョンを記載するだけ
```sh
$ echo "5.9" > .nvmrc
$ echo "lts/*" > .nvmrc # to default to the latest LTS version
$ echo "node" > .nvmrc # to default to the latest version
```

## 3. nvmの使い方

[GitHub - nvm-sh/nvm：Usage](https://github.com/nvm-sh/nvm#usage)

### 3-1. Node.jsのインストール・アンインストール

```sh
$ nvm install 14.7.0 # 指定したバージョンのインストール
$ nvm install node # 最新バージョンのインストール 
$ nvm install --lts # 最新LTSバージョンのインストール
```

```sh
$ nvm uninstall 14.7.0
```

### 3-2. Node.jsのバージョン切り替え

```sh
$ nvm use 16 # 16.x.xに切り替え
$ nvm use node # 最新バージョンに切り替え
$ nvm use --lts # 最新LTSバージョンに切り替え
```

### 3-3. デフォルトで利用する Node.js のバージョン設定

ターミナル起動時にデフォルトで有効になるバージョンを指定
```sh
$ nvm alias default 14.7.0 # 指定したバージョン
$ nvm alias default node # 最新バージョン
$ nvm alias default lts/* # 最新LTSバージョン
```

### 3-4. Node.jsのバージョン確認

現在使用中のバージョンを表示
```
$ nvm current
```

インストール済みのバージョンを表示
```
$ nvm ls
```

インストール可能なバージョンを表示
```
$ nvm ls-remote
```

---

【参考】

- [GitHub - nvm-sh/nvm](https://github.com/nvm-sh/nvm)
- [【nvm】nodeのバージョン管理をnodebrewからnvmに移行する&使い方 | offlo.in（おふろイン）](https://offlo.in/articles/node-version-nvm)
- [nvm で複数の Node.js バージョンを切り替えて使用する (Node Version Manager) - まくまく Node.js ノート](https://maku77.github.io/p/3x95seb/)
