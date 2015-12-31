+++
date = "2015-12-31T10:19:17+09:00"
title = "2015年のふりかえり"
+++

## 仕事

今年は7月頃に2年ほど携わっていたチームから異動して新しい環境になったのが一番大きな変化だった。異動前にPHPバージョンアップを成すことができたし、新しいチームでは新機能開発しながらも技術的な課題がサービスの発展を妨げることのないよう、継続的な改善ができる体制を取ることができた。今のチームは最高で、お互いに敬意を払いながらも得意領域で活躍し、不足しているところを補い合えているんじゃないかと思う。ただ、チームとしての成熟は変化に反応するための柔軟性を失う可能性も含んでいるので、新しい目標、例えば新しい技術への挑戦とか持っている技術の深掘りとかお互いを刺激し合える環境というのをつくっていきたいなあと思う。

## 技術

2014年から引き続きGo言語一色の年になった。[Top Go GitHub developers in Japan](http://github-awards.com/users?country=japan&language=Go&type=country)で3位に位置付けられてるのは素直に喜びたい。当たるか当たらないかに関係なく自分が必要と思ったらOSS前提でつくるようになれたし、発表も特に臆することなくできるようになったと思う。

今は、OSSや発表も個人ベースだったりGo言語界隈という感じで範囲や関係する人が限定的なので、来年はそのあたりをもっと広げて新しい世界を見たいと思っている。

### OSS

Go楽しい〜の一年だった。[argen](https://github.com/monochromegane/argen)とかの大きめのプロダクトもつくれたし、[The Platinum Searcher](https://github.com/monochromegane/the_platinum_searcher)の高速化も達成できたのはよかった。

- [hoi](https://github.com/monochromegane/hoi)([HoiとSlackで内緒話をする](http://blog.monochromegane.com/blog/2015/03/01/hoi-and-slack/))
- [argen](https://github.com/monochromegane/argen)([Go言語でActiveRecordライクなORMをつくった](http://blog.monochromegane.com/blog/2015/03/04/argen/))
- [mdt](https://github.com/monochromegane/mdt)([CSVテキストをMarkdown形式のテーブルに変換するツールをつくった](http://blog.monochromegane.com/blog/2015/05/17/csv-to-markdown-table/))
- [conflag](https://github.com/monochromegane/conflag)([Go言語でコマンドラインオプションをさくっと設定ファイル対応させるライブラリをつくった](http://blog.monochromegane.com/blog/2015/05/30/conflag/))
- [Torokko](https://github.com/monochromegane/torokko)([Goのデプロイを「もっと」簡単にする。ビルドプロキシCargo。改めTorokko。](http://blog.monochromegane.com/blog/2015/08/16/deploy-golang-by-cargo/))
- [mackerel-plugin-delayed-job-count](https://github.com/monochromegane/mackerel-plugin-delayed-job-count)
- [dsn](https://github.com/monochromegane/dsn)
- [go-bincode](https://github.com/monochromegane/go-bincode)([野良バイナリになっても大丈夫。Goのバイナリにソースコードを添付するツールをつくった。](http://blog.monochromegane.com/blog/2015/08/23/go-bincode/))
- [slack-incoming-webhooks](https://github.com/monochromegane/slack-incoming-webhooks)
- [memcache_key](https://github.com/monochromegane/memcache_key)
- [mysql_dump_slow](https://github.com/monochromegane/mysql_dump_slow)([MySQLのslow_logテーブルをサマライズするGemをつくった](http://blog.monochromegane.com/blog/2015/07/20/mysql-dump-slow/))
- [be_strong](https://github.com/monochromegane/be_strong)([
さくっとStrongParametersへ移行するbe_strongというgemをつくりました](http://tech.pepabo.com/2015/12/18/upgrade-to-strong-parameters/))
- [dragon-imports](https://github.com/monochromegane/dragon-imports)([goimportsを高速化するdragon-importsコマンドをつくった](http://blog.monochromegane.com/blog/2015/12/23/dragon-imports/))

- [V2! V2! Go言語製 高速検索ツールThe Platinum Searcherのv2をリリースしました](http://blog.monochromegane.com/blog/2015/12/08/the-platinum-searcher-v2/)
- [The Platinum Searcherを5倍高速化するためにやったこと](http://blog.monochromegane.com/blog/2015/12/15/how-to-speed-up-the-platinum-searcher-v2/)

### 発表

200-300人規模の発表に参加していくことができたので少しづつ東京での勉強会でも顔見知りが増えてきたり声を掛けてもらえるようになったのはうれしい。地元でのFukuoka.goがあんまり開催できてないので、やり方検討しなおして来年はなんとか再開していきたい。


<iframe src="http://www.storyboards.jp/widget/7xay4r" width="640" height="480"></iframe>

[Fukuoka.go#7 ソースコードリーディングvol.1を開催しました](http://blog.monochromegane.com/blog/2015/04/25/fukuoka-go-7/)


<script async class="speakerdeck-embed" data-id="baea21c51cb9413992b74786804e9109" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

[Go Conference 2015 summer で発表してきた](http://blog.monochromegane.com/blog/2015/06/23/go-conference-2015-summer/)

<script async class="speakerdeck-embed" data-id="dc3e7e96920d4412b24786d48114bd15" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

[第2回ペパボテックカンファレンスで発表してきた #pbtech](http://blog.monochromegane.com/blog/2015/07/05/pepabo-tech-conference-2-in-fukuoka/)

<script async class="speakerdeck-embed" data-id="ae6a096447944303a65456f6ef7af717" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

[「minne」技術戦略カンファレンスで発表してきた](http://blog.monochromegane.com/blog/2015/10/24/minne-tech-conference/)

## まとめ

ペパボに入って3年経って、とにかくコードを書きたいという気持ちはOSSやアウトプットで消化の仕方が分かってきた。来年は楽しみとしている技術を専門職として業務にフィードバックできるようにひとつ成長したいところ。なんにせよ一年おつかれさまでした。

