---
title: Git管理型WikiのGollumにマストドン認証を乗せるやつ
tags: gollum
date: 2018-10-18 17:55:00
---


昨日、[アビス丼のWikiページ](https://wiki.abyss.fun)をMediaWikiからGollumに移行しました。

別段何か理由があってMediaWikiをやめたかったというわけではないです。たまたまGollumを使って見たくなって手元で動いているWikiサービスがアビス丼wikiだっただけです。

ホントは別のWikiを立てる計画もあったようななかったような。って感じなんですが、Wiki立てまくってホントに信用できる情報はどこなんじゃい！ってなるのも嫌だったので、とりあえず見送った感じなんです。雑。

今回はとりあえずGollumを動かすまでの手順と、認証にマストドンアカウントを作るところあたりをざっくり紹介しようと思います。

## Gollumについて

Ruby製の軽量なWikiサービスを構築することができます。

記事のバージョン管理にgitを使っているので、バックアップとしてgithubに記事を上げておく。なんてこともできます。便利！

ただし見た目がシンプルすぎるのでカスタムCSS入れてデザイン変える必要が出てきたりとか、認証できるけどログアウトができなかったりとかします。微妙に不便！

まぁ、軽いのとマークダウン使えるのがウリってだけな感じがします。

それとOAuth2認証するのに便利すぎるライブラリであるomniauthをサポートしているのは、マストドンユーザーにとって嬉しいですね。

マストドン関連サービスとしてWikiを立てたい鯖缶各位が(いるかどうかは知らんけど)。いるようであればGollumの導入を検討してみるのもありなんじゃないかと思います。

## Gollumの導入手順

今回の肝であるGollum導入手順について紹介です。

本体だけであれば `gem install gollum` と打つだけですみますが。今回はOAuth2認証とカスタムCSS/JSについても書こうと思ってるのでちょっとややこしい感じです。

### 0. 構築環境

- ubuntu18.04LTS
- rbenv
- gollum本体はgemパッケージインストール
- omnigollumはソースコードビルドしてからインストール
- omniauth-mastodonのCLIENT_ID管理にActiveRecord(sqlite3)を使用
- カスタムCSS/JSのビルド用にNode.js/yarn環境も使用

omnigollumはgollumでOAuth2認証を有効にするためのモジュールで、その中に更にomniauth-mastodonを組み込むことになります。

bundler環境でも動かせそうですが、今回は本家README.mdに沿ってgemから導入する手順で紹介します。

### 1. 作業ディレクトリを用意する。

最初は「このディレクトリ掘ってその中にこのファイルを置いて…」みたいな手順にしようかと思ったんですが。  
面倒なのでGitHubに作業用ディレクトリをひとまとめにしたリポジトリを上げておいたので、そのまま使ってください。

```shell
$ cd ~/
$ git clone https://github.com/wd-shiroma/gollum-work.git
$ cd gollum-work
```

この中に入ってるファイル群をざっくり紹介すると、以下のような感じです。

```
./
+ dist ... 設定ファイル等のサンプルを入れてる
+ mastodon ... OAuth2認証でマストドンを使う時のスクリプトとか入れてある
+ util ... カスタムCSS/JSの作成支援用ユーティリティ(要node.js)
```

### 2. wiki用のリポジトリを作る

最初に述べたようにGollumはgitでバージョン管理をするwikiサービスなので、gitリポジトリを作成します。

```shell
$ mkdir repos
$ cd repos
$ git init
$ cd ..
```

### 3. 必要なaptパッケージを入れる

minimalな環境から導入する想定です。

```shell
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install -y gcc make sqlite3 libsqlite3-dev libjemalloc-dev
```
### 4. rbenvからrubyを入れる

マストドン界隈で今話題のjemallocを使ってるrubyを使えるようにします。  
(gollumくらいだったら大して性能に違いはないけど)

```shell
# rbenv入れる
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ source ~/.bashrc

# rbenv-build入れる
$ mkdir -p "$(rbenv root)"/plugins
$ git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build

# ruby入れる(rubyバージョンは最新のに変えてね)
$ RUBY_CONFIGURE_OPTS=--with-jemalloc rbenv install 2.5.2
$ rbenv global 2.5.2
```

### 5. 必要なgemパッケージを入れる

何も考えず、まとめてインストールしちゃいます。

```shell
$ gem install --no-ri --no-rdoc gollum omniauth omniauth-mastodon mastodon-api activerecord sqlite3 sqlite3-ruby dbd-sqlite3
```

GitHubなど、他の認証も入れたい場合は別途omniauth-githubなど追加で入れてください。

### 6. omnigollumを入れる

GollumのOAuth2プラグインomnigollumは、gemパッケージでも配布されています。  
しかしそのまま入れるとマストドンのロゴが表示されません。
僕の用意したリポジトリからcherry-pickしてロゴを取得してから、ソースコードビルドしてインストールします。

```shell
$ git clone https://github.com/arr2036/omnigollum.git
$ cd omnigollum
$ git add remote guskma https://github.com/wd-shiroma/omnigollum.git
$ git fetch guskma mastodon_logo
$ git cherry-pick guskma/mastodon_logo c58a4f7
$ gem build omnigollum.gemspec
$ gem install omnigollum-*.gem
$ cd ..
```

### 7. config.rbを作成する

作業ディレクトリの `dist` 配下に置いてあるサンプルコードをコピーして利用しましょう。

```shell
$ cp dist/omnigollum-config.rb config.rb
$ vim config.rb
```

マストドン認証を有効にするには48～50行目と53～66行目のコメントを解除してください。

GitHub認証を有効にするには28～34行目と48～50行目のコメントを解除してください。

### 8. CLIENT_ID/SECRETを管理するDBを用意する

今回sqlite3を使用してますが、他のDBを使いたい場合は各自で必要なgemを入れて `mastodon/instances.rb` で接続先DBを直して利用してください。

```shell
$ ./mastodon/create_db.sh
$ ls -l mastodon/stored.db # ファイルが有ることを確認
```

### 9. とりあえず起動させてみる

ここまでくれば実際に動くようになります。

```shell
$ gollum ./repos --config ./config.rb
```

正しく実行されれば `localhost:4567` が待受状態になります。

アクセスすると、初期ページである`/Home`を作成するページにリダイレクトされ、OAuth2認証画面が表示されるはずです。

## おまけの手順

### EX1. サービス化する

必要に応じてサービスとして起動させられるようにしましょう。

```shell
$ sudo cp dist/gollum.service /etc/systemd/system/
$ sudo vim /etc/systemd/system/gollum.service
```

gollumuserを自分のユーザ名に置換してください。

ここまでくれば、サービスとして起動できるようになります。

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl start gollum
```

### EX2. カスタムCSS/JSを導入する

Gollumはデフォルトで、カスタムCSS/JSを利用できる様になっています。

wikiリポジトリの `/custom.(css|js)` がカスタムCSS/JSとして認識されるんですが、直で書くのはダサいので、ビルドするだけのツールを同梱しました。

SCSSやES6以降の書き方が出来るし複数のファイルを一つにまとめることができるので、コレを使ったほうがいいと思います。

**※このutilはnode.jsとyarnを利用しています。今回こちらの導入手順については解説を省略します。**

```shell
$ cd util
$ yarn install
```

ソースコードを監視して、ファイルが更新されたら自動でビルドされるようにしてあります。

```shell
$ yarn run watch
```

`style/*.scss` 及び `javascript/*.js` ファイルを作成してください。

__GollumはjQuery1.7を利用しているので、そのままjQuery記法が使えます__

ビルドされたファイルはWikiリポジトリである `repos` の中に吐き出されます。

デフォルトでtwemojiを適用させるコードだけ用意してあります。煮るなり焼くなり好きにしてください。

## 終わりに

以上でgollumの導入手順は終了です。良きマストドンライフを。

## TIPS

以下は蛇足。

### - omniauthで取得できるmastodonのパラメータについて

Omnigollum::Model::OmnigollumUser#initializeあたりでomniauthのパラメータを取得する処理を入れている

https://github.com/arr2036/omnigollum/blob/master/lib/omnigollum.rb#L16

hashの中身がomniauthの取得したパラメータとなっている。

Omnigollumはその中から厳選して、 `:uid, :name, :email, :nickname, :provider` のパラメータをconfig.rbで使えるようにしている。

https://github.com/arr2036/omnigollum/blob/master/lib/omnigollum.rb#L12

omniauth-mastodonから渡されるパラメータは以下の通り

- :uid ... acct
- :name ... username
- :email ... (nil)
- :nickname ... username
- :provider ... 'mastodon'

Omnigollumではデフォルトで、:nameと:emailをgitのコミット時に使用するようにしているが、:emailが空のためエラーが出てしまう。  
エラーを回避するために、ダミーメールアドレスをデフォルト値としてconfig.rbに設定してやる必要がある。(:default_email)

コミットに必要なuser.nameとuser.emailで使用するパラメータの変更は:author_formatと:author_emailで `user.provider == 'mastodon'` の条件によってマストドンのみ:uidを使用するようにする

### - OAuthプロバイダ選択時のロゴ表示について

Omnigollumビルド前にpublic/images/mastodon_logo.pngを作成しておくことによりこの問題は回避することができる。

### - gollumのファビコン表示について

wikiリポジトリのルートディレクトリに`favicon.ico`を置いた後 **ページを一つ編集なり作成なりしてコミットを一つ進めてやらないと反映されない** かもしれない。  
コマンドラインから`git add`と`git commit`しても意味なかった。

### - ところでh1複数使用するのW3C的にはよくないらしいんですよ。

[h1要素は複数回使って良いのか!? HTML5.1に関するW3CとWHATWGの主張の違い - Togetter](https://togetter.com/li/1058977)

でもマークダウンで何か書く時って `# 大見出し` 結構使うじゃん？

Gollumとか、大見出し複数あるからh1大量にある状態なんだよね。

記事書き始めた時に、これどうしたらいいだろうなぁ。と思ってトゥートしたら軽くバズったし。やっぱみんな気になってるんだよね。

1ページに一つみたいなの、sectionタグとかそういうののほうがいいんじゃないの？とか。

マークダウンで `# 大見出し` タグはh2からにしてくれよ～とか。

…

ここまで考えて思考放棄した。もうh1使いまくっちゃえばいいや～。

紛らわしい定義をしたW3Cが悪い！僕やほかの方々は悪くない！