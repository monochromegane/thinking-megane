---
title: "半歩進むChef-Solo - Cookbookの共通化（recipeとdefinition）"
date: 2013-05-05
comments: true
tags: [ "vagrant", "chef" ]
---

# 半歩進むChef-Solo

最近、Vagrant + Chef-Soloによるローカル開発環境Boxイメージ構築自動化に取り組んでいます。
Cookbookをつくるうえでのノウハウがたまってきたので、まとめの意味も込めて「半歩進むChef-Solo」シリーズでエントリ書いていこうと思います。

今日はChefで行う「共通化」についてです。

<br/>
<hr/>

# Cookbookの共通化（recipeとdefinition）

Vagrant環境でChef-Soloによる構築自動化を行なっていると、複数の環境で似たような処理を行なっている箇所が出てくると思います。
例えば、同じResouceの組み合わせた処理が何箇所も出てきたり、特定のサーバソフトウェアに関連するセットアップをいろんなRecipeの中でやっていたり。

Chefにはこれらの処理をまとめるために、definitionとrecipeという仕組みがあります。


## definitionとrecipe

definitionはリソースを組み合わせた独自のリソースを定義できる仕組みです。
同じ処理を何箇所も定義している場合、definitionによる共通化が役立ちます。

recipeはChefに実行させる内容を定義する本体です。
レシピは、外部のレシピからその単位で呼び出すことができるため、大きな範囲で処理を共通化するのに役立ちます。

<br/>
<hr/>

# definitionによる共通化

## definitionを使う

### 1. definitionを定義する

apacheのインストール、初期セットアップを行うdefinitionを定義します。

`cookbooks/apache/definitions/apache_setup.rb`

```ruby
define :apache_setup do
  # インストール
  yum_package "httpd" do
    action :install
  end
  # 初期セットアップ
  template "/etc/httpd/conf/httpd.conf" do
    source "httpd.conf.erb"
  end
  # 自動起動
  service "httpd" do
    action [:enable]
  end
end
```

### 2. definitionを呼び出す（クックブック内）

定義したdefinitionをレシピから呼び出します。

`cookbooks/apache/recipes/default.rb`

```ruby
apache_setup # definitionで定義した独自リソースを呼び出す
```

### 3. definitionを呼び出す（クックブック外）

definitionはクックブック外からも呼び出すことができます。

`cookbooks/main/recipes/default.rb`


```ruby
apache_setup # apacheクックブックでで定義した独自リソースを呼び出す
```

<br/>

## パラメタによる汎用的なdefinition

definitionは以下のようにパラメタを渡すことができます。

```ruby
sample_definition "arg1" do
  arg2 "arg2"
end
```

definition内ではparamsという名前のハッシュで扱うことができます。

```
define :sample_definition, :arg2 => "default", :arg3 => "default" do
  params[:name] #=> "arg1"
  params[:arg2]   #=> "arg2"
  params[:arg3]   #=> "default"  
end
```

環境によって変化する箇所をパラメタ化しておくことで汎用的なdefinitionとすることができます。

Apacheの設定を行うdefinitionの場合、サーバ名、ドキュメントルートなどがパラメタ化の対象となるでしょう。


<br/>
<hr/>

# recipeによる共通化

## recipeを使う

### 1. recipeを定義する

apacheのインストール、初期セットアップを行うrecipeを定義します。

`cookbooks/apache/recipes/setup.rb`

```ruby
# インストール
yum_package "httpd" do
  action :install
end
# 初期セットアップ
template "/etc/httpd/conf/httpd.conf" do
  source "httpd.conf.erb"
end
# 自動起動
service "httpd" do
  action [:enable]
end
```

### 2. recipeを呼び出す（レシピ単位での呼び出し）

Vagrantfileなどのレシピ呼び出し機構からレシピ単位で呼び出す


`Vagrantfile`

```ruby
Vagrant::Config.run do |config|
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = "cookbooks"
    chef.add_recipe "apache::setup" # apacheクックブックのsetupレシピを呼ぶ
  end
end
```

この場合は、レシピの呼び出しをmainクックブックではなく、Vagrantが行うことになります。
単純なレシピの組み合わせでよい場合などはこの形式でもよいでしょう。 


### 3. recipeを呼び出す（他のレシピからの呼び出し）

他のレシピ（ここではmainクックブックのレシピ）から`include_recipe`リソースを使って呼び出す

`cookbooks/main/recipes/default.rb`

```ruby
# apacheクックブックのsetupレシピを呼ぶ 
include_recipe "apache::setup"
```

<br/>
## パラメタによる汎用的なrecipe

レシピに対して直接パラメタを渡すことはできません。
しかし、attributeという仕組みを用いることでレシピ内で環境ごとの変数を定義することができます。

### attributeを利用する

attributeに定義した変数はレシピ内で`node[:key]`の形式で呼ぶことができます。
Chef-Soloでは呼び出し時のjsonパラメタを渡すことができます。
Vagrantの場合、Vagrantfileで上記jsonパラメタを定義します。

`cookbooks/apache/attributes/default.rb`

```ruby
# apacheクックブックで定義するデフォルトの属性
default[:apache][:attr1] = "default"
```

`Vagrantfile`

```ruby
Vagrant::Config.run do |config|
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = "cookbooks"
    chef.add_recipe "apache::sample_attribute"
    chef.json = {:apache => {:attr1 => "attr1"}} # この環境での属性値を指定
  end
end
```

`cookbooks/apache/recipes/sample_attribute.rb`

```ruby
node.apache.attr1 #=> "attr1" jsonパラメタを指定しない場合は"default"
```

### 参考

ちなみにVagrant + Chef-SoloではVagrantfileに定義された内容はChef-Solo用のjson形式に変換されます。
変換後のファイルは仮想マシン上に配置されます。

`/tmp/vagrant-chef-1/dna.json`

```json
{
  "instance_role": "vagrant",
  "run_list": [
    "recipe[apache::sample_attribute]"
  ],
  "apache": {
    "attr1": "attr1"
  }
}
```

<br/>
<hr/>

# definitionとrecipeの使い分け

クックブック内での共通化はもちろんdefinitionを使います。
クックブックを超えた共通化ではdefinitionとrecipeの両方を用いることができます。

その場合の使い分けは、**パラメタの有無**によると考えています。

前述のようにrecipeではパラメタを渡すためにattributeの仕組みを用いる必要がありますが、これは変数名や、その変数が内部で何に使われているかという内部構造を意識した使い方をする必要があります。
逆にdefinitionは独自リソースとして、呼び出しパラメタを定義することができるため、公開インターフェースとして適しています。

自分でクックブックを作成するときは、**パラメタが必要ない処理はrecipeとして定義し、パラメタが必要となる処理はdefinitionを用いる**ことにしています。

<br/>
<hr/>

長くなってきたので今回はこの辺で。
次回はdefinitionを使う際のtipsなどを書いていこうと思います。


