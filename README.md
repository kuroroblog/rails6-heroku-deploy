# rails6-heroku-deploy
docker + vue + rails6 + heroku + mysqlを実装する。

# 事前準備
以下の動画URLの事前準備箇所を済ませておく。

動画URL : https://www.youtube.com/watch?v=uQf9968RWWo&t=2412s

# 各種サービスのversion

| サービス | version |
| ------------- | ------------- |
| docker  | クライアント(20.10.5), サーバー(20.10.5)  |
| vue  | 2.6.12  |
| rails  | 6.1.4  |
| heroku | heroku/7.56.1 darwin-x64 node-v12.21.0 |
| mysql | ローカル環境8系, heroku環境5系 |

# zipをダウンロードする
1. https://github.com/kuroroblog/rails6-heroku-deploy へアクセスする。
2. 緑色の「Code」と書かれたボタンを選択
3. 「Download ZIP」を選択
4. ダウンロードされたzipファイルをデスクトップへ移動
5. zipファイルをダブルクリック
6. ターミナルを開く。
7. ターミナルを活用して、zipを展開して生成されたフォルダへ移動する。(`$ cd Desktop/rails6-heroku-deploy-master`)

# Dockerイメージを用いて、rails6をローカル環境で立ち上げる
1. `$ docker-compose run web rails new . --force --database=mysql`を実行する。
2. src/config/database.ymlファイルを編集する。

```ruby:database.yml
default: &default
    adapter: mysql2
    encoding: utf8mb4
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
    username: root
-   password:
+   password: password
-   host: localhost
+   host: db
```

3. `$ docker-compose build`を実行する。
4. `$ docker-compose run web rails db:create`を実行する。
5. `$ docker-compose up`を実行する。

### 完成画像
<img width="1680" alt="screenshot 2021-08-01 10 36 49" src="https://user-images.githubusercontent.com/23373288/127756430-987e84f1-5784-4bca-8bb7-53dfe5013cc6.png">

### 参考 : https://www.youtube.com/watch?v=ltDdZAJli8c

## rails6へvueを追加し、ローカル環境で立ち上げる
1. `$ docker-compose exec web bundle exec rails webpacker:install:vue`を実行する。
2. `$ docker-compose run web yarn add vue@2.6.12 vue-loader@15.9.3 vue-template-compiler@2.6.12`を実行する。
3. routes.rbを編集する。

```ruby:routes.rb
Rails.application.routes.draw do
    # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
    get '/', to: 'users#index'
end
```

4. `$ docker-compose exec web bundle exec rails g controller users`を実行する。
5. users_controller.rbを編集する。

``` ruby:users_controller.rb
class UsersController < ApplicationController
    def index 
    end
end
```

6. app/views/users/index.html.erbファイルを作成する。
7. app/views/users/index.html.erbファイルを編集する。

``` ruby:index.html.erb
<h1>hello world</h1>
<%= javascript_pack_tag 'hello_vue.js' %>
```

8. `$ docker-compose down`を実行する。
9. `$ docker-compose build`を実行する。
10. `$ docker-compose up`を実行する。

### 完成画像
<img width="1680" alt="screenshot 2021-08-01 11 00 07" src="https://user-images.githubusercontent.com/23373288/127756734-1aad7cc0-fa4e-4212-b734-8f081298bf75.png">

## rails6でmysqlが利用できることを確認する
1. `$ docker-compose exec web bundle exec rails g model post title:string`を実行する。
2. `$ docker-compose exec web bundle exec rails db:migrate`を実行する。
3. users_controller.rbを編集する。

``` ruby:users_controller.rb
class UsersController < ApplicationController
   def index
       @posts = Post.order(created_at: :desc)
   end
end
```

4. app/views/users/index.html.erbファイルを編集する。

``` ruby:index.html.erb
<h1>hello world</h1>
<% @posts.each do |post| %>
   <p><%= post.title %></p>
<% end %>
<%= javascript_pack_tag 'hello_vue.js' %>
```

5. `$ docker-compose exec db bin/bash`を実行する。
6. `$ mysql -uroot -p`を実行する。
7. `$ password`を実行する。
8. `$ use app_development;`を実行する。
9. `$ INSERT INTO posts (title, created_at, updated_at) VALUES ('hoge', '9999-12-31 23:59:59.999999', '9999-12-31 23:59:59.999999');`を実行する。
10. `$ quit`を実行する。
11. `$ exit`を実行する。

## 完成画像
<img width="1680" alt="screenshot 2021-08-01 11 03 18" src="https://user-images.githubusercontent.com/23373288/127756776-0bea3d1c-4a29-494c-9f2f-98d2815e597f.png">

## herokuへデプロイする。

1. `$ docker-compose down`を実行する。
2. `$ heroku login`を実行する。
3. `$ heroku container login`を実行する。
4. `$ heroku create {アプリ名}`を実行する。({アプリ名}はお好みの名前をつけてください。)
5. `$ heroku addons:create cleardb:ignite -a {アプリ名}`を実行する。
6. src/config/database.ymlファイルを編集する。

```ruby:database.yml
production:
   <<: *default
-  database: app_production
+  database: <%= ENV['APP_DATABASE'] %>
-  username: app
+  username: <%= ENV['APP_DATABASE_USERNAME'] %>
   password: <%= ENV['APP_DATABASE_PASSWORD'] %>
+  host: <%= ENV['APP_DATABASE_HOST'] %>
```

6. `$ heroku config -a {アプリ名}`を実行する。(表示される`CLEARDB_DATABASE_URL: mysql://{APP_DATABASE_USERNAME}:{APP_DATABASE_PASSWORD}@{APP_DATABASE_HOST}/{APP_DATABASE}?reconnect=true`をメモしておく。)
7. 6でメモした情報を元に、`$ heroku config:add APP_DATABASE_USERNAME="{APP_DATABASE_USERNAME}" -a {アプリ名}`を実行する。
8. 6でメモした情報を元に、`$ heroku config:add APP_DATABASE_PASSWORD="{APP_DATABASE_PASSWORD}" -a {アプリ名}`を実行する。
9. 6でメモした情報を元に、`$ heroku config:add APP_DATABASE_HOST="{APP_DATABASE_HOST}" -a {アプリ名}`を実行する。
10. 6でメモした情報を元に、`$ heroku config:add APP_DATABASE="{APP_DATABASE}" -a {アプリ名}`を実行する。 
11. `$ heroku config:add RAILS_SERVE_STATIC_FILES='true' -a {アプリ名}`を実行する。
12. こちらのページへアクセスする。(https://tools.heroku.support/limits/boot_timeout)
13. {アプリ名}を選択して、Boot Timeout時間を120秒へ変更する。最後に「Change Boot Timeout」ボタンを選択する。
14. `$ heroku container:push web -a {アプリ名}`を実行する。
15. `$ heroku container:release web -a {アプリ名}`を実行する。
16. `$ heroku run bundle exec rake db:migrate RAILS_ENV=production -a {アプリ名}`を実行する。
17. `$ heroku open -a {アプリ名}`を実行する。

## herokuの本番環境mysqlへrecordを追加する

1. `$ heroku run rails c -a {アプリ名}`を実行する。
2. `$ Post.create(title:'title1')`を実行する。
3. `$ Post.create(title:'title2')`を実行する。
4. `$ exit`を実行する。
