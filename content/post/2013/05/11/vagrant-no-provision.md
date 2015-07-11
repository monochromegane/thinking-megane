---
title: "Vagrant起動時にChef-Soloの実行を省く"
date: 2013-05-11
comments: true
categories: vagrant chef
---

Vagrantの起動時にはChef-Soloが実行されますが、きちんと冪等性をもたせたレシピでもChef-Soloの実行には多少の待ちが発生してしまいます。

そこで今回は起動時間を短縮させるためのオプションを紹介します。

<br/><hr/>
## --no-provision オプションを使う

Vagrant起動時にChef-Soloの実行を省くには `vagrant up`時に`--no-provision`オプションをつけます

もちろん`vagrant reload`時にも使えます

<br/><hr/>
## --provision-with オプションを使う

`--no-provision`はすべてのprovisionの実行を省いてしまうため、`config.vm.provision :shell`など別のprovisionも実行されなくなってしまいます。

特定のprovisionを実行させたい場合は、`vagrant up`時に`--provision-with x,y,z`として実行したいprovisionを指定するとよいです。

```console
$ vagrant up --no-provision # すべてのprovisionが実行されない
$ vagrant up --provision-with shell # :shell provisionだけが実行される
```

<br/><hr/>
頻繁にChef-Soloを実行する必要がない開発用VMでは、上記のオプションをデフォルトにしてもいいかも。

