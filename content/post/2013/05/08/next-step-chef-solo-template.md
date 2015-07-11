---
title: "半歩進むChef-Solo - definitionでtemplateを使うとき気をつけること"
date: 2013-05-08
comments: true
tags: [ "chef" ]
---

# definitionでtemplateを使うとき気をつけること

Chefのdefinitionは、独自リソースを定義して、外部のCookbookからも呼び出せるため共通化に役立つ便利な仕組みです。
しかし、definition内でtemplateリソースを利用すると`Chef::Exceptions::FileNotFound`エラーが出る場合があります。

<br/>
## 現象

templateリソースを利用したdefinitionを外部のCookbookから呼び出した場合に`Chef::Exceptions::FileNotFound`が発生します。

<br/>
## 原因

templateリソースは、現在のCookbookのtemplatesディレクトリからテンプレートファイルを探すためです。

下記の場合だと呼び出し元となるmainクックブックのtemplatesディレクトリから探す挙動となってしまいます。

`cookbooks/apache/definitions/apache_setup.rb`

```ruby
define :apache_setup do
  template "/etc/httpd/conf/httpd.conf" do
    source "httpd.conf.erb"
  end
end
```

`cookbooks/main/recipes/default.rb`

```ruby
# この場合、mainクックブックのtemplatesディレクトリからhttpd.conf.erbを探してしまう
apache_setup
```

<br/>
## 対策

外部から呼ばれる可能性があるdefinition内のtemplateリソースには、検索対象となるCookbookを明示しておきます。

`cookbooks/apache/definitions/apache_setup.rb`

```ruby
define :apache_setup do
  template "/etc/httpd/conf/httpd.conf" do
    cookbook "apache" # apache/templatesディレクトリから検索する
    source "httpd.conf.erb"
  end
end
```

<br/>
以上です
