---
date: '2022-10-19T20:46:45+09:00'
draft: false
title: '【Rails】ユーザーフォロー機能を実装する'
tags: ["未分類"]
slug: '64'
---

この記事では、RailsでTwitterやInstagramのようなユーザーフォロー機能を実装する際のポイントを紹介します。

## 1. バージョン情報
- macOS：12.6
- Ruby：3.1.2
- Rails：6.1.6

## 2. 実装時のポイント

### 2-1. モデルの関連

#### Friendshipモデルの作成

```
$ rails g model Friendship follower_id:integer followed_id:integer --no-fixture --no-test-framework
```
- `follower_id`：フォロー(という行為)をしているユーザーのid
  - 自分にとってのフォロワーじゃなくてフォローをしている人という意味のフォロワー(⇄フォロイー)
- `followed_id`：フォローされてるユーザーのid
  - 意味的には`followee_id`でもいい

生成された`db/migrate/XXXX_create_friendships.rb`に以下を追記。
```ruby
add_index :friendships, :followed_id
add_index :friendships, [:follower_id, :followed_id], unique: true
```
- `add_index :friendships, [:follower_id, :followed_id], unique: true`が`:follower_id`で始まっているため、`add_index :friendships, :follower_id`は定義不要
- `unique: true`でデータベース側からユニーク制約をつけ、あるユーザーが同じユーザーを2回以上フォローすることを防ぐ(モデル側のバリデーションによる一意性チェックは後述)

```
$ rails db:migrate
```

#### User/Friendshipの関連付け

`app/models/friendship.rb`
```ruby
class Friendship < ApplicationRecord
  belongs_to :follower, class_name: "User"
  belongs_to :followed, class_name: "User"

  validates :follower_id, uniqueness: { scope: :followed_id }
end
```
- `validates :follower_id, uniqueness: { scope: :followed_id }`
  - モデル側のバリデーションによる一意性チェック
  - `scope`を指定することで`follower_id`と`followed_id`の組み合わせの一意性を保つ(あるユーザーが同じユーザーを2回以上フォローすることを防ぐ)

`app/models/user.rb`
```ruby
class User < ApplicationRecord
  has_many :active_friendships,  class_name: 'Friendship',
                                 foreign_key: :follower_id,
                                 dependent: :destroy,
                                 inverse_of: :follower
  has_many :passive_friendships, class_name: 'Friendship',
                                 foreign_key: :followed_id,
                                 dependent: :destroy,
                                 inverse_of: :followed
end
```
- `class_name`
  - `class_name`オプションは`has_many`で指定した関連付け名と実際のモデル名が違う場合に使用するもの。今回のように一つの`Friendship`モデルに対して二つの関連(`active_friendships`と`passive_friendships`)を持たせたい場合、`has_many :friendships`(で`user.friendships`)だとどちらの関連か区別できない。なので`has_many :active_friendships`(で`user.active_friendships`)とする。ただこのままだとRailsは存在しない`active_friendships`テーブルを探しに行くことになるので、実際に存在する`friendships`テーブルを使うには`has_many :active_friendships,  class_name: 'Friendship'`とする。以上により`active_friendships`という関連を実際のモデル名の`Friendship`と対応付けることができる。
  - [Railsガイド: Active Record の関連付け - 4.1.2.2 :class_name](https://railsguides.jp/association_basics.html#belongs-to%E3%81%AE%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3-class-name)
- `foreign_key`
  - 今回の場合、Railsは自動的に`friendships`テーブルの`user_id`を探しに行くが、`friendships`テーブルに`user_id`カラムはないので、`foreign_key`オプションで明示的に`follower_id`(または`followed_id`)を指定する必要がある
  - [Railsガイド: Active Record の関連付け - 4.1.2.5 :foreign_key
](https://railsguides.jp/association_basics.html#belongs-to%E3%81%AE%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3-foreign-key)
- `dependent: :destroy`
  - Userモデルのデータリソースが削除されるとそれに紐づくFriendshipモデルのデータリソースも同時に削除される
  - [Railsガイド: Active Record の関連付け - 4.1.2.4 :dependent](https://railsguides.jp/association_basics.html#belongs-to%E3%81%AE%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3-dependent)
- `inverse_of`
  - [Railsガイド: Active Record の関連付け - 3.5 双方向関連付け](https://railsguides.jp/association_basics.html#%E5%8F%8C%E6%96%B9%E5%90%91%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)

```ruby
# inverse_ofの挙動
irb> a = User.first
irb> b = a.active_friendships.first
irb> a.name == b.follower.name
=> true
irb> a.name = 'David'
irb> a.name == b.follower.name
=> true # inverse_ofを指定しないとfalse
```

```ruby
# user(自分)が相手をフォローした場合
user.active_friendships.first.follower #=> 自分
user.active_friendships.first.followed #=> 相手
user.passive_friendships.first.follower #=> 相手
user.passive_friendships.first.followed #=> 自分
```

### 2-2. フォロー数・フォロワー数の表示

`app/models/user.rb`
```ruby
class User < ApplicationRecord
  # 省略
  has_many :followings, through: :active_friendships, source: :followed
  has_many :followers, through: :passive_friendships, source: :follower # こっちのsourceオプションは省略可能
end
```
- `has_many :through`
  - フォローするユーザーとフォローされるユーザーの多対多
  - [Railsガイド: Active Record の関連付け - 2.4 has_many :through関連付け](https://railsguides.jp/association_basics.html#has-many-through%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)
- `source: :followed`
  - `source`オプションがないと、Railsは`friendships`テーブルから`following_id`を探しに行ってしまうので、`source: :followed`により`friendships`テーブルの`followed_id`を対象とする
  - [Railsガイド: Active Record の関連付け - 4.3.2.9 :source](https://railsguides.jp/association_basics.html#has-many%E3%81%AE%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3-source)
- ex)`user.followings`
  - userがフォローしたユーザーの一覧を取得
  - `active_friendships`テーブル(userがフォローした関連)の`followed_id`(userにフォローされたユーザー)からuserがフォローしたユーザーの一覧を取得する


`app/views/users/show.html.erb`
```html
<%= render 'stats', user: @user %>
```

`app/views/users/_stats.html.erb`
```html
<ul>
  <li><%= t('.followings', count: user.followings.count) %></li>
  <li><%= t('.followers', count: user.followers.count) %></li>
</ul>
```

`config/locales/ja.yml`
```yaml
ja:
  users:
    stats:
      followings: "%{count} フォロー"
      followers: "%{count} フォロワー"
```

### 2-3. フォロー一覧・フォロワー一覧画面

`config/routes.rb`
```ruby
  resources :users, only: %i[index show] do
    resources :followings, only: [:index], module: :users
    resources :followers, only: [:index], module: :users
  end
```

- `module: :users`
  - [Railsガイド: Rails のルーティング - 2.6 コントローラの名前空間とルーティング](https://railsguides.jp/routing.html#%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%A9%E3%81%AE%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93%E3%81%A8%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0)


| HTTP verb | Path    | Controller#Action   | Helper |
| ---- | ---- | ----- | ----- |
| GET    | /users/:user_id/followings | users/followings#index | user_followings_path |
| GET    | /users/:user_id/followers | users/followers#index | user_followers_path |


`app/views/users/show.html.erb`
```html
<%= render 'stats', user: @user %>
```

`app/views/users/_stats.html.erb`
```html
<ul>
  <li><%= link_to t('.followings', count: user.followings.count), user_followings_path(user) %></li>
  <li><%= link_to t('.followers', count: user.followers.count), user_followers_path(user) %></li>
</ul>
```

`app/controllers/users/followings_controller.rb`
```ruby
class Users::FollowingsController < ApplicationController
  def index
    @user = User.find(params[:user_id])
    @followings = @user.followings.order(id: :desc)
  end
end
```
- `order(id: :desc)`で新しいユーザーから表示

`app/controllers/users/followers_controller.rb`
```ruby
class Users::FollowersController < ApplicationController
  def index
    @user = User.find(params[:user_id])
    @followers = @user.followers.order(id: :desc)
  end
end
```

`app/views/users/followings/index.html.erb`
```html
<h1><%= t('.title') %></h1>
<p><%= User.model_name.human %>: <%= link_to @user.name, @user %></p>
<%= render 'users/users', users: @followings %>
<%= link_to t('views.common.back'), @user %>
```

`app/views/users/followers/index.html.erb`
```html
<h1><%= t('.title') %></h1>
<p><%= User.model_name.human %>: <%= link_to @user.name, @user %></p>
<%= render 'users/users', users: @followers %>
<%= link_to t('views.common.back'), @user %>
```

`app/views/users/_users.html.erb`
```html
<% if users.present? %>
  <table>
    <thead>
    <tr>
      <th><%= User.human_attribute_name(:email) %></th>
      <th><%= User.human_attribute_name(:name) %></th>
      <th></th>
    </tr>
    </thead>

    <tbody>
    <% users.each do |user| %>
      <tr>
        <td><%= user.email %></td>
        <td><%= user.name %></td>
        <td><%= link_to t('views.common.show'), user %></td>
      </tr>
    <% end %>
    </tbody>
  </table>
<% else %>
  <p>データがありません。</p>
<% end %>
```

`config/locales/ja.yml`
```yaml
ja:
  activerecord:
    models:
      user: ユーザ
    attributes:
      user:
        email: Eメール
        name: 氏名
  views:
    common:
      show: 詳細
      back: 戻る
  users:
    followings:
      index:
        title: フォロー
    followers:
      index:
        title: フォロワー
```


### 2-4. フォロー・フォロー解除機能

#### パターン1：resource(idなし)

`config/routes.rb`
```ruby
  resources :users, only: %i[index show] do
    resource :friendships, only: %i[create destroy]
    # 省略
  end
```

- `resource`
  - [Railsガイド: Rails のルーティング - 2.5 単数形リソース](https://railsguides.jp/routing.html#%E5%8D%98%E6%95%B0%E5%BD%A2%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9)


| HTTP verb | Path    | Controller#Action   | Helper |
| ---- | ---- | ----- | ----- |
| DELETE    | /users/:user_id/friendships | friendships#destroy | user_friendships_path |
| POST    | /users/:user_id/friendships | friendships#create | user_friendships_path |

`app/models/user.rb`
```ruby
class User < ApplicationRecord
  # 省略
  def following?(user)
    active_friendships.where(followed_id: user.id).exists?
  end

  def follow(user)
    active_friendships.find_or_create_by!(followed_id: user.id)
  end

  def unfollow(user)
    friendship = active_friendships.find_by(followed_id: user.id)
    friendship&.destroy!
  end
end
```
- `find_or_create_by!`
  - [Rails API: find_or_create_by!](https://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-find_or_create_by-21)
- `friendship&.destroy!`
  - [safe navigation operator (ぼっち演算子)](https://docs.ruby-lang.org/ja/latest/doc/news=2f2_3_0.html)

`app/controllers/friendships_controller.rb`
```ruby
class FriendshipsController < ApplicationController
  before_action :set_user

  def create
    current_user.follow(@user)
    redirect_to user_path(@user), notice: t('.notice')
  end

  def destroy
    current_user.unfollow(@user)
    redirect_to user_path(@user), notice: t('.notice')
  end

  private

  def set_user
    @user = User.find(params[:user_id])
  end
end

```


`app/views/users/show.html.erb`
```html
<%= render 'follow_form', user: @user %>
```

`app/views/users/_follow_form.html.erb`
```html
<% if current_user.following?(user) %>
  <%= form_with(url: user_friendships_path(user), method: :delete, local: true) do |f| %>
    <%= f.submit t('.destroy') %>
  <% end %>
<% elsif current_user != user %>
  <%= form_with(url: user_friendships_path(user), local: true) do |f| %>
    <%= f.submit t('.create') %>
  <% end %>
<% end %>
```

`config/locales/ja.yml`
```yaml
ja:
  users:
    follow_form:
      create: フォローする
      destroy: フォロー解除する
  friendships:
    create:
      notice: フォローしました。
    destroy:
      notice: フォロー解除しました。
```



#### パターン2：resources(idあり)

`config/routes.rb`
```ruby
  resources :users, only: %i[index show] do
    resources :friendships, only: %i[create destroy]
    # 省略
  end
```

| HTTP verb | Path    | Controller#Action   | Helper |
| ---- | ---- | ----- | ----- |
| POST    | /users/:user_id/friendships | friendships#create | user_friendships_path |
| DELETE    | /users/:user_id/friendships/:id | friendships#destroy | user_friendship_path |

`app/models/user.rb`
```ruby
class User < ApplicationRecord
  # 省略
  def following?(user)
    followings.include?(user)
  end
end
```

`app/controllers/friendships_controller.rb`
```ruby
class FriendshipsController < ApplicationController
  before_action :set_user

  def create
    current_user.active_friendships.create!(followed: @user)
    redirect_to @user, notice: t('.notice')
  end

  def destroy
    current_user.active_friendships.find(params[:id]).destroy
    redirect_to @user, notice: t('.notice')
  end

  private

  def set_user
    @user = User.find(params[:user_id])
  end
end
```

`app/views/users/show.html.erb`
```html
<%= render 'follow_form', user: @user %>
```

`app/views/users/_follow_form.html.erb`
```ruby
<% if current_user.following?(user) %>
  <% friendship = current_user.active_friendships.find_by(followed: user) %>
  <%= form_with(model: [user, friendship], method: :delete, local: true) do |f| %>
    <%= f.submit t('.destroy') %>
  <% end %>
<% elsif current_user != user %>
  <%= form_with(model: [user, current_user.active_friendships.build], local: true) do |f| %>
    <%= f.submit t('.create') %>
  <% end %>
<% end %>
```
- `form_with(model: [user, friendship], ~ )`
  - `form_with`はURLを生成するのに[polymorphic_path](https://api.rubyonrails.org/classes/ActionDispatch/Routing/PolymorphicRoutes.html)を使っている


---

【参考】

- uniquenessバリデーション
  - [Railsガイド: Active Record バリデーション - 2.12 uniqueness](https://railsguides.jp/active_record_validations.html#uniqueness)
  - [Rails API: Uniqueness Validations](https://api.rubyonrails.org/classes/ActiveRecord/Validations/ClassMethods.html#method-i-validates_uniqueness_of)
  - [Railsの技: 特定スコープ内でuniquenessバリデーションをかける（翻訳）](https://techracho.bpsinc.jp/hachi8833/2021_07_27/109827)
  - [uniqueness: scope を使ったユニーク制約方法の解説](https://qiita.com/j-sunaga/items/d7f0e944baad6e56206c)
- Railsのアソシエーションのオプション
  - [railsアソシエーションオプションのメモ](https://qiita.com/tomoharutt/items/e548186c763079327ed1)
