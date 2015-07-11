---
title: "pecoでVagrant操作を便利にする"
date: 2014-09-08
comments: true
categories: peco vagrant golang
---

開発環境で利用するVagrantのVMの数が増えてきたのでpecoでVM選んで起動や停止をできるようにしてみました。

# vagrant-peco

こんな感じで利用します。

```console
$ vagrant-peco up
```

![](https://raw.githubusercontent.com/monochromegane/vagrant-peco/master/images/vagrant-peco-up.gif)

<br />

## うれしさ

- pecoでVMを選択できる。Vagrantfileのあるディレクトリに移動しなくてよい
- VMを複数選択して一括で起動・停止できる
- 通常のvagrant global-statusよりも高速に表示
- direnvとの連携

<br />

## インストール

Vagrant1.6以降でつくったVMと`vagrant-peco`、`vagrant-global-status`が必要です。

```console
# vagrant-global-statusをインストール
$ go get github.com/monochromegane/vagrant-global-status/...

# vagrant-pecoをインストール
$ cd [PATH] # PATHの通ったディレクトリ
$ curl -O https://raw.githubusercontent.com/monochromegane/vagrant-peco/master/vagrant-peco
$ chmod +x ./vagrant-peco
```

<br />

# 仕組み

## vagrant global-status

Vagrantの1.6からglobal-statusプラグインが取り込まれ、一度起動したVMにIDが割り振られるようになりました。

IDは`vagrant global-status`コマンドで確認します。

```console
$ vagrant global-status
id       name          provider   state    directory
-------------------------------------------------------------------------------
14c9626  cross_compile virtualbox poweroff /Users/miyakey/Documents/vm/golang
dffc2a5  develop       virtualbox poweroff /Users/miyakey/Documents/vm/golang
ae3968b  web1          virtualbox poweroff /Users/miyakey/Documents/vm/dev
825680e  web2          virtualbox poweroff /Users/miyakey/Documents/vm/dev
e0aa5e9  db            virtualbox poweroff /Users/miyakey/Documents/vm/dev
```

このIDを使うことでVagrantfileのあるディレクトリに移動することなく任意のVMを操作できます。

```console
$ vagrant up 14c9626
```

<br />

## pecoと連携する

毎度、IDを選ぶのは面倒なので peco で選べるようにします。

先に公開されている@gongoさんの[pecrant](https://github.com/gongo/pecrant)でほぼばっちりです。

ただ、自分の環境だと`vagrant global-status`の起動が少し遅く感じたのと、Vagarntディレクトリごとの`direnv`定義を使いたかったので、以下のようにして解決しています。

### vagrant-global-status

- [vagrant-global-status](https://github.com/monochromegane/vagrant-global-status)

`vagrant global-status`の起動が遅い問題を解決するためにつくりました。  
同コマンドが参照するファイルを直接解析して結果を出力しています。ちょっとお行儀が悪いです。

自分の環境だと、1秒弱短縮できました。さくさく起動します。

```console
$ time vagrant global-status
vagrant global-status  0.97s user 0.09s system 99% cpu 1.072 total

$ time vagrant-global-status
vagrant-global-status  0.00s user 0.00s system 68% cpu 0.007 total # fast!!
```

### vagrant-peco

- [vagrant-peco](https://github.com/monochromegane/vagrant-peco)

Vagrant起動時に`direnv`を適用したかったのでつくりました。  
`vagrant-global-status`で選択したIDで操作する前に、該当ディレクトリの`.envrc`を読み込みます。

Vagrantfileはプロジェクト共通でマウントするソースファイルのディレクトリは個人ごとに環境変数で指定するような場合に都合がよいです。

<br />

---

vagrant-pecoは、だいぶ自分の環境にあわせた感じになってしまいましたが、VagrantのVMが増えてきた方、pecrant含めて、使ってみてはどうでしょうか。

いや〜、peco、便利。

