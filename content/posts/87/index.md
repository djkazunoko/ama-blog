---
date: '2023-01-21T22:59:49+09:00'
draft: false
title: '【Node.js】フラッシュ暗算ゲームを作成する'
tags: ["未分類"]
slug: '87'
---

CLIでフラッシュ暗算ができるnpmパッケージを作成しました。

[djkazunoko/flash-anzan](https://github.com/djkazunoko/flash-anzan)

npmパッケージの公開方法は以下を参照ください。

[【Node.js】npmパッケージの公開方法 - あまブログ](https://ama-tech.hatenablog.com/how-to-publish-npm-packages)

この記事では、JavaScriptでフラッシュ暗算ゲームを作成する方法を解説します。

## 1. 実行環境
- macOS：13.1
- Node.js：18.12.1
- npm：8.19.2
- [enquirer](https://www.npmjs.com/package/enquirer)：2.3.6

## 2. フラッシュ暗算ゲームの仕様
1. 桁数、表示回数、表示間隔を選択する
1. カウントダウンの後に問題を出題する
1. 回答を入力する
1. 正解を表示する

## 3. ソースコード
`index.js`

```javascript
#! /usr/bin/env node

const { prompt } = require("enquirer");

async function main() {
  const terms = await flashNumbers();
  await checkAnswer(terms);
}

async function flashNumbers() {
  const options = await getOptions();
  const terms = [];

  await countDown();

  for (let i = 0; i < options.displayCount; i++) {
    const prevNum = terms.slice(-1)[0];
    let num = getNumber(options);
    while (num === prevNum) {
      num = getNumber(options);
    }
    terms.push(num);

    await displayNumber(num);

    await new Promise((resolve) =>
      setTimeout(resolve, options.displayInterval * 1000)
    );
  }

  process.stdout.clearLine(0);
  process.stdout.cursorTo(0);

  return terms;
}

async function getOptions() {
  return await prompt([
    {
      type: "input",
      name: "digits",
      message: "Number of Digits",
      validate: isPositiveInteger,
      initial: 1,
    },
    {
      type: "input",
      name: "displayCount",
      message: "Display Count",
      validate: isPositiveInteger,
      initial: 10,
    },
    {
      type: "input",
      name: "displayInterval",
      message: "Display Interval(seconds)",
      validate: isPositiveNumber,
      initial: 1,
    },
  ]);
}

function isPositiveInteger(input) {
  const num = Number(input);
  return Number.isInteger(num) && num > 0
    ? true
    : "Please input a Positive Integer";
}

function isPositiveNumber(input) {
  const num = Number(input);
  return num > 0 ? true : "Please input a Positive Number";
}

async function countDown() {
  const texts = [
    "\x1b[31mReady\x1b[0m",
    "\x1b[33mSet\x1b[0m",
    "\x1b[32mGo!\x1b[0m",
  ];
  for (const text of texts) {
    process.stdout.clearLine(0);
    process.stdout.cursorTo(0);
    process.stdout.write(text);
    await new Promise((resolve) => setTimeout(resolve, 1000));
  }
}

function getNumber(options) {
  return Math.floor(
    Math.random() * Math.pow(10, options.digits) * 0.9 +
      Math.pow(10, options.digits - 1)
  );
}

async function displayNumber(num) {
  process.stdout.clearLine(0);
  process.stdout.cursorTo(0);
  process.stdout.write(String(num));
}

async function checkAnswer(terms) {
  const correctAnswer = terms.reduce((sum, term) => sum + term);
  const answer = await inputAnswer();
  const result =
    Number(answer.answer) === correctAnswer
      ? "\x1b[32mCorrect!\x1b[0m"
      : "\x1b[31mWrong...\x1b[0m";
  const yourAnswer =
    Number(answer.answer) === correctAnswer
      ? `\x1b[32m${answer.answer}\x1b[0m`
      : `\x1b[31m${answer.answer}\x1b[0m`;

  console.log(result);
  console.log(`your answer: ${yourAnswer}`);
  console.log(`correct answer: \x1b[32m${correctAnswer}\x1b[0m`);
  console.log(`${terms.join(" + ")} = ${correctAnswer}`);
}

async function inputAnswer() {
  const question = {
    type: "input",
    name: "answer",
    message: "Please enter your answer",
    validate: isPositiveInteger,
    initial: 10,
  };

  return await prompt(question);
}

main();
```

## 4. ポイント解説

### 4-1. enquirer

参考：[enquirer](https://github.com/enquirer/enquirer)

`prompt`メソッドの引数に`question`オブジェクトを渡す

- `question`オブジェクトのプロパティ：[Prompt Options](https://github.com/enquirer/enquirer#prompt-options)
  - `type`
    - promtのタイプを指定(ここでは[Input Prompt](https://github.com/enquirer/enquirer#input-prompt)を使用)
  - `name`
    - `prompt`メソッドの戻り値のオブジェクト(`{digits: '1', displayCount: '10', displayInterval: '1'}`)のプロパティのキーを指定
  - `message`
    - ターミナルに表示されるメッセージを指定
  - `validate`
    - ユーザーの入力値のバリデーションを行うメソッドを指定
    - メソッドの戻り値は真偽値と文字列。文字列をエラーメッセージに使う。
  - `initial`
    - `prompt`メソッドの戻り値のオブジェクト(`{digits: '1', displayCount: '10', displayInterval: '1'}`)のプロパティの値の初期値を指定



### 4-2. 正数、正の整数のバリデーション

* 正数(Positive Number)：0より大きい数(少数を含む)
* 整数(Integer)：自然数, 0, 負数の総称(少数を含まない)

#### 4-2-1. 正数のバリデーション

```javascript
function isPositiveNumber(input) {
  const num = Number(input);
  return num > 0 ? true : "Please input a Positive Number";
}
```
- [Number(input)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Number)で文字列を数値に変換
- `num > 0`で正数かどうかを判定

#### 4-2-2. 正の整数のバリデーション

```javascript
 function isPositiveInteger(input) {
  const num = Number(input);
  return Number.isInteger(num) && num > 0
    ? true
    : "Please input a Positive Integer";
}
```
- [Number(input)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Number)で文字列を数値に変換
- [Number.isInteger()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Number/isInteger)で整数かどうかを、`num > 0`で正数かどうかを判定

### 4-3. ターミナル出力の上書き表示

```javascript
async function displayNumber(num) {
  process.stdout.clearLine(0);
  process.stdout.cursorTo(0);
  process.stdout.write(String(num));
}
```
- [writeStream.clearLine()](https://nodejs.org/api/tty.html#writestreamclearlinedir-callback)で行全体を消す
- [writeStream.cursorTo()](https://nodejs.org/api/tty.html#writestreamcursortox-y-callback)でカーソル位置をx方向の0に戻す
- `process.stdout.write()`で改行なしで出力する

### 4-4. タイマー処理

```javascript
// 1秒おきに表示
const timer = (ms) => new Promise((resolve) => setTimeout(resolve, ms));
async function main() {
  for (let i = 1; i <= 10; i++) {
    await timer(1000);
    console.log(i);
  }
}
main();
```

### 4-5. 乱数生成

```javascript
function getNumber(options) {
  return Math.floor(
    Math.random() * Math.pow(10, options.digits) * 0.9 +
      Math.pow(10, options.digits - 1)
  );
}
```

例：4桁のランダムな数値を生成(`options.digits`が4の場合)

|     |   Math.random()  |* 10000|* 0.9|+ 1000|Math.floor()|
| --- | --- | --- | --- | --- | --- |
|最小値|0|0|0|1000|1000|
|最大値|0.99999999|9999.9999|8999.9991|9999.9991|9999|

- 2列目：`* 10000`→`* Math.pow(10, options.digits)`
- 3列目：`* 0.9`により最大値の一桁目が1になるが最後に`Math.floor()`で切り捨てるので問題なし
- 4列目：`+ 1000`→`+ Math.pow(10, options.digits - 1)`


### 4-6. 連続して同じ数字が表示されないようにする

```javascript
async function flashNumbers() {
  const terms = [];
  // 省略
  for (let i = 0; i < options.displayCount; i++) {
    const prevNum = terms.slice(-1)[0];
    let num = getNumber(options);
    while (num === prevNum) {
      num = getNumber(options);
    }
    terms.push(num);
    // 省略
  }
  // 省略
}
```
- `terms.slice(-1)[0]`で配列の末尾の値を取得
  - 初回は配列が空なので`prevNum`は`undefined`
- `while (num === prevNum)`で`num`が`prevNum`と異なる値になるまで`getNumber()`を繰り返す

---

【参考】

- [「フラッシュナンバー」のソースと解説](http://www.shurey.com/js/craft/flash/index.html)
- [Check if String is a Positive Integer in JavaScript | bobbyhadz](https://bobbyhadz.com/blog/javascript-check-if-string-is-positive-integer)
- [javascript - Validate that a string is a positive integer - Stack Overflow](https://stackoverflow.com/questions/10834796/validate-that-a-string-is-a-positive-integer)
- [Overwrite console output in Node — Javascript — Amit Dhamu — Software Engineer](https://amitd.co/code/javascript/overwrite-console-output-in-node)
- [javascript - Node.js console.log - Is it possible to update a line rather than create a new line? - Stack Overflow](https://stackoverflow.com/questions/17309749/node-js-console-log-is-it-possible-to-update-a-line-rather-than-create-a-new-l)
- [node.js - async await with setInterval - Stack Overflow](https://stackoverflow.com/questions/52184291/async-await-with-setinterval)
- [任意の桁数の数字の文字列をランダムに生成するJavaScriptプログラム – MIII.me](https://miii.me/6262.html)
