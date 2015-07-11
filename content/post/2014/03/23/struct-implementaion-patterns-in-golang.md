---
title: "Go言語での構造体実装パターン"
date: 2014-03-23
comments: true
tags: go golang
---

Go言語での構造体実装は、埋込や独自コンセプトのインターフェースといったGo言語独自の機能を理解して行う必要があります。
今年からGo言語を始めましたが理解が曖昧なままだと実装に迷うことが何度かありました。今回よい機会なので、Go言語での構造体実装パターンとしてまとめてみることにしました。

<br />
<hr />

# 構造体実装パターン

実装パターンの洗い出しとして、GoFデザインパターンをGo言語で実装する手法をとりました。
その中で繰り返し現れる実装をGo言語での構造体実装パターンとしてまとめてみました。

- コンストラクタ関数
- エクスポートによるアクセス許可
- インターフェースによるポリモフィズム
- 構造体によるポリモフィズム
- 構造体によるサブクラス・レスポンシビリティ
- 構造体による移譲
- 関数による移譲

以下、それぞれのパターンを解説していきます。

<br />

## コンストラクタ関数

Go言語には構造体のコンストラクタがないため、構造体の初期化を行うには構造体の属するパッケージにコンストラクタ関数を定義します。

慣例では関数名としてNew + 構造体名() が用いられています。

```go
type StructA struct {
	Name string
}

// 初期化時に行いたい処理
func (self *StructA) SomeInitialize() {
	// Some initialize
}

// コンストラクタ関数を定義
func NewStructA(name string) *StructA {
	structA := &StructA{Name: name}
	structA.SomeInitialize()
	return structA
}

func main() {
	_ = NewStructA("name")
}
```

コンストラクタ関数を用いることには以下の利点があります。

- 構造体の生成と同時に行いたい処理を利用側から隠す
- 埋込構造体を利用側から隠す（構造体の構造を利用側から隠す。埋込構造体の存在を隠すことにも使えます（後述））

<br />

## エクスポートによるアクセス許可

Go言語ではパッケージ外へのアクセス許可は、名前の先頭を大文字にすることで行います（エクスポート）。
これを利用することでシングルトンのようなインスタンス生成の制御を行うことが可能です。

```go
package singleton

// 構造体の名前を小文字にすることでパッケージ外へのエクスポートを行わない
type singleton struct {
}

// インスタンスを保持する変数も小文字にすることでエクスポートを行わない
var instance *singleton

// インスタンス取得用の関数のみエクスポートしておき、ここでインスタンスが
// 一意であることを保証する
func GetInstance() *singleton {
	if instance == nil {
	     instance = &singleton{}
	}
	return instance
}
```

省略書式`:=`を使うことで構造体自体をエクスポートしなくても利用できます。

```go
singleton := singleton.GetInstance()
```

<br />

## インターフェースによるポリモフィズム

Go言語には型の継承機能が用意されていませんが、インターフェースを用いることでポリモフィズムを実現できます。

```go
// インターフェースの定義
type SomeBehivor interface {
	DoSomething(arg string) string
}

// 構造体の定義
type StructA struct {
}

// インターフェースの実装
func (self *StructA) DoSomething(arg string) string {
	return arg
}

// ポリモフィズム
func main() {
	// インターフェース型の変数に格納できる
	var behivor SomeBehivor
	behivor = &StructA{}
	behivor.DoSomething("A")
}
```

構造体の埋込により擬似的な継承ができると説明しているものがありますが、これはあくまで透過的に構造体を利用できるだけで、"型"としてポリモフィズムを実現できるわけではないことに注意してください。


<br />

## 構造体によるポリモフィズム

ポリモフィズムを実現しつつ、構造体に共通の実装やメンバを定義したい場合があります。
この場合は、インターフェースを実装した構造体を用意して、それを埋め込む方法を取ります。

```go
// インターフェースの定義
type SomeBehivor interface {
	DoSomething() string
}

// 共通構造体の定義
type DefaultStruct struct {
	// 共通メンバ
	Name string
}

// 共通構造体にインターフェースを実装
func (self *DefaultStruct) DoSomething() string {
	return self.Name
}

// 共通構造体を埋め込む
type StructA struct {
	*DefaultStruct
}

func main() {
	// インターフェース型の変数に格納できる
	behivor := &StructA{&DefaultStruct{"A"}}

	// 共通実装とメンバを使うことができる
	behivor.DoSomething()
}
```

ただし、この場合、利用側が初期化時に埋込構造体を意識する必要があるため、前述のコンストラクタ関数を用意するとよいでしょう。

```go
// コンストラクタ関数
func NewStructA(name string) *StructA {
	return &StructA{&DefaultStruct{"A"}}
}
// 利用側は埋込構造体の存在を意識しない
behivor := NewStructA("A")
```

<br />

## 構造体によるサブクラス・レスポンシビリティ

Go言語では、埋め込まれた構造体がもとの構造体の実装を利用することができません（Dispatch backされないと言います）。

そのため、TemplateMethodパターンのように子クラスに処理の実装を任せる機能をつくる場合、埋め込まれた構造体のメソッドに対してレシーバとなる、もとの構造体の参照を渡す必要があります。

```go
type Behivor interface {
	DoSomethingByOther()
}

type Abstract struct {
}

// 内部で、もとの構造体の実装を利用するメソッド
func (self *Abstract) DoSomething(behivor Behivor) {
	behivor.DoSomethingByOther()

	// 仮にここで AbstractにBehivorを埋め込むなど、メソッドがある状態でコンパイルを通るようにして、
	// self.DoSomethingByOther()としてもpanic: runtime error: invalid memory address or nil pointer dereferenceが
	// 発生し、Dispatch backされないことがわかる
}

// 上で定義した構造体を埋め込む
type Concrete struct {
	*Abstract
}

// インターフェースを実装
func (self *Concrete) DoSomethingByOther() {
	// Do something
}

func main() {
	concrete := &Concrete{}
	// レシーバとなる自分自身を渡す
	concrete.DoSomething(concrete)
}
```

このレシーバを自ら指定する方法は、継承を利用できない言語で`Client-specified self`というパターンという名前で使われているようです。
ただし構造体間の依存関係が高くなるため、どうしても継承関係が必要な場合を除き、後述の構造体による移譲、または関数による移譲を検討するほうがよいでしょう。

<br />

## 構造体による移譲

Go言語では、処理の実装を適切な責務を持つ構造体に任せる移譲は簡単に実現できます。
それには構造体をメンバとして保持し、必要に応じて、その構造体のメソッドを呼ぶようにします。

また、Strategyパターンのようにインターフェースに対して移譲するような設計にしておくと依存性をより下げることができるでしょう。

```go
type Strategy interface {
	DoSomething()
}

type ConcreteStrategy struct {
}

func (self *ConcreteStrategy) DoSomething() {
	// Do something
}

type StructA struct {
	// 移譲先をメンバに保持
	strategy Strategy
}

func (self *StructA) Operation() {
	// 埋め込んだ構造体に処理を移譲
	self.strategy.DoSomething()
}
```

<br />

## 関数による移譲

Go言語では、関数をファーストクラスの型として扱えるため、移譲する処理を関数として渡す手法もとることができます。
Go言語の基本パッケージを見ていると、移譲処理を外部から注入する機能を提供する場合、関数型を渡すような設計になっているのが多いように思われます。
　
```go

// 関数型を定義
type DoSomething func()


type StructA struct {
}

// 関数型を引数で受け取るメソッドを定義
func (self *StructA) Operation(strategy DoSomething) {
	// 受け取った関数型に処理を移譲
	strategy()
}

func main() {
	structA := &StructA{}

	// 関数リテラルで引数として渡す
	structA.Operation(func(){
	     // Do something
	})
}
```

<br />
<hr />

# Go言語でのデザインパターン

今回、パターンの洗い出しに使ったGo言語でのデザインパターン実装はこちらのリポジトリに公開しています。

- [monochromegane/go_design_pattern](https://github.com/monochromegane/go_design_pattern)

パターン実装にあたっては、結城 浩さんの[増補改訂版Java言語で学ぶデザインパターン入門](http://www.amazon.co.jp/%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88Java%E8%A8%80%E8%AA%9E%E3%81%A7%E5%AD%A6%E3%81%B6%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3%E5%85%A5%E9%96%80-%E7%B5%90%E5%9F%8E-%E6%B5%A9/dp/4797327030/ref=sr_1_2?ie=UTF8&qid=1395468803&sr=8-2&keywords=%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)を参考にしました。
各デザインパターンの解説についてはそちらを参考にしてください。


