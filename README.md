# rails6-heroku-deploy
docker + vue + rails6 + heroku + mysqlを実装する。

## 事前準備
以下の動画URLの事前準備箇所を済ませておく。

動画URL : https://www.youtube.com/watch?v=uQf9968RWWo&t=2412s

## 各種サービスのversion

| サービス | version |
| ------------- | ------------- |
| docker  | クライアント(20.10.5), サーバー(20.10.5)  |
| vue  | 2.6.12  |
| rails  | 6.1.4  |
| heroku | heroku/7.56.1 darwin-x64 node-v12.21.0 |
| mysql | ローカル環境8系, heroku環境5系 |

## zipをダウンロードする
1. https://github.com/kuroroblog/rails6-heroku-deploy へアクセスする。
2. 緑色の「Code」と書かれたボタンを選択
3. 「Download ZIP」を選択
4. ダウンロードされたzipファイルをデスクトップへ移動
5. zipファイルをダブルクリック
6. ターミナルを開く。
7. ターミナルを活用して、zipを展開して生成されたフォルダへ移動する。(`$ cd Desktop/rails6-heroku-deploy-master`)

## Dockerイメージを用いて、rails6をローカル環境で立ち上げる
1. `$ docker-compose run web rails new . --force --database=mysql`を実行する。(railsアプリの雛形を作成する。参考 : https://docs.docker.jp/compose/reference/run.html)
2. src/config/database.ymlファイルを編集する。(docker-compose.ymlで定義する、password, hostへ変更する。)

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

3. `$ docker-compose build`を実行する。(docker-comopse.ymlファイルを変更したので、dockerイメージを作り直す。)
4. `$ docker-compose run web rails db:create`を実行する。(dbの作成を行う。参考 : https://docs.docker.jp/compose/reference/run.html)
5. `$ docker-compose up`を実行する。(コンテナを立ち上げる。参考 : https://docs.docker.jp/compose/reference/up.html)

### 完成画像
<img width="1680" alt="screenshot 2021-08-01 10 36 49" src="https://user-images.githubusercontent.com/23373288/127756430-987e84f1-5784-4bca-8bb7-53dfe5013cc6.png">

### 参考 : https://www.youtube.com/watch?v=ltDdZAJli8c

## rails6へvueを追加し、ローカル環境で立ち上げる
1. `$ docker-compose exec web bundle exec rails webpacker:install:vue`を実行する。(vueをインストールする。参考 : https://matsuand.github.io/docs.docker.jp.onthefly/compose/reference/exec/)
2. `$ docker-compose run web yarn add vue@2.6.12 vue-loader@15.9.3 vue-template-compiler@2.6.12`を実行する。(vue, vue-loader, vue-template-compilerのversionを変更する。)
3. routes.rbを編集する。

```ruby:routes.rb
Rails.application.routes.draw do
    # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
    get '/', to: 'users#index'
end
```

4. `$ docker-compose exec web bundle exec rails g controller users`を実行する。(controllerの雛形を作成する。 参考 : https://guides.rubyonrails.org/command_line.html#bin-rails-generate)
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

8. `$ docker-compose down`を実行する。(コンテナを停止する。参考 : https://docs.docker.jp/compose/reference/down.html)
9. `$ docker-compose build`を実行する。
10. `$ docker-compose up`を実行する。

### 完成画像
<img width="1680" alt="screenshot 2021-08-01 11 00 07" src="https://user-images.githubusercontent.com/23373288/127756734-1aad7cc0-fa4e-4212-b734-8f081298bf75.png">

## rails6でローカルのmysqlが利用できることを確認する
1. `$ docker-compose exec web bundle exec rails g model post title:string`を実行する。(modelの雛形を作成する。 参考 : https://guides.rubyonrails.org/command_line.html#bin-rails-generate)
2. `$ docker-compose exec web bundle exec rails db:migrate`を実行する。(migrationを行う。)
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

5. `$ docker-compose exec db bin/bash`を実行する。(dbサービスコンテナへログインする。)
6. `$ mysql -uroot -p`を実行する。
7. `$ password`を実行する。
8. `$ use app_development;`を実行する。
9. `$ INSERT INTO posts (title, created_at, updated_at) VALUES ('hoge', '9999-12-31 23:59:59.999999', '9999-12-31 23:59:59.999999');`を実行する。(posts tableへrecordの追加を行う。)
10. `$ quit`を実行する。
11. `$ exit`を実行する。

### 完成画像
<img width="1680" alt="screenshot 2021-08-01 11 03 18" src="https://user-images.githubusercontent.com/23373288/127756776-0bea3d1c-4a29-494c-9f2f-98d2815e597f.png">

## herokuへデプロイする。

1. `$ docker-compose down`を実行する。
2. `$ heroku login`を実行する。(herokuへログインする。)
3. `$ heroku container login`を実行する。(herokuのコンテナへログインする。)
4. `$ heroku create {アプリ名}`を実行する。({アプリ名}はお好みの名前をつけてください。herokuサービスを用いて、アプリの作成を行う。)
5. `$ heroku addons:create cleardb:ignite -a {アプリ名}`を実行する。(アプリへmysqlを装備する。)
6. src/config/database.ymlファイルを編集する。(herokuの本番環境mysqlとの接続のため、環境変数を整える。)

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

6. `$ heroku config -a {アプリ名}`を実行する。(表示される`CLEARDB_DATABASE_URL: mysql://{APP_DATABASE_USERNAME}:{APP_DATABASE_PASSWORD}@{APP_DATABASE_HOST}/{APP_DATABASE}?reconnect=true`をメモしておく。herokuのアプリ内で設定される環境変数を確認する。)
7. 6でメモした情報を元に、`$ heroku config:add APP_DATABASE_USERNAME="{APP_DATABASE_USERNAME}" -a {アプリ名}`を実行する。(アプリ内へAPP_DATABASE_USERNAME環境変数を設定する。)
8. 6でメモした情報を元に、`$ heroku config:add APP_DATABASE_PASSWORD="{APP_DATABASE_PASSWORD}" -a {アプリ名}`を実行する。(アプリ内へAPP_DATABASE_PASSWORD環境変数を設定する。)
9. 6でメモした情報を元に、`$ heroku config:add APP_DATABASE_HOST="{APP_DATABASE_HOST}" -a {アプリ名}`を実行する。(アプリ内へAPP_DATABASE_HOST環境変数を設定する。)
10. 6でメモした情報を元に、`$ heroku config:add APP_DATABASE="{APP_DATABASE}" -a {アプリ名}`を実行する。 (アプリ内へAPP_DATABASE環境変数を設定する。)
11. `$ heroku config:add RAILS_SERVE_STATIC_FILES='true' -a {アプリ名}`を実行する。(アプリ内へRAILS_SERVE_STATIC_FILES環境変数を設定する。precompileのために必要。参考 : https://railsguides.jp/asset_pipeline.html#%E3%82%A2%E3%82%BB%E3%83%83%E3%83%88%E3%82%92%E3%83%97%E3%83%AA%E3%82%B3%E3%83%B3%E3%83%91%E3%82%A4%E3%83%AB%E3%81%99%E3%82%8B)
12. こちらのページへアクセスする。(https://tools.heroku.support/limits/boot_timeout)
13. {アプリ名}を選択して、Boot Timeout時間を120秒へ変更する。最後に「Change Boot Timeout」ボタンを選択する。(時間を引き伸ばして、確実にdeployするため。)
14. `$ heroku container:push web -a {アプリ名}`を実行する。(Dockerイメージをherokuのコンテナへ展開する。)
15. `$ heroku container:release web -a {アプリ名}`を実行する。(herokuのコンテナへ展開した、Dockerイメージ情報を元に、サービスリリースする。)
16. `$ heroku run bundle exec rake db:migrate RAILS_ENV=production -a {アプリ名}`を実行する。(herokuの本番環境mysqlと接続して、migrationを行う。)
17. `$ heroku open -a {アプリ名}`を実行する。(アプリを開く。)

### 完成画像
<img width="1680" alt="screenshot 2021-08-01 11 50 42" src="https://user-images.githubusercontent.com/23373288/127757525-01107f79-19a3-4dc4-9756-db9c68000998.png">

### 参考
https://www.youtube.com/watch?v=uQf9968RWWo&t=2412s

## herokuの本番環境mysqlへrecordを追加する

1. `$ heroku run rails c -a {アプリ名}`を実行する。(herokuのアプリとインタラクティブモードを行う。参考 : https://www.atmarkit.co.jp/fdotnet/special/ironpython01/ironpython01_02.html)
2. `$ Post.create(title:'title1')`を実行する。(herokuの本番環境mysqlへrecordの追加を行う。)
3. `$ Post.create(title:'title2')`を実行する。
4. `$ exit`を実行する。

## 最終結果画像
<img width="1680" alt="screenshot 2021-08-01 11 52 21" src="https://user-images.githubusercontent.com/23373288/127757563-6999ccae-514c-44bd-b1d8-2a04c26cedd2.png">

## 参考文献
・https://www.youtube.com/watch?v=ltDdZAJli8c
・https://www.youtube.com/watch?v=uQf9968RWWo&t=2412s
・https://github.com/kiyodori/rails-docker-kyt
・https://qiita.com/keisukeYamagishi/items/444ef89590323af8a7ac
・https://railsdoc.com/rails
・https://qiita.com/TakahiroSakoda/items/5180ff9762ebddb0bd4d
・https://docs.docker.jp/compose/reference/build.html
・https://nishinatoshiharu.com/docker-volume-tutorial/#i
・https://stackoverflow.com/questions/65375391/dont-know-how-to-build-task-routes
・https://qiita.com/Hal_mai/items/aed97a6aba2293450a74
