---
layout: post
title: "Go言語でコマンドラインオプション使うなら。便利パッケージgo-flags。"
date: 2014-01-23 01:03
comments: true
categories: golang
---

Go言語でコマンドラインオプションを使うときは標準のflagsパッケージを使うことが多いと思いますが、[go-flags](https://github.com/jessevdk/go-flags)を使うとオプションの仕様を定義するだけでうまいことやってくれるので便利です。

<br />
<hr />

# go-flags

go-flagsはコマンドラインオプションのパーサーです。オプションの仕様を定義した構造体に、コマンドライン引数をバインドしたり、定義した仕様からヘルプを整形・表示してくれます。

標準のflagsパッケージに比べて、オプション指定方法が柔軟であること、指定されたオプションを構造体としてプログラム内で利用できること、値のバインドやヘルプの生成などの定型的な実装を行わないでよくなること、といいコトずくめなので使わない理由はないでしょう。

<br />

## インストール

```sh
$ go get github.com/jessevdk/go-flags
```

<br />

## 利用方法

オプションを定義した構造体を用意します。

```go
type Options struct {
    Verbose []bool `short:"v" long:"verbose" description:"Show verbose debug information"`
}
```

オプションを解析します。

```go
package main

import (
	flags "github.com/jessevdk/go-flags"
)

var opts Options

func main() {
	args, err := flags.Parse(&opts)
	if err != nil {
		os.Exit(1)
	}

	// go run main.go -vv arg1 のように起動された場合、以下のように値がバインドされる
	// opts.Verbose = [true, true]
	// args[0] = "arg1"
}
```

### ヘルプを表示する

`-h`もしくは`--help`で整形されたUsageが表示されます。引数の数が足りない場合などに明示的に表示したい、Usageの内容を変更したいという場合はパーサーを生成する必要があります。

```go
func main() {
	parser := flags.NewParser(&opts, flags.Default)
	parser.Name = "pt"
	parser.Usage = "[OPTIONS] PATTERN [PATH]"

	args, _ := parser.Parse()

	// 引数がひとつもなければヘルプを表示する
	if len(args) == 0 {
		parser.WriteHelp(os.Stdout)
		os.Exit(1)
	}
}
```

go-flagsを使っている[プラチナサーチャー](http://blog.monochromegane.com/blog/2014/01/16/the-platinum-searcher/)の場合、以下のようなヘルプが自動で表示されてます。便利！

```
Usage:
  pt [OPTIONS] PATTERN [PATH]

Application Options:
      --nocolor             Don't print color codes in results (Disabled by default)
      --nogroup             Don't print file name at header (Disabled by default)
  -l, --files-with-matches  Only print filenames that don't contain matches
      --vcs-ignore=         VCS ignore files (Default: .gitignore, .hgignore)
      --ignore=             Ignore files/directories matching pattern
      --depth=              Search up to NUM derectories deep (Default: 25)

Help Options:
  -h, --help                Show this help message
```

また、ヘルプはWindows向けにビルドした場合は、`-`ではなく、`/`になります。芸が細かくてよい感じ。

<br />

## コマンドラインオプションの定義

コマンドラインオプションの定義はフィールドタグとして用意されています。よく使うフィールドタグは以下のとおりです。

- `short`: 短いオプション名を定義。 short:"v"で、`-v`となる
- `long`: 長いオプション名を定義。 long:"verbose"で、`--verbose`となる
- `description`: 説明を定義。ヘルプに表示される文言となる
- `default`: 省略された場合のデフォルト値を定義
- `default-mask`: ヘルプに表示するデフォルト値をこの値でマスクする。ヘルプに表示したくない場合、"-"と指定。表示する文言を変えたい場合などにも使う。

その他、利用できるフィールドタグは以下を参考にしてください

- [GoDoc: go-flags](http://godoc.org/github.com/jessevdk/go-flags)


<br />
<hr />

go-flagsは他にもmapやfuncを格納できたりするのでドキュメントを眺めたり、ソース読んだりしても楽しいですね。

Go言語はコマンドラインツールつくることも多いと思うので、便利パッケージgo-flags、オススメです。


