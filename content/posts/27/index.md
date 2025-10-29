---
date: '2022-06-10T10:00:02+09:00'
draft: false
title: '【RuboCopエラー】Favor modifier if usage when having a single-line body. Another good alternative is the usage of control flow &&/||.'
tags: ["未分類"]
slug: '27'
---

## 1. 環境
- rubocop 1.30.0
- Ruby 3.1.0

## 2. エラー内容

```
Favor modifier if usage when having a single-line body. Another good alternative is the usage of control flow &&/||.
```

if文の中身が1行の場合は、後置ifを使用するか、`&&`または`||`を使用してください。

## 3. エラー対処法

```ruby
# bad
if some_condition
  do_something
end

# good
do_something if some_condition

# another good option
some_condition && do_something
```
---


- [RubyDoc.info: Class: RuboCop::Cop::Style::IfUnlessModifier – Documentation for rubocop (1.81.6) – RubyDoc.info](https://www.rubydoc.info/gems/rubocop/RuboCop/Cop/Style/IfUnlessModifier)
- [rubocop/ruby-style-guide: A community-driven Ruby coding style guide](https://github.com/rubocop/ruby-style-guide#if-as-a-modifier)
