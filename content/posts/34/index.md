---
date: '2022-06-24T21:48:46+09:00'
draft: false
title: '【Ruby 3.1】ボウリングのスコア計算プログラムを作る'
tags: ["未分類"]
slug: 34'
---

OOP版はこちら↓

[【Ruby3.1】ボウリングのスコア計算プログラムをオブジェクト指向で作る - あまブログ](https://ama-tech.hatenablog.com/oop-bowling-score-calculator-program-in-ruby)

## 1. 実行環境
- macOS Monterey 12.4
- Ruby 3.1.0

## 2. ボウリングのスコア計算プログラムの要件

- 1ゲーム = 10フレーム
- 1フレーム = 2投
- スペアのフレームの得点は次の1投の点を加算する。
- ストライクのフレームの得点は次の2投の点を加算する。
- 10フレーム目は1投目がストライクもしくは2投目がスペアだった場合、3投目が投げられる。
- ありえない投球数やありえない数字・記号がこない前提。
- 入力例：`6,3,9,0,0,3,8,2,7,3,X,9,1,8,0,X,6,4,5`

## 3. ソースコード

- ver1：自作→レビュー反映
- ver2：ver1→他の人のコードを反映

### 3-1. ver1

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

score = ARGV[0]
scores = score.split(',')
shots = []
scores.each do |s|
  if s == 'X'
    shots << 10
    shots << 0
  else
    shots << s.to_i
  end
end

frames = []
shots.each_slice(2) do |s|
  frames << s
end

frames_point = []
frames.each_with_index do |frame, i|
  frames_point[i] = frame.sum
  if i < 9
    if frame[0] == 10
      frames_point[i] += frames[i + 1][0]
      frames_point[i] += if frames[i + 1][0] == 10
                           frames[i + 2][0]
                         else
                           frames[i + 1][1]
                         end
    elsif frame.sum == 10
      frames_point[i] += frames[i + 1][0]
    end
  end
end
puts frames_point.sum

```

### 3-2. ver2

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

shots = ARGV[0].split(',').map { |s| s == 'X' ? 10 : s.to_i }

frame = []
frames = []
shots.each do |s|
  frame << s
  if frames.length < 10
    if frame.length >= 2 || s == 10
      frames << frame.dup
      frame.clear
    end
  else
    frames.last << s
  end
end

point = 0
(0..9).each do |n|
  point += frames[n].sum
  frames[n + 1] ||= []
  frames[n + 2] ||= []
  if frames[n][0] == 10
    point += (frames[n + 1] + frames[n + 2]).slice(0, 2).sum
  elsif frames[n].sum == 10
    point += frames[n + 1][0]
  end
end
puts point

```

## 4. ソースコード(ver2)の解説

### 4-1. 主な使用メソッド

- [String#split](https://docs.ruby-lang.org/ja/3.1/method/String/i/split.html)
- [Enumerable#map](https://docs.ruby-lang.org/ja/3.1/method/Enumerable/i/collect.html)
- [Array#length](https://docs.ruby-lang.org/ja/latest/method/Array/i/length.html)
- [Array#dup](https://docs.ruby-lang.org/ja/latest/method/Array/i/clone.html)
- [Array#clear](https://docs.ruby-lang.org/ja/3.1/method/Array/i/clear.html)
- [Array#last](https://docs.ruby-lang.org/ja/3.1/method/Array/i/last.html)
- [Array#slice](https://docs.ruby-lang.org/ja/3.1/method/Array/i/slice.html)
- [Array#sum](https://docs.ruby-lang.org/ja/3.1/method/Array/i/sum.html)

### 4-2. 解説

#### 4行目

```ruby
shots = ARGV[0].split(',').map { |s| s == 'X' ? 10 : s.to_i }
```

- `ARGV[0].split(',')`で入力された文字列を`,`で分割して文字列の配列にする
  - `"6,3,9,0,0,3,8,2,7,3,X,9,1,8,0,X,6,4,5"`が`["6", "3", "9", "0", "0", "3", "8", "2", "7", "3", "X", "9", "1", "8", "0", "X", "6", "4", "5"]`になる
- `.map { |s| s == 'X' ? 10 : s.to_i }`でXを数値の10に、それ以外を数値に変換する
- 結果、shotsには`[6, 3, 9, 0, 0, 3, 8, 2, 7, 3, 10, 9, 1, 8, 0, 10, 6, 4, 5]`が入る

#### 6~18行目

```ruby
frame = []
frames = []
shots.each do |s|
  frame << s
  if frames.length < 10
    if frame.length >= 2 || s == 10 # frameに2投またはストライクの1投が入ったら
      frames << frame.dup
      frame.clear
    end
  else # 10フレーム目
    frames.last << s
  end
end
```

shots`[6, 3, 9, 0, 0, 3, 8, 2, 7, 3, 10, 9, 1, 8, 0, 10, 6, 4, 5]`から、各フレーム毎に分割したframes`[[6, 3], [9, 0], [0, 3], [8, 2], [7, 3], [10], [9, 1], [8, 0], [10], [6, 4, 5]]`を作りたい

- `frames << frame.dup`について
  - `frames << frame`だとframeの参照が格納されるだけでオブジェクトそのものは格納されない
  - frame.dupを使うことでframeのオブジェクトそのもののコピーが作成され、framesに格納される

#### 21~30行目

```ruby
(0..9).each do |n|
  point += frames[n].sum # 倒したピンの本数を加算
  frames[n + 1] ||= []
  frames[n + 2] ||= []
  if frames[n][0] == 10 # ストライクの場合
    point += (frames[n + 1] + frames[n + 2]).slice(0, 2).sum # 次の2投を加算
  elsif frames[n].sum == 10 # スペアの場合
    point += frames[n + 1][0] # 次の1投を加算
  end
end
```

1~10フレーム目までの点数を加算

点数は倒したピンの本数 + スペアまたはストライクのボーナス

- `(frames[n + 1] + frames[n + 2]).slice(0, 2).sum`について
  - `frames[n + 1] + frames[n + 2]`で次の2フレームを合わせる
    - ストライクの次の2フレームが[6, 3]と[0, 4]なら、[6, 3, 0, 4]になる
    - ストライクの次の2フレームが[10]と[10]なら、[10, 10]になる
  - `.slice(0, 2)`で先頭から2つ→次の2投を加算
    - `[6, 3, 0, 4].slice(0, 2)`なら[6, 3]
    - `[10, 10].slice(0, 2)`なら[10, 10]
  - 連続ストライクのための施策
  - スペアはシンプルに次の1投だけなのでこのような施策はいらない

- `frames[n + 1] ||= []`と`frames[n + 2] ||= []`について
  - 9~10フレーム目で次の1フレームまたは2フレームがない場合の施策
  - nilガードを使う事で、frames[n + 1]またはframes[n + 2]が存在しない場合に、nilではなく空の配列が入るので、9~10フレーム目でも問題なくスペア・ストライクのボーナス加算ができる(10フレーム目は0点を加算してることになる)

---

【参考】

- [Rubyでオブジェクトの複製を作るならdupメソッドを | ポテパンスタイル](https://style.potepan.com/articles/27809.html)
