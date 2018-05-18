---
title: "HoiとSlackで内緒話をする"
date: 2015-03-01
comments: true
tags: [ "hoi", "golang", "slack", "takosan" ]
---

社内のチャットツールをSlackに移行中です。Slack使いやすくて満足度高いのですが、クラウドサービスゆえにこれまで社内IRCサーバ上でやりとりしていた情報の一部を載せてはいけなくなりました。

以前つくった[「ほい、これ」ってファイルを渡せる Hoi というツール](/blog/2014/07/17/hoi/)を使うことで外部から参照できないページのURLをやりとりに使えるのですが、やりとり用にファイルが必要だったり、URLを別途通知するという手番が面倒になってきました。(GH:EのGistでも同様ですね)

- [monochromegane/hoi](https://github.com/monochromegane/hoi)

面倒は解決しましょう。以下、Hoiのｿﾘｭｰｼｮﾝです。


## メッセージをやりとりする

今までのファイルやりとりに加え、メッセージにもURLを付与できるようになりました。
使い方は今までと同じで、引数が存在しないファイルの場合にメッセージと見なします。

```sh
$ hoi message
```

以下のようなURLが出力されます。

```
http://192.168.0.103:8082/uqhrip3fvy71x791s00mj2r24kb8yiwu/message.txt
```

message.txtにメッセージが記載されています。引数を複数にしたり、`""`で囲んで改行込にしてもOKです。


## URLを通知する

最後に`@account`をつけるとSlackにURLを通知できるようになりました。便利。

```sh
$ hoi message @you
```

通知にはSlackAPIもしくは[takosan](https://github.com/kentaro/takosan)が使えます。チームで使う場合は takosan があったほうが便利だと思います。たこさん最高。

こんな感じで通知されます。

![hoi-notification](/images/2015/03/hoi-notification.png)



## 設定

なお、通知先に関する設定は`~/.hoi/config.json`で行います。

```json
{
  "notification": {
    "from": "YOUR SLACK ACCOUNT",
    "to":   "takosan",
    "host": "TAKOSAN HOST NAME",
    "port": TAKOSAN PORT NUMBER,
  }
}
```

その他設定はこちらを参考のこと

- [Hoi Configuration](https://github.com/monochromegane/hoi#configuration)

---

これでクラウドサービスに載せたくない情報をさくっとこそっとやりとりできるようになりました。

インストールもさくっとできるよう各種揃えてます。よければご利用ください。

- `go get github.com/monochromegane/hoi/...`
- `brew tap monochromegane/hoi && brew install hoi`
- [バイナリはこちら。Windows/Linuxあります](https://github.com/monochromegane/hoi/releases)

