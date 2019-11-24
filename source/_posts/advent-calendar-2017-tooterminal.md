---
title: Tooterminalのススメ
date: 2017-12-7
tags:
---

----

この記事は[Mastodon 2 Advent Calendar 2017](https://adventar.org/calendars/2265)の7日目の記事として投稿しています。

----

今回の記事では、Tooterminalについて良く知らない人向けに、さらっと概要の説明から、使用用途別のコンフィグ投入例などを紹介していきたいと思います。

## Tooterminalについて

### これはなに？

Mastodon専用Webクライアントです。  
[Naumanni](https://github.com/naumanni/naumanni)や[Halcyon](https://halcyon.social)のような、Webブラウザで動かすことのできるクライアントの一つです。

### 公式クライアントと何が違うの？

同じWebブラウザなら公式が提供してるクライアントがありますよね。  
それでも他のクライアントが出るのは、公式だけではカバーしきれない部分がいくつかあるからなわけで。

- 複数のインスタンスをまとめて表示したい！
- TwitterのUIが好き！
- その他機能を充実させたい！

みたいな、独自の思想を持って作られている感じはあります。

Tooterminalはそもそも~~業務中にマストドンをしたい~~環境に溶け込んだクライアントを目指して作られたものです。  
そこから色々と機能を盛り込んでいくうちに、特定の用途によっては、公式クライアント以上に使い勝手の良いクライアントに成長したと思います。

### Tooterminalで何が出来るの？

以前の記事にも書きましたが、他のクライアントでは出来ないことを中心に改めて書いておきたいと思います。

- LTL、HTL、通知TLなど、複数TLをまとめて表示
- 豊富な正規表現フィルター機能
    - 複数フィルターの設定
    - 非表示フィルター以外のフィルター(強調フィルター)
- クライアント名、Website名の変更
- トゥートした当時のLTLを表示する機能(max_idの指定)
- ショートカットキーを組み合わせたファボ、ブースト機能
- アカウント情報の詳細表示(登録日、登録してから何日か、等)

具体的にどういう用途で使えばいいのかっていうのを今回の記事のメインにしていますので、もう少し読み進めていただければ幸いです。

### Tooterminalってどうやって使うの？

とりあえず[Tooterminal](https://wd-shiroma.github.io/tooterminal)にアクセスしてみましょう。

![なんじゃこりゃな感じのやつ](init.png)

チュートリアル的なものは無く、いきなり真っ黒な画面が表示されます。  
~~チュートリアル作るのが面倒くさかったわけではありません。本当です。~~

初期状態ではログインページは開かないし何が出来るか良く分からない画面になってます。

ここからはコンソール操作で、コマンドを入力していくことで少しずつ機能を設定していくイメージです。

このコンソールは、CiscoIOSという、Cisco社が開発しているネットワーク機器に実装されているファームウェアのコンソール画面を模した作りになっています。  
なので、操作方法はCisco社製NWスイッチ/ルータを触ったことがある人であれば、抵抗感少なく使い始めることが出来ると思います。

ネットワークエンジニアならCCNAを取得するのに触ったことがある人も多いかと思いますが、そうでない人は難しいところもあると思います。

どういうコマンドを打てばいいのか、という事については、[GithubリポジトリのREADME.md](https://github.com/wd-shiroma/tooterminal/blob/gh-pages/README.md)を参照してください。  
~~書くのが面倒くさかったわけではありません。本当です。~~

## 用途別、設定投入例

基本的な使い方は、[GithubリポジトリのREADME.md](https://github.com/wd-shiroma/tooterminal/blob/gh-pages/README.md)に書いてあるので、今回の記事では「公式クライアントでは出来ないこういう使い方が出来るよ！」な視点で初期状態からコンフィグを投入していく流れを紹介してみようと思います。

### クライアント名を変更してトゥートするクライアントとして利用する。

たまに見かけますが、クライアント名を自由に設定してる人いますよね？

![変わったクライアント名の人の例](appname.png)

Tooterminalでもこのように自由なクライアント名を設定することが出来るようになっています。

```console
Tooterminal# 
Tooterminal# configure terminal 
Tooterminal(config)# 
Tooterminal(config)# # クライアント名の設定
Tooterminal(config)# application name '深界四層 ナナチのアジト' ←スペースを挟む場合はクォーテーションで括る
Tooterminal(config)# 
Tooterminal(config)# # クライアントのURL設定
Tooterminal(config)# application website http://miabyss.com ←スペースがない場合はクオーテーション省略可能
Tooterminal(config)# 
Tooterminal(config)# exit
Tooterminal# 
Tooterminal# # 設定の確認
Tooterminal# show running-config 
{
    "application": {
        "name": "深界四層 ナナチのアジト", ←名前が変わっていることを確認
        "website": "http://miabyss.com", ←URLが変わっていることを確認
        "uris": "urn:ietf:wg:oauth:2.0:oob",
        "scopes": {
            "read": true,
            "write": true,
            "follow": true
        }
    },
    "terminal": {
        "length": 0
    },
    "instances": {
        "terminal": {
            "logging": {
                "favourite": true,
                "reblog": true,
                "mention": true,
                "following": true
            },
            "monitor": ［]
        }
    }
}

Tooterminal# 
Tooterminal# # 保存をすると、次回起動時にも同じ設定を利用することができます。
Tooterminal# write memory 
Building configuration...
[OK]
Tooterminal# 
Tooterminal# instance abyss
Input instance domain: abyss.fun

...初回アクセス時はここでログイン画面に遷移する...

Tooterminal# 
Tooterminal# instance abyss
Hello! ぐすくま@船団キャラバン @guskma
guskma@abyss.fun# 
guskma@abyss.fun# toot
```

この状態でトゥートをしてみると、クライアント名が「深界四層 ナナチのアジト」に変わります。

クライアントURLのリンクを踏めるクライアントを利用しているようであれば、website で設定したURLに遷移することもできます。

### LTLとHTLをまとめて見るためのクライアントとして利用する。

マストドンを使っていると必ず出てくる要望ですが。

__「HTLとLTLをまとめた一つのTLが欲しい！」__

僕もそう思います。なので、実装しました！

```console
Tooterminal# 
Tooterminal# configure terminal 
Tooterminal(config)# 
Tooterminal(config)# # トゥートを見やすくするための設定
Tooterminal(config)# # (入れなくても動くが、入ってると使いやすくなる)
Tooterminal(config)# 
Tooterminal(config)# # ユーザーアイコンを表示(動画GIFアイコンが常に動く設定)
Tooterminal(config)# instances status avatar animation 
Tooterminal(config)# 
Tooterminal(config)# # トゥート間に区切り線を入れる設定
Tooterminal(config)# instances status separator 
Tooterminal(config)# 
Tooterminal(config)# # 貼付画像のサムネを常に表示する
Tooterminal(config)# ins status thumbnail 
Tooterminal(config)# 
Tooterminal(config)# exit
Tooterminal# 
Tooterminal# # 保存はこまめにしようね！
Tooterminal# write memory 
Building configuration...
[OK]
Tooterminal# 
Tooterminal# instance abyss
Input instance domain: abyss.fun

...初回アクセス時はここでログイン画面に遷移する...

Tooterminal# 
Tooterminal# instance abyss
Hello! ぐすくま@船団キャラバン @guskma
guskma@abyss.fun# 
guskma@abyss.fun# # ホームタイムラインのトゥートは背景色を青くする。
guskma@abyss.fun# access-list 10 permit 'USER streaming updated'
guskma@abyss.fun# 
guskma@abyss.fun# # ローカルタイムラインの再生開始
guskma@abyss.fun# terminal monitor local
local Streaming start.

guskma@abyss.fun# 
guskma@abyss.fun# # ホームタイムラインの再生開始
guskma@abyss.fun# terminal monitor home 
home Streaming start.

guskma@abyss.fun# # 通知タイムラインの再生開始
guskma@abyss.fun# terminal monitor notification 
```

![HTLとLTLが一緒に流れてる例](ltl_htl.png)

ローカルタイムライン、ホームタイムライン、通知タイムラインがまとめて流れるTLになりました。

なお、ページ読み込み時は `terminal monitor` が実行されていない状態に戻ってしまいます。  
つまり、次回アクセス時にも再度 `terminal monitor` を打たなければならないということになります。

次回起動時から `terminal monitor` を自動実行してもらうためには、URLにパラメータを付加してやります。

```
https://wd-shiroma.github.io/tooterminal/?instance=abyss&terminal=local,home,notification
```

- instance=<インスタンス名>
    `tooterminal# instance <インスタンス名>`
    を自動発行します。
- terminal=<terminal types>
    `account@example.com# terminal monitor <terminal type>`  
    を自動発行します。
    カンマ区切りで入力することで、複数の `terminal monitor` を発行します。

### 「んなぁ」に即時反応できるクライアントとして利用する。

皆さん、TLに流れる「んなぁー」に反応したくなったことありますよね？

![んなぁー](nnaa.png)

しかし仕事中でTLをずっと見ていることはできない！なんてことはありませんか？

こんな時に、TLのキーワードフィルター機能である `access-list` を使えばさまざまな方法でフィルター通知をしてくれるようになります。

なお、「んなぁーって何？？？？？？？？？」と思われている方もいると思いますので、知りたい方は、本日[mstdn.jpのアドベントカレンダーで投稿したもう一つの記事](/2017/12/07/advent-calendar-2017-jp/)をご確認ください！！

宣伝失礼しました。

```console
guskma@abyss.fun# 
guskma@abyss.fun# # キーワードの設定と背景色の変更
guskma@abyss.fun# access-list 5 permit /んな[あぁ]/ color yellow 
guskma@abyss.fun# 
guskma@abyss.fun# # デスクトップ通知の設定(画面右下にポップアップが出るやつ)
guskma@abyss.fun# access-list 5 add notification 
guskma@abyss.fun# 
guskma@abyss.fun# # 合成音声をしゃべらせる（女性音声で「んなぁ」と流れる）
guskma@abyss.fun# access-list 5 add voice んなぁ
guskma@abyss.fun# 
guskma@abyss.fun# # ローカルタイムラインの監視を開始
guskma@abyss.fun# terminal monitor local
guskma@abyss.fun# 
guskma@abyss.fun# # 設定の確認
guskma@abyss.fun# show access-list 
Standard Status access list 5
    permit regexp /んな[あぁ]/, color yellow, notification, voice "んなぁ"
guskma@abyss.fun# 
```

これで画面を見ていなくても「んなぁ」が流れてきたら背景色黄色、デスクトップ通知、合成音声「んなぁ」が流れるフィルターが完成しました。

### その他の細々としたTIPS

項目として取り上げるほどではないが、使ってて便利(？)なコマンドの紹介をしておきます。

- ローカルタイムラインを遡る
    トゥートをダブルクリックすると、トゥートの詳細が表示されます。
    その中にある `> check the LTL of the time.` をクリックすると、当時のLTLを閲覧することができます。
    ![show status id ...](sh_stat_id.png)
- 連投する
    Tooterminalは公式クライアントと同様にCtrl+Enterで投稿できるわけですが、Ctrl+Enterを押してからPOSTが完了するまでの間、投稿メッセージはそのままで、特にキー入力制限などを設けていません。
    そのため、Ctrl押しっぱなしにしてEnterを連打することで、連投することが可能です。
    (誰が使うかは不明ですが、特に直す予定ありません。仕様です)

## 今後の追加予定機能

正直なところ、[abyss.fun](https://abyss.fun)の運営で若干忙しくなってきた今日この頃です。

追加したい機能はいろいろとあるので、今までから比べると更新は激落ちしますが、以下の機能は実装したいなぁとは思っています。

- リスト機能
- 設定およびACLのインポート/エクスポート機能
- 通知音の差し替え(ポコポコに近づけるorんなぁーする)

僕の業務＆インスタンス運営が忙しくならない以上は開発続いていくと思います。新しいMastodonの機能にもついていこうと思います。

半分以上僕が使いたいだけのクライアントですが、この記事を見て興味が持ってくれる人が出てきてくれたらうれしいです。

今日のアドベントカレンダーは以上です。  
次回はYukiya│Naycharlさんのアドベントカレンダーです。ありがとうございました。
