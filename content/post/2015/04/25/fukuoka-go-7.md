---
title: "Fukuoka.go#7 ソースコードリーディングvol.1を開催しました"
date: 2015-04-25
comments: true
tags: [ "golang", "fukuoka.go" ]
---

今年入ってちょっと休憩していたFukuoka.goなんですが、[@nobkz](https://twitter.com/nobkz)さんがソースコードリーディングやろうという企画を持ちかけてくれたので久々に開催しました。

- [Fukuoka.go#7 ソースコードリーディングの会vol.1](https://fukuokago.doorkeeper.jp/events/23212)

今回は発表はしましたが、会場準備などの裏方だったので参加者としても勉強会を楽しむことができたのが個人的にはうれしかったです。

---

# 発表

## ダウンタイム・ゼロを支える技術 in Go

Goでアプリをダウンタイムなしで運用するための方法あれこれを発表しました。

<iframe src="http://www.storyboards.jp/widget/7xay4r" width="640" height="480"></iframe>

アプリでがんばるところとインフラでがんばるところのバランスは難しいですね。
ベストなやり方を取れるように検討することは大事だと思います。


## 並列とGo

[@nobkz](https://twitter.com/nobkz)さんの発表です。

<iframe src="//slides.com/nobukazuhanada/go/embed" width="640" height="480" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

Go初学者向けにコードリーティングやりやすくなるようにgoroutineまわりなどを解説してくれました。


## lexer実装について (@keizo\_bookman)

[@keizo\_bookman](https://twitter.com/keizo_bookman)さんの発表。

資料まだアップしていないとのこと。goroutineとchannelを使ったきれいなlexer実装のお話でした。

---

# ソースコードリーディング vol.1

ソースコードリーディングは3〜4人のチームに分かれて Go製のApacheBenchを読みました。

- [rakyll/boom - HTTP(S) load generator, ApacheBench (ab) replacement, written in Go](https://github.com/rakyll/boom)

30分ぐらいでざっと読んでそのあとチームごとにプロダクトの概要や気になった点、よいと思った点などを発表していく流れ。

- チームで読むことで自分の考えを言葉に出して伝えるので理解が深まる
- 時間制限があるので全体を把握するため最短で読む方法を模索する
- なぜそうなっているのかなど有識者の意見を聞ける

などなど。チームで読むというのはとてもおもしろいのでまたやっていきたい。

大きなプロダクトだと機能や範囲を絞らないと読みきれないと思うのでお題のチョイスは重要。
今回のboomは初回としてはgoroutine扱いながらもコンパクトなプロダクトだったので[@nobkz](https://twitter.com/nobkz)さん良いチョイスだったと思います。


## 様子です

雰囲気など感じてもらえれば。

- [Fukuoka.go#7 ソースコードリーディングの会vol.1 - Togetterまとめ](http://togetter.com/li/812438)

---

# 所感

勉強会やコミュニティを続けていくのはひとりでは息切れしてしまうので、今回みたいに別の勉強会とかで知り合った人から企画あげてもらえたのはうれしかったです。
また、今回は集客を積極的にやってくれた仲間もいて、つくづく繋がりなのだなあと思えたFukuoka.goでした。

またFukuoka.goぼちぼち続けていきます。福岡でGoに興味がある方気軽にご参加ください〜。
ソースコードリーディングだけではなくてLT大会、もくもく会もまたやっていこうと思います。

ʕ◔ϖ◔ʔ < Go!

