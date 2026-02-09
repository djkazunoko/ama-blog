---
date: '2022-10-12T21:22:05+09:00'
draft: false
title: '【Rails】ActiveStorageで画像アップロード機能を実装する'
tags: ["未分類"]
slug: '63'
---

この記事では、Rails 5.2からの標準機能であるActiveStorageを使って画像アップロード機能を実装するポイントを紹介します。

## 1. バージョン情報
* macOS：12.6
* Ruby：3.1.2
* Rails：6.1.6
* image_processing：1.12.2
* active_storage_validations：0.9.8

## 2. 実装時のポイント

### 2-1. Active Storageのセットアップ

以下を実行して、migrationファイルを作成します。

```
$ rails active_storage:install
```
- `db/migrate/XXXX_create_active_storage_tables.active_storage.rb`が作成される

migrationを実行します。

```
$ rails db:migrate
```

### 2-2. 画像アップロードと表示

各userにアバター画像を添付したい場合は、以下のようにUserモデルを定義します。

```ruby
class User < ApplicationRecord
  has_one_attached :avatar
end
```

以下のように書くことでフォームからアバター画像をアップロードできます。

```html
<div class="field">
  <%= f.label :avatar %>
  <%= f.file_field :avatar %>
</div>
```
- ストロングパラメータに`avatar`の追加が必要

以下のように書くことでアバター画像を表示できます。

```html
<%= image_tag user.avatar if user.avatar.attached? %>
```
- `avatar.attached?`で特定のuserがアバター画像を持っているかどうかを調べられます。

### 2-3. 画像のリサイズ

画像のリサイズのために[image_processing](https://rubygems.org/gems/image_processing/versions/1.9.0?locale=ja) gemが必要です。

`Gemfile`の`image_processing` gemのコメントを解除します。

```ruby
# Use Active Storage variant
gem 'image_processing', '~> 1.2'
```

`image_processing` gemを使うために、macに`imagemagick`と`vips`をインストールします。

```
$ brew install imagemagick vips
```
- [GitHub - image_processing - Installation](https://github.com/janko/image_processing#installation)


`variant`メソッドで、添付ファイルごとに特定のサイズ違いの画像を生成できます。

```html
<%= image_tag user.avatar.variant(resize_to_fit: [50, 50]) %>
```
- [ImageProcessing::MiniMagick - README](https://github.com/janko/image_processing/blob/master/doc/minimagick.md#readme)
- [https://qiita.com/wann/items/c6d4c3f17b97bb33936f:title]

### 2-4. バリアントプロセッサの変更

Active Storageのデフォルトのバリアントプロセッサは[MiniMagick](https://github.com/minimagick/minimagick)ですが、[Vips](https://github.com/libvips/libvips)も指定可能です。

`Vips`に切り替えるには、`config/application.rb`に以下の設定を追加します。

```ruby
config.active_storage.variant_processor = :vips
```
- [Active Storage の概要 - 9 画像を変形する - Railsガイド](https://railsguides.jp/active_storage_overview.html#%E7%94%BB%E5%83%8F%E3%82%92%E5%A4%89%E5%BD%A2%E3%81%99%E3%82%8B)
- [config.active_storage.variant_processor - Railsガイド](https://railsguides.jp/configuring.html#config-active-storage-variant-processor)
- [variant.rb - Rails API](https://api.rubyonrails.org/files/activestorage/app/models/active_storage/variant_rb.html)

設定した値はコンソールからも確認できます。
```
$ rails c
irb(main):001:0> Rails.application.config.active_storage.variant_processor
=> :vips
```

### 2-5. N+1問題の解決

Active Storageでは、画像ファイルを親子モデルのアソシエーションとして関係付けるため、N+1問題を引き起こす可能性があります。

このN+1問題を解決するために`with_attached_属性名`スコープを使用します。

以下のように、`app/controllers/users_controller.rb`で`with_attached_avatar`とすることでN+1問題を回避することができます。

```ruby
class UsersController < ApplicationController
  def index
    @users = User.all.with_attached_avatar
  end
  # 省略
end
```
- [ActiveStorage::Attachment](https://api.rubyonrails.org/v7.0/classes/ActiveStorage/Attachment.html)
- [N+1を解決する - Rails 5.2新機能を先行チェック！ TechRacho](https://techracho.bpsinc.jp/hachi8833/2018_02_06/52179)
- [Active StorageのN+1問題に対処する - シュッと開発日記](https://shuttodev.hatenablog.com/entry/2019/09/10/012916)
- [activestorage/lib/active_storage/attached/model.rb](https://github.com/rails/rails/blob/main/activestorage/lib/active_storage/attached/model.rb#L73)

### 2-6. バリデーション

アップロード可能なファイルの種類を限定する方法を紹介します。

#### 方法1. active_storage_validationsを使用する

[active_storage_validations](https://rubygems.org/gems/active_storage_validations/versions/0.6.1?locale=ja) gemを使用します。

`Gemfile`に以下を追記して、`bundle install`を実行します。

```ruby
gem 'active_storage_validations'
```

`app/models/user.rb`に以下を追記します。

```ruby
class User < ApplicationRecord
  has_one_attached :avatar
  validates :avatar, content_type: ['image/png', 'image/jpeg', 'image/gif']
end
```

#### 方法2. 独自のバリデーションヘルパーを実装する


`app/models/user.rb`に以下を追記します。

```ruby
class User < ApplicationRecord
  has_one_attached :avatar

  validate :image_check

  private

  def image_check
    return unless avatar.attached?
    return if avatar.image?

    errors.add(:avatar, 'のContent Typeが不正です')
  end
end
```
- [activestorage/app/models/active_storage/blob.rb](https://github.com/rails/rails/blob/main/activestorage/app/models/active_storage/blob.rb#L195)

フォーム側からファイルの種類に制限をかけるために`accept`を使用します。

```html
<%= f.file_field :avatar, accept: 'image/png,image/gif,image/jpeg' %>
```

---

【参考】

- [Active Storage の概要 - Railsガイド](https://railsguides.jp/active_storage_overview.html)
