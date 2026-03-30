---
date: '2022-11-23T20:52:06+09:00'
draft: false
title: 'Node.jsでESモジュールを使う'
tags: ["未分類"]
slug: '74'
---

この記事では、Node.jsでESモジュール(ECMAScriptモジュール, JavaScriptモジュール)を使う方法を紹介します。

- バージョン情報
  - node：v16.18.0

## 方法1：モジュールファイルの拡張子を`mjs`にする

`foo.mjs`
```javascript
export const foo = "foo";
```

`main.mjs`
```javascript
import { foo } from "./foo.mjs";
console.log(foo);
```

```
$ node main.mjs
foo
```

## 方法2：`package.json`に`"type": "module"`を指定する

```
$ tree
.
├── foo.js
├── main.js
└── package.json
```

`package.json`
```json
{
  "type": "module"
}
```

`foo.js`
```javascript
export const foo = "foo";
```

`main.js`
```javascript
import { foo } from "./foo.js";
console.log(foo);
```

```
$ node main.js
foo
```

---

【参考】

- [Modules: ECMAScript modules - Enabling | Node.js v19.1.0 Documentation](https://nodejs.org/api/esm.html#enabling)
- [Modules: Packages - Determining module system | Node.js v19.1.0 Documentation](https://nodejs.org/api/packages.html#determining-module-system)
- [javascript - SyntaxError: Cannot use import statement outside a module - Stack Overflow](https://stackoverflow.com/questions/58384179/syntaxerror-cannot-use-import-statement-outside-a-module)
- [How to Use ES Modules in Node.js](https://dmitripavlutin.com/ecmascript-modules-nodejs/)
