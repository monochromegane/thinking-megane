---
title: "Fukuoka.go#1を開催しました。"
date: 2014-07-06
comments: true
tags: [ "golang", "fukuoka.go" ]
---

7月4日に **Fukuoka.go#1** を開催しました。

Fukuoka.goは福岡でGo言語に興味のあるひとが集まる場所を目指して開催する会です。

今回が初開催でしたが、参加者18名（キャンセル含めると申込自体は28名）で、福岡にも着実にGo熱が来てるなと感じています。

ということで、今回の開催した内容をまとめておこうと思います。

<br />

## タイムテーブル

[イベント概要](http://connpass.com/event/6887/)

- 19:35 - 19:45 自己紹介タイム(Go歴を教えてください)
- 19:45 - 20:20 LT: 速習Go ([@monochromegane](https://twitter.com/monochromegane))
- 20:20 - 20:40 LT: GAE/Goの始め方 ([@laco0416](https://twitter.com/laco0416))
- 20:40 - 21:40 もくもく会

<br />

## 自己紹介タイム(Go歴を教えてください)

今回の参加者は18名で、Go歴は長くて半年、大半がGoのチュートリアルである、[A Tour of Go](http://go-tour-jp.appspot.com/#1)をやり終えた（または途中までやった）ぐらいの経験の方が多かったです。

<br />
<hr />

# LT

## 速習Go ([@monochromegane](https://twitter.com/monochromegane))

自分のほうから、速習Goというタイトルで、Go環境の作り方から基本的な文法規則と注意したほうがよい点、ハマることが多い点などを説明させてもらいました。

- [LT資料: 速習Go](https://gist.github.com/monochromegane/8bb73390f2ebd9d325f4)

GoのPlaygroundもつけてGo環境がない人でも試せるようにした点はよかったですが、足りない分はもくもく会でカバーする、という方針だったのでかなり駆け足になってしまいました。

もう少し、ライブコーディング的なものもやったほうが興味ひけたかなあと思いますが、次回に活かそうと思います。

<br />

## GAE/Goの始め方 ([@laco0416](https://twitter.com/laco0416))

[@laco0416](https://twitter.com/laco0416)さんによるLTは、「GAE/Goの始め方」でした。

- [LT資料: GAE/Goの始め方](https://docs.google.com/presentation/d/1jz4h1-V4CCL0emQjxl-3BL12lGonol1A0dZrEc96oTw/pub?start=false&loop=false&delayms=3000#slide=id.p)

統合開発環境での開発のノウハウや、GAE/Goで気をつける点などをLTしていただきました。
Go言語はシンプルなHTTPサーバを立てるのは非常に簡単なのでGAEと組み合わせることで簡易なAPIサーバであればさくっとつくってしまえるみたいです。

<br />
<hr />

# もくもく会

各々で作業を持ちよって、もくもくする会をしました。今回はLTでわからなかったことを調べたり話しあったりする方々が多かったようです。

こんな感じのことを話しました。

<br />

## メソッドのレシーバ変数に self を使わないほうがよい理由とは

速習GoのLTのときに話はしたけど、明確な根拠が出せなかったので調べなおしました。

当初は、selfだとレシーバとして何を指しているのかわからないということだと考えていたのですが、まあ定義には型名もきちんとあるわけで...

<blockquote class="twitter-tweet" data-cards="hidden" lang="ja"><p><a href="https://twitter.com/monochromegane">@monochromegane</a> 以前Qiitaに記事書きましたのでよかったらどうぞ。元ネタは go-nuts MLの <a href="https://t.co/UGabHgGUSV">https://t.co/UGabHgGUSV</a> です / &quot;Goのメソッドレシーバの命名慣習 - Qiita&quot; <a href="http://t.co/xMncnSKmXv">http://t.co/xMncnSKmXv</a></p>&mdash; Hiroaki Nakamura (@hnakamur2) <a href="https://twitter.com/hnakamur2/statuses/485056364033499138">2014, 7月 4</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

いつもありがとうございます！

自分なりに整理してみました。

### 1. レシーバ変数はメソッド内で不変ではない

レシーバ変数に同じ型の別の値を設定することができてしまうので、意味的に整合性がとれるのは`self`ではなくて 型名をもとにした名前になります。  
自分の中ではこれが一番納得できる理由でした。
  
### 2. Goの標準ライブラリではレシーバ変数には self ではなく型名の先頭または二文字程度を使っている

理由といか命名慣習がなぜそうなっているのかですが、上述のとおり型名をもとにした名前なので、Client型であれば`c`とか`cl`とかにするようです。  
型名をそのまま小文字(`client`)にしないのはエクスポートしない小文字で始まる型を使う場合に同じになってしまうから、かなと思ってます。

### 3. メソッドは関数として呼ぶことができる

これも知らなかったのですが、型のメソッドは型を引数に渡して関数として実行することができるようです。

```go
type Foo int

func (f Foo) Bar() {
	fmt.Printf("My receiver is %v\n", f)
}

func main() {
	a := Foo(46)
	a.Bar()
	b := Foo(51)
	Foo.Bar(b)
}
```

であれば、レシーバ変数は引数ともみなすことができるわけで、selfとつけるのは少し違うのかなとも思えます。

<br />

## 構造体定義の型名のあとの文字列にはどうやってアクセスするんだろう

[@laco0416](https://twitter.com/laco0416)さんのLTで JSON文字列を構造体に変換する際に、JSONのキーと構造体のフィールド名をマッピングするため、このような定義をしていました。

```go
type Person struct {
	Name string `json:"name"`
}
```

型名のあとに`json:"name"`という文字列があります。プログラムからこの値にアクセスするやり方がわからなかったので調べました。

- この定義は`タグ`というようです。
- reflectパッケージを使って型情報からアクセスします。

先ほどの構造体定義であれば以下のようにアクセスできるようです。

```go
person := Person{"hoge"}

// リフレクションを使って型情報を取得する
t := reflect.TypeOf(person)

// フィールド情報を取得する
f, _ := t.FieldByName("Name")

// フィールドからタグ情報を取得する
tag := f.Tag

// タグ文字列全体
fmt.Println(tag) // json:"name"

// key:"value"形式で定義した場合、key名を指定して取得できる
fmt.Println(tag.Get("json")) // name
```

`key1:"value1" key2:"value2"`のように複数の情報を持たせることもできるのでうまく使えば面白いかもしれません。

ただし、定義自体は文字列であり、リフレクションを使ってアクセスされるため静的型チェックが効かず、実行時エラーになるため乱用は避けたほうがよさそうです。

<br />

## Go言語の日付フォーマットが独特

Go言語の日付や時刻のフォーマットに使う書式はよくある%Y%m%dとかではなく、2006とか01とかを使います。

```go
fmt.Println(time.Now().Format("2006/01/02 15:04:05 MST"))
```

最初は戸惑いますが、以下の様な順番になっているというのがわかれば、意外とわかりやすいかもしれません。

1. 月 01 or Jan
2. 日 02
3. 時 3 or 15
4. 分 04
5. 秒 05
6. 年 2006
7. 標準時 -0700 or MST

timeパッケージに定数も用意されているようです。

```go
const (
        ANSIC       = "Mon Jan _2 15:04:05 2006"
        UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
        RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
        RFC822      = "02 Jan 06 15:04 MST"
        RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
        RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
        RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
        RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
        RFC3339     = "2006-01-02T15:04:05Z07:00"
        RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
        Kitchen     = "3:04PM"
        // Handy time stamps.
        Stamp      = "Jan _2 15:04:05"
        StampMilli = "Jan _2 15:04:05.000"
        StampMicro = "Jan _2 15:04:05.000000"
        StampNano  = "Jan _2 15:04:05.000000000"
)
```

KitchenとかRubyDateとかあっておもしろいですね。

<br />

---

# 所感など

LTでGo言語の情報を教え合ったり、もくもく会で気軽に質問とかできたので、Go言語についてわいわいするという今回の目的は達成できたんじゃなかろうかと勝手に思っています。

わいわいする中でも、ちゃんと学びがあるのもよかったですね。

そんな、**楽しくも学びのある Fukuoka.go** は月一回を目安に定期的に開催していきます。  
Goに興味がある方も、Go談義がしたい方も、どなたでもお気軽にご参加ください

LTも絶賛募集しています！  
Go言語でこんなのやってみたというのがあればぜひぜひ教えてください。


