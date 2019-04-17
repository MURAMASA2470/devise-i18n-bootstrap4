# 実行環境

OS...MacOS(High Sierra)
ruby...2.5.0
bundler...1.16.3
rails...5.2.1
devise...4.5.0

# deviseの導入

Gemfileに記述します

```:Gemfile
#devise本体
gem 'devise'

#deviseの日本語化用
gem 'devise-i18n'
gem 'font-awesome-rails'

#Bootstrap4
gem 'bootstrap', '~> 4.1.1'
gem 'jquery-rails'
```

bundle install を実行

```:コマンドライン
$ bundle install 
```

# deviseの設定ファイルの生成

deviseの設定ファイルを生成します

```:コマンドライン
$ bundle exec rails g devise:install
```

実行すると以下のように出力がされます

```erb:コマンドライン
Running via Spring preloader in process 17674
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

===============================================================================
```

出力された指示通りに設定を行なっていきます

適当な場所に以下を追記します

```ruby:config/environments/development.rb

config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```
deviseではログイン成功時にアプリケーションの`root`に飛ぶので、適当に**コントローラー**を作成しておきます

```:コマンドライン
$ bundle exec rails g controller Home index
```

そして`routes.rb`に`root_url`を追記します

```ruby:config/routes.rb
root to: "home#index"
```

エラーや認証時のflashメッセージを出力できるように、共通`Layout`の`body`内に以下を追記します

```erb:app/views/layouts/application.html.erb

<% if notice %>
  <p class="notice"><%= notice %></p>
<% elsif alert %>
  <p class="alert"><%= alert %></p>
<% end %>
```

# deviseのviewを生成

普通にdeviseのviewを生成してしまうと`devise`の`Bootstrap4`化が難しいため、下の方にある[deviseの日本語化](#deviseの日本語化)で一緒に説明します

# deviseのmodelを生成

今回はモデル名を`user`します

```:コマンドライン
$ bundle exec rails g devise user
```
このコマンドで`model`と`migrate`ファイルが生成されました
生成された`db/migrate/yyyymmddmmss_devise_create_users.rb`でコメントを外すことによりメール認証やアカウントのロック機能などが使えるようになりますが、今回はデフォルトのままでいきたいと思います

migrateを実行して、DBに反映します

```:コマンドライン
$ bundle exec rails db:migrate
```

# Bootstrap4の実装

共通`CSS`の`application.css`を`application.scss`にリネームします

```:コマンドライン
$ mv app/assets/stylesheets/application.css app/assets/stylesheets/application.scss
```
そして`Bootstrap`を読み込むため`application.scss`に以下を追記します

```scss:app/assets/stylesheets/application.scss
@import "bootstrap";
```

次に`Bootstrap`と依存関係を`application.js`に追記する

```js:app/assets/javascripts/application.js
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

これで`Bootstrap`の実装は完了しました
次にdeviseの日本語化に入ります

# deviseの日本語化

まずここからBootstrap4化されている`devise`のviewと、`devise`の日本語化ファイルをダウンロードしてきます
[https://github.com/MURAMASA2470/devise-i18n-bootstrap4/](https://github.com/MURAMASA2470/devise-i18n-bootstrap4/)

ダウンロードしてきたフォルダの中身を、現在作業中のrailsアプリのフォルダに全て上書きします

すると`app/views/`の中に`devise`というフォルダと`config/locales/`の中に`devise.views.ja.yml`というファイルができたと思います

`devise`の中には今回導入する`devise`の`view`ファイル群が入っています
`devise.views.ja.yml`は以下の様な日本語用の設定が書かれています

```yml:config/locales/devise.views.ja.yml
ja:
  activerecord:
    attributes:
      user:
        confirmation_sent_at: "パスワード確認送信時刻"
        confirmation_token: "パスワード確認用トークン"
        confirmed_at: "パスワード確認時刻"
        created_at: "作成日"
        current_password: "現在のパスワード"
        current_sign_in_at: "現在のログイン時刻"
        current_sign_in_ip: "現在のログインIPアドレス"
        email: Eメール
        encrypted_password: "暗号化パスワード"
        failed_attempts: "失敗したログイン試行回数"
        last_sign_in_at: "最終ログイン時刻"
        last_sign_in_ip: "最終ログインIPアドレス"
        locked_at: "ロック時刻"
        password: "パスワード"
        password_confirmation: "パスワード（確認用）"
        remember_created_at: "ログイン記憶時刻"
        remember_me: "ログインを記憶する"
        reset_password_sent_at: "パスワードリセット送信時刻"
        reset_password_token: "パスワードリセット用トークン"
        sign_in_count: "ログイン回数"
        unconfirmed_email: "未確認Eメール"
        unlock_token: "ロック解除用トークン"
        updated_at: "更新日"
　　　　・
　　　　・
　　　　・
```

このファイルを書き換えることで、自分の好きな日本語に変えることもできます

次に、`Application`クラスで日本語化を反映させます
以下を追記してください

```ruby:config/application.rb
config.i18n.default_locale = :jack_o_lantern:
```

これで`devise`の導入と`Bootstrap4`化と`日本語化`はひとまず完了しました

下の[おまけ](#おまけ)を実装することにより、より本格的なユーザ認証になります

# サーバー再起動で確認

```:コマンドライン
bundle exec rails server
```

#おまけ
## ログアウトボタンの設置

共通`Layout`の`body`内に以下を追記します

**`user_signed_in?`と`destroy_user_session_path`の`user`の部分は上の方で作ったモデル名と同じにしてください**

```erb:app/views/layouts/application.html.erb
  <% if user_signed_in? %>
    <p><%= link_to "ログアウト", destroy_user_session_path, method: :delete %></p>
  <% end %>
```



## ログインしていないときの処理

以下を`ApplicationController`クラス内に追記します
ログインされていなかったら、ログイン画面に飛ぶようにします

**ここでも`authenticate_user!`の`user`の部分は上の方で作ったモデル名と同じにしてください**

```ruby:app/controllers/application_controller.rb
before_action :authenticate_user!
```
