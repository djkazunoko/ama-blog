---
date: '2022-09-15T22:44:50+09:00'
draft: false
title: '【Rails】i18nで日本語化する方法'
tags: ["未分類"]
slug: '58'
---

この記事では、Railsに同梱されているi18n gemを使ってアプリケーションを多言語化する方法を紹介します。

## 1. 実行環境
- macOS：12.5.1
- Ruby：3.1.2
- Rails：6.1.7
- i18n：1.12.0

## 2. 手順

### 2-1. i18nモジュールの設定

使用する言語のリストとデフォルトで使用する言語を設定します。

`config/initializers/locale.rb`に以下を追記します。

```ruby
# 英語と日本語の利用を許可する
I18n.available_locales = [:en, :ja]

# デフォルトの言語を日本語にする
I18n.default_locale = :ja
```

### 2-2. ロケールファイルのダウンロード

使用する言語のデフォルトのロケールファイルを[svenfuchs/rails-i18n](https://github.com/svenfuchs/rails-i18n/)からダウンロードします。

**日本語**
```
$ curl -s https://raw.githubusercontent.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml -o config/locales/ja.yml
```

**英語**
```
$ curl -s https://raw.githubusercontent.com/svenfuchs/rails-i18n/master/rails/locale/en.yml -o config/locales/en.yml
```

上記のコマンドにより`config/locales/ja.yml`と`config/locales/en.yml`が作成されます。

以降はこれらのファイルに翻訳を追加していきます。

### 2-3. Active Recordモデルで翻訳を行なう

`config/locales/ja.yml`に以下を追記します。
```yaml
ja:
  activerecord:
    models:
      book: 本
    attributes:
      book:
        title: タイトル
        memo: メモ
```

`model_name.human`と`human_attribute_name`を使ってモデル名とカラム名の訳語を表示します。

```ruby
Book.model_name.human #=> 本
Book.human_attribute_name(:title) #=> タイトル
Book.human_attribute_name(:memo) #=> メモ
```

- [Rails 国際化（i18n）API - 4.7.2 Active Modelのメソッド](https://railsguides.jp/i18n.html#active-model%E3%81%AE%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89)

`config/locales/en.yml`に英訳を追加します。
```yaml
en:
  activerecord:
    models:
      book: Book
    attributes:
      book:
        title: Title
        memo: Memo
```

### 2-4. その他の翻訳

`config/locales/ja.yml`に以下を追記します。
```yaml
ja:
  views:
    common:
      show: 詳細
      title_new: "%{name}の新規作成"
  controllers:
    common:
      notice_create: "%{name}が作成されました。"
```

`I18n.translate`メソッドを使って訳文を参照します。

`I18n.translate()`は`I18n.t()`または`t()`のように省略可能です。

```ruby
t('views.common.show') #=> 詳細
t('views.common.title_new', name: Book.model_name.human) #=> 本の新規作成
t('controllers.common.notice_destroy', name: Book.model_name.human) #=> 本が作成されました。
```

- [Rails 国際化（i18n）API - 3.3 訳文に変数を渡す](https://railsguides.jp/i18n.html#%E8%A8%B3%E6%96%87%E3%81%AB%E5%A4%89%E6%95%B0%E3%82%92%E6%B8%A1%E3%81%99)

`config/locales/en.yml`に英訳を追加します。
```yaml
en:
  views:
    common:
      show: Show
      title_new: "New %{name}"
  controllers:
    common:
      notice_create: "%{name} was successfully created."
```

### 2-5. URLクエリパラメータでロケールを切り替え

- [Rails 国際化（i18n）API - 2.2 リクエスト間でロケールを管理する](https://railsguides.jp/i18n.html#%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E9%96%93%E3%81%A7%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB%E3%82%92%E7%AE%A1%E7%90%86%E3%81%99%E3%82%8B)

`app/controllers/application_controller.rb`に以下を追記します。
```ruby
class ApplicationController < ActionController::Base
  around_action :switch_locale

  def switch_locale(&action)
    locale = params[:locale] || I18n.default_locale
    I18n.with_locale(locale, &action)
  end

  def default_url_options
    { locale: I18n.locale }
  end
end
```

上記の設定により、`http://localhost:3000/books?locale=ja`で日本語に、`http://localhost:3000/books?locale=en`で英語に切り替えることができます。

`default_url_options`により全てのURLのクエリパラメータに自動的にロケール情報が含まれるようになります。

- [Rails 国際化（i18n）API - 2.2.2 URL paramsを元にロケールを設定する](https://railsguides.jp/i18n.html#url-params%E3%82%92%E5%85%83%E3%81%AB%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)
- [Action Controller の概要 - 4.4 default_url_options](https://railsguides.jp/action_controller_overview.html#default-url-options)

---

【参考】

- [Rails 国際化（I18n）API - Railsガイド](https://railsguides.jp/i18n.html)
- [i18nについて | 酒と涙とRubyとRailsと](https://morizyun.github.io/ruby/rails-function-i18n-internationalization.html)
- [【Rails】 I18n入門書~日本語化対応の手順と応用的な使い方 | Pikawaka](https://pikawaka.com/rails/i18n)
