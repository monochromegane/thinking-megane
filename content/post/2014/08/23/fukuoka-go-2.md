---
title: "Fukuoka.go#2+Fukuoka.rbを開催しました。"
date: 2014-08-23
comments: true
tags: [ "golang", "fukuoka.go", "ruby" ]
---


8月21日に **Fukuoka.go#2+Fukuoka.rb** を開催しました。

Fukuoka.goは福岡でGo言語に興味のあるひとが集まる場所を目指して開催する会です。

- [Fukuoka.go#2+Fukuoka.rb](http://connpass.com/event/7559/)

今回はFukuoka.rbと合同開催になったこともあり、参加者26名（キャンセル含めると申込自体は35名）と前回より多くの方に来ていただきました。福岡のGo熱は衰えず！です！

今回の内容をまとめておきます。


## タイムテーブル

- 19:30 - 21:30 LT大会

当初もくもく会も予定していましたが、LT希望者が多く、最終的にはオールLTのLT大会になってしまいました。

<hr />

# LT

## ルビーストのためのGo (@linyows)

[@linyows](https://twitter.com/linyows)さんによるLT。

RubyとGoを比較しながらGoを学ぶ。


<div style="speakerdeck">
<script async class="speakerdeck-embed" data-id="244385d00bc4013224d21eb14f30e1c6" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>
</div>

自分の知っている言語(知識)と比較していくと知識がメタ化、抽象化されて応用力がついていってよいですよね。

GoのBDD形式のテスティングフレームワーク、`onsi/ginkgo`以外はまだ使ったことがないものもあったので調べてみようと思いました。


## Hello GoDoc! (@laco0416)

[@laco0416](https://twitter.com/laco0416)さんによるLT。

GoのドキュメントホスティングサービスとしてのGoDocとコメントの書式について紹介。


<iframe src="//www.slideshare.net/slideshow/embed_code/38209599?rel=0" width="512" height="421" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/laco0416/hello-godoc-38209599" title="Hello GoDoc!" target="_blank">Hello GoDoc!</a> </strong> from <strong><a href="http://www.slideshare.net/laco0416" target="_blank">laco0416</a></strong> </div>

GoDocサービスがあることで、開発者はコメント記載して、リポジトリのURLを叩くだけで自動でドキュメントを公開できて非常に便利。

Exampleも書けるのはいいですね。
バッジやLintも用意されているのでどんどん使っていきましょう（参加者中GoDoc利用者はゼロでした）。



## 最近やっていること (@udzura)

[@udzura](https://twitter.com/udzura)さんによるLT。

DockerでPuppet/Chefのテストをするにあたって頑張ったお話。

[![udzura\_saikin](/images/2014/08/udzura_saikin.png)](http://www.storyboards.jp/viewer/64t76r)

現場で得られたノウハウが盛り込まれた便利情報でした。失敗したイメージはタグをつけてコミットしておくの使えそう。

そのうち、Fukuoka.goでもDockerのソースコードリーディングとかもやってみたいなあと考えてます。


## 実践Go - ツールの作成から配布まで (@monochromegane)

[@monochromegane](https://twitter.com/monochromegane)によるLT。

Fukuoka.goで毎回やっているGo入門用LTです。

前回は構文を学ぶ[速習Go](https://gist.github.com/monochromegane/8bb73390f2ebd9d325f4)をやったので今回は実践Goとして、簡単なツールをつくって配布する部分を紹介しました。

<div style="speakerdeck">
<script async class="speakerdeck-embed" data-id="943c17900bbf013224d21eb14f30e1c6" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>
</div>

ライブコーディングしながらGOPATHまわりの話やツールつくる上でのTipsなど話してみました。

Go書いたことない方にも雰囲気が伝わっていればうれしいです。

入門用LT、次回はHTTPサーバの実装を通して型とか構造体とか紹介するようなのを考えてます。お楽しみに。


## gyowitter の ご紹介 (@morygonzalez)

[@morygonzalez](https://twitter.com/morygonzalez)さんによるLT。

gyowitterというGo製のYo/Twitter連携サービスの紹介。

<div style="speakerdeck">
<script async class="speakerdeck-embed" data-id="ccf4a2000bc7013260fd7e2d2d4e67ff" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>
</div>

圧倒的なLT力で会場を沸かせてくれました。

GoのHTTPサーバを利用しているとのことですが、全然落ちてないとのこと。すばらしい〜。


## Togetter

今回、つぶやきも盛り上がっていたので、まとめてみました。

- [Fukuoka.go#2+Fukuoka.rb - Togetterまとめ](http://togetter.com/li/710020)

<hr />

# 所感など

参加していただいた方々、LTしていただいた方々、会場設営手伝っていただいた方々、**ありがとうございます！**

今回はLT大会でしたが、参加した方のお話やつぶやきを見る限りでは、Goに対して興味を持ってもらえたり、情報を得られたりしたようなので、Go言語についてわいわいするという目的は今回も達成できたんじゃないかなと思ってます。

それから、Fukuoka.rbとの合同だったのでお互いの言語、コミュニティに対して興味を持つことができたのもよかったですね。

次回は、要望の多かったもくもく会も時間とってやろうと思います。

<hr />

**楽しくも学びのある Fukuoka.go** は月一回を目安に定期的に開催しています。  
Goに興味がある方も、Go談義がしたい方も、どなたでもお気軽にご参加ください。

LTも絶賛募集しています！  
Go言語でこんなのやってみたというのがあればぜひぜひ教えてください。

