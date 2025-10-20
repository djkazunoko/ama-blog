---
date: '2022-05-07'
draft: false
title: '【Rails/MySQL】bundle installでgem mysql2がインストールできない時の解決法'
tags: ["未分類"]
slug: '9'
---

## 1. 開発環境
- macOS Monterey：12.3.1
- Ruby：3.1.0
- Ruby on Rails：6.1.5
- Bundler：2.3.12
- MySQL：8.0.28(Homebrew3.4.10でインストール)

## 2. エラー時の状況
1. RailsでMySQLを使うために`rails new <app> -d mysql`を実行
1. bundle installの所でエラーが発生(mysql2のインストールに失敗していた)

## 3. エラーログ
```zsh
$ rails new <appname> -d mysql
省略
         run  bundle install
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies....
省略
Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

    current directory: /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/mysql2-0.5.3/ext/mysql2
/Users/kazunoko/.rbenv/versions/3.1.0/bin/ruby -I /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0 -r ./siteconf20220503-1924-xwtu0t.rb extconf.rb
checking for rb_absint_size()... yes
checking for rb_absint_singlebit_p()... yes
checking for rb_wait_for_single_fd()... yes
-----
Using mysql_config at /usr/local/bin/mysql_config
-----
checking for mysql.h... yes
checking for errmsg.h... yes
checking for SSL_MODE_DISABLED in mysql.h... yes
checking for SSL_MODE_PREFERRED in mysql.h... yes
checking for SSL_MODE_REQUIRED in mysql.h... yes
checking for SSL_MODE_VERIFY_CA in mysql.h... yes
checking for SSL_MODE_VERIFY_IDENTITY in mysql.h... yes
checking for MYSQL.net.vio in mysql.h... yes
checking for MYSQL.net.pvio in mysql.h... no
checking for MYSQL_ENABLE_CLEARTEXT_PLUGIN in mysql.h... yes
checking for SERVER_QUERY_NO_GOOD_INDEX_USED in mysql.h... yes
checking for SERVER_QUERY_NO_INDEX_USED in mysql.h... yes
checking for SERVER_QUERY_WAS_SLOW in mysql.h... yes
checking for MYSQL_OPTION_MULTI_STATEMENTS_ON in mysql.h... yes
checking for MYSQL_OPTION_MULTI_STATEMENTS_OFF in mysql.h... yes
checking for my_bool in mysql.h... no
-----
Don't know how to set rpath on your system, if MySQL libraries are not in path mysql2 may not load
-----
-----
Setting libpath to /usr/local/Cellar/mysql/8.0.28_1/lib
-----
creating Makefile

current directory: /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/mysql2-0.5.3/ext/mysql2
make DESTDIR\= clean

current directory: /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/mysql2-0.5.3/ext/mysql2
make DESTDIR\=
compiling client.c
client.c:178:24: warning: 'rbimpl_tainted_str_new_cstr' is deprecated: taintedness turned out to be a wrong idea. [-Wdeprecated-declarations]
  VALUE rb_sql_state = rb_tainted_str_new2(mysql_sqlstate(wrapper->client));
                       ^
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/intern/string.h:1742:29: note: expanded from macro 'rb_tainted_str_new2'
#define rb_tainted_str_new2 rb_tainted_str_new_cstr  /**< @old{rb_tainted_str_new_cstr} */
                            ^
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/intern/string.h:1615:7: note: expanded from macro 'rb_tainted_str_new_cstr'
      rbimpl_tainted_str_new_cstr :             \
      ^
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/intern/string.h:1401:1: note: 'rbimpl_tainted_str_new_cstr' has been explicitly marked deprecated here
RBIMPL_ATTR_DEPRECATED(("taintedness turned out to be a wrong idea."))
^
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/attr/deprecated.h:36:53: note: expanded from macro 'RBIMPL_ATTR_DEPRECATED'
# define RBIMPL_ATTR_DEPRECATED(msg) __attribute__((__deprecated__ msg))
                                                    ^
client.c:787:14: warning: incompatible pointer types passing 'VALUE (void *)' (aka 'unsigned long (void *)') to parameter of type 'VALUE (*)(VALUE)' (aka 'unsigned long (*)(unsigned long)')
[-Wincompatible-pointer-types]
  rb_rescue2(do_send_query, (VALUE)&args, disconnect_and_raise, self, rb_eException, (VALUE)0);
             ^~~~~~~~~~~~~
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/iterator.h:388:26: note: passing argument to parameter 'b_proc' here
VALUE rb_rescue2(VALUE (*b_proc)(VALUE), VALUE data1, VALUE (*r_proc)(VALUE, VALUE), VALUE data2, ...);
                         ^
client.c:795:16: warning: incompatible pointer types passing 'VALUE (void *)' (aka 'unsigned long (void *)') to parameter of type 'VALUE (*)(VALUE)' (aka 'unsigned long (*)(unsigned long)')
[-Wincompatible-pointer-types]
    rb_rescue2(do_query, (VALUE)&async_args, disconnect_and_raise, self, rb_eException, (VALUE)0);
               ^~~~~~~~
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/iterator.h:388:26: note: passing argument to parameter 'b_proc' here
VALUE rb_rescue2(VALUE (*b_proc)(VALUE), VALUE data1, VALUE (*r_proc)(VALUE, VALUE), VALUE data2, ...);
                         ^
3 warnings generated.
compiling infile.c
compiling mysql2_ext.c
compiling result.c
compiling statement.c
statement.c:49:24: warning: 'rbimpl_tainted_str_new_cstr' is deprecated: taintedness turned out to be a wrong idea. [-Wdeprecated-declarations]
  VALUE rb_sql_state = rb_tainted_str_new2(mysql_stmt_sqlstate(stmt_wrapper->stmt));
                       ^
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/intern/string.h:1742:29: note: expanded from macro 'rb_tainted_str_new2'
#define rb_tainted_str_new2 rb_tainted_str_new_cstr  /**< @old{rb_tainted_str_new_cstr} */
                            ^
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/intern/string.h:1615:7: note: expanded from macro 'rb_tainted_str_new_cstr'
      rbimpl_tainted_str_new_cstr :             \
      ^
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/intern/string.h:1401:1: note: 'rbimpl_tainted_str_new_cstr' has been explicitly marked deprecated here
RBIMPL_ATTR_DEPRECATED(("taintedness turned out to be a wrong idea."))
^
/Users/kazunoko/.rbenv/versions/3.1.0/include/ruby-3.1.0/ruby/internal/attr/deprecated.h:36:53: note: expanded from macro 'RBIMPL_ATTR_DEPRECATED'
# define RBIMPL_ATTR_DEPRECATED(msg) __attribute__((__deprecated__ msg))
                                                    ^
1 warning generated.
linking shared-object mysql2/mysql2.bundle
ld: library not found for -lssl
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [mysql2.bundle] Error 1

make failed, exit code 2

Gem files will remain installed in /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/mysql2-0.5.3 for inspection.
Results logged to /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/extensions/x86_64-darwin-21/3.1.0/mysql2-0.5.3/gem_make.out

  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/builder.rb:95:in `run'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/builder.rb:44:in `block in make'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/builder.rb:36:in `each'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/builder.rb:36:in `make'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/ext_conf_builder.rb:63:in `block in build'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/tempfile.rb:317:in `open'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/ext_conf_builder.rb:26:in `build'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/builder.rb:161:in `build_extension'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/builder.rb:195:in `block in build_extensions'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/builder.rb:192:in `each'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/ext/builder.rb:192:in `build_extensions'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/3.1.0/rubygems/installer.rb:847:in `build_extensions'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/rubygems_gem_installer.rb:71:in `build_extensions'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/rubygems_gem_installer.rb:28:in `install'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/source/rubygems.rb:204:in `install'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/installer/gem_installer.rb:54:in `install'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/installer/gem_installer.rb:16:in `install_from_spec'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/installer/parallel_installer.rb:186:in `do_install'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/installer/parallel_installer.rb:177:in `block in worker_pool'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/worker.rb:62:in `apply_func'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/worker.rb:57:in `block in process_queue'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/worker.rb:54:in `loop'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/worker.rb:54:in `process_queue'
  /Users/kazunoko/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.12/lib/bundler/worker.rb:91:in `block (2 levels) in create_threads'

An error occurred while installing mysql2 (0.5.3), and Bundler cannot continue.

In Gemfile:
  mysql2
         run  bundle binstubs bundler
Could not find gem 'mysql2 (~> 0.5)' in locally installed gems.
       rails  webpacker:install
Could not find gem 'mysql2 (~> 0.5)' in locally installed gems.
Run `bundle install` to install missing gems.
```

## 4. エラーの原因と思しき箇所

```
ld: library not found for -lssl
```
- opensslのパスが見つからないと言っている
    - gemのmysql2はopensslが必須であり、bundleのビルド時にopensslのパスを設定する必要がある
- macに標準でopenssl入ってなかったっけ？
    - mac OS High SierraからデフォルトのOpenSSLが[LibreSSL](https://www.libressl.org/)に変わった

## 5. 解決法
`bundle config`でbundlerの設定ファイルにopensslのパスを設定して、`bundle install`で再度mysql2をインストールする

### 5-1. Homebrewでopensslをインストール
```
$ brew install openssl
```

### 5-2. bundle configコマンドでopensslのパスを指定
[mysql2公式レポジトリのissue](https://github.com/brianmario/mysql2/issues/1005#issuecomment-433219580)から以下のコマンドを実行
```
$ bundle config --global build.mysql2 --with-opt-dir="$(brew --prefix openssl)"
```
- [brew --prefix(公式ドキュメント)](https://docs.brew.sh/Manpage#--prefix---unbrewed---installed-formula-)
- [bundle config(公式ドキュメント)](https://bundler.io/man/bundle-config.1.html)

### 5-3. bundle installしてmysql2をインストール
```
$ bundle install
```

無事にmysql2がインストールされたら解決です。

___

【メモ】

- OpenSSL
    - SSL/TSL(暗号通信プロトコル)機能を実装したオープンソースライブラリ
    - mac OS High SierraからデフォルトのOpenSSLが[LibreSSL](https://www.libressl.org/)に変わった
- bundle configコマンド
    - bundlerの設定ファイルを作成
    - bundle config --globalで`~/.bundle/config`に作成
    - bundle config --localで`app/.bundle/config`に作成
    - 設定の優先順位
        - 1.ローカルのアプリケーション(app/.bundle/config)
        - 2.環境変数
        - 3.ユーザーのホームディレクトリ(~/.bundle/config)
___

【参考】

- エラー解決時の参考
    - [bundle installでmysql2がエラーになる件](https://qiita.com/SAYJOY/items/dd7c8fc7a3647e7ff969)
    - [gem mysql2がbundle installできない時の解決法（ERROR: Error installing mysql2:）](https://qiita.com/ryouzi/items/b89965b76cef546e3046)
    - [RailsプロジェクトでMySQLがbundle installできなかった](https://qiita.com/akito19/items/e1dc54f907987e688cc0)
    - [mysql2 のインストール時に ld: library not found for -lssl  が発生した場合の対処 Mac版](https://qiita.com/ffggss/items/2a6e965c8c3133a92eb9)
    - [Gem native extension build fails after upgrade to MacOS 10.4 Mojave #1005](https://github.com/brianmario/mysql2/issues/1005)
    - [Error when trying to install app with mysql2 gem](https://stackoverflow.com/questions/30834421/error-when-trying-to-install-app-with-mysql2-gem/39628463#39628463)
- OpenSSLについて
    - [MacでOpenSSLを使う](http://tec-shi.com/mac/596/)
    - [macOS High Sierra（OSX）のOpenSSLをデフォルトのLibreSSLからOpenSSLに変更する](https://qiita.com/moroi/items/53d60d1d6885795a0f6f)
- Bundlerについて
    - [Bundlerでビルドオプションを指定する](https://qiita.com/thunders/items/101c6b329830fb1fb27d)
- ldコマンドについて
    - [MacでPCLビルドエラーへの対処[ld: library not found for]](https://qiita.com/ysuzuki19/items/044a85efb03f3c290a95)
    - [IBM_ldコマンド](https://www.ibm.com/docs/ja/aix/7.1?topic=l-ld-command)
    - [ldコマンド manpage](https://linuxjm.osdn.jp/html/GNU_binutils/man1/ld.1.html)
- /usr/local/optについて
    - [Why is there a /usr/local/opt directory created by Homebrew and should I use it?](https://stackoverflow.com/questions/35337601/why-is-there-a-usr-local-opt-directory-created-by-homebrew-and-should-i-use-it)
    - [How to Build Software Outside Homebrew with Homebrew keg_only Dependencies](https://docs.brew.sh/How-to-Build-Software-Outside-Homebrew-with-Homebrew-keg-only-Dependencies)
