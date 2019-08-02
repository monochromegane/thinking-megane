+++
date = "2019-08-02T15:33:36+09:00"
title = "福岡でGoConを開催した"
tags = [ "golang", "gocon" ]
+++


7/13に[Go Conference'19 Summer in Fukuoka](https://fukuoka.gocon.jp/)を開催した。
諸々落ち着いたので、初めて200名規模の大きなカンファレンスを開催するまでに駆け抜けた日々を振り返っておく。

# 準備

Fukuoka.goの主催の一人としてGoConの福岡開催は是非ともやりたい気持ちがあり、Go Conference 2018 Autumnで登壇した際の懇親会で盛り上がったのが去年の11/25。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">ホテル到着。移動で朝早かったので懇親会途中までとさせていただきました。お誘いありがとうございます。色々な話が聞けて楽しかったです！ カンファレンスも濃い話から体系的な整理がなされたものまであり来れて良かったです。自分の登壇内容も反応いただけて嬉しい限り。次は福岡版どうですか <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a></p>&mdash; モノクロメガネ研究員 (@monochromegane) <a href="https://twitter.com/monochromegane/status/1066697776912691200?ref_src=twsrc%5Etfw">2018年11月25日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 11/30にFukuoka.go運営でやっていきのIssue起票。
- 12/04に会場と時期のおおまかなあたりがついたので正式にGoCon地方開催を東京GoCon運営に打診。
- 12/14に企画書を元に関係者に一通り話しが通った状態へ。

と良い感じのスピード感で進み出し、後は運営スタッフ一同で

- 1月はマスコットのデザインやらLPの準備やら
- 2月はGo1.12のリリパでイベント開催を初めて告知。GoCon運営スタッフも増えた。
- 3月はCfPをオープン。スポンサー内容や金額感について具体的な検討と募集を始めた
- 4月はCfPへのアーリーフィードバック、CfP草の根宣伝活動、スポンサーとの個別やりとり
- 5月はCfPの選考と結果通知、タイムテーブルと参加枠を決めてconnpassを公開。
- 6月は集客草の根宣伝活動、Tシャツやバックパネル、会場の軽食や飲み物、スピーカーディナーなどについて検討・確認・発注
- 7月は当日までの作業の整理をして確定していないものを一つづつ対応（受付方法、司会進行、会場レイアウトや設営日、名札作成、ノベルティ配布準備、アンケート、市長、段ボール、張り紙 etc）

と、ひたすら準備を進めた。
大規模カンファレンス開催のノウハウがない中でも、準備は共同主催の[@linyows](https://twitter.com/linyows)が皆を引っ張った。
作業分担は、@linyowsが打ち出したデザインやおもてなしの世界観に皆がイイねをしながら、できる人が具体化を進めていく形に落ち着いた。
@linyows自身がガンガンとタスクをこなしていく（会社では傭兵と呼ばれている...）ので、僕自身は具体化を担当する他、方針の見直し案や進捗整理からの抜け漏れ対応といった補佐役に徹した。
[@seike460](https://twitter.com/seike460)をはじめとする運営スタッフとSlackでワイワイしながらだんだんと形になっていく過程も含めて（大変だったけど）楽しめたと思う。
開催時期については会場都合で少し延ばして7月としたが、今思えばこれぐらいの準備期間がないと納得いく形にするのは無理だったかもしれない。

---

個人的に大変だったのは、連絡まわりで、いくつかの連絡経路を使ってたくさんの関係者と個別または全体で効率よく相談、告知するのは骨が折れた。
他のカンファレンス運営でどういう風にやっているのかを聞いてみたい。
反対にCfPの選考は多様なプロポーザルを見る機会として有益であったと思う。
また、より良いプロポーザルにするためのアドバイスをアーリーフィードバックという形で進めることができたのも非常に良かった。
特に、このフィードバックを通してプロポーザルの説得力が格段に上がっていく例などを見るのは嬉しかった。

準備は前日と当日の朝が一番大変であった。
会場のリニューアル直後であったことも手伝い、搬入されるスポンサー物品を含む会場設営などはその場で詳細を決定して進めなければならなかったため、開場までの時間がない中で慌てて進めた。
特に当日朝は雨がすごかったので準備頑張ってるぞ感がすごかった気がする。

# 当日

当日は幸いなことに@linyowsと僕は専任のタスクを持たずに各種ジャッジや遊撃手的な動きをできたことで全体に柔軟に対応できたと思う。
受付がひと段落して、高島市長のスペシャルセッション前後の応対が終わって、ビールや生ハムの提供が軌道にのるまでは、イベントがスムーズに進むように、心地よく過ごしてもらえるように裏側では様々な物品や情報が行き交っており、とにかく忙しかった（けど何をやったかは覚えていない）みたいな感じだった。
これが16:00ぐらいで、ビールが入ったこともあって一瞬寝落ちしたのを写真に撮られてしまった。

![gocon_00](/images/2019/08/gocon_00.jpg)

この後、少しだけセッションを聞けたけども、楽しみにしていたセッションをほぼ聞けなかったのでここは次への課題だねと振り返りで話した。
セッションは最後まで盛況で、懇親会まで盛り上がっていたので嬉しい気持ちで眺めていた。


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">経費精算用エビデンスはこちら <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://t.co/zCsDLKBXJW">pic.twitter.com/zCsDLKBXJW</a></p>&mdash; L I N Y O W S 🤡 (@linyows) <a href="https://twitter.com/linyows/status/1150059130633068544?ref_src=twsrc%5Etfw">July 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# スピーカーディナー

カンファレンス後のスピーカーディナーでは豪華なメンバと素敵な料理に囲まれてこれまた楽しいひと時であったものの、各種支払いや幹事的な動きをしていたらあっという間に終わってしまったので、ちょっと残念だった。
ただ、テーブルの右も左もみんなが新しいつながりを作って飯食って酒飲んでGoの話して笑っているのを見て、「アーーーこれはやって良かったんではーーーーーー！！」と一人泣きそうになっていたのは秘密である。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://twitter.com/hashtag/spekerdinner?src=hash&amp;ref_src=twsrc%5Etfw">#spekerdinner</a> <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://twitter.com/hashtag/nulab?src=hash&amp;ref_src=twsrc%5Etfw">#nulab</a> マジすごい！圧倒的感謝！！！！ <a href="https://t.co/oZi3hfXHON">pic.twitter.com/oZi3hfXHON</a></p>&mdash; L I N Y O W S 🤡 (@linyows) <a href="https://twitter.com/linyows/status/1150008128953413632?ref_src=twsrc%5Etfw">July 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# カンファレンスを終えて

まず、@linyowsにスタッフ一同からお礼を伝えたいです。ありがとう！
今回のイベントは@linyowsなしでは考えられない。
@linyowsの行動力、大人力に対して尊敬の念が更に高まりました。Fukuoka.goを共同で主催してくれることが非常に心強いです。これからもよろしくお願いします。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://twitter.com/monochromegane?ref_src=twsrc%5Etfw">@monochromegane</a> さん、<a href="https://twitter.com/linyows?ref_src=twsrc%5Etfw">@linyows</a> さん、スタッフの皆様、スポンサーの皆様、発表者の皆様、参加者の皆様、お疲れ様でした＆ありがとうGoざいました！q@w@p <a href="https://twitter.com/hashtag/gocon?src=hash&amp;ref_src=twsrc%5Etfw">#gocon</a> <a href="https://twitter.com/hashtag/fukuokago?src=hash&amp;ref_src=twsrc%5Etfw">#fukuokago</a> <a href="https://t.co/hzSWpgZ4ij">pic.twitter.com/hzSWpgZ4ij</a></p>&mdash; Ryuji Iwata (@qt_luigi) <a href="https://twitter.com/qt_luigi/status/1150069073427886080?ref_src=twsrc%5Etfw">July 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

僕たちのソフトウェアの仕事は作った後に動く期間が長いので，こういった半年以上の準備とわずか1日のイベントという形式の経験はなかなか貴重だったし，大変ながらも充実した時間を過ごせて楽しかった。

# カンファレンスへの想い


どうして集う形の勉強会やカンファレンスにするかというのは、何度か聞かれていて、[この辺り](https://pr.forkwell.com/2018-10-01-community_lovers/)でも答えていたのだけど、まず単純に集まって楽しく話す仲間がいることの嬉しさがあって、その背景として「場やコミュニティは実在しないので。定期的にやることが大事」という考えがあった。
コミュニティというネットワークを継続させていくためには、接合点である参加者の間の接続を活性化する必要がある。
そのためにはネットワークが活性化するように情報を行き交いさせるんだけれども、均等な一方向の情報発信は刺激に乏しいこともある。
人間も単純な部分は残っているので、実際に会ったとか、その場の熱量とかに影響されて発信側も受信側も一時的に接続が活性化する。
こういう励起された状態は、例えばカンファレンスを地元でやろうとか、触発されてツールを作って見たとか、ネットワークへの新しい情報の元になってコミュニティに還元される。
勉強会やカンファレンスは、そういうやり方での重みの活性化を担っていると思う。

今回のカンファレンスがGo言語コミュニティや福岡にとって上のような影響を与えることができたのであればとても嬉しい。
何より、カンファレンス運営を通して、スタッフみんながエキサイティングな経験を楽しんでいたことがとても嬉しい。
なぜなら、僕たちが楽しみ続けることがまず大事だから〜〜。お疲れ様でした！！！

そしてオフィシャルのイベントレポートが非常によくまとまっています．こちらも是非ご覧ください！

- [The Official event report of Go Conference '19 Summer in Fukuoka](https://fukuoka.gocon.jp/report/)

# フォトギャラリー

最後に、誰得 モノクロメガネ フォトギャラリーを置いておきます。いやぁ、楽しんでるなあ〜

![gocon_01](/images/2019/08/gocon_01.jpg)
![gocon_02](/images/2019/08/gocon_02.jpg)
![gocon_03](/images/2019/08/gocon_03.jpg)
![gocon_04](/images/2019/08/gocon_04.jpg)
![gocon_05](/images/2019/08/gocon_05.jpg)
![gocon_06](/images/2019/08/gocon_06.jpg)
![gocon_07](/images/2019/08/gocon_07.jpg)
![gocon_08](/images/2019/08/gocon_08.jpg)

Photo by [@atani](https://twitter.com/atani)

---

𝝣Go

