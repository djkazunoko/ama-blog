---
date: '2022-08-19T21:35:50+09:00'
draft: false
title: '【Ruby】Sinatraでメモアプリを作る(DB編)'
tags: ["未分類"]
slug: '55'
---

この記事では、RubyのWebアプリケーションフレームワークであるSinatraを使って、シンプルなメモアプリを作成します。

データ保存先のDBにはPostgreSQLを使用します。

今回は、以下の記事で作成したメモアプリのデータ保存先をPostgreSQLに変更します。

[【Ruby】Sinatraでメモアプリを作る(JSONファイル編) | あまブログ](https://ama-blog.com/51/)

PostgreSQLのインストール方法は以下を参照ください。

[【macOS】PostgreSQLの基本操作 | あまブログ](https://ama-blog.com/22/)

## 1. 実行環境

- macOS：12.5
- Ruby：3.1.0
- Bundler：2.3.12
- PostgreSQL：14.3
- sinatra：2.2.2
- webrick：1.7.0
- sinatra-contrib：2.2.2
- pg：1.4.3

## 2. 手順

### 2-1. pgのインストール

RubyからPostgreSQLを使うために必要なgem、[pg](https://github.com/ged/ruby-pg)をインストールします。

`Gemfile`に以下を追加します。

```ruby
+ gem "pg"
  gem "sinatra"
```

以下のコマンドを実行してgemをインストールします。
```
$ bundle install
```

### 2-2. DB接続とテーブル定義

`memo.rb`でpgを使えるようにします。

```ruby
- require 'json'
+ require 'pg' 
```

`memo.rb`の以下の部分を削除します。

```ruby
FILE_PATH = 'public/memos.json'

def get_memos(file_path)
  File.open(file_path) { |f| JSON.parse(f.read) }
end

def set_memos(file_path, memos)
  File.open(file_path, 'w') { |f| JSON.dump(memos, f) }
end
```

`memo.rb`に以下を追加します。

```ruby
def conn
  @conn ||= PG.connect(dbname: 'postgres')
end

configure do
  result = conn.exec("SELECT * FROM information_schema.tables WHERE table_name = 'memos'")
  conn.exec('CREATE TABLE memos (id serial, title varchar(255), content text)') if result.values.empty?
end
```
- [PG.connect](https://deveiate.org/code/pg/PG.html#method-c-connect)でDBに接続し、インスタンス変数`@conn`に接続情報をキャッシュ
- [configuration](https://github.com/sinatra/sinatra#configuration)にはアプリ起動時に一度だけ行う処理を記述
- PostgreSQLのシステムテーブル[tables](https://www.postgresql.org/docs/current/infoschema-tables.html)からmemosテーブルの存在を確認
- `if result.values.empty?`でmemosテーブルがなければテーブルを作成

### 2-3. メモ一覧の表示

`memo.rb`に以下を追加します。

```ruby
# configure do
# ~

def read_memos
  conn.exec('SELECT * FROM memos')
end

# get '/' do
```
- [PG::Connection.exec](https://deveiate.org/code/pg/PG/Connection.html#method-i-exec)

`get '/memos'`のルーティングブロックを以下に変更します。

```ruby
get '/memos' do
  @memos = read_memos
  erb :index
end
```

`views/index.erb`のメモ一覧表示部分を以下に変更します。
```ruby
<ul>
  <% @memos.each do |memo| %>
    <li>
      <a href="/memos/<%= memo["id"] %>"><%= CGI.escapeHTML(memo["title"]) %></a>
    </li>
  <% end %>
</ul>
```

### 2-4. 特定のメモの表示

`memo.rb`に以下を追加します。

```ruby
# def read_memos
# ~

def read_memo(id)
  result = conn.exec_params('SELECT * FROM memos WHERE id = $1;', [id])
  result.tuple_values(0)
end

# get '/' do
```
- [PG::Connection.exec_params](https://deveiate.org/code/pg/PG/Connection.html#method-i-exec_params)
  - SQL文でプレースホルダを使う場合に使用

`get '/memos/:id'`のルーティングブロックを以下に変更します。
```ruby
get '/memos/:id' do
  memo = read_memo(params[:id])
  @title = memo[1]
  @content = memo[2]
  erb :show
end
```

### 2-5. メモの作成

`memo.rb`に以下を追加します。

```ruby
# def read_memo(id)
# ~

def post_memo(title, content)
  conn.exec_params('INSERT INTO memos(title, content) VALUES ($1, $2);', [title, content])
end

# get '/' do
```

`post '/memos'`のルーティングブロックを以下に変更します。
```ruby
post '/memos' do
  title = params[:title]
  content = params[:content]
  post_memo(title, content)
  redirect '/memos'
end
```

### 2-6. メモの編集

`memo.rb`に以下を追加します。

```ruby
# def post_memo(title, content)
# ~

def edit_memo(title, content, id)
  conn.exec_params('UPDATE memos SET title = $1, content = $2 WHERE id = $3;', [title, content, id])
end

# get '/' do
```

`get '/memos/:id/edit'`と`patch '/memos/:id'`のルーティングブロックを以下に変更します。
```ruby
get '/memos/:id/edit' do
  memo = read_memo(params[:id])
  @title = memo[1]
  @content = memo[2]
  erb :edit
end

patch '/memos/:id' do
  title = params[:title]
  content = params[:content]
  edit_memo(title, content, params[:id])
  redirect "/memos/#{params[:id]}"
end
```

### 2-7. メモの削除

`memo.rb`に以下を追加します。

```ruby
# def edit_memo(title, content, id)
# ~

def delete_memo(id)
  conn.exec_params('DELETE FROM memos WHERE id = $1;', [id])
end

# get '/' do
```

`delete '/memos/:id'`のルーティングブロックを以下に変更します。
```ruby
delete '/memos/:id' do
  delete_memo(params[:id])
  redirect '/memos'
end
```

以上で終了です。

---

【参考】

- [ged/ruby-pg: A PostgreSQL client library for Ruby](https://github.com/ged/ruby-pg)
- [PG: The Ruby PostgreSQL Driver](https://deveiate.org/code/pg/)
  - [PG::Result](https://deveiate.org/code/pg/PG/Result.html)
  - [PG::Connection](https://deveiate.org/code/pg/PG/Connection.html)
- https://www.ownway.info/Ruby/pg/about
- [安全なウェブサイトの作り方 - 1.1 SQLインジェクション | 情報セキュリティ | IPA 独立行政法人 情報処理推進機構](https://www.ipa.go.jp/security/vuln/websecurity/sql.html)
- [SQLインジェクション - Wikipedia](https://ja.wikipedia.org/wiki/SQL%E3%82%A4%E3%83%B3%E3%82%B8%E3%82%A7%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3)
