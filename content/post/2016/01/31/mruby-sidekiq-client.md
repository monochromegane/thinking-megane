+++
date = "2016-01-31T14:44:11+09:00"
title = "mrubyからSidekiqに非同期ジョブを登録するmrbgemをつくった"
tags = [ "mruby" ]
+++

ngx_mrubyでHTTPリクエストに対して非同期処理をしたかったので、`mruby-sidekiq-client`という mrbgem をつくりました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fmruby-sidekiq-client" title="monochromegane/mruby-sidekiq-client" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/mruby-sidekiq-client"&gt;monochromegane/mruby-sidekiq-client&lt;/a&gt;</iframe>

mrubyからSidekiqのジョブ形式でRedisに非同期ジョブを登録し、別途用意したSidekiqのサーバーが登録されたジョブを捌いていくという方式です。これにより、非同期処理は通常のRailsやCRubyのコードで書くことができ、mruby側の処理はシンプルに保つことができます。

## 使い方

RailsなどでSidekiqを使っている方にはお馴染みのやり方です。

### 1. 非同期ジョブにしたいWorkerクラスを定義します

```rb
class HardWorker
  include Sidekiq::Worker
end
```

※ `perform`メソッドはmruby側では実装する必要はありません。Sidekiqサーバー側のRubyコードに実装してください。

### 2. 非同期ジョブを登録する処理を定義します。

```rb
HardWorker.perform_async('bob', 5)
```

指定時間後に実行するジョブも登録できます

```rb
HardWorker.perform_in(300, 'bob', 5) # 300秒後に実行
```

mruby側はこれだけです。あとは先ほどのWorkerクラスと同じクラスをSidekiqサーバー側に追加し、performメソッドに非同期ジョブの内容を記述します。


## ngx_mrubyで使う

mruby_userdataを使って、リクエストごとにRedis再接続しないようにしておきます。

```
http {

    (snip)

    mruby_init_code '
        userdata = Userdata.new "redis_data_key"
        userdata.redis = Redis.new "localhost", 6379
    ';

    server {

        (snip)

        location /hello {
             mruby_content_handler_code '
                 userdata = Userdata.new "redis_data_key"
                 Sidekiq.redis = userdata.redis

                 class MyWorker; include Sidekiq::Worker; end

                 (snip)

                 MyWorker.perform_async(args)
             ';
        }

```

## Sidekiqサーバー

SidekiqサーバーはPlain Rubyの環境でも動作するため、ngx_mrubyとRedis、Rubyだけでシンプルなジョブキューをつくることができます。
手順はこちらを参考にしてください。

- [Sidekiq example - Plain old ruby](https://github.com/mperham/sidekiq/blob/master/examples/por.rb)

## インストール

`build_config.rb` に以下を記述してください。

```rb
conf.gem :github => 'monochromegane/mruby-secure-random'
conf.gem :github => 'monochromegane/mruby-sidekiq-client'
```

[mruby-secure-random](https://github.com/monochromegane/mruby-secure-random)は`SecureRandom`をmrubyで使えるようにするためにつくったmrbgemです。SidekiqのJIDを算出するために利用しています。ランダムデバイスとして `/dev/urandom` しか対応していないため Windows環境だと動きません。PullRequestお待ちしております。


## まとめ

こんな感じでmrubyからSidekiqと連携できるようにしてみました。シンプルなジョブキューであればRails使わずにngx_mrubyとRedis、Rubyだけでつくれてしまうので便利では〜と個人的には思っています。

あわせて、今後、mruby使った動的なフロントサーバーがサービスのどの要件までを担当すべきかといった構成は常に考えていかなければいけないだろうなあとも感じています。適材適所それぞれの利点を最大限に活かせる構成を検討したいところ。

