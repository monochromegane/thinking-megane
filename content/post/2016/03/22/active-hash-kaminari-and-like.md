+++
date = "2016-03-22T20:46:48+09:00"
title = "静的データを扱うActiveHashでページングとライク検索するgem達をつくった"
tags = [ "ruby" ]
+++

静的データをActiveRecord的に扱えて便利な[ActiveHash](https://github.com/zilkey/active_hash)ですが、ページングとライク検索が必要になったのでgemをつくりました。

---

ページングを行えるようにする `active_hash-kaminari` と、

<iframe src="//hatenablog-parts.com/embed?url=https://github.com/monochromegane/active_hash-kaminari" title="monochromegane/active_hash-kaminari" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"><a href="https://github.com/monochromegane/active_hash-kaminari">monochromegane/active_hash-kaminari</a></iframe>

ライク検索を行えるようにする `active_hash-like` です。

<iframe src="//hatenablog-parts.com/embed?url=https://github.com/monochromegane/active_hash-like" title="monochromegane/active_hash-like" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"><a href="https://github.com/monochromegane/active_hash-like">monochromegane/active_hash-like</a></iframe>

# active_hash-kaminari

ページングを行いたいActiveHashのクラスに`Paginatable`モジュールを`prepend`します。

```rb
class Country < ActiveHash::Base
  prepend ActiveHash::Paginatable
  ...
end
```

ページングには[Kaminari](https://github.com/amatsuda/kaminari)を使っています。モジュールをprependすることにより、検索結果が `Kaminari::PaginatableArray` でラップされるようになり、ページングが可能になります。

```rb
Country.all.page(1).per(10)
```

もちろんViewでも使えます。

```rb
<%= paginate @counties %>
```

# active_hash-like

gemをインストールすると、ActiveHashで`like`が使えるようになります。

```rb
class Country < ActiveHash::Base; end

Country.like(name: 'Cana%')
```

複数条件のうち、ひとつをlikeで検索する場合はwhere内`ActiveHash::Match::Like` マッチャーを使います。

```rb
Country.where(name: ActiveHash::Match::Like.new('Cana%'))
```

## Custom Matcher

`ActiveHash::Match::Like`はカスタムマッチャーのひとつで、マッチャーは独自でつくることが可能です。

マッチャーは`call`メソッドを持つ必要があります。

```rb
class MyCustomMatcher
  attr_accessor :pattern

  def initialize(pattern)
    self.pattern = pattern
  end

  def call(value)
    # Case ignore matcher
    value.upcase == pattern.upcase
  end
end

Country.where(name: MyCustomMatcher.new('pattern'))
```

`call`メソッドを持つ`Proc`オブジェクトも使うことができます。

```rb
Country.where(name: ->(value){value == 'some value'})
```

## OR 条件

あいまい検索という括りで`OR`検索も行うことができるようになります。

```rb
Country.where(name: 'Canada', or: {field1: 'foo', field2: 'bar'})
#=> name = 'Canada' and (field1 = 'foo' or field2 = 'bar')
```

OR条件の対象のカラムが同じ場合（IN相当）はまだ未実装なのでカスタムマッチャーを使ってください。

```rb
Country.where(name: ->(val){val == 'Canada' || val == 'US'})
```

# まとめ

ActiveHash、静的データを扱うのにとても便利なので、active_hash-kaminariとactive_hash-likeと組み合わせて更に便利に使ってみてはどうでしょうか。

