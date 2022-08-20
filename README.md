
# blog (チラ裏)

https://kaaaanon96.github.io/blog/

## 構成周り

Hugoで構築。  
テーマは[Mainroad](https://github.com/Vimux/Mainroad)を使わせていただきました。

## ローカルで動かすまで (macOS)

1. `$brew install hugo`でHugoのinstall
2. `git clone --recursive [このレポジトリのURL]`でsubmodule(サイトのテーマ)ごとまとめてclone
3. `hugo server`した後「http://localhost:1313/blog/」にアクセスすることでページの確認ができる。
4. 終わり

## デプロイ周り

GitHub Actionsで自動的にbuildしてくれるようになってるので記事を書いてローカルで確認できたらcommitしてpushするだけ。

[GutHub Actionの参考](https://gohugo.io/hosting-and-deployment/hosting-on-github/#readout) (公式)

