# rails6-heroku-deploy
docker + vue + rails6 + heroku + mysqlを実装する。

# 各種サービスのversion

| サービス | version |
| ------------- | ------------- |
| docker  | クライアント(20.10.5), サーバー(20.10.5)  |
| vue  | 2.6.12  |
| rails  | 6.1.4  |
| heroku | heroku/7.56.1 darwin-x64 node-v12.21.0 |
| mysql | ローカル環境8系, heroku環境5系 |

# zipをダウンロードする。
1. https://github.com/kuroroblog/rails6-heroku-deploy へアクセスする。
2. 緑色の「Code」と書かれたボタンを選択
3. 「Download ZIP」を選択
4. ダウンロードされたzipファイルをデスクトップへ移動
5. zipファイルをダブルクリック
6. ターミナルを開く。
7. ターミナルを活用して、zipを展開して生成されたフォルダへ移動する。(`$ cd Desktop/rails6-heroku-deploy-master`)

# Dockerイメージを用いて、rails6をローカル環境で立ち上げる。
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

## rails6へvueを追加し、ローカル環境で立ち上げる。
1. `$ docker-compose exec web bundle exec rails webpacker:install:vue`を実行する。
2. `$ docker-compose run web yarn add vue@2.6.12 vue-loader@15.9.3 vue-template-compiler@2.6.12`を実行する。
3. routes.rbを編集する。

```ruby:routes.rb
Rails.application.routes.draw do
    # For details on the DSL available within this file, see     https://guides.rubyonrails.org/routing.html
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

