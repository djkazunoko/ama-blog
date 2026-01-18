---
date: '2022-09-22T13:56:54+09:00'
draft: false
title: '【Rails】deviseでユーザー認証機能を実装する'
tags: ["未分類"]
slug: '62'
---

この記事では、devise gemを使ってRailsアプリケーションにユーザー認証機能を実装する方法を紹介します。

## 1. 実行環境
- macOS：12.5.1
- Ruby：3.1.2
- Rails：6.1.7
- devise：4.8.1
- devise-i18n：1.10.2


## 2. 手順

### 2-1. サンプルアプリの作成

以下のコマンドを実行して、サンプルアプリを作成します。

```
$ rails new sample_app
$ cd sample_app
$ rails generate scaffold book title:string memo:text
$ rails db:migrate
```

### 2-2. deviseのセットアップ

`Gemfile`に以下を追記して、`bundle install`を実行します。

```ruby
gem 'devise'
```

アプリケーションのルートディレクトリで以下のコマンドを実行して、deviseをセットアップします。

```
$ rails generate devise:install
```

`config/environments/development.rb`に以下を追記して、デフォルトの url オプションを定義します。

```ruby
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

### 2-3. ルートルーティングの設定

`config/routes.rb`に以下を追記して、ルートルーティングを設定します。

```ruby
Rails.application.routes.draw do
  root to: 'books#index'
end
```

deviseはサインインやパスワード変更などを行った後のデフォルトのリダイレクト先にルート(`/`)を使用します。

上記の設定により、リダイレクト先が`/`から`/books`に変更されます。

### 2-4. フラッシュメッセージの表示

`app/views/layouts/application.html.erb`を以下のように編集します。

```html
<%# 省略 %>
<body>
  <% if notice.present? %>
    <p id="notice"><%= notice %></p>
  <% end %>
  <% if alert.present? %>
    <p id="alert"><%= alert %></p>
  <% end %>
  <%= yield %>
</body>
```

`app/views/books/index.html.erb`と`app/views/books/show.html.erb`の以下の行を削除します。


```html
<p id="notice"><%= notice %></p>
```

### 2-5. Userモデルの作成

以下のコマンドを実行して、Userモデルを作成します。

```
$ rails generate devise user
$ rails db:migrate
```

上記のコマンドにより、Devise がアカウントの作成,ログイン,ログアウトなどに関するすべてのコードやルーティングを生成します。

- [$ rails generate devise MODEL](https://github.com/heartcombo/devise#controller-filters-and-helpers:~:text=In%20the%20following,the%20Devise%20controller.)

### 2-6. ログインしていない時に登録した内容を確認できないようにする

`app/controllers/application_controller.rb`に以下を追記します。

```ruby
class ApplicationController < ActionController::Base
  before_action :authenticate_user!
end
```

上記の設定により、ログインしていない時の各ページへのアクセスは全てログイン画面に遷移するようになります。

- [Controller filters and helpers - devise](https://github.com/heartcombo/devise#controller-filters-and-helpers)
- [Devise::Controllers::Helpers](https://github.com/heartcombo/devise/blob/main/lib/devise/controllers/helpers.rb)

`app/views/layouts/application.html.erb`にアカウント編集とログアウトのリンクを追加します。

```html
<body>
  <% if user_signed_in? %>
    <div class="menu-container">
      <div class="title">Logged in as <%= current_user.email %></div>
      <ul>
        <li>
          <%= link_to 'Edit profile', edit_user_registration_path %>
        </li>
        <li>
          <%= link_to 'Logout', destroy_user_session_path, method: :delete %>
        </li>
      </ul>
    </div>
  <% end %>
  <% if notice.present? %>
    <p id="notice"><%= notice %></p>
  <% end %>
  <% if alert.present? %>
    <p id="alert"><%= alert %></p>
  <% end %>
  <%= yield %>
</body>
```

### 2-7. ユーザー一覧画面とユーザー詳細画面の作成

以下のコマンドを実行して、Userモデルに`name`と`self_introduction`カラムを追加します。

```
$ rails generate migration add_name_and_self_introduction_to_users name:string self_introduction:text
$ rails db:migrate
```

`config/routes.rb`に以下のルーティングを追記します。

```ruby
resources :users, only: %i(index show)
```

以下のコマンドを実行して、`users#index`, `users#show`のルーティングが追加されたことを確認します。

```
$ rails routes
# 省略
users GET    /users(.:format)   users#index
user GET    /users/:id(.:format)   users#show
```

以下のコマンドを実行して、usersコントローラーを作成します。

```
$ rails generate controller users index show --skip-routes --no-helper --no-assets --no-test-framework
```

生成された`app/controllers/users_controller.rb`を以下のように編集します。

```ruby
class UsersController < ApplicationController
  def index
    @users = User.all
  end

  def show
    @user = User.find(params[:id])
  end
end
```

次に、ユーザ一ー覧画面とユーザー詳細画面を編集します。

`app/views/users/index.html.erb`を以下のように編集します。

```html
<h1>ユーザー</h1>

<table>
  <thead>
    <tr>
      <th>Eメール</th>
      <th>氏名</th>
      <th></th>
    </tr>
  </thead>

  <tbody>
    <% @users.each do |user| %>
      <tr>
        <td><%= user.email %></td>
        <td><%= user.name %></td>
        <td><%= link_to '詳細', user %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

`app/views/users/show.html.erb`を以下のように編集します。

```html
<h1>ユーザーの詳細</h1>

<p>
  <strong>Eメール:</strong>
  <%= @user.email %>
</p>

<p>
  <strong>氏名:</strong>
  <%= @user.name %>
</p>

<p>
  <strong>自己紹介文:</strong>
  <%= @user.self_introduction %>
</p>

<% if current_user == @user %>
  <%= link_to '編集', edit_user_registration_path %> |
<% end %>
<%= link_to '戻る', users_path %>
```

`app/views/layouts/application.html.erb`にユーザー一覧画面へのリンクを追加します。

```html
    <%# 省略%>
    <% if user_signed_in? %>
      <div class="menu-container">
        <div class="title">メニュー</div>
        <ul>
          <li>
            <%= link_to '本', books_path %>
          </li>
          <li>
            <%= link_to 'ユーザー', users_path %>
          </li>
        </ul>
        <div class="title">Logged in as <%= current_user.email %></div>
        <ul>
          <li>
            <%= link_to 'Edit profile', edit_user_registration_path %>
          </li>
          <li>
            <%= link_to 'Logout', destroy_user_session_path, method: :delete %>
          </li>
        </ul>
      </div>
    <% end %>
    <%# 省略%>
```

### 2-8. サインアップ画面とアカウント編集画面の編集

Gemfileに以下を追記し、bundle installを実行して[devise-i18n](https://rubygems.org/gems/devise-i18n/versions/1.7.0?locale=ja)をインストールします。

```ruby
gem 'devise-i18n'
```

以下のコマンドを実行して、devise:i18n:viewsを生成します。

```
$ rails g devise:i18n:views -v registrations
```
上記により、[devise-i18n/app/views/devise/registrations/edit.html.erb](https://github.com/tigrish/devise-i18n/tree/master/app/views/devise/registrations)の`edit.html.erb`と`new.html.erb`が生成されます。

- `$ rails g devise:views -v registrations` の場合
  - [Configuring views - devise](https://github.com/heartcombo/devise#configuring-views)
  - [devise/app/views/devise/registrations](https://github.com/heartcombo/devise/tree/main/app/views/devise/registrations)の`edit.html.erb`と`new.html.erb`が生成される


`app/views/devise/registrations/_profile_fields.html.erb`を作成します。
```html
<div class="field">
  <%= f.label :name %>
  <%= f.text_field :name %>
</div>

<div class="field">
  <%= f.label :self_introduction %>
  <%= f.text_area :self_introduction %>
</div>
```

`app/views/devise/registrations/new.html.erb`を以下のように編集します。
```html
  <%# 省略 %>
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>

  <%# この1行を追加 %>
  <%= render 'devise/registrations/profile_fields', f: f %>

  <div class="field">
    <%= f.label :password %>
    <% if @minimum_password_length %>
    <em><%= t('devise.shared.minimum_password_length', count: @minimum_password_length) %></em>
    <% end %><br />
    <%= f.password_field :password, autocomplete: "new-password" %>
  </div>
  <%# 省略 %>
```

`app/views/devise/registrations/edit.html.erb`を以下のように編集します。
```html
  <%# 省略 %>
  <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
    <div><%= t('.currently_waiting_confirmation_for_email', email: resource.unconfirmed_email) %></div>
  <% end %>

  <%# この1行を追加 %>
  <%= render 'devise/registrations/profile_fields', f: f %>

  <div class="field">
    <%= f.label :password %> <i>(<%= t('.leave_blank_if_you_don_t_want_to_change_it') %>)</i><br />
    <%= f.password_field :password, autocomplete: "new-password" %>
    <% if @minimum_password_length %>
      <br />
      <em><%= t('devise.shared.minimum_password_length', count: @minimum_password_length) %></em>
    <% end %>
  </div>
  <%# 省略 %>
```

次にストロングパラメータ(Strong Parameters)を追加します。

`app/controllers/application_controller.rb`に以下を追記します。

```ruby
class ApplicationController < ActionController::Base
  before_action :authenticate_user!
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    keys = %i[name self_introduction]
    devise_parameter_sanitizer.permit(:sign_up, keys: keys)
    devise_parameter_sanitizer.permit(:account_update, keys: keys)
  end
end
```
上記により、サインアップ時 (`Devise::RegistrationsController#create`)とアカウント編集時(`Devise::RegistrationsController#update`)のみ`name`カラムと`self_introduction`カラムの更新を許可します。

- [Strong Parameters - devise](https://github.com/heartcombo/devise#strong-parameters)
- `devise_parameter_sanitizer`
  - [devise/lib/devise/controllers/helpers.rb](https://github.com/heartcombo/devise/blob/main/lib/devise/controllers/helpers.rb#L158-L160)
  - [devise/lib/devise/parameter_sanitizer.rb](https://github.com/heartcombo/devise/blob/6d32d2447cc0f3739d9732246b5a5bde98d9e032/lib/devise/parameter_sanitizer.rb)
- `devise_controller?`
  - [devise/lib/devise/controllers/helpers.rb](https://github.com/heartcombo/devise/blob/main/lib/devise/controllers/helpers.rb#L151-L153)

### 2-9. i18nで日本語化

i18nの基本的な使い方は以下を参照ください。

[【Rails】i18nで日本語化する方法 | あまブログ](https://ama-blog.com/58/)

はじめに、使用する言語のリストとデフォルトで使用する言語を設定します。

`config/initializers/locale.rb`を作成し、以下を追記します。

```ruby
I18n.available_locales = [:en, :ja]
I18n.default_locale = :ja
```

次に、日本語のロケールファイルを作成します。

`config/locales/ja.yml`を作成し、以下を追記します。

```yaml
ja:
  activerecord:
    models:
      book: 本
    attributes:
      user:
        name: 氏名
        self_introduction: 自己紹介文
  views:
    common:
      show: 詳細
      edit: 編集
      back: 戻る
      title_show: "%{name}の詳細"
      sign_out: ログアウト
  layouts:
    application:
      menu: メニュー
      sign_in_as: "%{email} としてログイン中"
  # devise-i18n-viewsをオーバーライド
  devise:
    registrations:
      edit:
        title: "アカウント編集"
```

- devise-i18n gemのインストールにより、[devise-i18n/rails/locales/ja.yml](https://github.com/tigrish/devise-i18n/blob/33e30855bf67be52e56121a4b0f1c284129a69c0/rails/locales/ja.yml)の翻訳も反映されます。

最後に、各viewページに翻訳を反映させていきます。

`app/views/layouts/application.html.erb`を以下のように編集します。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>SampleApp</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <% if user_signed_in? %>
      <div class="menu-container">
        <div class="title"><%= t('.menu') %></div>
        <ul>
          <li>
            <%= link_to Book.model_name.human, books_path %>
          </li>
          <li>
            <%= link_to User.model_name.human, users_path %>
          </li>
        </ul>
        <div class="title"><%= t('.sign_in_as', email: current_user.email) %></div>
        <ul>
          <li>
            <%= link_to t('devise.registrations.edit.title'), edit_user_registration_path %>
          </li>
          <li>
            <%= link_to t('views.common.sign_out'), destroy_user_session_path, method: :delete %>
          </li>
        </ul>
      </div>
    <% end %>
    <% if notice.present? %>
      <p id="notice"><%= notice %></p>
    <% end %>
    <% if alert.present? %>
      <p id="alert"><%= alert %></p>
    <% end %>
    <%= yield %>
  </body>
</html>
```

`app/views/users/index.html.erb`を以下のように編集します。

```html
<h1><%= User.model_name.human %></h1>

<table>
  <thead>
    <tr>
      <th><%= User.human_attribute_name(:email) %></th>
      <th><%= User.human_attribute_name(:name) %></th>
      <th></th>
    </tr>
  </thead>

  <tbody>
    <% @users.each do |user| %>
      <tr>
        <td><%= user.email %></td>
        <td><%= user.name %></td>
        <td><%= link_to t('views.common.show'), user %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

`app/views/users/show.html.erb`を以下のように編集します。

```html
<h1><%= t('views.common.title_show', name: User.model_name.human) %></h1>

<p>
  <strong><%= User.human_attribute_name(:email) %>:</strong>
  <%= @user.email %>
</p>

<p>
  <strong><%= User.human_attribute_name(:name) %>:</strong>
  <%= @user.name %>
</p>

<p>
  <strong><%= User.human_attribute_name(:self_introduction) %>:</strong>
  <%= @user.self_introduction %>
</p>

<% if current_user == @user %>
  <%= link_to t('views.common.edit'), edit_user_registration_path %> |
<% end %>
<%= link_to t('views.common.back'), users_path %>
```

### 2-10. サインイン,サインアウト,アカウント編集後のリダイレクト先の変更

`app/controllers/application_controller.rb`に以下を追記します。

```ruby
class ApplicationController < ActionController::Base
  # 省略
  private

  def after_sign_in_path_for(resource_or_scope)
    books_path
  end

  def after_sign_out_path_for(resource_or_scope)
    new_user_session_path
  end

  def signed_in_root_path(resource_or_scope)
    user_path(current_user)
  end
end
```

上記では、deviseに実装されている`after_sign_in_path_for`メソッド、`after_sign_out_path_for`メソッド、`signed_in_root_path`メソッドをオーバーライドしています。

これにより、サインイン後に`/books`、サインアウト後に`/users/sign_in`、アカウント編集後に`/users/:id`にリダイレクトします。

- [Controller filters and helpers - devise](https://github.com/heartcombo/devise#controller-filters-and-helpers:~:text=You%20can%20also%20override%20after_sign_in_path_for%20and%20after_sign_out_path_for%20to%20customize%20your%20redirect%20hooks.)
- `after_sign_in_path_for`
  - [devise/lib/devise/controllers/helpers.rb](https://github.com/heartcombo/devise/blob/main/lib/devise/controllers/helpers.rb#L215-L217)
- `after_sign_out_path_for`
  - [devise/lib/devise/controllers/helpers.rb](https://github.com/heartcombo/devise/blob/main/lib/devise/controllers/helpers.rb#L225-L230)
- `signed_in_root_path`
  - [devise/lib/devise/controllers/helpers.rb](https://github.com/heartcombo/devise/blob/main/lib/devise/controllers/helpers.rb#L169-L186)

### 2-11. letter_opener_webで送信メールをブラウザ上で確認できるようにする
[【Rails】letter_opener_webで送信メールをブラウザ上で確認する - あまブログ](https://ama-tech.hatenablog.com/letter_opener_web)

---

【参考】

- [Devise で作成した User モデル用のコントローラーの index, show アクションを追加 | EasyRamble](https://easyramble.com/create-users-index-show-on-devise.html)
- [How To: Change the redirect path after destroying a session i.e. signing out · heartcombo/devise Wiki](https://github.com/heartcombo/devise/wiki/How-To:-Change-the-redirect-path-after-destroying-a-session-i.e.-signing-out)
- [How To: Customize the redirect after a user edits their profile · heartcombo/devise Wiki](https://github.com/heartcombo/devise/wiki/How-To:-Customize-the-redirect-after-a-user-edits-their-profile)
- [Rails: Devise: redirect after sign in by role - Stack Overflow](https://stackoverflow.com/questions/7638920/rails-devise-redirect-after-sign-in-by-role)
