---
date: '2022-08-04T21:12:59+09:00'
draft: false
title: '【Ruby3.1】lsコマンドを作る'
tags: ["未分類"]
slug: '50'
---

この記事では、RubyでLinuxのlsコマンドを実装する方法を解説します。

gemを使わずにRubyの標準ライブラリのみで実装します。

OOP版はこちら↓

[【Ruby3.1】lsコマンドを作る(OOP版) - あまブログ](https://ama-tech.hatenablog.com/ls-command-in-ruby-oop)

## 1. 実行環境
- macOS：12.5
- Ruby：3.1.0

## 2. 作成するlsコマンドの要件

今回はlsコマンドの以下の機能を実装の対象とします。

- オプションなしのlsコマンド
  - 横に最大3列を維持して表示
  - ファイルの並び順は列ごとに辞書式順序にソートされる
- `-a`オプション
- `-r`オプション
- `-l`オプション

**ファイルの並び順**

```zsh
# OK
0 4 8
1 5 9
2 6 
3 7

# NG
0 1 2
3 4 5
6 7 8
9
```

以下の機能は実装の対象外とします。

- 引数にファイルやディレクトリを指定可能にする
- mac拡張属性(@マーク)の表示

## 3. lsコマンドの仕様

macOS標準のlsコマンドの仕様は以下の通りです。(ソースコードは[こちら](https://opensource.apple.com/source/file_cmds/file_cmds-116.10/ls/ls.c.auto.html))

(今回の要件に必要な箇所のみ)

- 引数なし
  - カレントディレクトリの内容を表示
- `-a`オプション
  - ファイル名が`.`で始まるファイルを含めて表示
- `-l`オプション
  - ディレクトリ内の各ファイルのブロック数の合計を1行目に表示(`total <blocks>`)
  - 2行目以降に各ファイルをロングフォーマットで表示
- `-r`オプション
  - 逆順で表示

### 3-1. ロングフォーマットについて

`-l`オプション指定時に表示される以下のような形式をロングフォーマットと呼びます。

```zsh
dr-xr-xr-x   3 root  wheel  4539  7 30 07:10 dev
```

ロングフォーマットに含まれるファイルの情報(属性)は以下の7つです。

1. ファイルタイプとファイルモード
1. ハードリンク数
1. 所有者名
1. グループ名
1. ファイルサイズ
1. タイムスタンプ
1. ファイル名

#### 3-1-1. ファイルタイプとファイルモード

- ファイルタイプとファイルモードを10桁のアルファベットで表示(`drwxr-xr-x`)
  - 1桁目：ファイルタイプ
  - 2桁目~10桁目：ファイルモード
- [File::Stat#ftype](https://docs.ruby-lang.org/ja/3.1/method/File=3a=3aStat/i/ftype.html)でファイルタイプを取得
- [File::Stat#mode](https://docs.ruby-lang.org/ja/3.1/method/File=3a=3aStat/i/mode.html)でファイルモードを取得

lsコマンドの`-l`オプションで表示されるファイルタイプとファイルモードの詳細については以下の記事を参照ください。

[【ls -l】ファイルタイプとファイルモードの記号の意味 | あまブログ](https://ama-blog.com/48/)

また、[File::Stat#mode](https://docs.ruby-lang.org/ja/3.1/method/File=3a=3aStat/i/mode.html)が返すファイルモードの数値と記号表記の対応については以下の記事を参照ください。

[【Ruby】File::Stat#modeが返すファイルモードの数値を記号表記に変換する | あまブログ](https://ama-blog.com/49/)

#### 3-1-2. ハードリンク数

- ファイルのハードリンク数を表示
  - シンボリックリンクはカウントされない
  - [File::Stat#nlink](https://docs.ruby-lang.org/ja/3.1/method/File=3a=3aStat/i/nlink.html)でファイルのハードリンク数を取得

#### 3-1-3. 所有者名

- ファイルの所有者名を表示
  - [File::Stat#uid](https://docs.ruby-lang.org/ja/3.1/method/File=3a=3aStat/i/uid.html)でファイルの所有者のUIDを取得
  - ユーザ名とユーザID(UID)の対応は`/etc/passwd`で管理
  - [Etc.#getpwuid](https://docs.ruby-lang.org/ja/latest/method/Etc/m/getpwuid.html)でpasswdデータベースからUIDを検索、[Etc::Passwd#name](https://docs.ruby-lang.org/ja/latest/method/Etc=3a=3aPasswd/i/name.html)でユーザ名を返す


#### 3-1-4. グループ名

- ファイルの所有グループ名を表示
  - [File::Stat#gid](https://docs.ruby-lang.org/ja/3.1/method/File=3a=3aStat/i/gid.html)でファイルの所有グループのGIDを取得
  - グループ名とグループID(GID)の対応は`/etc/group`で管理
  - [Etc.#getgrgid](https://docs.ruby-lang.org/ja/latest/method/Etc/m/getgrgid.html)でgroupデータベースからGIDを検索、[Etc::Group#name](https://docs.ruby-lang.org/ja/latest/method/Etc=3a=3aGroup/i/name.html)でグループ名を返す

#### 3-1-5. ファイルサイズ

- ファイルサイズをバイト単位で表示
- ファイルがキャラクタデバイスまたはブロックデバイスの場合、ファイルサイズの代わりにデバイス番号を16進数で表示
  - [File::Stat#rdev](https://docs.ruby-lang.org/ja/3.1/method/File=3a=3aStat/i/rdev.html)でデバイスファイルのデバイス番号を取得

#### 3-1-6. タイムスタンプ

- ファイルの最終更新時刻を表示
  - 表示形式
    - デフォルト：`<月> <日> <時間>`
    - 最終更新時刻が6ヶ月以上前または未来の日付の場合：`<月> <日> <年>`
  - [File::Stat#mtime](https://docs.ruby-lang.org/ja/3.1/method/File=3a=3aStat/i/mtime.html)でファイルの最終更新時刻を取得

#### 3-1-7. ファイル名

- ファイル名を表示
- ファイルがシンボリックリンクの場合、リンク先のパスも表示
  - 例：`etc -> private/etc`


## 4. ソースコード

- ver1：自作→レビュー反映
- ver2：ver1→他の人のコードを反映

### 4-1. ver1

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

require 'optparse'
require 'etc'

params = ARGV.getopts('alr')

def display_files(params)
  files = get_files(params)
  if params['l']
    display_long_format(files)
  else
    display_sort_by_column(files)
  end
end

def get_files(params)
  files = if params['a']
            Dir.glob('*', File::FNM_DOTMATCH, base: './')
          else
            Dir.glob('*', base: './')
          end
  files = files.reverse if params['r']
  files
end

def display_sort_by_column(files)
  number_of_elements = files.size
  max_number_of_words = files.map(&:size).max
  max_number_of_columns = 3
  number_of_rows = calc_number_of_rows(number_of_elements, max_number_of_columns)
  number_of_rows.times do |i|
    i.step(number_of_elements - 1, number_of_rows) do |n|
      print files[n].ljust(max_number_of_words + 2)
    end
    print "\n"
  end
end

def calc_number_of_rows(number_of_elements, max_number_of_columns)
  if (number_of_elements % max_number_of_columns).zero?
    number_of_elements / max_number_of_columns
  else
    (number_of_elements / max_number_of_columns) + 1
  end
end

def display_long_format(files)
  long_formats = get_long_formats(files)
  max_widths = get_max_widths(long_formats)
  number_of_blocks = get_number_of_blocks(long_formats)
  puts "total #{number_of_blocks}"
  long_formats.each do |long_format|
    print "#{long_format[:file_mode]} "
    print "#{long_format[:number_of_links].rjust(max_widths[:link])} "
    print "#{long_format[:owner_name].ljust(max_widths[:owner])}  "
    print "#{long_format[:group_name].ljust(max_widths[:group])}  "
    print "#{long_format[:file_size].rjust(max_widths[:file_size])} "
    print "#{long_format[:last_modified_time]} "
    print "#{long_format[:pathname]}\n"
  end
end

def get_long_formats(files)
  long_formats = []
  files.each do |file|
    file_stat = File.lstat(file)
    long_format = {
      file_mode: get_file_mode(file_stat),
      number_of_links: file_stat.nlink.to_s,
      owner_name: Etc.getpwuid(file_stat.uid).name,
      group_name: Etc.getgrgid(file_stat.gid).name,
      file_size: get_file_size(file_stat),
      last_modified_time: get_last_modified_time(file_stat),
      pathname: get_pathname(file),
      blocks: file_stat.blocks
    }
    long_formats << long_format
  end
  long_formats
end

def get_max_widths(long_formats)
  links = []
  owners = []
  groups = []
  file_sizes = []
  long_formats.each do |long_format|
    links << long_format[:number_of_links]
    owners << long_format[:owner_name]
    groups << long_format[:group_name]
    file_sizes << long_format[:file_size]
  end
  {
    link: links.map(&:size).max,
    owner: owners.map(&:size).max,
    group: groups.map(&:size).max,
    file_size: file_sizes.map(&:size).max
  }
end

def get_number_of_blocks(long_formats)
  blocks = []
  long_formats.each do |long_format|
    blocks << long_format[:blocks]
  end
  blocks.sum
end

def get_file_mode(file_stat)
  file_mode_numeric = file_stat.mode.to_s(8).rjust(6, '0')
  file_type_symbolic = get_file_type_symbolic(file_stat.ftype)
  file_permissions_symbolic = get_file_permissions_symbolic(file_mode_numeric)
  "#{file_type_symbolic}#{file_permissions_symbolic}"
end

def get_file_type_symbolic(file_type)
  {
    'fifo' => 'p',
    'characterSpecial' => 'c',
    'directory' => 'd',
    'blockSpecial' => 'b',
    'file' => '-',
    'link' => 'l',
    'socket' => 's'
  }[file_type]
end

def get_file_permissions_symbolic(file_mode_numeric)
  file_permissions_symbolic = []
  file_mode_numeric.slice(3, 3).each_char do |file_permission_numeric|
    file_permission_symbolic = {
      '0' => '---',
      '1' => '--x',
      '2' => '-w-',
      '3' => '-wx',
      '4' => 'r--',
      '5' => 'r-x',
      '6' => 'rw-',
      '7' => 'rwx'
    }[file_permission_numeric]
    file_permissions_symbolic << file_permission_symbolic
  end
  get_special_permissions(file_mode_numeric, file_permissions_symbolic)
  file_permissions_symbolic.join
end

def get_special_permissions(file_mode_numeric, file_permissions_symbolic)
  case file_mode_numeric.slice(2)
  when '1'
    file_permissions_symbolic[2] = if file_permissions_symbolic[2].slice(2) == 'x'
                                     file_permissions_symbolic[2].gsub(/.$/, 't')
                                   else
                                     file_permissions_symbolic[2].gsub(/.$/, 'T')
                                   end
  when '2'
    file_permissions_symbolic[1] = if file_permissions_symbolic[1].slice(2) == 'x'
                                     file_permissions_symbolic[1].gsub(/.$/, 's')
                                   else
                                     file_permissions_symbolic[1].gsub(/.$/, 'S')
                                   end
  when '4'
    file_permissions_symbolic[0] = if file_permissions_symbolic[0].slice(2) == 'x'
                                     file_permissions_symbolic[0].gsub(/.$/, 's')
                                   else
                                     file_permissions_symbolic[0].gsub(/.$/, 'S')
                                   end
  end
end

def get_file_size(file_stat)
  if file_stat.rdev != 0
    "#{file_stat.rdev_major}, #{file_stat.rdev_minor}"
  else
    file_stat.size.to_s
  end
end

def get_last_modified_time(file_stat)
  if Time.now - file_stat.mtime >= (60 * 60 * 24 * (365 / 2.0)) || (Time.now - file_stat.mtime).negative?
    file_stat.mtime.strftime('%_m %_d  %Y')
  else
    file_stat.mtime.strftime('%_m %_d %H:%M')
  end
end

def get_pathname(file)
  if File.symlink?(file)
    "#{file} -> #{File.readlink(file)}"
  else
    file
  end
end

display_files(params)
```

### 4-2. ver2

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

require 'optparse'
require 'etc'

COLUMN_NUMBER = 3

MODE_MAP = {
  '0' => '---',
  '1' => '--x',
  '2' => '-w-',
  '3' => '-wx',
  '4' => 'r--',
  '5' => 'r-x',
  '6' => 'rw-',
  '7' => 'rwx'
}.freeze

def exec
  params = ARGV.getopts('alr')

  paths = get_paths(dotmatch: params['a'])
  list_paths(paths, long_format: params['l'], reverse: params['r'])
end

def get_paths(dotmatch: false)
  dotmatch ? Dir.glob('*', File::FNM_DOTMATCH) : Dir.glob('*')
end

def list_paths(paths, long_format: false, reverse: false)
  paths = paths.reverse if reverse
  long_format ? list_long(paths) : list_short(paths)
end

def list_long(paths)
  long_formats = paths.map { |path| get_long_format(path) }
  max_length_map = get_max_length_map(long_formats)
  block_total = long_formats.map { |long_format| long_format[:blocks] }.sum

  puts "total #{block_total}"
  long_formats.each { |long_format| print_long_format(long_format, max_length_map) }
end

def get_long_format(path)
  path_stat = File.lstat(path)
  {
    type: format_type(path_stat.ftype),
    mode: format_mode(path_stat.mode),
    nlink: path_stat.nlink.to_s,
    username: Etc.getpwuid(path_stat.uid).name,
    groupname: Etc.getgrgid(path_stat.gid).name,
    bitesize: get_bitesize(path_stat),
    mtime: get_mtime(path_stat),
    pathname: get_pathname(path),
    blocks: path_stat.blocks
  }
end

def format_type(type)
  {
    'fifo' => 'p',
    'characterSpecial' => 'c',
    'directory' => 'd',
    'blockSpecial' => 'b',
    'file' => '-',
    'link' => 'l',
    'socket' => 's'
  }[type]
end

def format_mode(mode)
  mode_octal = mode.to_s(8)
  permissions_numeric = mode_octal.slice(-3..-1).split(//)
  permissions_symbolic = permissions_numeric.map { |n| MODE_MAP[n] }
  add_special_permissions(mode_octal, permissions_symbolic).join
end

def add_special_permissions(mode_octal, permissions_symbolic)
  case mode_octal.slice(-4)
  when '1'
    add_sticky_bit(permissions_symbolic)
  when '2'
    add_sgid(permissions_symbolic)
  when '4'
    add_suid(permissions_symbolic)
  end
  permissions_symbolic
end

def add_sticky_bit(permissions_symbolic)
  permissions_symbolic[2] = if permissions_symbolic[2].slice(2) == 'x'
                              permissions_symbolic[2].gsub(/.$/, 't')
                            else
                              permissions_symbolic[2].gsub(/.$/, 'T')
                            end
end

def add_sgid(permissions_symbolic)
  permissions_symbolic[1] = if permissions_symbolic[1].slice(2) == 'x'
                              permissions_symbolic[1].gsub(/.$/, 's')
                            else
                              permissions_symbolic[1].gsub(/.$/, 'S')
                            end
end

def add_suid(permissions_symbolic)
  permissions_symbolic[0] = if permissions_symbolic[0].slice(2) == 'x'
                              permissions_symbolic[0].gsub(/.$/, 's')
                            else
                              permissions_symbolic[0].gsub(/.$/, 'S')
                            end
end

def get_bitesize(path_stat)
  if path_stat.rdev != 0
    "0x#{path_stat.rdev.to_s(16)}"
  else
    path_stat.size.to_s
  end
end

def get_mtime(path_stat)
  if Time.now - path_stat.mtime >= (60 * 60 * 24 * (365 / 2.0)) || (Time.now - path_stat.mtime).negative?
    path_stat.mtime.strftime('%_m %_d  %Y')
  else
    path_stat.mtime.strftime('%_m %_d %H:%M')
  end
end

def get_pathname(path)
  if File.symlink?(path)
    "#{path} -> #{File.readlink(path)}"
  else
    path
  end
end

def get_max_length_map(long_formats)
  {
    nlink: long_formats.map { |long_format| long_format[:nlink].size }.max,
    username: long_formats.map { |long_format| long_format[:username].size }.max,
    groupname: long_formats.map { |long_format| long_format[:groupname].size }.max,
    bitesize: long_formats.map { |long_format| long_format[:bitesize].size }.max
  }
end

def print_long_format(long_format, max_length_map)
  print [
    "#{long_format[:type]}#{long_format[:mode]} ",
    "#{long_format[:nlink].rjust(max_length_map[:nlink])} ",
    "#{long_format[:username].ljust(max_length_map[:username])}  ",
    "#{long_format[:groupname].ljust(max_length_map[:groupname])}  ",
    "#{long_format[:bitesize].rjust(max_length_map[:bitesize])} ",
    "#{long_format[:mtime]} ",
    "#{long_format[:pathname]}\n"
  ].join
end

def list_short(paths)
  element_number = paths.size.to_f
  max_length = paths.map(&:size).max
  row_number = (element_number / COLUMN_NUMBER).ceil
  lines = Array.new(row_number) { [] }
  paths.each_with_index do |path, index|
    line_number = index % row_number
    lines[line_number].push(path.ljust(max_length + 2))
  end
  lines.each { |line| puts line.join }
end

exec
```

---

【参考】

- https://linuxjm.osdn.jp/html/gnumaniak/man1/ls.1.html
- [ls Man Page - macOS - SS64.com](https://ss64.com/mac/ls.html)
