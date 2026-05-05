---
date: '2023-01-29T12:10:04+09:00'
draft: false
title: '【Node.js】CLI版メモアプリを作る'
tags: ["未分類"]
slug: '89'
---

この記事では、Node.jsでメモの追加・一覧・参照・削除ができるコマンドラインアプリを作成します。

データの保存先にはJSONファイルを使い、JavaScriptの`class`構文を使って作成します。

## 1. 実行環境
- macOS：13.0.1
- Node.js：18.12.1
- npm：8.19.2
- [enquirer](https://www.npmjs.com/package/enquirer)：2.3.6
- [minimist](https://www.npmjs.com/package/minimist)：1.2.7

## 2. アプリの要件
以下の機能を持つメモアプリを作成します。

### 2-1. メモの追加
標準入力に入ってきたテキストを新しいメモとして追加する。
```sh
$ echo 'メモの内容' | app.js
```

### 2-2. メモの一覧
それぞれのメモの最初の行のみを表示する。
```sh
$ app.js -l
メモ1の1行目
メモ2の1行目
メモ3の1行目
```

### 2-3. メモの参照
選んだメモの全文が表示される。
```sh
$ app.js -r
Choose a note you want to see:
  メモ1の1行目
  メモ2の1行目
> メモ3の1行目

メモ3の1行目
メモ3の2行目
メモ3の3行目
```

### 2-4. メモの削除
選んだメモが削除される。
```sh
$ app.js -d
Choose a note you want to delete:
  メモ1の1行目
  メモ2の1行目
> メモ3の1行目

Successfully deleted !!
```

## 3. ソースコード

`app.js`
```javascript
#!/usr/bin/env node

const MemoController = require("./memo-controller");
const argv = require("minimist")(process.argv.slice(2));

class App {
  constructor(argv) {
    this.argv = argv;
  }

  exec() {
    try {
      if (Object.keys(this.argv).length >= 3) {
        throw new Error("Only one option is available.");
      } else if (Object.keys(this.argv).some(this.#isNotOption)) {
        throw new Error("Only options -r, -l and -d are available.");
      } else if (this.argv.l) {
        MemoController.list();
      } else if (this.argv.r) {
        MemoController.select();
      } else if (this.argv.d) {
        MemoController.delete();
      } else {
        MemoController.add();
      }
    } catch (err) {
      console.log(err.message);
    }
  }

  #isNotOption(value) {
    return value != "l" && value != "r" && value != "d" && value != "_";
  }
}

const app = new App(argv);
app.exec();
```

`memo-controller.js`
```javascript
const readline = require("node:readline");
const { once } = require("node:events");
const { prompt } = require("enquirer");
const FileController = require("./file-controller");
const filePath = "./memos.json";

class MemoController {
  static async add() {
    try {
      process.stdin.setEncoding("utf8");
      const lines = [];
      const rl = readline.createInterface({
        input: process.stdin,
      });

      rl.on("line", (line) => {
        lines.push(line);
      });

      await once(rl, "close");

      const fileController = new FileController(filePath);
      const memos = await fileController.read();

      const ids = Object.keys(memos).map((x) => parseInt(x));
      const id = ids.length > 0 ? Math.max(...ids) + 1 : 1;
      memos[id] = lines;

      await fileController.write(memos);
    } catch (err) {
      console.log(err.message);
    }
  }

  static async list() {
    try {
      const fileController = new FileController(filePath);
      const memos = await fileController.read();
      if (!Object.keys(memos).length) {
        throw new Error("Please create at least one note.");
      }

      for (const id in memos) {
        console.log(memos[id][0]);
      }
    } catch (err) {
      console.log(err.message);
    }
  }

  static async select() {
    try {
      const fileController = new FileController(filePath);
      const memos = await fileController.read();
      if (!Object.keys(memos).length) {
        throw new Error("Please create at least one note.");
      }

      const question = {
        type: "select",
        name: "memoId",
        message: "Choose a note you want to see:",
        choices: [],
        result() {
          return this.focused.value;
        },
      };

      const answer = await this.#getAnswer(memos, question);
      const memo = memos[answer.memoId];
      for (const line of memo) {
        console.log(line);
      }
    } catch (err) {
      console.log(err.message);
    }
  }

  static async delete() {
    try {
      const fileController = new FileController(filePath);
      const memos = await fileController.read();
      if (!Object.keys(memos).length) {
        throw new Error("Please create at least one note.");
      }

      const question = {
        type: "select",
        name: "memoId",
        message: "Choose a note you want to delete:",
        choices: [],
        result() {
          return this.focused.value;
        },
      };

      const answer = await this.#getAnswer(memos, question);
      delete memos[answer.memoId];
      await fileController.write(memos);
      console.log("Successfully deleted !!");
    } catch (err) {
      console.log(err.message);
    }
  }

  static async #getAnswer(memos, question) {
    try {
      for (const id in memos) {
        const obj = { name: memos[id][0], message: memos[id][0], value: id };
        question.choices.push(obj);
      }
      return await prompt(question);
    } catch (err) {
      console.log(err.message);
    }
  }
}

module.exports = MemoController;
```

`file-controller.js`
```javascript
const fs = require("node:fs/promises");

class FileController {
  constructor(filePath) {
    this.filePath = filePath;
  }

  async read() {
    try {
      if (!(await this.#exists(this.filePath))) {
        await fs.writeFile(this.filePath, "{}");
      }
      const json = await fs.readFile(this.filePath, { encoding: "utf8" });
      return JSON.parse(json);
    } catch (err) {
      console.log(err.message);
    }
  }

  async write(obj) {
    try {
      const json = JSON.stringify(obj);
      await fs.writeFile(this.filePath, json);
    } catch (err) {
      console.log(err.message);
    }
  }

  async #exists(filePath) {
    try {
      await fs.access(filePath);
      return true;
    } catch {
      return false;
    }
  }
}

module.exports = FileController;
```

## 4. ポイント解説

### 4-1. fsモジュール(JSONファイルの読み書き)

- 参考：[fs Promises API](https://nodejs.org/api/fs.html#promises-api)

#### ファイルの読み込み
```javascript
// file-controller.js
async read() {
  try {
    if (!(await this.#exists(this.filePath))) {
      await fs.writeFile(this.filePath, "{}");
    }
    const json = await fs.readFile(this.filePath, { encoding: "utf8" });
    return JSON.parse(json);
  } catch (err) {
    console.log(err.message);
  }
}
```
- ファイルが存在しなければ、`await fs.writeFile(this.filePath, "{}")`で空のオブジェクトを持つファイルを作成
- `await fs.readFile(this.filePath, { encoding: "utf8" });`でJSON文字列を取得して、`JSON.parse(json);`でオブジェクトを返す

```javascript
// memo-controller.js
const memos = await fileController.read();
```
- `memos`にはオブジェクトが入る

#### ファイルの書き込み
```javascript
// file-controller.js
async write(obj) {
  try {
    const json = JSON.stringify(obj);
    await fs.writeFile(this.filePath, json);
  } catch (err) {
    console.log(err.message);
  }
}
```
- `JSON.stringify(obj);`でオブジェクトをJSON文字列に変える
- `await fs.writeFile(this.filePath, json);`でファイルを書き換える

```javascript
// memo-controller.js
await fileController.write(memos);
```
- `memos`オブジェクトを渡して実行

#### ファイルの存在確認
```javascript
// file-controller.js
async #exists(filePath) {
  try {
    await fs.access(filePath);
    return true;
  } catch {
    return false;
  }
}
```
- [fs Promises API](https://nodejs.org/docs/latest/api/fs.html#fs_promises_api)には[fs.existsSync()](https://nodejs.org/docs/latest/api/fs.html#fsexistssyncpath)を置き換えるメソッドがないため、[fsPromises.access()](https://nodejs.org/docs/latest/api/fs.html#fspromisesaccesspath-mode)を応用してファイルの存在確認メソッドを実装。(参考：[Node.js — Check If a Path or File Exists](https://futurestud.io/tutorials/node-js-check-if-a-file-exists))
- [fsPromises.access()](https://nodejs.org/docs/latest/api/fs.html#fspromisesaccesspath-mode)はファイルのアクセスに成功した場合Promiseは値なしでresolveされ、ファイルのアクセスに失敗した場合PromiseはErrorオブジェクトでrejectされる。


### 4-2. readlineモジュール

- 参考：[Readline Callback API](https://nodejs.org/api/readline.html#callback-api)

```javascript
// memo-controller.js
static async add() {
  try {
    process.stdin.setEncoding("utf8");
    const lines = [];
    const rl = readline.createInterface({
      input: process.stdin,
    });

    rl.on("line", (line) => {
      lines.push(line);
    });

    await once(rl, "close");

    const fileController = new FileController(filePath);
    const memos = await fileController.read();

    const ids = Object.keys(memos).map((x) => parseInt(x));
    const id = ids.length > 0 ? Math.max(...ids) + 1 : 1;
    memos[id] = lines;

    await fileController.write(memos);
  } catch (err) {
    console.log(err.message);
  }
}
```
- `rl`が改行(`\n`)を受け取り`line`イベントが発行されるたびに`lines.push(line);`で標準入力の各行を配列に追加する
- `await once(rl, "close");`→`close`イベントに一度だけ応答し、`rl`インスタンスが終了する
- `ids`にはメモのid(数値)を要素に持つ配列が入る
- `ids.length > 0`で配列が要素を持つか判定し、要素があれば`Math.max(...ids) + 1`で最大値に+1をした値を、空なら1を`id`に格納する

### 4-3. enquirer

- 参考：[enquirer](https://github.com/enquirer/enquirer)

```javascript
// memo-controller.js
static async select() {
  try {
    const fileController = new FileController(filePath);
    const memos = await fileController.read();
    if (!Object.keys(memos).length) {
      throw new Error("Please create at least one note.");
    }

    const question = {
      type: "select",
      name: "memoId",
      message: "Choose a note you want to see:",
      choices: [],
      result() {
        return this.focused.value;
      },
    };

    const answer = await this.#getAnswer(memos, question);
    const memo = memos[answer.memoId];
    for (const line of memo) {
      console.log(line);
    }
  } catch (err) {
    console.log(err.message);
  }
}

static async #getAnswer(memos, question) {
  try {
    for (const id in memos) {
      const obj = { name: memos[id][0], message: memos[id][0], value: id };
      question.choices.push(obj);
    }
    return await prompt(question);
  } catch (err) {
    console.log(err.message);
  }
}
```
- `question`オブジェクト([Prompt Options](https://github.com/enquirer/enquirer#prompt-options))
  - `type: "select"`→[Select Prompt](https://github.com/enquirer/enquirer#select-prompt)を指定
  - `name: "memoId"`→promtメソッドの戻り値であるanswerオブジェクトのプロパティのキーに`memoId`を指定(`answer.memoId`のように使う)
    - promtメソッドはここでいう`#getAnswer`メソッド内の`await prompt(question)`のこと
  - `message: "Choose a note you want to see:"`→ターミナルに表示されるメッセージ
  - `choices: []`→choiceオブジェクトを要素に持つ配列、[ArrayPrompt](https://github.com/enquirer/enquirer#arrayprompt)のプロパティの一つ
    - choiceオブジェクトのプロパティについて→[Choice properties](https://github.com/enquirer/enquirer#choice-properties)
  - `result()`→`return this.focused.value;`とすることで、`answer.memoId`の値にchoiceオブジェクトの`value`プロパティの値が入る(参考：[Select Prompt - Examples](https://github.com/enquirer/enquirer/blob/master/docs/prompts/select.md#examples))
    - `return this.focused.value;`がないとデフォルトで`answer.memoId`の値にはchoiceオブジェクトの`name`プロパティの値が入る
