---
date: '2022-06-10T09:53:31+09:00'
draft: false
title: '【RuboCopエラー】Use the return of the conditional for variable assignment and comparison.'
tags: ["未分類"]
slug: '26'
---

rubocopで`Use the return of the conditional for variable assignment and comparison.`のエラーが出た時の対処法。

## 1. 環境
- rubocop 1.30.0
- Ruby 3.1.0

## 2. エラー内容

```
Use the return of the conditional for variable assignment and comparison.
```

変数の代入と比較には、条件式の戻り値を使用してください。

## 3. エラー対処法

```ruby
# bad
if foo
  bar = 1
else
  bar = 2
end

# good
bar = if foo
        1
      else
        2
      end
```

---

- [RubyDoc.info: Class: RuboCop::Cop::Style::ConditionalAssignment – Documentation for rubocop (1.81.1) – RubyDoc.info](https://www.rubydoc.info/gems/rubocop/RuboCop/Cop/Style/ConditionalAssignment)
- [rubocop/ruby-style-guide: A community-driven Ruby coding style guide](https://github.com/rubocop/ruby-style-guide)
- [Ruby: Use the return of the conditional for variable assignment and comparison - Stack Overflow](https://stackoverflow.com/questions/48731014/ruby-use-the-return-of-the-conditional-for-variable-assignment-and-comparison)
- [【Rubocop】Use the return of the conditional for variable assignment and comparison.エラーの対処法 - 紙一重の積み重ね](https://www.yokoyan.net/entry/2019/03/08/180000)
