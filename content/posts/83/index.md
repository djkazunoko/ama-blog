---
date: '2022-11-30T16:39:45+09:00'
draft: false
title: '【Node.js】カレンダーのプログラムを作る'
tags: ["未分類"]
slug: '83'
---

以下のカレンダーのプログラムをJavaScriptで、nodejsで実行するコマンドラインのプログラムとして作り直します。

[【Ruby 3.1】カレンダーのプログラムを作る | あまブログ](https://ama-blog.com/24/)

## 1. 環境

- macOS：13.0.1
- node：v18.12.1

## 2. ソースコード

```javascript
#!/usr/bin/env node

const argv = require("minimist")(process.argv.slice(2));
const today = new Date();
const month = argv.m || today.getMonth() + 1;
const year = argv.y || today.getFullYear();
const startOfMonth = new Date(year, month - 1);
const endOfMonth = new Date(year, month, 0);

console.log(`      ${month}月 ${year}`);
console.log("日 月 火 水 木 金 土");
process.stdout.write(" ".repeat(startOfMonth.getDay() * 3));
for (const d = startOfMonth; d <= endOfMonth; d.setDate(d.getDate() + 1)) {
  let day = String(d.getDate()).padStart(2, " ");
  const color_reverse = "\x1b[7m";
  const color_reset = "\x1b[0m";
  if (
    d.getFullYear() == today.getFullYear() &&
    d.getMonth() == today.getMonth() &&
    d.getDate() == today.getDate()
  ) {
    day = `${color_reverse}${day}${color_reset}`;
  }
  process.stdout.write(`${day} `);
  if (d.getDay() == 6) {
    process.stdout.write("\n");
  }
}
process.stdout.write("\n\n");
```

---

【参考】

- [Date - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Date)
- [minimist - npm](https://www.npmjs.com/package/minimist)
- [Loop through a date range with JavaScript - Stack Overflow](https://stackoverflow.com/questions/4345045/loop-through-a-date-range-with-javascript)
- [【Node.js】 コンソール(CLI)出力に色や装飾をつける方法](https://note.affi-sapo-sv.com/nodejs-console-color-output.php)
