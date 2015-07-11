---
title: "Go言語でActiveRecordライクなORMをつくった"
date: 2015-03-04
comments: true
tags: golang activerecord argen generate
---

Goで DataMapperじゃなく、ActiveRecordライクにDB操作したいと思ってつくってみました。

`go/parser`と`go/ast`でソースを解析、個々の構造体ごとにARなコードを生成します。

<br />
---

# argen

**A**ctive**R**ecord** Gen**eratorで`argen`です。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fargen" title="monochromegane/argen" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/argen"&gt;monochromegane/argen&lt;/a&gt;</iframe>

<br />

## クイックスタート

テーブルを表す構造体に`+AR`アノテーションをマークします。

```go
//+AR
type User struct {
	Name string
}
```

ファイルに対して`argen`コマンドを実行します。

```sh
$ argen xxx.go
```

ActiveRecordライクなメソッドが定義された`*_gen.go`が出力されます。あとはそれを使ってコード書いていくだけです。

こんな感じで使います。

```go
db, _ := sql.Open("sqlite3", "foo.db")
Use(db)

u := User{Name: "test", Age: 20}
u.Save()
//// INSERT INTO users (name, age) VALUES (?, ?); [test 20]

User{}.Where("name", "test").And("age", ">", 20).Query()
//// SELECT users.id, users.name, users.age FROM users WHERE name = ? AND age > ?; [test 20]
```

<br />

## アソシエーション

構造体に関連を表すメソッドを定義しておくことでアソシエーション系のメソッドを出力します。

```go
func (m User) hasManyPosts() *ar.Association {
	return nil
}
```

こんな感じで使います。

```go
user, _ := User{}.Create(UserParams{Name: "user1"})
user.BuildPost(PostParams{Name: "post1"}).Save()

// Get the related records
user.Posts()
//// SELECT posts.id, posts.user_id, posts.name FROM posts WHERE user_id = ?; [1]

// Join
user.JoinsPosts()
//// SELECT users.id, users.name, users.age FROM users INNER JOIN posts ON posts.user_id = users.id;

```

<br />

## バリデーション

構造体に検証ルールを表すメソッドを定義しておくことでバリデーション系のメソッドを出力します。

```go
func (u User) validatesName() ar.Rule {
	// Define rule for "name" column
	return ar.MakeRule().Format().With("[a-z]")
}
```

バリデーションは構造体のSave時に実行されるように組み込まれます。エラーは以下のように扱うことができます。

```go
u := User{Name: "invalid name !!1"}
_, errs := u.IsValid()
if errs != nil {
	fmt.Errorf("%v\n", errs.Messages)
	//// map[name:[is invalid]]
}
```

OnCreateなどの実行トリガー、独自バリデーションにも対応しています。

<br />

## go generate

Go1.4から導入された`go generate`を使うことで複数ファイルに対して一括でargenを適用させることができます。

ファイルの先頭に以下を定義。

```go
//go:generate argen
```

あとは定義を変更したときに`go generate`コマンドを実行すればOK。

<br />
---

#### ジェネレーターか〜

はい。

「設計書からアプリケーションを全て生成します」みたいなものではなくて、最小限の定義に対して定型的なコードを生成してくれるという範囲で扱う分には悪いものではないと思っています。

特にGo言語だと現時点でジェネリクスがサポートされていないため、今回のような任意の構造体に共通のメソッドを持たせつつ、扱える型は個々のものにするといった場合は、構造体の埋込やインターフェースだと呼び出し側の型アサーションが必要になるので、ジェネレーターを利用してしまうのもひとつのやり方かなと思います。

(任意の型に対応するためにリフレクションを使うこともできますが、残念ながら実行速度に影響が出ますし、実行時エラーの可能性がどうしても高くなってしまいます)

このあたり、よりよいやり方についてコメントもらえるとうれしいです。

<br />
---

`argen`、まだまだライブラリとしては不足がありますが、よければご利用ください。PRお待ちしております。

- [monochromegane/argen](https://github.com/monochromegane/argen)
