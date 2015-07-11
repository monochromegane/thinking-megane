---
layout: post
title: "Chef-Solo + Knife-Solo + Berkshelf  環境のつくりかた"
date: 2013-08-20
comments: true
categories: chef chef-solo knife-solo berkshelf
---
Chef-Solo + Knife-Solo + Berkshelf 環境を構築するときに少しはまったので、まとめておきます。

ブログ投稿時点（2013/08/20）では knife-solo のバージョンだけ気をつけておけば大丈夫です。
knife-solo のバージョンが古いと2回目以降の実行時にうまくいかないためです。
その他の構築自体は難しくありません。

以下、構築手順です。

# 1. 前提

構築対象となるリモート側のサーバへはすでにSSHによるログインが可能であるものとします。

<br />

# 2. chef, knife-solo, berkshelf のインストール

以下のGemfileを用意して`bundle install`を行います。

`Gemfile`

```ruby
source "https://rubygems.org"

gem 'chef'
gem 'knife-solo', '>= 0.3.0.pre5'
gem 'berkshelf'
```

** knife-solo のバージョンについて **

指定しない場合、0.2.0系がインストールされますが、2回目以降の実行時にエラーとなります。
原因などはこちらが詳しいです。

[Hack like a rolling stone - knife-solo 0.2.0 で rsync エラーによって苦しまないためのたったひとつの方法](http://tk0miya.hatenablog.com/entry/2013/04/18/011339)

<br />

# 3. cookbookの準備

## Chefのリポジトリをつくる

```console
$ knife solo init chef-repo
```

chef-repo内に以下の構成が生成されます。必要に応じてGit管理を行ってください。

```console
 $ tree chef-repo/
 chef-repo/
 ├── cookbooks
 ├── data_bags
 ├── nodes
 ├── roles
 └── site-cookbooks
```

** Cookbookを置く場所について **

自分でつくるCookbookは`site-cookbooks`以下に、公開Cookbookは`cookbooks`以下に置きます。

knife-soloでは`cookbooks`配下は`.gitignore`で除外対象となっており、後述のBerkshelfなどの利用を前提としているようです。

** knife-solo はsshコマンドのオプションが使えます。証明書の指定などがある場合に便利です。 **

<br />

# 4. Cookbookの作成

## 自分でCookbookをつくる場合

今回は環境構築をメインにするので、雛形作成の手順のみ。

```console
$ knife cookbook create xxx -o site-cookbooks/
``` 

<br />

## 公開Cookbookを利用する場合

リポジトリ直下に`Berksfile`を作成する

```ruby
site :opscode
cookbook "public_cookbook_name"
```

<br />

# 5. リモート側でChef-Soloを実行する

## 実行するレシピの指定

`nodes`以下に`severname.json`が作成されているので、run_list に実行したいレシピを指定する。

```json
{
  "run_list": [
    "public_cookbook_name::recipe_name", "cookbook_name::recipe_name"
  ]
}
```

** 公開Cookbookも忘れずに追加しておきます **

<br />

## リモート側でのChef-Solo実行準備

```console
$ knife solo prepare username@servername
```

これにより、リモート側にChefがインストールされます。

<br />

## リモート側でのChef-Solo実行

```console
$ knife cook username@servername
```

これによりリモート側にCookbookが送信され、run_listで指定したレシピが実行されます。


<br />

# 6. Tipsなど

## knife-soloでattributesを使う

nodes/servername.jsonに追加

```json
{
  "run_list": [
    "cookbook_name::recipe_name"
  ],
  "hoge": {
    "fuga": "piyo"
  }
}
```

この例ではrecipe内で`node.hoge.fuga`として値を利用できます。

<br />
---

Vagrant + Chef-Solo の環境構築は手軽にできますが、いざVagrant以外に適用しようとすると意外と面倒です。

今回のようにknife-soloをかませておくことで、対象のサーバを選ばずにChef-Soloを実行でき、とても便利です。

環境構築を行うときは、Berkshelfによる公開Cookbookの管理とあわせた今回の構成をおすすめします。

