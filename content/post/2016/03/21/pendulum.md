+++
date = "2016-03-21T18:29:18+09:00"
title = "Treasure Dataのスケジュールジョブをコードで管理するPendulumというgemをつくった"
tags = [ "ruby" ]
+++

Treasure Dataに収集したデータを集計・出力するためにジョブをスケジュール登録するにあたり、ブラウザコンソールやAPIから直接行うと履歴管理やレビューができないといった課題を解決するために `Pendulum` というgemをつくりました。

<iframe src="//hatenablog-parts.com/embed?url=https://github.com/monochromegane/pendulum" title="monochromegane/pendulum" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"><a href="https://github.com/monochromegane/pendulum">monochromegane/pendulum</a></iframe>

PendulumはDSLで記述された定義に従い、Treasure Dataのスケジュールジョブを管理します。
定義ファイルをGit管理することで、履歴管理やGitHubと連携したコードレビューが可能になります。

余談ですが、Pendulumは振り子という意味で、定期的な実行という意味と宝探しのダウジング的な意味から連想しています。ペンデュラム。響きがカッコイイ。

## 使い方

`Schedfile`という名前で定義ファイルを用意して、

```rb
schedule 'my-schedule-job' do
  database 'db_name'
  query    'select count(time) from access;'
  cron     '30 0 * * *'
  result   'td://@/db_name/count'
end
```

`pendulum`コマンドを実行するだけです。

```sh
# dry-run で 適用内容を確認
$ pendulum --apikey='...' -a --dry-run
# 適用
$ pendulum --apikey='...' -a
```

AWSのRoute53に対するRoadworker的なものを想像してもらえるとよいかと思います。

### エクスポート

既存のスケジュールジョブがある場合はエクスポートもできます。

```sh
$ pendulum --apikey='...' -e
```

実行ディレクトリに `Schedfile` と `queries` ディレクトリが生成されジョブの定義とクエリが出力されています。

修正後、dry-runによるapplyを行ってみると、差分が検出されていることがわかります。

```sh
$ pendulum --apikey='...' -a --dry-run
Update schedule: my-scheduled-job (dry-run)
  set cron=@hourly
```

### Schedfile

APIで利用する名称と同じものが使えます。

```rb
schedule 'test-scheduled-job' do
  database    'db_name'
  query       'select count(time) from access;'
  retry_limit 0
  priority    :normal
  cron        '30 0 * * *'
  timezone    'Asia/Tokyo'
  delay       0
  result_url  'td://@/db_name/count'
end
```

`priority`は`:very_high`など、`cron`は`:daily`といった読みやすい値も使えます。

#### query_file

クエリは`query_file`を使って別ファイルのクエリを読み込めます。

```rb
query_file 'queries/test-scheduled-job.hql'
```

集計用のクエリは長くなることが多いと思うので、こちらを使っていくことになるでしょう。

#### result

Result Exportの先を定義する、`result_url` をわかりやすくするための `result` もあります。

```rb
schedule 'test-scheduled-job' do
  database   'db_name'
  ...
  result :td do
    database 'db_name'
    table    'table_name'
  end
end
```

現時点では出力先として`Treasure Data`と`PostgreSQL`、カスタムのresultをサポートしています。他の出力先をresult記法で書きたいときはPullRequestをお待ちしております。

### force

`Schedfile`に定義されていないスケジュールジョブは通常は`Undefined schedule`となり削除対象とはなりません。削除が必要な場合は`--force`オプションを使ってください。

また、Treasure DataのAPI都合上、result_url内のパスワードは`***`とマスキングされており差分が比較できません。もしパスワードを変更した場合も同様に`--force`オプションを使って適用してください。

### config

`environments/default.yml`などを用意することで、DSL内で`settings`経由で取得することができます。

```rb
...
  result :postgresql do
    ...
    password settings.password
  end
```

また、`-E` オプションによる環境ごとの設定ファイルにも対応しています。

## まとめ

コード管理により履歴管理やレビューといったフローに乗せることができるので大変便利になりました。そのあたりの運用で困っているかたは使ってみてはどうでしょうか。

