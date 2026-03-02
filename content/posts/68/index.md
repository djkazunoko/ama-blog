---
date: '2022-10-31T12:25:00+09:00'
draft: false
title: '【Ruby3.1】ボウリングのスコア計算プログラムをオブジェクト指向で作る'
tags: ["未分類"]
slug: '68'
---

非OOP版はこちら↓

[【Ruby 3.1】ボウリングのスコア計算プログラムを作る | あまブログ](https://ama-blog.com/34/)

## 1. 実行環境

- macOS：12.6
- Ruby：3.1.0

## 2. ソースコード

`bowling.rb`
```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

require_relative 'game'

shots = ARGV[0].split(',')

game = Game.new(shots)
puts game.score
```

`game.rb`
```ruby
# frozen_string_literal: true

require_relative 'frame'

class Game
  def initialize(shots)
    @shots = shots
  end

  def score
    @frames = build_frames
    game_score = 0
    (0..9).each do |idx|
      frame = Frame.new(@frames[idx])
      game_score += frame.score
      @frames[idx + 1] ||= []
      @frames[idx + 2] ||= []
      game_score += calc_bonus_point(idx, frame)
    end
    game_score
  end

  def build_frames
    frame = []
    frames = []
    @shots.each do |s|
      frame << s
      if frames.length < 10
        if frame.length >= 2 || s == 'X'
          frames << frame.dup
          frame.clear
        end
      else
        frames.last << s
      end
    end
    frames
  end

  def calc_bonus_point(idx, frame)
    if frame.strike?
      next_two_shots = (@frames[idx + 1] + @frames[idx + 2]).slice(0, 2)
      bonus_point = next_two_shots.sum { |s| Shot.new(s).score }
    elsif frame.spare?
      next_shot = @frames[idx + 1][0]
      bonus_point = Shot.new(next_shot).score
    else
      bonus_point = 0
    end
  end
end
```

`frame.rb`
```ruby
# frozen_string_literal: true

require_relative 'shot'

class Frame
  def initialize(frame)
    @first_shot = Shot.new(frame[0])
    @second_shot = Shot.new(frame[1])
    @third_shot = Shot.new(frame[2])
  end

  def score
    [
      @first_shot.score,
      @second_shot.score,
      @third_shot.score
    ].sum
  end

  def strike?
    @first_shot.score == 10
  end

  def spare?
    score == 10
  end
end
```

`shot.rb`
```ruby
# frozen_string_literal: true

class Shot
  def initialize(mark)
    @mark = mark
  end

  def score
    @mark == 'X' ? 10 : @mark.to_i
  end
end
```
