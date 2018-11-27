+++
date = "2018-11-27T16:58:11+09:00"
title = "Go Conference 2018 Autumnで発表してきた(3年ぶり3回目)"
tags = [ "golang", "gocon" ]
+++


11/25に[Go Conference 2018 Autumn](https://gocon.jp/)というイベントで`フィードバック制御によるGoroutine起動数の最適化`を発表してきました．

Goでの開発時に常に悩んでいたGoroutine起動数に関してフィードバック制御という切り口でKaburayaアーキテクチャと題して解決策を検討したものです．
直前に[WSA研向けに研究として再定義した](/blog/2018/11/25/wsa_3_kaburaya/)ことでGoroutineにこだわらない，より汎用的な問題に対するアプローチとして検討し直すことができ，まだ最適解は模索中ですが，面白いと思っていただける発表ができたのではと考えています．

<script async class="speakerdeck-embed" data-id="3fb10a250cd44b8799eac767baec5cc9" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fkaburaya" title="monochromegane/kaburaya" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/kaburaya"&gt;monochromegane/kaburaya&lt;/a&gt;</iframe>

歴史ある制御工学という分野の手法を用いることで蓄積されたノウハウを活用できると考えていましたが，発表後に早速有意義なアドバイスなど頂くことができ，今後の精度，速度向上並びに汎用化に向けて面白くなってきそうです．

ちなみに開発にあたってGoランタイムを調べていた時の資料も`#gocon`ハッシュタグで放流したところ，こちらも好評でしたので改めて置いておきます．

<script async class="speakerdeck-embed" data-id="4046d92f81c545ab99e5b974c874d4e0" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

## 発表を終えて

GoConでの登壇は今回で3回目となりました．ここ数年，自分のキャリアや興味範囲が広がっていく中でも，Go言語はずっと相棒として付き合ってくれる言語でしたし，Go言語を中心に色々な付き合いができていると感じています．
そしてその契機となったGoConと，これを継続して開催してくれている運営の方々に本当に感謝いたします．

主催するFukuoka.goでも微力ながらコミュニティの活発化に繋がるよう引き続き"[楽しみながら](https://press.forkwell.com/entry/2018/10/01/community_lovers)"やっていきたいと思います．

次は海外カンファレンスで登壇するぞう．

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">ホテル到着。移動で朝早かったので懇親会途中までとさせていただきました。お誘いありがとうございます。色々な話が聞けて楽しかったです！ カンファレンスも濃い話から体系的な整理がなされたものまであり来れて良かったです。自分の登壇内容も反応いただけて嬉しい限り。次は福岡版どうですか <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; モノクロメガネ研究員 (@monochromegane) <a href="https://twitter.com/monochromegane/status/1066697776912691200?ref_src=twsrc%5Etfw">2018年11月25日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">GoConお疲れ様でした！いわたプロと知り合えたので何か一緒にやりたいなと思ってマス．Fukuoka.goはいつも10名ほどが発表してまして，例えば岡山のコミュニティの皆さんへ中継もしくは双方向の合同トーク大会とか一回どうでしょうか？ お気軽にご検討いただければ <a href="https://twitter.com/qt_luigi?ref_src=twsrc%5Etfw">@qt_luigi</a></p>&mdash; モノクロメガネ研究員 (@monochromegane) <a href="https://twitter.com/monochromegane/status/1067258144995201024?ref_src=twsrc%5Etfw">2018年11月27日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


## Gopherくん

今回で登壇特典のGopherくん全色コンプリート！！

![gophers](/images/2018/11/gophers.jpg)

## 過去の発表

- [Go Conference 2014 spring で発表してきた](https://blog.monochromegane.com/blog/2014/06/08/go-conference-2014-spring/) - [発表資料: pt & Goroutine](https://speakerdeck.com/monochromegane/pt-and-goroutines)
- [Go Conference 2015 summer で発表してきた](https://blog.monochromegane.com/blog/2015/06/23/go-conference-2015-summer/) - [発表資料: Generative Programming in Go](https://speakerdeck.com/monochromegane/generative-programming-in-go)


