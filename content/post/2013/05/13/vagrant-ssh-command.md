---
title: "ちょっと便利なvagrant sshのコマンドオプション"
date: 2013-05-13
comments: true
tags: [ "vagrant", "chef" ]
---

`vagrant ssh`にはコマンドオプションがあり、sshで仮想マシンにログインしなくても実行結果を取得することができます。

### コマンドオプション

vagrant ssh のコマンドオプションは`-c command`です。
`-c`オプションを使うことで、SSHコマンドを直接実行することができます。

例えば、こんな使い方。
<br/><hr/>

### DHCPな仮想マシンのIPアドレスを知る

```console
$ vagrant ssh -c ifconfig
```

grepなりで取得結果を整形すれば、仮想マシンのWebサーバへのブラウザアクセスなどに利用できます。

<br/><hr/>

### Chef-Soloのレシピを個別に実行する

Vagrant + Chef-Solo環境であれば、以下のコマンドで個別にレシピを実行することができます

```console
# Vagrantfileで定義したレシピを全て実行
$ vagrant ssh -c "cd /tmp/vagrant-chef-1 && sudo chef-solo -c solo.rb -j dna.json"

# レシピを指定して実行
$ vagrant ssh -c "cd /tmp/vagrant-chef-1 && sudo chef-solo -c solo.rb -o cookbookName::recipeName"
```

<br/><hr/>
ログインの手間を省けるので、ちょっとした確認などに便利ではないでしょうか。
