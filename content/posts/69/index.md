---
date: '2022-10-31T23:39:15+09:00'
draft: false
title: '【Ruby】Minitestの使い方'
tags: ["未分類"]
slug: '69'
---

この記事では、minitestを使ってRubyのコードをテストする方法を紹介します。

Rubyのテストフレームワーク、Test::Unit・test-unit・minitestの違いについては以下を参照ください。

- [Ruby標準のテスティングフレームワークで手軽にテストコードを書く方法 #RSpec - Qiita](https://qiita.com/jnchito/items/ff4f7a23addbd8dbc460)
- [Rubyのテスティングフレームワークの歴史（2014年版） - 2014-11-06 - ククログ](https://www.clear-code.com/blog/2014/11/6.html)

## minitestの使い方

minitestを使うための基本的なルールは以下です。

1. `require 'minitest/autorun'`を書く
1. `Minitest::Test`を継承するクラス内にテストを書く
1. メソッド名を`test_`から始める

```
$ tree
.
├── sample.rb
└── sample_test.rb
```

`sample.rb`
```ruby
class Sample
  def self.hello
    'Hello!'
  end
end
```

`sample_test.rb`
```ruby
require 'minitest/autorun'
require_relative 'sample'

class SampleTest < Minitest::Test
  def test_hello
    assert_equal 'Hello!', Sample.hello
  end
end
```

```
$ ruby sample_test.rb
Run options: --seed 20412

# Running:

.

Finished in 0.001143s, 874.8906 runs/s, 874.8906 assertions/s.
1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

---

【参考】

- [minitest/unit](https://docs.ruby-lang.org/ja/latest/library/minitest=2funit.html)
- [Module: Minitest::Assertions](https://www.rubydoc.info/gems/minitest/Minitest/Assertions)

