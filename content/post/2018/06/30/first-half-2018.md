+++
date = "2018-06-30T11:30:12+09:00"
tags = [ "ふりかえり" ]
title = "First Half of 2018"
+++

上期も昨年度から引き続き研究開発に従事した．これまでやってきたいくつかの研究が繋がり始めて"なめらかなマッチング"というおぼろげながら大きな研究テーマが見えてきたように思う．
推薦システムに代表されるような要望と提案のマッチングをなめらかにするためには，マッチング観点からの研究と，それをサービスとして的確かつ即時に提供できるような実装や導入の観点からの両方が必要であり，マッチングを支える仕組み全体に対する研究を進めていきたいと考えるようになった．

また，上期はこのテーマに基づく各研究内容を実装するにあたって技術的な深堀りを進めることができたこと，継続してアウトプットを続けることができたのも良かった．
マッチング観点からは消費者行動や情報探索プロセスも考慮しながら検討している．実装面からは，近似近傍探索を高速かつ高精度に提供するためにSannyやSmux，go-avxなどこれまでよりレイヤを掘り下げながら目的を達成するための最適な実装を検討することができるようになった．
アウトプットについても，これらの内容を元に継続的にブログや登壇を行った．
技術的な必要性などをきちんと記述することで反響が大きくなるものもあり論理的な思考やそれを用いて人に伝えることの重要性も感じた．

それでも，特に論文のような専門的なアウトプット過程において，力不足は常に感じていた．
論文執筆において，初稿はなんとか書き上げられるようになったものの，共著者や査読からの指摘点を踏まえてより良いものにするための議論が上手くできず，徐々にこれがストレスとなってしまっていった．
技術的な背景による新規性についてはサーベイの積み重ねで少しづつでも回答できるが，研究として新規性や有用性を謳う部分，これらを論述する部分の議論において，そもそも前提となる能力が足りないようにも思えた．
論文執筆を続けていく中でこれらの能力が上がるようコツがつかめてくるのかもしれないが，自分としては一旦この足りない能力についてきちんと向き合った上で，今後の研究開発へ取り組んでいきたいと思っている．
そのため下期はこれまでの研究開発成果を導入・課題発見の部分に力を注ぎながら，論文執筆に向けたスキルを高めていきたい．

# 実績

以下，上期実績を列挙する．

## 論文

3本，ジャーナルについては事前査読制度への投稿のみで終わったこともあり，上述した対策を進めた上で再度挑戦したい．

- [三宅 悠介, 松本 亮介, Sanny: 大規模ECサイトのための精度と速度を両立した分散可能な近似近傍探索エンジン, 研究報告インターネットと運用技術（IOT）,2018-IOT-42(2),1-7 (2018-06-21) , 2188-8787](http://id.nii.ac.jp/1001/00190207/)
- フォーラム投稿1本
- ジャーナル論文執筆1本（事前査読のみ）

## 発表

技術イベントでの発表の他，研究会での発表，大学での登壇など計6回．

- [ペパボにおける研究所の役割 - 研究と開発の相乗効果 -](https://speakerdeck.com/monochromegane/think-about-research-and-development) @香川大学
- [Go言語でオンライン外れ値検出エンジンSmartSifterを実装した](https://speakerdeck.com/monochromegane/smartsifter-in-golang) @Fukuoka.go#10
- [Sanny: 大規模ECサイトのための精度と速度を両立した分散可能な近似近傍探索エンジン](https://speakerdeck.com/monochromegane/wsa-2-sanny) @WSA研#2
- [分散アプリケーションにおける複数端末利用を考慮したプライベートデータの管理](https://speakerdeck.com/monochromegane/wsa-1-kaleidoscope) @Hi-Ether Meetup #4 Fukuoka
- [近似近傍探索エンジン Sannyを支える技術](https://speakerdeck.com/monochromegane/sanny-inside) @Fukuoka.go#11
- [Sanny: 大規模ECサイトのための精度と速度を両立した分散可能な近似近傍探索エンジン](https://speakerdeck.com/monochromegane/iot42-sanny) @IOT42

## OSS

なめらかなマッチングの実装面の観点から高速かつ高精度，かつ拡張性も備えた近似近傍探索の実装とそれらの要素技術の検討を行っていった．
Smuxやgo-avxなどこれまで扱ってこなかったレイヤで課題に対処できるようになってきたと思う．

- [Sanny](https://github.com/monochromegane/sanny): Scalable Approximate Nearest Neighbors in Golang optimized for embedding vectors.
- [go-avx](https://github.com/monochromegane/go-avx): AVX(Advanced Vector Extensions) binding for golang.
- [Smux](https://github.com/monochromegane/smux): smux is a socket multiplexer written in Golang. It provides fast communication by efficiently a single connection.
- [SmartSifter](https://github.com/monochromegane/smartsifter): SmartSifter (On-line outlier detection) by Golang.
- [Guardian](https://github.com/monochromegane/guardian): Monitor file changes and execute custom commands for each event.
- [Gannoy(NGT ver.)](https://github.com/monochromegane/gannoy/tree/ngt): Approximate nearest neighbor search server and dynamic index written in Golang.

## コミュニティ活動

福岡のGo言語コミュニティ主催として継続的なイベント開催を進めた．
研究開発とはまた別の技術の付き合い方ができる活動であり，大切な活動となっている．
また，分散アプリケーションも興味があるため仮想通貨コミュニティの地方開催もお手伝いさせていただいた．

- [Fukuoka.go#10](https://fukuokago.connpass.com/event/81056/) 主催
- [Fukuoka.go#11](https://fukuokago.connpass.com/event/87684/) 主催
- [Hi-Ether Meetup #4 Fukuoka](https://techplay.jp/event/668228?pw=HplUa3Vs) 運営協力

## ブログ

実装紹介を主に7本．もう少し気軽に考えをまとめてアウトプットするようにしたい．

- [2018, 認知的転回](https://blog.monochromegane.com/blog/2018/01/01/2018-the-turn/)
- [情報探索プロセスのモデルについて: 「情報検索のためのユーザインタフェース」を読んで](https://blog.monochromegane.com/blog/2018/01/07/information-retrieval-process-model/)
- [情報要求を取り巻く技術とプロセスモデル](https://blog.monochromegane.com/blog/2018/01/10/information-need-association-chart/)
- [論文生成モデルの検討を通して論文執筆を進めてみる](https://blog.monochromegane.com/blog/2018/02/20/generate-paper-model/)
- [Go言語でオンライン外れ値検出エンジンSmartSifterを実装した](https://blog.monochromegane.com/blog/2018/02/25/smartsifter/)
- [Go言語でTCPやソケット通信を多重化，高速化するsmux(ソケットマルチプレクサ)をつくった](https://blog.monochromegane.com/blog/2018/05/03/smux/)
- [Sanny: 大規模ECサイトのための精度と速度を両立した分散可能な近似近傍探索エンジン](https://blog.monochromegane.com/blog/2018/05/16/wsa_2_sanny/)

## 読書

半期で購入した本と読み直した本を重ねてみた．物理本が54冊．電子書籍を含めるともう少しなりそう．
ベイズは相変わらず分からない．
これだけ学んだのだという自信が半分，本当に自分の知識として議論へ活かせるようにせねばなという反省が半分．

![books](/images/2018/06/books.jpg)

# まとめ

上期はとにかく濃くて長い半年だった．
論文執筆など自分が上手くできないことに対してストレスと感情の起伏が大きくなってしまって相変わらず未熟だなあと反省しきりであったが，こうして客観的に実績としてまとめてみると進んでいるところももちろんあって，適切に都度自分を認めてあげなくてはなあと思えたのはよかった．

もちろん力不足で悔しい思いをしているのは事実であるので，下期は研究開発への付き合い方も見直しながら，まわりも見る余裕を持って，どんどん影響を与えていけるよう成長できればなあと思う．
