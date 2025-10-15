---
date: '2022-04-27'
draft: false
title: '【Linuxコマンド】grep・egrep・fgrepの違い'
tags: ["未分類"]
slug: '4'
---

本稿では、Linuxのgrep・egrep・fgrepコマンドの違いを解説します。

- grep
    - パターンを基本正規表現(BRE：Basic Regular Expression)として扱う
- egrep
    - パターンを拡張正規表現(ERE:Extended Regular Expression)として扱う
    - grep -Eと同じ
- fgrep
    - パターンを固定文字列として扱うため、文字列をそのまま検索できる(正規表現を認識しない)
    - grep -Fと同じ

以下のファイルを使用してそれぞれのコマンドの違いを見ていきます。

```zsh
$ cat check_file
file
(f|g)ile
'\(f\|g\)ile'
```

## 1. grep

grepで使われるBRE(基本正規表現)では`{`,`}`,`(`,`)`,`|`,`+`,`?`を通常の文字列として扱います。

そのため、これらのメタ文字をちゃんとメタ文字として扱うには逆にエスケープが必要です。
```zsh
$ grep -C 0 '(f|g)ile' check_file
(f|g)ile
$ grep -C 0 '\(f\|g\)ile' check_file
file
```
1つ目の検索のように`(`,`|`,`)`をエスケープしない場合、BREはこれらのメタ文字を通常の文字列として扱うため、`(f|g)ile`という文字列を検索します。

2つ目の検索のように`(`,`|`,`)`をエスケープすると、BREはこれらをメタ文字として扱うため、メタ文字の意味通りに`file`または`gile`という文字列を検索します。

## 2. egrep

egrepで使われるERE(拡張正規表現)ではメタ文字をそのままメタ文字として扱うため、正規表現を使った検索に向いています。

```zsh
$ egrep -C 0 '(f|g)ile' check_file
file
$ egrep -C 0 '\(f\|g\)ile' check_file
(f|g)ile
```
1つ目の検索のように`(`,`|`,`)`をエスケープしない場合、EREはメタ文字の意味通りに`file`または`gile`という文字列を検索します。

2つ目の検索のように`(`,`|`,`)`をエスケープすると、EREはこれらのメタ文字を通常の文字列として扱うため、`(f|g)ile`という文字列を検索します。

## 3. fgrep

fgrepはメタ文字を認識しないため、直接文字列を検索するのに向いています。

```zsh
$ fgrep -C 0 '(f|g)ile' check_file
(f|g)ile
$ fgrep -C 0 '\(f\|g\)ile' check_file
'\(f\|g\)ile'
```
1つ目の検索では、fgrepはメタ文字を通常の文字列として扱うため、`(f|g)ile`という文字列を検索します。

2つ目の検索では、fgrepは`\`も通常の文字列として扱うため、`\(f\|g\)ile`という文字列を検索します。

---

【参考】

- [What’s Difference Between Grep, Egrep and Fgrep in Linux?](https://www.tecmint.com/difference-between-grep-egrep-and-fgrep-in-linux/)
- [Man Page of grep](https://linuxjm.osdn.jp/html/GNU_grep/man1/grep.1.html)
- [Linuxのコマンドで使われる正規表現の違い](https://azisava.sakura.ne.jp/programming/0016.html)
