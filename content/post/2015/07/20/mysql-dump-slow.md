+++
date = "2015-07-20T21:15:28+09:00"
title = "MySQLのslow_logテーブルをサマライズするGemをつくった"
tags = [ "mysql", "ruby" ]
+++

MySQLにはログファイルに出力したスロークエリをサマライズする`mysqldumpslow`コマンドがあって重宝していたのですが、出力先をテーブルに変更すると使えなくなってしまったので、同じことができるmysql\_dump\_slowというGemをつくりました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fmysql_dump_slow" title="monochromegane/mysql_dump_slow" class="embed-card embed-webcard" scrolling="no" frame    border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/mysql_dump_slow"&gt;monochromegane/mysql_dump_slow&lt;/a&gt;</iframe>

# 使い方

ActiveRecordで取得したスローログを渡して使います。

```ruby
# ActiveRecordでmysql.slow_logテーブルのレコードを取得します
logs = SlowLog.all

# スローログをサマライズします
summary = MysqlDumpSlow.summarize(logs)
summary.each do |counter|
  # mysqldumpslowコマンド形式で出力することができます
  counter.to_mysqldumpslow
  # => Count: 2  Time=100s (200s)  Lock=200s (400s)  Rows=300 (600),  2hosts
  #      SELECT * FROM T
end
```

`to_mysqldump`を使って、サマリした結果ごとに`mysqldumpslow`コマンドの形式で出力することができます。

社内ではこんな感じで前日分のスロークエリをSlackに通知するrakeタスクに組み込んで使っています。

![mysql\_dump\_slow](/images/2015/07/mysql_dump_slow.png)

このように某清貧会会長に煽られるので緊張感もってスロークエリの撲滅に取り組んでいます。

## getter

自分で結果を整形して使いたい場合は、それぞれの total, average の値を取得できます。

```ruby
counter.total_query_time   # => 100000
counter.average_query_time # => 100000
```

以下の値を取得可能です。

- total\_count
- [ total | average ]\_query\_time
- [ total | average ]\_lock\_time
- [ total | average ]\_rows\_set
- user\_hosts

## サマリについて

`mysqldumpslow`と同じ方式でサマライズします。

具体的には、同じクエリに対してそれぞれの実行時間、ロック時間、取得行数を集計し、合計、平均を求めます。

また、クエリについてはパラメタを抽象化しており、条件の値が違うだけのクエリであれば同じクエリと見なします。

```sql
SELECT * FROM `users` WHERE `users`.`name` = 'hoge' AND `users`.`age` > 20
```

```sql
SELECT * FROM `users` WHERE `users`.`name` = 'fuga' AND `users`.`age` > 30
```

のようなクエリは

```sql
SELECT * FROM `users` WHERE `users`.`name` = 'S' AND `users`.`age` > N
```

のように抽象化され、同じクエリと見なします。

# インストール

RubyGemsに公開しているのでGemfileに書いてbundle installすればOKです。

```ruby
gem 'mysql_dump_slow'
```

# まとめ

ログファイルの場合、解析するためにファイルを取得する必要があって少し手間がかかっていたのが、テーブルに出力することでクエリで取得できるようになって取り扱いしやすくなりました。

実装にあたっては同僚の[@mizoR](https://github.com/mizoR)にいろいろアドバイスもらえて助かりました。Thanks!

スロークエリが常に通知される状態になったことで、どんどん撲滅していくぞという空気が生まれるんじゃないかなと思ってます。よければ使ってみてください。

