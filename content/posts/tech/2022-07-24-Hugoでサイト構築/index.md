---
title: "Hugoでサイト構築"
description: ""
date: 2022-07-24T13:21:27+09:00
categories: ["技術関連"]
tags: ["Hugo"]

<!-- draft: true -->

---

Go言語製の静的サイトジェネレーターである「[Hugo](https://gohugo.io/)」を使ってこのブログの作成を行ったので、作成の手順等を残しておこうと思います。

筆者はMac環境で行ったので記事の内容もそれに準じます。  

## ゴール

GitHub Pagesにてカスタムドメインを使用せずにサイトの公開をする。  
また、公開をする為のビルドをGitHub Actionsで自動化できるようにする。



## サイトの作成から設定まで

全体の大まかな流れは公式の[Quick Start](https://gohugo.io/getting-started/quick-start/)を参考にしました。

まずHugoのコマンドが無いと始まらないのでインストールします。

```sh
$ brew install hugo
```

次にインストールしたHugoでサイトの作成を行います。  
好きなディレクトリで以下ようにコマンドを叩きます。「**test-site**」の部分は自分の好きな名前で大丈夫です。  

```sh
$ hugo new site test-site
```

これでサイトの作成が完了しました。
実際に作成されたサイトの雛形が以下になります。

```sh
$ tree
.
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── public
├── resources
│   └── _gen
│       ├── assets
│       └── images
├── static
└── themes

```

サイトの雛形ができたので、Gitの初期化もやっておきます。  
レポジトリは好きな名前で作成してついでにpushできるようにしておくといいと思います。  
(GitHub Pagesでの公開を行う際に無課金でprivateレポジトリだと公開ができないような気がしたのでそこら辺は気をつけてください。)

```sh
$ cd test-site
$ git init
```

次にサイトのテーマを入れていきます。  
サイトのテーマは[こちら](https://themes.gohugo.io/)から好きなものを探して使用します。

今回はこのブログの構築に使用した[Mainroad](https://github.com/Vimux/Mainroad)を使用します。

テーマの導入はGitのsubmoduleを使うのが推奨されているみたいなので、自動で生成されたthemesディレクトリ以下に下記のコマンドで入れます。

```sh
$ git submodule add https://github.com/vimux/mainroad themes/mainroad
```

テーマのインストールができたら、次にサイトの設定を行います。  

Mainroadは、[公式ドキュメント](https://mainroad-demo.netlify.app/docs/getting-started/)によると細かな設定を追加しなくても使えるらしいので、使用するテーマの指定のみconfig.tomlに追加します。

```toml
# デフォルトの設定以下に追加
theme = "mainroad"
```

テーマの設定までできたら、一度サーバを起動してみます。  
サーバの起動は以下のコマンドでできます。

```sh
$ hugo server
```

まだ記事を書いていないので細かい部分の確認はできませんが、ヘッダー等から選択したテーマのデザインでそれっぽいページになっていることが確認できると思います。

これでひとまず記事を書くための準備が完了しました。


実際にサイトの公開をするには、`config.toml`をもっと書き換えてデザインや細かな挙動の変更を行ったり自分で書いたcssをあてたりと色々あると思いますが、ここでは割愛させていただきます。  

設定の内容や詳細については[こちら](https://gohugo.io/getting-started/configuration/)で確認できます。  
また、使用しているテーマによって独自の設定があったり、設定ファイルの置き方等が変わる場合があるみたいなので、テーマのREADMEやドキュメントを確認して進めるのがいいと思います。  
筆者はREADMEのexampleをとりあえずまるごとコピーしてからいらないものを消したり設定を書き換えたりする感じで進めました。

一部分にはなりますが、筆者は現状こんな感じの設定を書いています。

```toml
# ページネーションする際の1ページあたりの記事数
paginate = 6
# 日本語のカウントできるように
hasCJKLanguage = true
# サマリーの文字数を80文字に
summaryLength = 80

# メニューバーの設定
[[Menus.main]]
  identifier = "home"
  Name = "Home"
  URL = '/'
  weight = 1
[[Menus.main]]
  identifier = "posts"
  Name = "Posts"
  URL = "/posts/"
  weight = 2
[[Menus.main]]
  identifier = "about"
  Name = "About"
  URL = '/about'
  weight = 3

[Params]
  # 記事内の目次を非表示に
  toc = false
  # GoogleFontsでNoto Sans JPを入れるように
  googleFontsLink = "https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@500;700&display=swap"
  # 独自cssの指定(フォントサイズ等を変えてる)
  customCSS = ["css/custom.css"] # Include custom CSS files

[Params.style.vars]
  # Mainroadのテーマカラー変更
  highlightColor = "#a8c088"

  # Font familyの変更
  fontFamilyPrimary = "'Noto Sans JP', 'Open Sans', Helvetica, Arial, sans-serif"
  fontFamilySecondary = "SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New', monospace"

# 配置・表示項目設定
[Params.sidebar]
  home = "right"
  list = "right"
  single = "right"
  widgets = ["recent", "categories", "taglist"]
```


## 記事を書く

とりあえずサイトのデザインがぽくなったので記事を書いてみます。  
記事用ファイルの作成は以下のコマンドからできます。 

```sh
$ hugo new [記事名など].md
```

このコマンドを叩くと、`content/[記事名など].md`という形でcontentディレクトリ以下にファイルが生成されます。

new以降の部分はパスの指定等もできるので自分が記事を管理しやすい形で指定をするといいと思います。
ただし、content以下のファイルパスが記事のURLになるので命名には少し注意したほうがいいかもしれません。

今回は`content/posts`以下にtest記事を作成しようと思うのでコマンドは以下のようになります。

```sh
$ hugo new posts/test.md
```

生成したファイルについてですが、設定を変更していない場合先頭は下のようになっていると思います。

```md
---
title: "Test"
date: 2022-07-24T20:23:02+09:00
draft: true
---

```

この`---`で囲まれた部分は[Front Matter](https://gohugo.io/content-management/front-matter/)と言い、ページのタイトルやカテゴリー、公開設定等もろもろの設定をここに記述したりします。  

コマンドでファイルを生成した場合`archetypes/default.md`をテンプレートとして作成されているため、 
ドキュメント等を参考に予めFont Matterの設定などを記述しておくことで自分の使いやすいようにすることができます。


本文は以下のようにFornt Matter以下に記述することになると思います。  

```md
---
title: "Test"
date: 2022-07-24T20:23:02+09:00
draft: true
---

test


```

実際に本文を書き込んだら、サーバを起動して書いた記事の確認をしてみましょう。

この時、先程のようにサーバを起動すると恐らく作成した記事の確認ができないと思います。  
これは`draft: true`の部分が原因で、オプション無しのサーバ起動だと下書き状態の記事を読み込んでくれません。

対応としては、今の筆者(初心者)が知る限りだと

1. `hugo server -D` と言う感じにオプションをつけて起動する。
2. `draft: true`の部分を`false`にするか、コメントアウト/削除をする

の2つになります。  
本番に上げる際は当然draft設定は解除する必要があるので気をつけましょう。

自分の好きな方で対応を行いページにアクセスすると記事の確認ができると思います🎉🎉🎉


## 公開する

実際に記事がかけたら、GitHub Pagesで公開をしてみます。

まず、GitHub Pagesでの公開をするにあたり設定を変更する必要があるので、`config.toml`を以下のように変更します。  
デフォルトの設定から一切変更を行っていない場合は、ついでに他の部分も変更しておくといいと思います。

```toml
baseURL = 'https://[GitHubのユーザー名].github_.io/[レポジトリ名]/' # 自分に合わせた形で変更
languageCode = 'en-us' # 言語(日本語ならja)
title = 'My New Hugo Site' # ブログ名
theme = "mainroad"
canonifyurls = true # 追加
```

次に、GitHubにpushしたら勝手にビルドをしてくれるようにします。自分でビルドするのはめんどくさいので。

ありがたいことにGitHub Actionsの例が公式ドキュメントに書いてあるので[こちら](https://gohugo.io/hosting-and-deployment/hosting-on-github/#branches-for-github-actions)を参考にワークフローを書きます。  
~~ちなみに上記リンクのページにはGitHub Pages公開に必要なことが大体記述されているためこの記事の「公開する」以降の部分はなくても問題なかったりします。~~

公式ドキュメントの内容をそのまま書いた場合pushをトリガーとしてビルドした結果を`gh-pages`ブランチに自動で置いておいてくれます。

ワークフローが書けたらGitHubにpushしてみましょう。  
pushしてから少し待てば`gh-pages`ブランチに見覚えのないファイル群が生成されているはずです。

実際にファイルが生成されているのが確認できたら、あとはGitHub Pagesの設定を行うだけです。

レポジトリのSettingにてPagesの項目を選択し、Source設定にてブランチを`gh-pages`、フォルダを`root`と指定しsaveボタンを押せば設定完了です。

設定が正しく行えていた場合、baseURLにて指定したURL、またはGitHub Pagesの設定ページ上部にかかれていたリンクにアクセスすることで作成したサイトの確認ができると思います。

めでたしめでたし。

## 終わりに

実際にブログの構築とこの記事を書くところまでやってみた感じですが、正直なところ初期の設定は種類の多さ等、少し面倒な部分はありました。  
一方でカスタマイズが色々ときくみたいなので運用を続けるにあたって気になる部分ができたら好き勝手できるのはとてもいいのかなと思います。

記事を書く際は、普段自分が使用しているエディタでドキュメントを書くのと同じ感覚で書いていける上に、記事を書き換えるとすぐにページに反映されるためサクサクと書き進めることができるのはとても良かったです。


## 参考(公式ドキュメント以外)

- https://terashim.com/posts/create-hugo-blog-and-customize-mainroad-theme/


---

