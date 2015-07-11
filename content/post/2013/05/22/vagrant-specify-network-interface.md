---
layout: post
title: "Vagrantで起動時のネットワークインターフェースを指定するには"
date: 2013-05-22
comments: true
categories: vagrant
---

VagrantのPublicNetwork(Bridged)ネットワーク構成のときに、ホストマシンのネットワークインターフェースが複数あると毎回の起動時に選択するのが面倒です。

```console
$ vagrant up
-- 中略 --
[default] Available bridged network interfaces:
1) en1: Thunderbolt Ethernet
2) en0: Wi-Fi (AirPort)
What interface should the network bridge to? 1
[default] Preparing network interfaces based on configuration...
-- 後略 --
```

Vagrantfileに設定を行うことで、これを回避することができます。

<br />
<hr />
### ネットワークインターフェースの指定

Vagrantfileの`config.vm.network`に`bridge`オプションを加える。

configuration version が 2の場合
```ruby
config.vm.network :public_network, :bridge => "en1: Thunderbolt Ethernet"
```

configuration version が 1の場合
```ruby
config.vm.network :bridged, :bridge => "en1: Thunderbolt Ethernet"
```

`bridge`オプションには、起動時に表示されるネットワークインターフェース名をそのまま記述すること。


** 注：** `en1:`から必要

<br />
<hr />

これで起動時の手間をちょっとだけ省くことができました。


`bride`オプションはVagrantのドキュメント(V1)には記載がありましたが、ドキュメントがV2になってからはなぜか記載されていません。なぜだろう...


