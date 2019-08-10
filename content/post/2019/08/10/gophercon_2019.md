+++
date = "2019-08-10T20:52:57+09:00"
title = "GopherCon 2019で初の海外カンファレンス登壇をしてきました"
tags = [ "golang", "gophercon" ]
+++

7/24から27にかけてアメリカ、サンディエゴで開催された[GopherCon 2019](https://www.gophercon.com/home)で人生初となる海外カンファレンスに登壇してきました。

GopherConはGo関連で最大級の国際カンファレンスです。
6年目となる今年は世界中から1,800名のGopherが参加し、200名以上の応募の中から選ばれた36名がスピーカーとして登壇しました。
今回のGopherConではPre-Conference Workshopと呼ばれるカンファレンス前日に終日行われるワークショップと、カンファレンス期間中に行われる25分のキーノートセッション、そして45分のチュートリアルセッションがありました。
その中で、僕は「Optimazation for Number of goroutines Using Feedback Control」というタイトルで45分の[チュートリアルセッション](https://www.gophercon.com/agenda/speakers/442434)を務めました。

# スライド

<script async class="speakerdeck-embed" data-id="aa53e4353d9b4efc9064eefba40e13b7" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

# スピーカーノート

- [GopherCon 2019: Optimization for Number of goroutines Using Feedback Control](https://blog.monochromegane.com/blog/2019/07/25/gophercon_2019_kaburaya/)

# 発表内容

今回の発表は以前東京で開催されたGo Conference 2018 Autumnで発表した[フィードバック制御によるGoroutine起動数の最適化](https://speakerdeck.com/monochromegane/optimization-of-goroutine-numbers-by-feedback-control)を英語化したものです。
また、今回向けに実際に[The Platinum Searcher](https://github.com/monochromegane/the_platinum_searcher)に組み込んで有効性を評価する実験結果を追加しています。

発表資料については、45分という時間で母国語を使わずにできるだけわかりやすく伝えるため、結果的にほぼ刷新する形となりましたが、母国語の慣れに頼らずとも説明可能なわかりやすい資料になったのではないかと自分では思っています。
具体的には、まずイントロダクションとバックグラウンドの章を少し手厚くしました。
今回の手法は、フィードバック制御を用いてgoroutine数を継続的に最適に保つことを目的としています。
しかしながら、そもそもGo言語の並行処理においては、goroutineのコストの低さとランタイムがgoroutineを効果的に切り替えることから、その必要性はないのではないかと考えることができます。
そこで、リソース競合や枯渇などの影響も受けてしまう可能性があること、それらを考慮した汎用的な並行設計に対する難しさを、実際にプラチナサーチャーによるパフォーマンスチューニングの例を踏まえながら前提を合わせるようにしました。
また、手法の説明では、簡単なアプローチから出発して少しづつ課題を解決していくようにすることで、比較的複雑な箇所の説明の際に飛躍がないように気をつけました。

発表について、普段は資料のみで臨んでもそれなりに話せると自負していますが、初の英語での登壇となるため今回は発表内容のスクリプトを用意して、伝えるべきことを全て伝えられるようにしました。
また、不慣れな言語でも自分自身が納得して説明できるように、できるだけ平易な文で説明することで、発表資料と同様に曖昧な箇所や飛躍がある点を潰すことができたと思います。

本番の発表は、4スクリーンある300-400人規模の会場ではありましたが、とても熱心に頷いてくれる方がいたり日本から来たメンバが前に座っていたりして、幸いにも落ち着いて自分のペースで登壇することができたと思います。
発表では、あまり一本調子にならないように、文の中でも意味や文法上切れる箇所は間をおいたり、定型の言い回し（I found that など）なるべく流暢にしたり、強調したいところは少しゆっくり大きく言うなど、「今の」英語力でできる工夫をしていきました。
一方で、個別の単語の発音などはまだまだ課題意識があるので、今後、そちらも重点的に取り組んでいきたいと感じました。

質疑の時間は登壇時間には含まれなかったのですが、終了して台を降りると同時に7人ぐらいに囲まれて質問責めに会い、発表に興味を持ってくれたことをとても嬉しく思いました。
質問としては、発表内容について理解があっているか確認するものと、実装上の質問などがあり、資料やKaburayaのコードを見ながら説明していきました。
それでも、今の英語力では、発表内容以外の部分については拙い説明になることも多々あり、これも今後改めて英語をやるモチベーションが高まりました。
また、発表後もカンファレンス期間中に何度も感想や質問をもらう機会があり、これがとても嬉しかったです。

### 会場と発表の様子

<iframe class="embed_iframe" src="https://s.insta360.com/p/28b6fbdd988f775d97328c5179e8061a?e=true&locale=en-us" frameborder="0" width="666" height="413"></iframe>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">.<a href="https://twitter.com/monochromegane?ref_src=twsrc%5Etfw">@monochromegane</a> is giving talk about Go&#39;s concurrency mechanism and his proposal to optimize the number of goroutine. <a href="https://twitter.com/hashtag/gophercon?src=hash&amp;ref_src=twsrc%5Etfw">#gophercon</a> <a href="https://t.co/LXnnx5vQ6D">pic.twitter.com/LXnnx5vQ6D</a></p>&mdash; Yoshi Yamaguchi 🇯🇵 (@ymotongpoo) <a href="https://twitter.com/ymotongpoo/status/1154500168181379072?ref_src=twsrc%5Etfw">July 25, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### 発表に関する反応

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Thanks to Yusuke Miyake for speaking to us on the Optimization for Number of goroutines Using Feedback Control! The <a href="https://twitter.com/hashtag/gophercon?src=hash&amp;ref_src=twsrc%5Etfw">#gophercon</a> tutorial sessions are killin&#39; it! <a href="https://t.co/4I5ihE1wKG">pic.twitter.com/4I5ihE1wKG</a></p>&mdash; GopherCon (@GopherCon) <a href="https://twitter.com/GopherCon/status/1154502084634406917?ref_src=twsrc%5Etfw">July 25, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This was a really good talk, <a href="https://twitter.com/monochromegane?ref_src=twsrc%5Etfw">@monochromegane</a> is very smart! Also it was really interesting hearing about the fukuoka Golang community. I need to read slides again. <a href="https://t.co/bdGZK7p6DX">https://t.co/bdGZK7p6DX</a> <a href="https://t.co/82GKvlFseO">pic.twitter.com/82GKvlFseO</a></p>&mdash; Jamal Yusuf (@JamalYusuf\_) <a href="https://twitter.com/JamalYusuf_/status/1154587450242502656?ref_src=twsrc%5Etfw">July 26, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr"><a href="https://twitter.com/monochromegane?ref_src=twsrc%5Etfw">@monochromegane</a> talking about dynamic concurrency scaling with a PID controller and dynamic semaphores. <a href="https://twitter.com/hashtag/golang?src=hash&amp;ref_src=twsrc%5Etfw">#golang</a> <a href="https://twitter.com/hashtag/gophercon?src=hash&amp;ref_src=twsrc%5Etfw">#gophercon</a> <a href="https://t.co/dufxZNhKEZ">https://t.co/dufxZNhKEZ</a> <a href="https://t.co/5eyRazxg2g">pic.twitter.com/5eyRazxg2g</a></p>&mdash; Andy Walker (@flowchartsman) <a href="https://twitter.com/flowchartsman/status/1154503310327439360?ref_src=twsrc%5Etfw">July 25, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Really interesting thoughts on controlling <a href="https://twitter.com/hashtag/golang?src=hash&amp;ref_src=twsrc%5Etfw">#golang</a> concurrency dynamically while a program is running. Thanks to Yusuke Miyake for traveling from Japan to present this at <a href="https://twitter.com/hashtag/gophercon?src=hash&amp;ref_src=twsrc%5Etfw">#gophercon</a>! <a href="https://t.co/WWfsJo5nNi">pic.twitter.com/WWfsJo5nNi</a></p>&mdash; Daniel Whitenack (@dwhitena) <a href="https://twitter.com/dwhitena/status/1154500909713383424?ref_src=twsrc%5Etfw">July 25, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



# カンファレンス

会場はカリフォルニア サンディエゴのホテル [Marriott Marquis San Diego Marina](https://www.gophercon.com/page/1388212/hotel)でした。
サンディエゴが比較的温暖な気候であること、ホテルもいわゆるリゾートホテルのような感じであったことも手伝って、期間中はとても快適に過ごせました。

GopherConのタイムテーブルは朝が早いものの（最初のセッションがAM9:00から）、2-3セッションごとに30-40分の空き時間が設けられており、この時間に登壇者とディスカッションしたり、ポップアップミーティングスペースやスポンサーブースでコミュニティや企業への関係を作ったりすることができました。
日本でも学術系のシンボジウムだとこういったタイムテーブルは見かけるものの、GopherConではより積極的に関係を作っていこう、そしてそのために運営がその接点を作る機会（場所、時間）やホスピタリティを提供していこうという気概が感じられました。
また、Welcome Partyが本物の空母の上で開催されたのはめちゃくちゃ驚きました。こういう「楽しんでるぞ」というのが一貫して感じられるのも良かったです。
実際に #gophercon ハッシュタグを追っていると、たくさんの知見や関係を得たという感想と合わせて一種のバケーションを楽しんだような満足感を持っている方が多かったようです。
ちょうど、GoCon福岡を2週間前に開催したところであったため、「カンファレンス」をどういう設計で進めるのかという観点はとても勉強になりました。

トークについては、どれも最新の情報だけでなく、スピーカーの方の実際の経験に基づく興味深く、かつ面白いセッションばかりでとてもためになりました。
それでも、自分の発表前後の緊張や疲れ、時差ボケなどで万全の状態で全てを聞くことはできなかったのが悔やまれますが、幸い個別のトークについては今後動画も公開されるようです。
[Agenda](https://www.gophercon.com/agenda)を見て興味あるセッションを是非ご覧ください。

# スピーカーディナー

発表は7/25でしたが、実は7/26の夜にスピーカーディナーなるものが控えていました。
登壇者ばかりが集う3時間の立食パーティーを英語で乗り切るという、ある意味では登壇よりもプレッシャーのかかるイベントでした。
それでも、とてもよい機会であるし無駄にはしたくないと、できるだけ話に加わり自分の意見や要望を伝えることができたと思います。
特に日本や福岡のコミュニティ活動やカンファレンスに何人か興味を持ってもらうことができたのは非常に有意義でした。
また、ワークショップを担当したエンジニアの方とは日本に戻ってからも連絡を取り合っており、今後何か面白いことが一緒にできるといいなあと考えています。

とにかくこの3時間は僕の英語に対する心理的な苦手意識を取り除いてくれる絶好の機会となりました。
同時にやはり英語力の足りなさを痛感したことで、引き続き英語もやっていきたいと強く思えました。

# おわりに

2014年の東京のGoConference 2014 springで発表したのが僕にとって初めての全国区のカンファレンス登壇でした。
そこから、自分のキャリアや興味範囲が広がっていく中でも、Go言語はずっと相棒として付き合ってくれる言語でしたし、Go言語を中心に色々な付き合いができていると感じています。
また、ここ数年の研究職としての試行錯誤がなければ今回の発表内容はありませんでした。
その意味で、今回のGopherCon 2019は現時点の僕の集大成で臨んだカンファレンスとなりました。
幸いにも、発表に対して興味を持ってもらえたこと、一定の反響があったことを、非常に嬉しく思っています。
同時に、このような大きな海外のカンファレンスで発表できたことは（2014にGoConで発表した時と同じように）素直に自信につながりました。
（当日聞いたのですが、スピーカーのうち、ワークショップを行われた約10名の応募枠は別だったとのことで今回は改めて狭き門を抜けることができたんだなあと感じています）

この登壇に向けてアドバイスや協力をいただいた皆様、本当に感謝いたします。
このカンファレンスで得た様々な知見や関係、そして至らなかった点は次へ活かし、この契機となったGoコミュニティの活発化に繋がるよう引き続き”楽しみながら“やっていきたいと思います。

### 俺たちが日本代表だ！

![gophercon_japan](/images/2019/08/gophercon_japan.jpg)

LTセッションで登壇した[@hajimehoshi](https://twitter.com/hajimehoshi)さんと[@hgsgtk](https://twitter.com/hgsgtk)さんと共に。

---

そして最後に、Russ Coxと隣に扱われる機会ってそうそうないので記念に[スピーカー一覧](https://www.gophercon.com/page/1388210/speakers)のスクショを置いておきます、えへへ。（ただのアルファベット順だけど）

![gophercon_russ](/images/2019/08/gophercon_russ.jpg)
