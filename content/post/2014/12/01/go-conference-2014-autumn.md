---
title: "Go Conference 2014 autumn に行ってきました"
date: 2014-12-01
comments: true
tags: golang
---

11/30に開催された[Go Conference 2014 autumn](http://gocon.connpass.com/event/9748/)に行ってきました。所感メインですが忘れないうちにまとめておきます。

<br />
---

# キーノート

## Simplicity is Complicated

**Rob Pike**氏のキーノート `Simplicity is Complicated`はGo言語が`単純さ`を提供するために、どのように設計されているのかGo言語を構成する各要素の例を挙げながら説明されていました。

可読性と表現力のバランスを取りながら単純で組み合わせ可能な機能を提供するというのは、とても大変なことだと思います。
機能追加という複雑さの導入は誘惑だからです。このあたり今後GitHubへの移行でコントリビュートしやすくなる中でどう進んでいくのか興味深いですね。

それから、今回のキーノートは1時間ありましたが、資料は日本語に訳されたものが常に投影されている状態で、英語のヒアリングが得意じゃない自分はとても助かりました。
聞きかじった話だとこれはPike氏からの発案とのことで、ヒアリングや翻訳という手段に惑わされず、内容という本質を提供したいというPike氏の考えだったんじゃないか、こういうところもまた、Goの思想と一貫しているなと「勝手に」解釈しました。（英語がんばります）

写真は、Rob Pike氏と、複雑になりすぎたGopherの図です。

![rob\_pike](/images/2014/12/rob_pike.png)
![gopher](/images/2014/12/gopher.png)

<br />
## Goに入ってはGoに従え

鵜飼さんのキーノートは[Goに入ってはGoに従え](http://ukai-go-talks.appspot.com/2014/gocon.slide#1)ということで、Googleのreadability approverという役を通して得た、Goらしく書くノウハウを紹介してくれました。

今年1年間Go言語を書いたり、各種ドキュメントを参考にしたりしてたことが簡潔にまとめられていて、納得感の高いキーノートでした。
特に[エラーまわりの使い分け](http://ukai-go-talks.appspot.com/2014/gocon.slide#14)は自分の中で指針がなかったので特に参考になりました。

Goらしく書くことに関して各種ドキュメントやブログエントリがありますが、方針が同じでどれも納得しやすいというのもGo言語のコンセプトがそれを助けているのかなと感じています。

<br />
---

# 印象に残ったプレゼンテーション

プレゼンテーションでは、@y\_matsuwitterさんの[Golang @ISUCON](https://speakerdeck.com/ymatsuwitter/golang-at-isucon)や、@groseの[NSQの話](http://www.slideshare.net/guregu/nsqcentric-architecture-gocon-autumn-2014)が特におもしろかったです。もっとGoroutineを使いこなしていきたい。

それから、@methaneさんの[pprofの話](http://www.slideshare.net/InadaNaoki/gocon2014-pprof)も収穫がありました。[pt](https://github.com/monochromegane/the_platinum_searcher)をもっと速くするのに今度試してみようと思います。

<br />
---

# 楽しいカンファレンスでした！

技術的なエントリについてはまた別途あげますが、ひとまず所感です。
とても素晴らしいカンファレンスを開いてくれた運営の方々、登壇していただいた方々、ありがとうございます！

やっぱりGoはいいですね。

@qt\_luigiさんの[Golang JP Communityの紹介](https://docs.google.com/presentation/d/1UTi4uqt4sOrQ1dHJE0y8UB9BR9iqDK7dLP57QAfrOX4/pub?slide=id.g4f1d3881c_00)でFukuoka.goが紹介されたのもうれしかったです。

今回は登壇するネタがなかったのが悔しいですが、刺激や収穫をもらったので福岡でまた次回に向けたGo研鑽を積んでいきます。

<br />
---

カンファレンス内容についてはこちらがよくまとめられているのであわせてご覧ください。

- [Go Conference 2014 autumn を終えて #gocon](http://ymotongpoo.hatenablog.com/entry/2014/12/01/080131)
- [\[まとめ\] Go Conference 2014 autumn](http://qiita.com/yoheimuta/items/81763237dc41ae33e891)


