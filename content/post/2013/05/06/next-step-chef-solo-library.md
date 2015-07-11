---
title: "半歩進むChef-Solo - Cookbookの共通化（library）"
date: 2013-05-06
comments: true
categories: chef vagrant
---

# Cookbookの共通化（library）

[前回エントリ](http://blog.monochromegane.com/blog/2013/05/05/next-step-chef-solo-recipe-and-definition/)ではrecipeとdefinitionを用いたCookbookの共通化の手順を紹介しました。
今回はChefのもうひとつの共通化の仕組みである**library**を紹介します。

<br/><hr/>

# libraryって

libraryはRubyコードを用いて、Chefに新しいクラスやメソッドを追加することができる仕組みです。

libraryはクックブック内のlibraries/library_name.rbに定義することで自動で読み込まれ、recipes, attributes, file, definitions, providers, definitionsで利用することができます。

libraryの用途は以下の様なものがあります。

- ファイルに格納されている属性値へのアクセス
- ループのようなプログラムテクニックの利用
- Chefのレシピから直接呼び出せるような独自名前空間の作成（Chef::Recipeの名前空間をきれいな状態に保つ）
- データベースへの接続
- LDAPプロバイダとの接続
- その他、Rubyでできることはなんでも

<br/><hr/>

# libraryを使う

## 基本

### 1. libraryを定義する

一番かんたんな利用法は、Chef::Recipeクラスにメソッドを追加する方法です。
例としてChefのキャッシュディレクトリを取得するメソッドを定義してみます。

`libraries/cache.rb`

```ruby
def cache_path
  Chef::Config[:file_cache_path]
end
```

マニュアルではChef::Recipeクラスに明示的に定義するようになっていますが、上記の記法でもChef::Recipeクラスに定義されるようです。

### 2. libraryを使う

レシピ内などで呼び出すことができます。

`recipes/default.rb`

```ruby
remote_file "#{cache_path}/sample.tar.gz" do
  # 省略
end
```

<br/>
## 名前空間

libraryが大きくなるようであれば、名前空間の分割を考える必要があります。
以下のようにChef::Recipe::XXXのように名前空間を分けて定義するか、後述のmodule化を検討してください。

`libraries/cache.rb`

```ruby
class Chef::Recipe::Cache
  def self.path
    Chef::Config[:file_cache_path]
  end
end
```

`recipes/default.rb`

```ruby
remote_file "#{Cache.path}/sample.tar.gz" do
  # 省略
end
```


<br/>
## module構成

名前空間の分割にはmoduleを利用することもできます。
opscodeでの利用例はmodule構成にして、helperとして定義する例が多いようです。

`libraries/helper.rb`

```ruby
module Helper
  def cache_path
    Chef::Config[:file_cache_path]
  end
end
```

`recipes/default.rb`

```ruby
# moduleのinclude
::Chef::Recipe.send(:include, Helpers) 
# メソッドを利用する
remote_file "#{cache_path}/sample.tar.gz" do
  # 省略
end
```

<br/>
## attirubuteへのアクセス

マニュアルにはChef::Recipeクラスの@node変数経由で取得する記述になっていますが、通常通り`node.attr`もしくは`node[attr]`の取得方法で取得することができます。

<br/>
## resourceの利用

library内でもresource類を使うことができます。

<br/><hr/>

前回のrecipe, definitionに続き、共通化という観点でlibraryを紹介しました。

次回はdefinition利用時のtipsを紹介する予定です。


