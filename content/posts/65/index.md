---
date: '2022-10-25T21:21:15+09:00'
draft: false
title: '【Rails】ポリモーフィック関連付けを使ってコメント投稿機能を実装する'
tags: ["未分類"]
slug: '65'
---

この記事では、Railsの[ポリモーフィック関連付け](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)を使ってコメント投稿機能を実装する際のポイントを紹介します。

## 1. バージョン情報
- macOS：12.6
- Ruby：3.1.2
- Rails：6.1.6


## 2. 前提条件
- Userモデル(ユーザー)
- Bookモデル(本)のCRUD
- Userモデルに紐づいたReportモデル(日報)のCRUD


## 3. 実装時のポイント

### 3-1. ポリモーフィック関連付け

**BookモデルとReportモデルに従属するCommentモデルの作成**

```
$ rails g model comment user:references commentable:references{polymorphic} content:text --no-test-framework
```

`app/models/comment.rb`
```ruby
class Comment < ApplicationRecord
  belongs_to :user
  belongs_to :commentable, polymorphic: true

  validates :content, presence: true # 追記
end
```

`db/migrate/XXXX_create_comments.rb`
```ruby
class CreateComments < ActiveRecord::Migration[6.1]
  def change
    create_table :comments do |t|
      t.references :user, null: false, foreign_key: true
      t.references :commentable, polymorphic: true, null: false
      t.text :content

      t.timestamps
    end
  end
end
```

マイグレーションを実行
```
$ rails db:migrate
```

`db/schema.rb`
```ruby
  create_table "comments", force: :cascade do |t|
    t.integer "user_id", null: false
    t.string "commentable_type", null: false
    t.integer "commentable_id", null: false
    t.text "content"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["commentable_type", "commentable_id"], name: "index_comments_on_commentable"
    t.index ["user_id"], name: "index_comments_on_user_id"
  end
```
- `commentable_type`カラムにセットされるのは参照する親モデルのクラス名(`Book`または`Report`)

`app/models/book.rb`
```ruby
class Book < ApplicationRecord
  has_many :comments, as: :commentable, dependent: :destroy # 追記
end
```

`app/models/report.rb`
```ruby
class Report < ApplicationRecord
  belongs_to :user
  has_many :comments, as: :commentable, dependent: :destroy # 追記

  validates :title, presence: true
  validates :content, presence: true
end
```

### 3-2. コメント作成

**ルーティングの設定**

`config/routes.rb`
```ruby
Rails.application.routes.draw do
  # 省略
  resources :books do
    resources :comments, only: :create, module: :books
  end
  resources :reports do
    resources :comments, only: :create, module: :reports
  end
end
```
- `module: :books`と`module: :reports`
  - [Railsガイド: Rails のルーティング - 2.6 コントローラの名前空間とルーティング](https://railsguides.jp/routing.html#%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%A9%E3%81%AE%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93%E3%81%A8%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0)


| HTTP verb | Path    | Controller#Action   | Helper |
| ---- | ---- | ----- | ----- |
| POST    | /books/:book_id/comments | books/comments#create | book_comments_path |
| POST    | /reports/:report_id/comments | reports/comments#create | report_comments_path |


**Commentsコントローラーの作成**

```
$ rails g controller Comments --no-assets --no-helper --no-test-framework
```

`app/controllers/comments_controller.rb`
```ruby
class CommentsController < ApplicationController
  def create
    @comment = @commentable.comments.build(comment_params)
    @comment.user = current_user
    if @comment.save
      redirect_to @commentable, notice: t('controllers.common.notice_create', name: Comment.model_name.human)
    else
      # コメント作成失敗時の処理は後述
    end
  end

  private

  def comment_params
    params.require(:comment).permit(:content)
  end
end
```

**Books::Commentsコントローラーの作成**

`app/controllers/books/comments_controller.rb`
```ruby
class Books::CommentsController < CommentsController
  before_action :set_commentable

  private

  def set_commentable
    @commentable = Book.find(params[:book_id])
  end
end
```

**Reports::Commentsコントローラーの作成**

`app/controllers/reports/comments_controller.rb`
```ruby
class Reports::CommentsController < CommentsController
  before_action :set_commentable

  private

  def set_commentable
    @commentable = Report.find(params[:report_id])
  end
end
```

**コメント投稿フォームの作成**

`app/views/comments/_form.html.erb`
```html
<%= form_with model: [commentable, comment] do |f| %>
  <% if comment.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= I18n.t("errors.messages.not_saved",
                  count: comment.errors.count,
                  resource: comment.class.model_name.human.downcase) %>
      </h2>
      <ul>
        <% comment.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <%= f.text_field :content, required: true %>
  <%= f.submit t('.create') %>
<% end %>
```
- `form_with model: [commentable, comment]`
  - form_withはURLを生成するのに[polymorphic_path](https://api.rubyonrails.org/classes/ActionDispatch/Routing/PolymorphicRoutes.html)を使っている

**本と日報の詳細画面からコメントを投稿できるようにする**

`app/controllers/books_controller.rb`
```ruby
class BooksController < ApplicationController
  # 省略
  def show
    @comment = Comment.new
  end
end
```
- フォームに渡す`@comment`を定義

`app/controllers/reports_controller.rb`
```ruby
class ReportsController < ApplicationController
  # 省略
  def show
    @comment = Comment.new
  end
end
```
- フォームに渡す`@comment`を定義


`app/views/books/show.html.erb`
```html
<%= render 'comments/form', commentable: @book, comment: @comment %>
```
- フォーム側で`book_comments_path`を生成

`app/views/reports/show.html.erb`
```html
<%= render 'comments/form', commentable: @report, comment: @comment %>
```
- フォーム側で`report_comments_path`を生成

**i18nで日本語化**

`ja.yml`
```yaml
ja:
  activerecord:
    models:
      comment: コメント
  controllers:
    common:
      notice_create: "%{name}が作成されました。"
  comments:
    form:
      create: コメントする
```

### 3-3. コメント表示

**コメント一覧パーシャルの作成**

`app/views/comments/_comments.html.erb`
```html
<div class="comments-container">
  <strong><%= Comment.model_name.human %>:</strong>
  <% if comments.any? %>
    <ul>
      <% comments.each do |comment| %>
        <% if comment.persisted? %>
          <li>
            <%= comment.content %>
            <small>
              (<%= link_to comment.user.name_or_email, comment.user %> - <%= l comment.created_at, format: :short %>)
            </small>
          </li>
        <% end %>
      <% end %>
    </ul>
  <% else %>
    (<%= t('.no_comments') %>)
  <% end %>
</div>
```
- `if comments.any?`
  - コメントが1件もなければ`t('.no_comments')`
  - [Rails API: any?](https://api.rubyonrails.org/classes/ActiveRecord/Associations/CollectionProxy.html#method-i-any-3F)
- `if comment.persisted?`
  - コメントがDBに保存されていれば表示
  - [Rails API: persisted?](https://api.rubyonrails.org/classes/ActiveModel/API.html#method-i-persisted-3F)
- `l comment.created_at, format: :short`
  - `10/24 11:32`の形式で表示
  - [Railsガイド: Rails 国際化（i18n）API - 3.4 日付・時刻フォーマットを追加する](https://railsguides.jp/i18n.html#%E6%97%A5%E4%BB%98%E3%83%BB%E6%99%82%E5%88%BB%E3%83%95%E3%82%A9%E3%83%BC%E3%83%9E%E3%83%83%E3%83%88%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)

`app/models/user.rb`
```ruby
class User < ApplicationRecord
  # 省略
  def name_or_email
    name.empty? ? email : name
  end
end
```
- `name_or_email`
  - ユーザー名が入力されていればユーザー名を表示、未入力ならメールアドレスを表示

`app/controllers/books_controller.rb`
```ruby
class BooksController < ApplicationController
  # 省略
  def show
    @comment = Comment.new
    @comments = @book.comments # 追記
  end
end
```
- コメント一覧パーシャルに渡す`@comments`を定義

`app/controllers/reports_controller.rb`
```ruby
class ReportsController < ApplicationController
  # 省略
  def show
    @comment = Comment.new
    @comments = @report.comments # 追記
  end
end
```
- コメント一覧パーシャルに渡す`@comments`を定義

`app/views/books/show.html.erb`
```html
<%= render 'comments/comments', comments: @comments %>
```

`app/views/reports/show.html.erb`
```html
<%= render 'comments/comments', comments: @comments %>
```

**i18nで日本語化**

`ja.yml`
```yaml
ja:
  activerecord:
    models:
      comment: コメント
  comments:
    comments:
      no_comments: コメントはまだありません
```

### 3-4. コメント作成失敗時の処理

**コメント作成失敗時に本または日報の詳細画面に遷移する**

`app/controllers/comments_controller.rb`
```ruby
class CommentsController < ApplicationController
  def create
    @comment = @commentable.comments.build(comment_params)
    @comment.user = current_user
    if @comment.save
      redirect_to @commentable, notice: t('controllers.common.notice_create', name: Comment.model_name.human)
    else
      # コメント作成失敗時の処理を追記
      @comments = @commentable.comments
      render_commentable_show
    end
  end

  # 省略
end
```

`app/controllers/books/comments_controller.rb`
```ruby
class Books::CommentsController < CommentsController
  before_action :set_commentable

  private

  # 省略

  def render_commentable_show
    @book = @commentable
    render 'books/show'
  end
end
```

`app/controllers/reports/comments_controller.rb`
```ruby
class Reports::CommentsController < CommentsController
  before_action :set_commentable

  private

  # 省略

  def render_commentable_show
    @report = @commentable
    render 'reports/show'
  end
end
```

**i18nで日本語化**

`ja.yml`
```yaml
ja:
  activerecord:
    models:
      comment: コメント
  controllers:
    common:
      notice_create: "%{name}が作成されました。"
```

### 3-5. コメント削除・編集

**ルーティングの設定**

`config/routes.rb`
```ruby
Rails.application.routes.draw do
  # 省略
  resources :books do
    resources :comments, only: %i[create destroy edit update], module: :books
  end
  resources :reports do
    resources :comments, only: %i[create destroy edit update], module: :reports
  end
end
```

| HTTP verb | Path    | Controller#Action   | Helper |
| ---- | ---- | ----- | ----- |
| POST    | /books/:book_id/comments | books/comments#create | book_comments_path |
| GET    | /books/:book_id/comments/:id/edit | books/comments#edit | edit_book_comment_path |
| PATCH    | /books/:book_id/comments/:id | books/comments#update | book_comment_path |
| DELETE    | /books/:book_id/comments/:id | books/comments#destroy | book_comment_path |

**edit・update・destroyアクションの追加**

`app/controllers/comments_controller.rb`
```ruby
class CommentsController < ApplicationController
  before_action :set_comment, only: %i[edit update destroy]

  def edit; end

  def create
    # 省略
  end

  def update
    if @comment.update(comment_params)
      redirect_to @commentable, notice: t('controllers.common.notice_update', name: Comment.model_name.human)
    else
      render :edit
    end
  end

  def destroy
    @comment.destroy

    redirect_to @commentable, notice: t('controllers.common.notice_destroy', name: Comment.model_name.human)
  end

  private

  def set_comment
    @comment = Comment.find(params[:id])
  end

  # 省略
end
```

**コメント削除・編集リンクの追加**

`app/views/comments/_comments.html.erb`
```html
<div class="comments-container">
  <strong><%= Comment.model_name.human %>:</strong>
  <% if comments.any? %>
    <ul>
      <% comments.each do |comment| %>
        <% if comment.persisted? %>
          <li>
            <%= comment.content %>
            <small>
              (<%= link_to comment.user.name_or_email, comment.user %> - <%= l comment.created_at, format: :short %>)
            </small>
            <% if current_user == comment.user %>
              <%= link_to t('views.common.edit'), edit_polymorphic_path([commentable, comment]) %> |
              <%= link_to t('views.common.destroy'), polymorphic_path([commentable, comment]), method: :delete, data: { confirm: t('views.common.delete_confirm') } %>
            <% end %>
          </li>
        <% end %>
      <% end %>
    </ul>
  <% else %>
    (<%= t('.no_comments') %>)
  <% end %>
</div>
```
- `edit_polymorphic_path([commentable, comment])`
  - `"/books/1/comments/1/edit"`または`"/reports/1/comments/1/edit"`
- `polymorphic_path([commentable, comment])`
  - `"/books/1/comments/1"`または`"/reports/1/comments/1"`

`app/views/books/show.html.erb`
```html
<%= render 'comments/comments', commentable: @book, comments: @comments %>
```
- `commentable: @book`を追記

`app/views/reports/show.html.erb`
```html
<%= render 'comments/comments', commentable: @report, comments: @comments %>
```
- `commentable: @report`を追記

**i18nで日本語化**

`ja.yml`
```yaml
ja:
  activerecord:
    models:
      comment: コメント
  views:
    common:
      edit: 編集
      destroy: 削除
      delete_confirm: よろしいですか？
  controllers:
    common:
      notice_update: "%{name}が更新されました。"
      notice_destroy: "%{name}が削除されました。"
  comments:
    comments:
      no_comments: コメントはまだありません
```
