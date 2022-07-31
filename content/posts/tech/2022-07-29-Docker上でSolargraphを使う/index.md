---
title: "Docker上でSolargraphを使う"
description: ""
<!-- date: 2022-07-29T23:00:24+09:00 -->
date: 2022-07-31T00:00:00+09:00
categories: ["技術関連"]
tags: ["Ruby", "Docker"]

<!-- draft: true -->

---

LSPが登場し様々なLanguage Serverが使えるようになってから、開発する際にはとりあえずLanguage Serverを用意して使えるようにしています。

ローカル環境で使っているぶんにはいいですが、Dockerで構築した環境にて動かすのは少し面倒なので自分が使っているやり方をメモを残しておこうと思います。


## ゴール

Docker Composeで構築した環境上で動作するSolargraphを外から使えるようにする。

## 環境

過去に自分で使っていた環境の一部を切り出して持ってきているので、この記事の内容をコピペしても動かない可能性があります。あくまでどうやったら動く可能性があるかについての参考としてお使いいただけますと幸いです。

ファイル構成、`Dockerfile`、`docker-compose.yml`はそれぞれ以下の様になっているものとします。

```sh
$ tree -L 1
.
├── api    # Railsはこの下
├── docker # Dockerfileの保存場所
└── docker-compose.yml
```

```Docker
# docker/api/Dockerfile
FROM ruby:3.0.3

RUN apt-get update -qq && apt-get install -y build-essential libpq-dev

ENV APP_ROOT /app
WORKDIR $APP_ROOT

COPY Gemfile $APP_ROOT
COPY Gemfile.lock $APP_ROOT
RUN bundle install

```

```yml
# docker-compose.yml
version: '3'

services:
  api:
    build: ./docker/api
    volumes:
      - ./api:/app
      - bundle:/usr/local/bundle
    command: /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    ports:
      - '23000:3000'
    tty: true
    stdin_open: true
volumes:
  bundle:
```

DockerfileにENTRYPOINTやCMDが書いてありませんが、Railsでよくある`server.pid`を削除するシェルスクリプトや、サーバー起動用のコマンド等であればあっても問題ないと思います。

## Solargraphを動かせるようにする

まずSolargraphを導入していない場合はGemfileに追記をしてインストールを行います。

```ruby
group :development do
  gem 'solargraph', require: false
end
```

次に、`docker-compose.yml`を以下のようにしてSolargraph用のサービスを用意します。共通化せずにRailsのサービス設定をコピペして、変更が必要な部分だけ適宜書き換えても大丈夫です。

恐らく使用する環境によると思いますが、筆者の環境ではエディタを閉じるとSolargraphのサービスが正常終了してしまうので、`restart: always`をつけるようにしています。

```yml
version: '3'

x-api-base: &api-base
    build: ./docker/api
    volumes:
      - ./api:/app
      - bundle:/usr/local/bundle
    tty: true
    stdin_open: true

services:
  api:
    <<: *api-base
    command: /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    ports:
      - '23000:3000'
  solargraph:
    <<: *api-base
    command: bundle exec solargraph socket --host=0.0.0.0 --port=7658
    ports:
      - '17658:7658'
    restart: always

volumes:
  bundle:

```

これで`docker-compose up`した際にRailsアプリと同様の環境でSolargraphサービスが立ち上がるようになります。

あとは使用しているLSPクライアントに、Solargraphを使用する場合は`localhost:17658`との通信を行うような設定を記述することで使用ができるようになります。

例えばvim-lspを使っている場合であれば、以下のように記述をすることで接続ができるようになると思います。

```vim
if strlen(system('docker ps -q -f name="solargraph"')) > 0 
    au User lsp_setup call lsp#register_server({
        \ 'name': 'solargraph-docker',
        \ 'tcp': "localhost:17658",
        \ 'initialization_options': {"diagnostics": "true"},
        \ 'whitelist': ['ruby'],
        \ })
endif
```

接続設定を書けたら実際にファイルを開いて試しに書いてみましょう。

正しく設定が行えていれば基本的な構文の補完は効くと思いますが、他のファイルで定義したクラス等については補完が効かないと思います。  
(補完が効く場合は下記の設定は読み飛ばしてしまって大丈夫です。)

LSPの定義内容や実装方法についてほとんど理解していませんが、Solargraphへのメッセージにファイルパスが含まれており筆者の環境では絶対パスが使用されていることを確認しています。

エディタはローカル環境、SolargraphはDocker環境と違う環境で動いているのでほとんどの場合でファイルの所在に関する情報はうまく伝わらず、補完等一部の機能がうまく動きません。

ローカルのパスとDocker上のパスを一致させれば問題ないですがめんどくさいので、筆者は以下のようなコマンドを叩きシンボリックリンクを貼って対応しています。

```sh
docker-compose exec solargraph mkdir -p `pwd`
docker-compose exec solargraph ln -s /app `pwd`/api
```

恐らくこれでいい感じにSolargraphが動くようになると思います。


## 終わりに

今回はSolargraphを動かせるようにしましたが、TCPsocketでの通信が行えるLanguage Serverであれば同様の方法で動かせるようになると思います。

他の通信方法ではまだうまくできていないのでもう少しLSPについて勉強したり、情報収集をしていい感じに動かせるようにしたいです。

この記事が快適なRubyライフを送る手助けになれば幸いです。


## おまけ

Docker環境だとdownする度にシンボリックリンクを張り直したり、GemのYARDドキュメント生成のコマンドをいちいち叩くのは面倒だと思います。

なので筆者は以下の様にMakefileで動かせるようにしています。Makefileおすすめです。


```make
EXEC_BASE:= docker-compose exec

lsp-init: solargraph-init

solargraph-init: rdoc2yard yard-gems
	$(EXEC_BASE) solargraph mkdir -p `pwd`
	$(EXEC_BASE) solargraph ln -s /app `pwd`/api

rdoc2yard:
	$(EXEC_BASE) solargraph bundle exec solargraph bundle

yard-gems:
	$(EXEC_BASE) solargraph bundle exec yard gems

.PHONY: lsp-init solargraph-init rdoc2yard yard-gems

```
