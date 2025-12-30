---
date: '2022-09-16T12:26:45+09:00'
draft: false
title: '【Rails】kaminariでページング処理を実装する'
tags: ["未分類"]
slug: '59'
---

この記事では、kaminari gemを使ってページング処理(ページネーション)を実装する方法を紹介します。

## 1. 実行環境
- macOS：12.5.1
- Ruby：3.1.2
- Rails：6.1.7
- kaminari：1.2.2

## 2. 手順

### 2-1. kaminari gemのインストール

Gemfileに以下を追記します。

```ruby
gem 'kaminari'
```

以下のコマンドを実行してkaminari gemをインストールします。
```
bundle install
```

### 2-2. ページネーションを実装

コントローラーに以下を追記します。
```ruby
def index
  @books = Book.order(:id).page(params[:page])
end
```
- デフォルトで25件/ページ
- Book.page(params[:page])だと並び順が変わることがあるため、必ず`order`をつける
  - [Active Record クエリインターフェイス - 4 並び順](https://railsguides.jp/active_record_querying.html#%E4%B8%A6%E3%81%B3%E9%A0%86)
  - [order - ActiveRecord::QueryMethods](https://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html#method-i-order)



ビューファイルに以下を追記します。

```ruby
<%= paginate @books %>
```

以上でページネーションの実装は完了です。

### 2-3. 日本語化

kaminariが表示する"First"や"Last"のような表示を日本語化します。

`config/locales/ja.yml`に以下を追記します。
```yaml
ja:
  views:
    pagination:
      first: '最初'
      last: '最後'
      previous: '前'
      next: '次'
      truncate: '...'
```
- [I18n and Labels - GitHub - amatsuda/kaminari](https://github.com/amatsuda/kaminari#i18n-and-labels)

---

【参考】

- [GitHub - amatsuda/kaminari](https://github.com/amatsuda/kaminari)
- [kaminari徹底入門 #Ruby - Qiita](https://qiita.com/nysalor/items/77b9d6bc5baa41ea01f3)
- [【Rails】 kaminariの使い方を理解してページネーションを実装しよう | Pikawaka](https://pikawaka.com/rails/kaminari)
