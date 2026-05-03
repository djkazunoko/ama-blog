---
date: '2023-01-23T15:28:17+09:00'
draft: false
title: '【RUBY_FUNCTION_NAME_STRING】rbenv installでBUILD FAILEDが出てRuby 3.1.1がインストールできない時の対処法'
tags: ["未分類"]
slug: '88'
---

## 1. 環境

- MacBook Pro (13-inch, 2020)
- macOS Ventura 13.1
- Homebrew：3.6.20(※)
- xcode-select：2396(※)

※ エラー解決後のバージョン

## 2. 発生した問題

rbenvでRuby 3.1.1をインストールしようとしたら以下のエラーが発生。

```zsh
$ rbenv install 3.1.1
Downloading openssl-3.0.5.tar.gz...
-> https://dqw8nmjcqpjn7.cloudfront.net/aa7d8d9bef71ad6525c55ba11e5f4397889ce49c2c9349dcea6d3e4f0b024a7a
Installing openssl-3.0.5...
Installed openssl-3.0.5 to /Users/kazunoko/.rbenv/versions/3.1.1

Downloading ruby-3.1.1.tar.gz...
-> https://cache.ruby-lang.org/pub/ruby/3.1/ruby-3.1.1.tar.gz
Installing ruby-3.1.1...
ruby-build: using readline from homebrew

BUILD FAILED (macOS 13.1 using ruby-build 20220726)

Inspect or clean up the working tree at /var/folders/m9/r1bz5qns0nj2ysmyrqwpn3j00000gn/T/ruby-build.20230123081732.42760.Fk3kTB
Results logged to /var/folders/m9/r1bz5qns0nj2ysmyrqwpn3j00000gn/T/ruby-build.20230123081732.42760.log

Last 10 log lines:
                                                                       ^
In file included from compile.c:40:
./vm_callinfo.h:216:16: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
    if (debug) rp(ci);
               ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
2 errors generated.
make: *** [compile.o] Error 1
```

出力された`/var/folders/m9/r1bz5qns0nj2ysmyrqwpn3j00000gn/T/ruby-build.20230123081732.42760.log`を確認(最後の方だけ抜粋)。
```zsh
In file included from debug.c:27:
./vm_callinfo.h:175:9: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
        rp(ci);
        ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
In file included from debug.c:27:
./vm_callinfo.h:216:16: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
    if (debug) rp(ci);
               ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
2 errors generated.
make: *** [debug.o] Error 1
make: *** Waiting for unfinished jobs....
In file included from compile.c:40:
./vm_callinfo.h:175:9: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
        rp(ci);
        ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
In file included from compile.c:40:
./vm_callinfo.h:216:16: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
    if (debug) rp(ci);
               ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
2 errors generated.
make: *** [compile.o] Error 1
```

## 3. 解決方法

ruby-buildのwikiの[Suggested build environment](https://github.com/rbenv/ruby-build/wiki#suggested-build-environment)に従って以下のコマンドを実行。

```zsh
$ brew install openssl@3 readline libyaml gmp
$ export RUBY_CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl@3)"
```
10分以上かかって色々インストールが完了し、環境変数の設定も完了。

再度Ruby 3.1.1をインストールしてみるも、以下のエラーが発生。

```zsh
$ rbenv install 3.1.1
To follow progress, use 'tail -f /var/folders/m9/r1bz5qns0nj2ysmyrqwpn3j00000gn/T/ruby-build.20230123093647.99361.log' or pass --verbose
Downloading ruby-3.1.1.tar.gz...
-> https://cache.ruby-lang.org/pub/ruby/3.1/ruby-3.1.1.tar.gz
Installing ruby-3.1.1...
ruby-build: using readline from homebrew
ruby-build: using gmp from homebrew

BUILD FAILED (macOS 13.1 using ruby-build 20221225)

Inspect or clean up the working tree at /var/folders/m9/r1bz5qns0nj2ysmyrqwpn3j00000gn/T/ruby-build.20230123093647.99361.VH2vnQ
Results logged to /var/folders/m9/r1bz5qns0nj2ysmyrqwpn3j00000gn/T/ruby-build.20230123093647.99361.log

Last 10 log lines:
                                                                       ^
In file included from compile.c:40:
./vm_callinfo.h:216:16: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
    if (debug) rp(ci);
               ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
2 errors generated.
make: *** [compile.o] Error 1
```

出力された`/var/folders/m9/r1bz5qns0nj2ysmyrqwpn3j00000gn/T/ruby-build.20230123093647.99361.log`を確認(最後の方だけ抜粋)。
```zsh
In file included from debug.c:27:
./vm_callinfo.h:175:9: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
        rp(ci);
        ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
In file included from debug.c:27:
./vm_callinfo.h:216:16: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
    if (debug) rp(ci);
               ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
2 errors generated.
compiling dln_find.c
make: *** [debug.o] Error 1
make: *** Waiting for unfinished jobs....
In file included from compile.c:40:
./vm_callinfo.h:175:9: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
        rp(ci);
        ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
In file included from compile.c:40:
./vm_callinfo.h:216:16: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
    if (debug) rp(ci);
               ^
./internal.h:94:72: note: expanded from macro 'rp'
#define rp(obj) rb_obj_info_dump_loc((VALUE)(obj), __FILE__, __LINE__, RUBY_FUNCTION_NAME_STRING)
                                                                       ^
2 errors generated.
make: *** [compile.o] Error 1
```

ログにある`RUBY_FUNCTION_NAME_STRING`で調べてみて、ruby-buildのissuesの[Ruby installation fails in macOS Catalina #1409](https://github.com/rbenv/ruby-build/issues/1409)がヒット。

[このコメント](https://github.com/rbenv/ruby-build/issues/1409#issuecomment-752223239)にある以下のコマンドを実行。

```zsh
$ sudo rm -rf /Library/Developer/CommandLineTools          
$ sudo xcode-select --install
```

ダイアログの指示に従いCommand Line Tools for Xcodeをインストール。

再度Ruby 3.1.1をインストール。
```zsh
$ rbenv install 3.1.1
To follow progress, use 'tail -f /var/folders/m9/r1bz5qns0nj2ysmyrqwpn3j00000gn/T/ruby-build.20230123104923.9665.log' or pass --verbose
Downloading ruby-3.1.1.tar.gz...
-> https://cache.ruby-lang.org/pub/ruby/3.1/ruby-3.1.1.tar.gz
Installing ruby-3.1.1...
ruby-build: using readline from homebrew
ruby-build: using gmp from homebrew
Installed ruby-3.1.1 to /Users/kazunoko/.rbenv/versions/3.1.1
```

無事インストールできました！

---

【参考】

- [rbenv install で BUILD FAILED - インゲージ開発者ブログ](https://blog.ingage.jp/entry/2022/11/22/093000)
- [`require': cannot load such file -- openssl (LoadError) というエラーが出てRuby 2.7.6がインストールできない場合の対処法 - Qiita](https://qiita.com/jnchito/items/eb9cf24cbd29e1471e45)
- [M1 Macbook Proにrbenv経由でRubyをインストールしようとしたらエラーになってハマった - 行動すれば次の現実](https://blog.furu07yu.com/entry/rbenv-ruby-install-on-m1)
- [macでRubyをビルドしたときにRUBY_FUNCTION_NAME_STRINGのerrorが出た場合 | Blog](https://blog.nksm.in.net/2022-03-01-mac%E3%81%A7Ruby%E3%82%92%E3%83%93%E3%83%AB%E3%83%89%E3%81%97%E3%81%9F%E3%81%A8%E3%81%8D%E3%81%ABRUBY_FUNCTION_NAME_STRING%E3%81%AEerror%E3%81%8C%E5%87%BA%E3%81%9F%E5%A0%B4%E5%90%88/)
- [brew upgrade でのエラー対処からCommand Line Toolsについてまとめてみる｜TechRacho by BPS株式会社](https://techracho.bpsinc.jp/wingdoor/2021_04_09/104821)
- [MacのhomebrewでOpenSSLがビルドエラーになる場合の対処方法｜TechRacho by BPS株式会社](https://techracho.bpsinc.jp/hachi8833/2016_09_09/25454)
