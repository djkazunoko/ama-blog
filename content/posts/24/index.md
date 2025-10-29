---
date: '2022-06-08T09:24:49+09:00'
draft: false
title: '【Ruby 3.1】カレンダーのプログラムを作る'
tags: ["未分類"]
slug: '24'
---

## 1. 環境
- macOS Monterey 12.4
- Ruby 3.1.0

## 2. カレンダープログラムの要件
- -mで月を、-yで年を指定できる
- 引数を指定しない場合は、今年・今月のカレンダーが表示される
- macに入っているcalコマンドと同じ見た目になっている
- 今日の日付の部分の色が反転する
- どのような引数が与えられようが、cal コマンドと同じ表示結果になる

## 3. ソースコード

### 3-1. 試作品(自作→レビュー反映)

```ruby
#!/usr/bin/env ruby

require 'date'
require 'optparse'

today = Date.today
options = ARGV.getopts("", "m:#{today.month}", "y:#{today.year}")

if options["m"].to_i >= 1 && options["m"].to_i <= 12
  inputed_month = options["m"].to_i
else
  puts "cal: #{options["m"]} is neither a month number (1..12) nor a name"
  return
end

if options["y"].to_i >= 1 && options["y"].to_i <= 9999
  inputed_year = options["y"].to_i
else
  puts "cal: year `#{options["y"]}' not in range 1..9999"
  return
end

first_date = Date.new(inputed_year, inputed_month, 1)
last_date = Date.new(inputed_year, inputed_month, -1)
space = "   "

puts "      #{inputed_month}月 #{inputed_year}"
puts "日 月 火 水 木 金 土"

print space * first_date.wday

def color_reverse(text)
  "\e[30m\e[47m#{text}\e[0m"
end

(first_date..last_date).each do |full_date|
  day_of_week = full_date.wday
  if full_date == today
    print color_reverse(full_date.day).rjust(16)
  else
    print full_date.day.to_s.rjust(2)
  end
  print " "
  puts "" if day_of_week == 6
end

puts ""


```

### 3-2. 最終形(自作→レビュー反映→他の人のコードを見る)

```ruby
#!/usr/bin/env ruby

require 'date'
require 'optparse'

params = ARGV.getopts('m:y:')
month = (params['m'] || Date.today.month).to_i
year = (params['y'] || Date.today.year).to_i

if month < 1 || month > 12
  puts "cal: #{params['m']} is neither a month number (1..12) nor a name"
  return
end

if year < 1 || year > 9999
  puts "cal: year `#{params['y']}' not in range 1..9999"
  return
end

start_of_month = Date.new(year, month, 1)
end_of_month = Date.new(year, month, -1)

puts "      #{month}月 #{year}"
puts "日 月 火 水 木 金 土"
print " " * 3 * start_of_month.wday
(start_of_month..end_of_month).each do |day|
  format = day == Date.today ? "\e[7m%2d\e[0m " : '%2d '
  printf format, day.day
  puts "\n" if day.wday == 6
end
puts "\n\n"
```

## 4. ソースコード(最終形)の解説

### 4-1. 主な使用メソッド

- [Arguable#getopts](https://docs.ruby-lang.org/ja/3.1/method/OptionParser=3a=3aArguable/i/getopts.html)
- [Date.today](https://docs.ruby-lang.org/ja/latest/method/Date/s/today.html)
- [Date#mon](https://docs.ruby-lang.org/ja/latest/method/Date/i/mon.html)
- [Date#year](https://docs.ruby-lang.org/ja/latest/method/Date/i/year.html)
- [Date#wday](https://docs.ruby-lang.org/ja/latest/method/Date/i/wday.html)
- [Date#day](https://docs.ruby-lang.org/ja/latest/method/Date/i/day.html)
- [Kernel.#printf](https://docs.ruby-lang.org/ja/3.1/method/Kernel/m/printf.html)

### 4-2. 解説

```ruby
# 6行目
params = ARGV.getopts('m:y:')

# 引数に「-m」と「-y」をとる
```

```ruby
# 7~8行目
month = (params['m'] || Date.today.month).to_i
year = (params['y'] || Date.today.year).to_i

# それぞれの引数が指定されなかった場合、monthに今月の月、yearに今年の年が入る
```

```ruby
# 10~13行目
if month < 1 || month > 12
  puts "cal: #{params['m']} is neither a month number (1..12) nor a name"
  return
end

# -mオプションで1~12以外が指定された場合、エラーメッセージを表示して処理を終了
```

```ruby
# 15~18行目
if year < 1 || year > 9999
  puts "cal: year `#{params['y']}' not in range 1..9999"
  return
end

# -yオプションで1~9999以外が指定された場合、エラーメッセージを表示して処理を終了
```

```ruby
# 20~21行目
start_of_month = Date.new(year, month, 1)
end_of_month = Date.new(year, month, -1)

# 月の初日と最終日
```

```ruby
# 25行目
print " " * 3 * start_of_month.wday

# 月の初日の曜日によって表示する位置を調整
# start_of_month.wdayには曜日を表す数値(日曜から0~6)が入る
```

```ruby
# 26行目
(start_of_month..end_of_month).each do |day|

# dayに<Date: 2017-09-20 ...>の形式で月の初日から最終日までが入る
```

```ruby
# 27~28行目
format = day == Date.today ? "\e[7m%2d\e[0m " : '%2d '
printf format, day.day

# 条件演算子によりdayが今日の日付の場合"\e[7m%2d\e[0m "が、それ以外の場合'%2d 'がformatに代入される
# printfにより、指定したフォーマットでday.day(日付)が表示される(今日の日付の場合に色を反転して表示)
```

- [エスケープシーケンスについて](https://ama-tech.hatenablog.com/ansi-escape-code-in-ruby)
- [sprintf フォーマット](https://docs.ruby-lang.org/ja/latest/doc/print_format.html)
  - [フォーマット指定子一覧](https://www.k-cube.co.jp/wakaba/server/format.html)



```ruby
# 29行目
puts "\n" if day.wday == 6

# 土曜日で改行
```
