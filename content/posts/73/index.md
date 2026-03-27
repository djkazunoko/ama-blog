---
date: '2022-11-14T12:39:15+09:00'
draft: false
title: '【ES2015】JavaScriptの関数の書き方'
tags: ["未分類"]
slug: '73'
---

## 1. 関数宣言(Function declaration)

```javascript
function calcRectArea(width, height) {
  return width * height;
}

console.log(calcRectArea(5, 6)); // => 30
```

### 1-1. 関数宣言の巻き上げ

関数宣言は巻き上げられる
```javascript
hello(); // => "Hello"

function hello(){
    return "Hello";
}
```

関数式は巻き上げられない
```javascript
hello(); // => TypeError: hello is not a function

const hello = function(){
    return "Hello";
};
```

## 2. 関数式(Function expression)

### 2-1. 名前付き関数(Named function)

- 名前付き関数は再帰的に関数を呼び出す際などに利用する
```javascript
// factorialは関数の外から呼び出せる名前
// innerFactは関数の外から呼び出せない名前
const factorial = function innerFact(n) {
    if (n === 0) {
        return 1;
    }
    // innerFactを再帰的に呼び出している
    return n * innerFact(n - 1);
};
console.log(factorial(3)); // => 6
```

### 2-2. 無名関数(Anonymous Function)

- 無名関数はコールバック関数としてよく使われる
```javascript
const x = function(a, b) {
   return a * b;
};
```

## 3. アロー関数(Arrow function expression)

- アロー関数は関数式の代替構文(機能的な違いはいくつかある)
- ES2015から追加された
```javascript
const x = (a, b) => {
  return a * b;
};
```

### 3-1. アロー関数の省略記法

- 関数の仮引数が1つのときは`()`を省略できる
- 関数の処理が1つの式である場合に、ブロック(`{}`)と`return`文を省略できる
  - その式の評価結果を`return`の返り値とする

```javascript
// 仮引数の数と定義
const fnA = () => { /* 仮引数がないとき */ };
const fnB = (x) => { /* 仮引数が1つのみのとき */ };
const fnC = x => { /* 仮引数が1つのみのときは()を省略可能 */ };
const fnD = (x, y) => { /* 仮引数が複数のとき */ };
// 値の返し方
const fnE = x => { return x + x; }; // ブロックの中でreturn
const fnF = x => x + x;            // 1行のみの場合はreturnとブロックを省略できる
const fnG = x => {
  let y = 10;
  return x + y;
};  // 関数の処理が複数行ある場合はreturnとブロックは省略できない
```

## 4. メソッド定義(Method definition)

- オブジェクトのプロパティである関数をメソッドと呼ぶ
- JavaScriptにおいて関数とメソッドの機能的な違いはない
- `オブジェクト.メソッド名()`でメソッドを呼び出す

```javascript
const obj = {
  method1: function () {
    return "this is method1";
  }, // `function`キーワードでのメソッド
  method2: () => "this is method2" // Arrow Functionでのメソッド
};
console.log(obj.method1());  // => "this is method1"
console.log(obj.method2());  // => "this is method2"
```

次のように空オブジェクトの`obj`を定義してから、`method`プロパティへ関数を代入してもメソッドを定義できる

```javascript
const obj = {};
obj.method = function() {
};
```

### 4-1. メソッドの短縮記法

- ES2015からメソッドの短縮記法が追加された
- オブジェクトリテラルの中で`メソッド名(){ /*メソッドの処理*/ }` と書くことができる

```javascript
const obj = {
    method() {
        return "this is method";
    }
};
console.log(obj.method()); // => "this is method"
```
- この書き方はオブジェクトのメソッドだけではなく、クラスのメソッドと共通の書き方となっているため、メソッドを定義する場合は、この短縮記法に統一する

---

【参考】

- [関数と宣言 · JavaScript Primer #jsprimer](https://jsprimer.net/basic/function-declaration/)
- [O'Reilly Japan - 初めてのJavaScript 第3版 - 6章 関数](https://www.oreilly.co.jp/books/9784873117836/)
- [JavaScript Function Definitions](https://www.w3schools.com/js/js_function_definition.asp)
- [関数 - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions)
- [function - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/function)
- [function 式 - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/function)
- [アロー関数式 - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [メソッド定義 - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Method_definitions)
- [Function() コンストラクター - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/Function)
