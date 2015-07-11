---
layout: post
title: "[Ruby] ThorでWebAPIサービスを立ち上げるGem\"トールハンマー\"をつくった"
date: 2013-06-15 14:25
comments: true
categories: thor ruby rails gem
---

コマンドラインツールをつくって、もっと多くのひとに気軽に利用してもらいたいとき、Webアプリとして作りなおすのが面倒だなーと思うことがあります。

コマンドラインツールをそのままWebアプリとして使えるようにする仕組みが欲しくて、そんな Gem、`ThorHammer`トールハンマーをつくりました。

<br />
<hr />

# Thorって？

[Thor](http://whatisthor.com/)（トール、ソー、雷神）はRubyでコマンドラインツールをかんたんにつくるためのGemです。

<br />
<hr />

# ThorHammerって？

[ThorHammer](https://github.com/monochromegane/thor_hammer)はThorでつくったCLIをWebAPIにするGemです。

マイティー・ソーが武器のハンマーをつかって、雷を広げて攻撃するように、ThorでつくったCLI機能をWebに広げるGemです。※1

![Thor Hammer]({{ root_url }}/images/2013/06/thor_hammer.png) 

<br />
<hr />

# どう使うの？

ThorHammerは、RailsアプリとしてWebAPIを公開します。

<br />

## インストール

Railsのプロジェクトを作成して、Gemfileに以下を追記します。

```ruby
gem 'thor_hammer'
gem 'Your Thor CLI' # WebAPIとして公開するThorのCLI
```

あとは`bundle install`でOKです。

<br />

## ジェネレータ

ThorHammerは、WebAPIを作成するためのジェネレータを提供します。

```console
$ rails g thor_hammer:api ThorCliのクラス名 公開APIのパス
```

引数に指定する値は以下を参考にしてください。

### ThorCliのクラス名

第一引数は、Thor CLI の起点となるクラス名を指定します。
起点となるクラス名は、Thor を継承したクラスとして作成されているはずです。
また、モジュール名も必要になります。

以下のような Thor CLI の場合、`SampleThorCli::Runner`を指定します。
```ruby
module SampleThorCli
  class Runner < Thor
    # 中略
  end
end
```

### 公開APIのパス

第二引数は、WebAPIの公開時のパスを指定します。
省略時は、第一引数を`underscore`したものが適用されます。

* 省略時

`rails g thor_hammer:api SampleThorCli::Runner` => `http://hostname/sample_thor_cli/runner`

* 指定時

`rails g thor_hammer:api SampleThorCli::Runner api` => `http://hostname/api`

<br />

## アクセス

以上で、あなたのThor CLI がWebAPIとして公開する準備が整いました。
`rails server`でRailsアプリを起動し、アクセスしてください。

```console
$ curl http://hostname/api/subcommand/arg1,arg2
```

CLIとして利用した場合と同じ結果をそのまま取得することができます。（JSON, XML形式への対応は今後予定）


### subcommand

サブコマンドは、あなたのThor CLIのサブコマンドを指定します。

### arg1,arg2

サブコマンドに引数があれば、指定することができます。
引数が複数の場合は、`,`を間に挟んで指定します。

<br />
<hr />

今はThor限定ですが、Rakeタスクなんかもできるようにして、CIと組み合わせると面白くなるかも。

<br />
<hr />

※1. はい、もちろん後付けです。

