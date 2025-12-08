---
date: '2022-08-18T22:00:26+09:00'
draft: false
title: '【Ruby3.1】wcコマンドを作る'
tags: ["未分類"]
slug: '54'
---

この記事では、RubyでLinuxのwcコマンドを実装する方法を解説します。

gemを使わずにRubyの標準ライブラリのみで実装します。

## 1. 実行環境

- macOS：12.5
- Ruby：3.1.0

## 2. 作成するwcコマンドの要件

- オプションなし
  - 標準入力とファイルの入力を受け取る
- `-c`オプション
- `-l`オプション
- `-w`オプション
  - 単語の区切り文字は半角スペース、タブ、改行のみに対応

以下の機能は実装の対象外とします。

- ノーブレークスペースを含むマルチバイト文字の対応

## 3. wcコマンドの仕様

macOS標準のwcコマンドの仕様は以下の通りです。(ソースコードは[こちら](https://opensource.apple.com/source/text_cmds/text_cmds-99/wc/wc.c.auto.html))

(今回の要件に必要な箇所のみ)

- 引数なし
  - 標準入力を受け取る
  - Ctrl + D(EOF)を受け取るまで入力を受け付ける
- 引数にファイルを指定
  - ファイルを受け取る
  - 複数ファイルを指定した場合、出力の最終行に各ファイルの合計を表示
    - `<合計の行数> <合計の単語数> <合計のバイト数> total`
- オプションなし
  - `<行数> <単語数> <バイト数> <ファイル名>`の順で表示
- `-c`オプション
  - バイト数を表示する
- `-l`オプション
  - 行数を表示する(改行コードの数)
- `-w`オプション
  - 単語数を表示する
  - 半角スペース、タブ、改行で区切られた文字列の数(全角スペースはmacOS標準のwcコマンドも非対応)

## 4. ソースコード

- ver1：自作→レビュー反映
- ver2：ver1→他の人のコードを反映

### 4-1. ver1

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

require 'optparse'

def exec
  options = parse_options
  options = { c: true, l: true, w: true } if options.empty?
  paths = ARGV
  paths.empty? ? display_word_count_from_stdin(options) : display_word_count_from_files(paths, options)
end

def parse_options
  opt = OptionParser.new
  options = {}
  opt.on('-c')
  opt.on('-l')
  opt.on('-w')
  opt.parse!(ARGV, into: options)
  options
end

def display_word_count_from_stdin(options)
  lines = $stdin.readlines
  word_count_map = build_word_count_map(lines)
  display_word_count(word_count_map, options)
end

def build_word_count_map(lines, path = '')
  {
    number_of_lines: count_line(lines),
    number_of_words: count_word(lines),
    bytesize: count_bytesize(lines),
    path: path
  }
end

def count_line(lines)
  lines.size
end

def count_word(lines)
  lines.sum { |line| line.split(/[ \t\n]+/).size }
end

def count_bytesize(lines)
  lines.sum(&:bytesize)
end

def display_word_count(word_count_map, options)
  word_counts = []
  word_counts << format_word_count(word_count_map[:number_of_lines]) if options[:l]
  word_counts << format_word_count(word_count_map[:number_of_words]) if options[:w]
  word_counts << format_word_count(word_count_map[:bytesize]) if options[:c]
  word_counts << " #{word_count_map[:path]}" unless word_count_map[:path].empty?
  puts word_counts.join
end

def format_word_count(word_count)
  word_count.to_s.rjust(8)
end

def display_word_count_from_files(paths, options)
  word_count_maps = build_word_count_maps(paths)
  word_count_maps.map { |word_count_map| display_word_count(word_count_map, options) }
  display_total_word_count(word_count_maps, options) if paths.size >= 2
end

def build_word_count_maps(paths)
  paths.map do |path|
    lines = File.readlines(path)
    build_word_count_map(lines, path)
  end
end

def display_total_word_count(word_count_maps, options)
  total_word_count_map = build_total_word_count_map(word_count_maps)
  display_word_count(total_word_count_map, options)
end

def build_total_word_count_map(word_count_maps)
  {
    number_of_lines: word_count_maps.sum { |word_count_map| word_count_map[:number_of_lines] },
    number_of_words: word_count_maps.sum { |word_count_map| word_count_map[:number_of_words] },
    bytesize: word_count_maps.sum { |word_count_map| word_count_map[:bytesize] },
    path: 'total'
  }
end

exec
```

### 4-2. ver2

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

require 'optparse'

def exec
  options = parse_options
  options = { c: true, l: true, w: true } if options.empty?
  paths = ARGV
  counts = build_counts(paths: paths)
  counts.map { |count| display_count(count: count, options: options) }
  display_total_count(counts: counts, options: options) if paths.size >= 2
end

def parse_options
  opt = OptionParser.new
  options = {}
  opt.on('-c')
  opt.on('-l')
  opt.on('-w')
  opt.parse!(ARGV, into: options)
  options
end

def build_counts(paths:)
  if paths.empty?
    [build_count(text: $stdin.read)]
  else
    paths.map do |path|
      build_count(text: File.read(path), path: path)
    end
  end
end

def build_count(text:, path: '')
  {
    line_count: text.count("\n"),
    word_count: text.split(/\s+/).size,
    bytesize: text.bytesize,
    path: path
  }
end

def display_count(count:, options:)
  formatted_counts = []
  formatted_counts << format_count(count: count[:line_count]) if options[:l]
  formatted_counts << format_count(count: count[:word_count]) if options[:w]
  formatted_counts << format_count(count: count[:bytesize]) if options[:c]
  formatted_counts << " #{count[:path]}" unless count[:path].empty?
  puts formatted_counts.join
end

def format_count(count:)
  count.to_s.rjust(8)
end

def display_total_count(counts:, options:)
  total_count = build_total_count(counts: counts)
  display_count(count: total_count, options: options)
end

def build_total_count(counts:)
  {
    line_count: counts.sum { |count| count[:line_count] },
    word_count: counts.sum { |count| count[:word_count] },
    bytesize: counts.sum { |count| count[:bytesize] },
    path: 'total'
  }
end

exec
```

---

【参考】

- [wc(1) - Linux man page](https://linux.die.net/man/1/wc)
- https://linuxjm.osdn.jp/html/gnumaniak/man1/wc.1.html
- http://itdoc.hitachi.co.jp/manuals/3021/3021313320/JPAS0343.HTM
- [Rubyでwcコマンドを作った - どぼじょのIT学習ブログ](https://mistyrinth.hatenablog.com/entry/2019/03/15/201754)
- [wcコマンドのソースコードを読んでみた - いづいづブログ](https://izumii19.hatenablog.com/entry/2019/05/31/105636)
- [正規表現 (Ruby 3.4 リファレンスマニュアル)](https://docs.ruby-lang.org/ja/latest/doc/spec=2fregexp.html)
- [Rubyアソシエーション: 入出力](https://www.ruby.or.jp/ja/tech/development/ruby/tutorial/060_in_output)
- [Ruby 標準入力から複数行読み取りたい - んなまにのメモ帳](https://nnnamani.hateblo.jp/entry/2016/08/14/150900)
