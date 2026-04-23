---
date: '2022-12-02T10:07:43+09:00'
draft: false
title: '【Ruby3.1】lsコマンドを作る(OOP版)'
tags: ["未分類"]
slug: '84'
---

非OOP版はこちら↓

[【Ruby3.1】lsコマンドを作る | あまブログ](https://ama-blog.com/50/)

## 1. 実行環境

- macOS：13.0.1
- Ruby：3.1.0

## 2. ソースコード

`ls.rb`

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

require_relative 'command'

LS::Command.new(ARGV).list_files
```

`command.rb`

```ruby
# frozen_string_literal: true

require 'optparse'
require 'pathname'
require_relative 'file_stat'
require_relative 'long_formatter'
require_relative 'short_formatter'

module LS
  class Command
    def initialize(argv)
      @params = argv.getopts('alr')
      @target_dir = Pathname(argv[0] || '.')
    end

    def list_files
      files = build_files
      @params['l'] ? LongFormatter.new(files).list_files : ShortFormatter.new(files).list_files
    end

    private

    def build_files
      pattern = @target_dir.join('*')
      paths = @params['a'] ? Dir.glob(pattern, File::FNM_DOTMATCH) : Dir.glob(pattern)
      paths = paths.reverse if @params['r']
      paths.map { |path| FileStat.new(path) }
    end
  end
end
```

`file_stat.rb`

```ruby
# frozen_string_literal: true

require 'etc'
require_relative 'mode_formatter'

module LS
  class FileStat
    TYPE_MAP = {
      'fifo' => 'p',
      'characterSpecial' => 'c',
      'directory' => 'd',
      'blockSpecial' => 'b',
      'file' => '-',
      'link' => 'l',
      'socket' => 's'
    }.freeze

    attr_reader :basename, :type, :mode, :nlink, :username, :groupname, :bytesize, :mtime, :pathname, :blocks

    def initialize(path)
      file_stat = File.lstat(path)
      @basename = File.basename(path)
      @type = TYPE_MAP[file_stat.ftype]
      @mode = ModeFormatter.new(file_stat.mode).mode
      @nlink = file_stat.nlink.to_s
      @username = Etc.getpwuid(file_stat.uid).name
      @groupname = Etc.getgrgid(file_stat.gid).name
      @bytesize = get_bytesize(file_stat)
      @mtime = get_mtime(file_stat)
      @pathname = get_pathname(path)
      @blocks = file_stat.blocks
    end

    private

    def get_bytesize(file_stat)
      if file_stat.rdev != 0
        "0x#{file_stat.rdev.to_s(16)}"
      else
        file_stat.size.to_s
      end
    end

    def get_mtime(file_stat)
      if Time.now - file_stat.mtime >= (60 * 60 * 24 * (365 / 2.0)) || (Time.now - file_stat.mtime).negative?
        file_stat.mtime.strftime('%_m %_d  %Y')
      else
        file_stat.mtime.strftime('%_m %_d %H:%M')
      end
    end

    def get_pathname(path)
      if File.symlink?(path)
        "#{File.basename(path)} -> #{File.readlink(path)}"
      else
        File.basename(path)
      end
    end
  end
end
```

`mode_formatter.rb`

```ruby
# frozen_string_literal: true

module LS
  class ModeFormatter
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

    attr_reader :mode

    def initialize(mode)
      @mode_octal = mode.to_s(8)
      permissions_numeric = @mode_octal.slice(-3..-1).split(//)
      @permissions_symbolic = permissions_numeric.map { |n| MODE_MAP[n] }
      @mode = add_special_permissions.join
    end

    private

    def add_special_permissions
      case @mode_octal.slice(-4)
      when '1'
        add_sticky_bit
      when '2'
        add_sgid
      when '4'
        add_suid
      end
      @permissions_symbolic
    end

    def add_sticky_bit
      @permissions_symbolic[2] = if @permissions_symbolic[2].slice(2) == 'x'
                                   @permissions_symbolic[2].gsub(/.$/, 't')
                                 else
                                   @permissions_symbolic[2].gsub(/.$/, 'T')
                                 end
    end

    def add_sgid
      @permissions_symbolic[1] = if @permissions_symbolic[1].slice(2) == 'x'
                                   @permissions_symbolic[1].gsub(/.$/, 's')
                                 else
                                   @permissions_symbolic[1].gsub(/.$/, 'S')
                                 end
    end

    def add_suid
      @permissions_symbolic[0] = if @permissions_symbolic[0].slice(2) == 'x'
                                   @permissions_symbolic[0].gsub(/.$/, 's')
                                 else
                                   @permissions_symbolic[0].gsub(/.$/, 'S')
                                 end
    end
  end
end
```

`long_formatter.rb`

```ruby
# frozen_string_literal: true

module LS
  class LongFormatter
    def initialize(files)
      @files = files
      @max_length_map = max_length_map(@files)
      @block_total = @files.map(&:blocks).sum
    end

    def list_files
      puts "total #{@block_total}"
      @files.each { |file| print_long_format(file, @max_length_map) }
    end

    private

    def max_length_map(files)
      {
        nlink: files.map { |file| file.nlink.size }.max,
        username: files.map { |file| file.username.size }.max,
        groupname: files.map { |file| file.groupname.size }.max,
        bytesize: files.map { |file| file.bytesize.size }.max
      }
    end

    def print_long_format(file, max_length_map)
      print [
        "#{file.type}#{file.mode} ",
        "#{file.nlink.rjust(max_length_map[:nlink])} ",
        "#{file.username.ljust(max_length_map[:username])}  ",
        "#{file.groupname.ljust(max_length_map[:groupname])}  ",
        "#{file.bytesize.rjust(max_length_map[:bytesize])} ",
        "#{file.mtime} ",
        "#{file.pathname}\n"
      ].join
    end
  end
end
```

`short_formatter.rb`

```ruby
# frozen_string_literal: true

module LS
  class ShortFormatter
    COLUMN_NUMBER = 3

    def initialize(files)
      @files = files
    end

    def list_files
      element_number = @files.size.to_f
      max_length = @files.map { |file| file.basename.size }.max
      row_number = (element_number / COLUMN_NUMBER).ceil
      lines = Array.new(row_number) { [] }
      @files.each_with_index do |file, index|
        line_number = index % row_number
        lines[line_number].push(file.basename.ljust(max_length + 2))
      end
      lines.each { |line| puts line.join }
    end
  end
end
```
