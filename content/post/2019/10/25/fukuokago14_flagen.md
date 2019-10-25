+++
date = "2019-10-25T17:24:43+09:00"
title = "コマンドラインオプションをパースするコードをコマンドラインオプションから生成するツールをつくった"
tags = [ "golang" ]
+++


コマンドラインオプションの形式は決まったけれども、パース処理を実装するために各言語やライブラリのドキュメントを読むことを繰り返していたので、この手間を省くためのツールをつくりました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fflagen" title="monochromegane/flagen" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/flagen"&gt;monochromegane/flagen&lt;/a&gt;</iframe> 

flagenは、先に決めたコマンドラインオプションから、これを解析するための各言語用のコードを出力するツールです。
オプション名から変数名や変数の型、デフォルト値が決定されるため、汎用的なボイラーテンプレートと比較して編集の手間が少なくなります。
また、テンプレートによって任意の出力を行えるため、エディタや自前のボイラーテンプレート出力ツールとの連携が容易です。
使いやすくするため、プリセットのテンプレートとしてGo、Ruby、Python、Shellのものを提供しています。

# 使い方

使い方は、テンプレートと実際に使う時のコマンドラインオプションを渡すだけです。

```sh
$ flagen YOUR_TEMPLATE YOUR_COMMAND_LINE_OPTIONS...
```

例えば、

```sh
$ flagen go --dist erlang -e k/l --lambda 1.5 -k 1 -v
```

と指定すると、Go用のコマンドラインオプションの解析処理を出力します。

```go
var (
	dist	string
	e	string
	lambda	float64
	k	int
	v	bool
)

func init() {
	flag.StringVar(&dist, "dist", "erlang", "usage of dist")
	flag.StringVar(&e, "e", "k/l", "usage of e")
	flag.Float64Var(&lambda, "lambda", 1.5, "usage of lambda")
	flag.IntVar(&k, "k", 1, "usage of k")
	flag.BoolVar(&v, "v", false, "usage of v")
}
```

指定されたコマンドラインオプションから変数名、オプション名、型、デフォルト値が設定されることで編集の手間が極力ない状態になっています。

Python、Ruby、Shellの例は[GodocのExamples](https://godoc.org/github.com/monochromegane/flagen#pkg-examples)を参考にしてください。

# テンプレート

テンプレートはGoの[text/template](https://golang.org/pkg/text/template/)を利用して解析されます。
テンプレート内では、`.Flags`（解析したオプションの情報）と`.Args`（残りのコマンドライン引数）が利用可能です。
`.Flags`は`Name`と`Value`をもち、`Value`は更に`Type`と`Get`をもちます。

以下は、解析したオプションの情報を列挙するシンプルなテンプレート(my.tmpl)です。

```
{{ range $flag := .Flags -}}
  {{ $flag.Name }}={{ $flag.Value.Get}}({{ $flag.Value.Type }})
{{ end }}
```

```sh
$ flagen my.tmpl --dist erlang -e k/l --lambda 1.5 -k 1 -v
```

このテンプレートから以下の出力を得ることができます

```
dist=erlang(string)
e=k/l(string)
lambda=1.5(float)
k=1(int)
v=false(bool)
```

また、テンプレート内では文字列のケース変換のための関数を利用することができます。
主に変数を言語の命名規約に合わせるのに使えます。
使える関数は、[こちら](https://godoc.org/github.com/monochromegane/flagen#pkg-variables) で確認ください。

# 連携

## Vim

flagenの結果は標準出力を使っているため、エディタとの連携も容易です。
例えば、Vimでは以下により、カーソル位置に結果を挿入することができます。

```
:r!flagen YOUR_TEMPLATE YOUR_COMMAND_LINE_OPTIONS...
```

## ボイラーテンプレート出力ツール

flagenはライブラリとして利用することができるため、自前のボイラーテンプレート出力ツールがGo製であれば以下のように呼び出すことができます。

```go
	tmpl, err := flagen.NewTemplate(args[0])
	if err != nil {
		return err
	}
	return tmpl.Execute(outStream, args[1:])
```

また、独自の関数が必要な場合は `flagen.TemplateFuncMap`に設定することでテンプレート内で利用することができます。

# ワークアラウンド

## 曖昧なフラグ

flagenはオプションに値が指定されていないときにboolだと見なすため、以下のようにboolフラグで終わって引数がある場合に判断がつきません。

```sh
$ flagen TEMPLATE --bool-flag arg1
```

想定どおりにするためには値としてtrue or falseを受け取ることを明示する必要があります。

```sh
$ flagen TEMPLATE --bool-flag=false arg1
```

# まとめ

様々な実装が提供されているコマンドラインオプションの解析処理を利用形式から動的に生成するジェネレーターとしてflagenをつくりました。実際にいくつかの言語のテンプレートを用意してエディタと連携させることでCLI開発の効率が改善しています。

今後はflagen自体のオプションとしてprefixなどを提供すれば構造体の変数に設定する用途などのテンプレートとの相性もよくなりそうだと考えています。
便利なテンプレート追加のプルリクエストやイシュー、ボイラーテンプレートのツールへ組み込んだ報告などお待ちしています。

# Fukuoka.go

このツールは[Fukuoka.go#14+Umeda.go](https://fukuokago.connpass.com/event/146447/)で発表しました。
発表資料はこちらです。

<script async class="speakerdeck-embed" data-id="ece61cadb91e46febd8b692e32a8679f" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

