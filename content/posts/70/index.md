---
date: '2022-11-01T22:37:08+09:00'
draft: false
title: '【RuboCopエラー】Use a guard clause instead of wrapping the code inside a conditional expression.'
tags: ["未分類"]
slug: '70'
---

## バージョン情報
- rubocop 1.34.1
- Ruby 3.1.2

## エラー内容

```
Use a guard clause instead of wrapping the code inside a conditional expression.
```

→例外処理の条件分岐のネストが深くなるのを防ぐためにGuard Clause(ガード節)を使いましょう。

## エラー対処

```ruby
# これを
def image_check
  unless avatar.image?
    errors.add(:avatar, 'エラーメッセージ')
  end
end

# こうする
def image_check
  return if avatar.image?

  errors.add(:avatar, 'エラーメッセージ')
end
```

---

【参考】

- [Ruby Style Guide: Nested Conditionals](https://github.com/rubocop/ruby-style-guide#no-nested-conditionals)
- [Class: RuboCop::Cop::Style::GuardClause](https://www.rubydoc.info/gems/rubocop/RuboCop/Cop/Style/GuardClause)
- [Guard clause instead of wrapping the code inside a conditional expression Rails - Stack Overflow](https://stackoverflow.com/questions/31153934/guard-clause-instead-of-wrapping-the-code-inside-a-conditional-expression-rails)
- [[Ruby/Rails] 例外で深くなったネストをGuard Clauseですっきりさせる｜TechRacho by BPS株式会社](https://techracho.bpsinc.jp/hachi8833/2016_10_11/26950)
