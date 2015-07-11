---
title: "Go言語のDB接続情報をRailsのdatabase.ymlから借用するライブラリをつくった"
date: 2015-07-05
comments: true
categories: golang rails database
---

最近、Rails資産のあるサーバーでGoツールを動かすことがあったので、DB接続情報を共通で使えるようにしたいと思い小さなライブラリをつくりました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fdsn" title="monochromegane/dsn" class="embed-card embed-webcard" scrolling="no" frame    border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/dsn"&gt;monochromegane/dsn&lt;/a&gt;</iframe>

Railsのdatabase.ymlからGoのsql.Openに渡すDataSourceNameを出力します。

<br />
## 使い方

database.ymlのパスと環境(productionやdevelopment)をパラメタとして渡してあげるだけです。

```go
name, dsn, _ := dsn.FromRailsConfig("myapp/config/database.yml", "production")
fmt.Printf("[%s] %s", name, dsn)
// => [mysql] username:password@tcp(localhost:3306)/dbname
```

DSN、書式覚えるの厳しいので分かりやすいフォーマットの設定ファイルから読み込めるようになるのもうれしい感じです。

<br />

## 機能

今のところ、MySQL, PostgreSql, Sqlite に対応しています。

<br />
---

# 実装と今後

単純にYAMLを解析しているだけなので、ERBが含まれる項目はうまく動作しません。

そのうち、環境変数からやフラグからもDSNを生成できるようにするつもりですが、まだ自分の用途では特に必要になってないのでつくってません。よければPRお待ちしております。
