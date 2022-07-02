+++
date = "2022-07-02T11:59:22+09:00"
title = "gonum/matパッケージを直感的に操作するMatrix Adapterをつくった"
tags = [ "golang" ]
+++

gonum/matによる行列計算を幾分か直感的に扱える薄いラッパーを作りました。
具体的には、計算結果用に空の行列を予め用意するのではなく、計算結果を戻り値で受け取れるように統一します。

- [Matrix Adapter](https://github.com/monochromegane/mat): Small adapter which provides method signatures that allow intuitive operation with fewer lines of code for gonum/mat.

# 背景と課題感

[Gonumプロジェクトのmatパッケージ](https://github.com/gonum/gonum/tree/master/mat)はGo言語での行列計算ライブラリを提供してくれています。
こういった正確さと速度の求められる数値計算の処理には、（趣味は別として）自作ではなく、多くの人に使われ実績のあるライブラリを採用したいことから、このパッケージを利用しています。

非常に便利に使わせていただいている一方で、自分の場合、スムーズに書けないことがありました。
理由としては、計算結果の受け取り方法に対する一貫性の崩れがあるだろうと考えています。

matパッケージでは、予め空の行列を用意し、これをレシーバーとして計算対象となる行列を引数で渡します。

```go
var c mat.Dense
c.Product(a, b)
```

元の行列が維持されるこの設計は非常にありがたいです。
しかしながら、一部の関数（例えば `Dense.T`）は結果が戻り値として得られます。

```go
ct := c.T()
```

この一貫性の崩れは不変と可変の混在にあるのではないかと考えています。
gonum/matパッケージでは、[引数で渡す行列は計算操作に対して不変](https://pkg.go.dev/gonum.org/v1/gonum/mat#hdr-Invariants)とされています。
これは、関数のパラメータの方が、`mat.Matrix`インターフェースであることからも読み取れます。
実際、このインターフェースには、`Dims()`, `At()`といった参照系かレシーバーを壊さない`T()`のみが用意されています。

一方で、このインターフェースを実装した`mat.Dense`などは自身をレシーバーとして内容を変更する操作を認めています。
この差異により、mat.Matrixインターフェースのシグネチャの一つである`T()`は戻り値を返すが、Denseの関数ではレシーバーを変更するという振る舞いの違いがあるのではないかと思われました。

gonum/matパッケージではほとんどの関数が空の行列を予め用意する方式に従っているため、慣れれば問題ない程度ではありますが、久しぶりに使う場合など、多少まごつくことが何度かありました。
個人的に、Go言語で何か書くときは迷わず書きたいという欲求があり、迷わないため一貫性のあるラッパーを作ることとしました。

# Matrix Adapter

今回は、計算結果を戻り値で受け取れるように統一します。
これにより、利用側からはDenseらも不変であるかのように扱え、一貫性という側面から認知負荷が減ると考えるためです。

Matrix Adapterを適用した場合、先ほどの例は以下のようになります。
`c`がレシーバーではなく、先ほどは引数となっていた`a`がレシーバーとなっていることに注意してください。
`c`は戻り値として受け取ることになりました。

```go
c := a.Product(b)
```

## 実装と使い方

Adapterの実装は単純明快で、元の構造体を埋め込み（embedding）し、結果を戻り値で返すように関数のシグネチャを変更しました。
変更が不要なものは移譲されます。
いわゆるGoFのデザインパターンにおけるAdapterパターンというやつです。

```go
type Dense struct {
	*mat.Dense
}

func (m *Dense) Add(b mat.Matrix) *Dense {
	var dense mat.Dense
	dense.Add(m.Dense, b)
	return &Dense{&dense}
}
```

現状、DenseとVecDenseに対するAdapterを提供しています。
既存と同名の`NewDense()`と`NewVecDense()`を使って、ラップされたDenseやVecDenseを作成できます。

```go
package main

import (
	"fmt"

	"github.com/monochromegane/mat/adapter"
)

func main() {
	m := adapter.NewDense(2, 2, []float64{1.0, 2.0, 3.0, 4.0})
	fmt.Printf("%v\n", m)
	// ⎡1  2⎤
	// ⎣3  4⎦
}
```

ちなみに、提供されるDenseとVecDenseは`fmt.Stringer`インターフェースを実装し、行列の内容を整形して出力するようにしています。

ここで、少し実践的な例として、リッジ回帰によるパラメータ推定（$\hat{\theta} = (X^{\top}X + \lambda I)^{-1} X^{\top}Y$）を実装し、比較してみます。

### Matrix Adapterでの実装

```go
	X, Y := syntheticData(N, theta) // Return adapter.Dense

	I := mat.NewDiagDense(3, []float64{1.0, 1.0, 1.0})
	reg := DenseCopyOf(I).Scale(lambda)

	XTXinv, _ := X.Transpose().Product(X).Add(reg).Inverse()
	XTY := X.Transpose().Product(Y)

	estimated := XTXinv.Product(XTY)
```

### gonum/matでの実装

```go
	X, Y := syntheticData(N, theta) // Return adapter.Dense

	XTXinv := mat.NewDense(3, 3, nil)
	XTXinv.Product(X.T(), X)

	I := mat.NewDiagDense(3, []float64{1.0, 1.0, 1.0})
	reg := mat.DenseCopyOf(I)
	reg.Scale(lambda, reg)

	XTXinv.Add(XTXinv, reg)
	XTXinv.Inverse(XTXinv)

	XTY := mat.NewDense(3, 1, nil)
	XTY.Product(X.T(), Y)

	estimated := mat.NewDense(3, 1, nil)
	estimated.Product(XTXinv, XTY)
```

この例では、gonum/matでの実装に比べてMatrix Adapterでの実装が、結果用の`Dense`の準備が省略できたこと、各関数で受け取る引数が減ったことなどから、幾分か簡潔に記述できているかと思います。

なお、Adapterでの実装では、Method Chainによる行数の短縮も見てとれます。
これについては、戻り値を返す実装としたことで副次的に得られた、本Adapterの特徴です。
Go言語におけるMethod Chainは、errorを含む多値の戻り値との相性から、積極的に採用されていないと認識しています。
ただ、今回は元のパッケージが各計算においてerrorを返さないものが多く、多値にならない関数が多くできたため、結果としてMethod Chainがつながる場合ができています（`Inverse()`などerrorを返すものもあるため全部は繋げません）。
この辺りのエラー処理については、一考の価値があると思いますが、現時点で本Adapterの対象外（従来パッケージを踏襲）としています。

## 相互運用性

Adapterの提供する関数でも、引数は`mat.Matrix`（や`mat.Vector`）を使うため、既存のラップしていないものを入力として受け取れます。
また、これらのAdapterは、`mat.Dense`（や`mat.VecDense`）構造体を埋め込んでいるため、`mat.Matrix`（や`mat.Vector`）の実装を満たします。
よって、ラップされた`adapter.Dense`（や`adapter.VecDense`）を既存の関数の入力として渡すことができます。

また、`T()`は`mat.Matrix`として維持しなければならない関数であることから、同等の`Transpose()`を提供しています。
これにより、転置行列がレシーバーとなるような呼び出しであっても、`adapter.Dense`として振る舞うことができるようになり、Matrix Adapterとの親和性が向上します。

```go
// X.T().Product(Y) is invalid due to mat.Matrix has no method Product.
XTY := X.Transpose().Product(Y) // Valid
```

# まとめ

本エントリでは、gonum/matパッケージを直感的に操作するための[Matrix Adapter](https://github.com/monochromegane/mat)を紹介しました。
本Adapterの作成とエントリ執筆において、「直感的でない」と主観的に感じる理由について、不変と可変の混在に起因するものではないかなと言語化できたのがよかったかなあと思います。

自分の利用範囲だと、DenseとVecDenseで事足りる場合が多いのですが、必要に応じて対象を増やしていこうかと思います。

# 参考

- [Go + Gonum を使った行列計算まとめ](https://po3rin.com/blog/gonum)
    - 関数の直感性を上げるための独自関数や、行列のフォーマットなどを参考にさせていただきました。
- [gorgonia/tensor](https://github.com/gorgonia/tensor)
    - gonum/matではないGo言語の行列計算パッケージ。戻り値で結果を受け取れたりerrorも返すので本Adapterの目指すところに近いとは思います。速度含めて検証が必要なのもあり、今回は慣れて使っている方が多いであろうgonum/matをラップする方式を採用しました。
