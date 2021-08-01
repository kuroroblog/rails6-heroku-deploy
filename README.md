# rails6-heroku-deploy
docker + vue + rails6 + herokuを実装する。

# 各種サービスのversion

| サービス | version |
| ------------- | ------------- |
| docker  | クライアント(20.10.5), サーバー(20.10.5)  |
| vue  | 2.6.12  |
| rails  | 6.1.4  |
| heroku | heroku/7.56.1 darwin-x64 node-v12.21.0 |

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

参考 : https://www.youtube.com/watch?v=ltDdZAJli8c
