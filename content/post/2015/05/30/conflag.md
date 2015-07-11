---
layout: post
title: "Go言語でコマンドラインオプションをさくっと設定ファイル対応させるライブラリをつくった"
date: 2015-05-30 21:45
comments: true
categories: golang conflag
---

Goでコマンドラインオプションのあるツールを提供していると設定ファイルに定義できるようにしてほしいという要望が来ますが、意外とオプションと設定ファイルで定義が冗長になったりして煩わしいのでライブラリにしてみました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fconflag" title="monochromegane/conflag" class="embed-card embed-webcard" scrolling="no" frame    border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/conflag"&gt;monochromegane/conflag&lt;/a&gt;</iframe>

conf+flagで`conflag`です。

いつものコードに **3行** 追加するだけでコマンドラインオプションを設定ファイルに書いておけるようになります。

<br />

## 使い方

以下のような感じで、flag.Parse()を実行する前に、設定ファイルからflagに設定する値を呼び出すコードを追加します。

```go
// フラグを定義します
var procs int
flag.IntVar(&procs, "procs", runtime.NumCPU(), "GOMAXPROCS")

// コマンドラインオプションを解析する前に設定ファイルからフラグに値を設定します
if args, err := conflag.ArgsFrom("/path/to/config.toml"); err == nil {
	flag.CommandLine.Parse(args)
}

// コマンドラインオプションを解析します
flag.Parse()
```

設定ファイルをつくります。ここではTOML形式の設定ファイルを使いました（その他の対応フォーマットは後述）

```
procs = 2
```

この状態でコマンドラインオプションを渡さずに実行すると、設定ファイルに定義していた`2`がflagに設定されます。

便利っぽい。


### flagの優先順位

flagの優先順位は

`コマンドラインオプション` > `設定ファイルの値` > `flagのデフォルト値` となります。

上の例で言えば、こんな感じです。

```sh
$ myapp -procs 3 # 設定ファイルあり procs => 3
$ myapp          # 設定ファイルあり procs => 2
$ myapp          # 設定ファイルなし procs => runtime.NumCPU() (flagのデフォルト値)
```

### オプションの位置

そのツール用の設定ファイルがコマンドラインオプションだけを設定するものとは限らないので、適宜階層化されていると思います。
conflagでは`ArgsFrom`関数に渡す引数で設定ファイル内でコマンドラインオプションを定義している場所を指定できます。

このような設定ファイル構造の場合

```
[options]
flag = "value"

[other settings]
hoge = "fuga"
```

第二引数に`options`を指定します。

```go
// [options]の配下のみを解析します
conflag.ArgsFrom("/path/to/config.toml", "options")
```

階層が深い場合は、第三引数以降に続けて指定してください。

### ファイルフォーマット

現在のところファイルフォーマットは`TOML`と`JSON`に対応しています。
YAMLやHCLなども使いたいと思われた方、PullRequestチャンスです。

<br />

## インストール

```sh
$ go get github.com/monochromegane/conflag
```

です。

<br />

---

# 実装について

実装すごい簡単です。

flagパッケージはデフォルトのFlagSetをCommandLineという名前で持っているのでそれに対してコマンドラインオプションを渡すことでflag.Parse()相当の処理が行えます。

あとは設定ファイルからコマンドラインオプション一覧を生成して渡すだけ。
conflagは生成するところまでを担当します。

各種コマンドラインオプションの便利パッケージでも内部でflagパッケージ使っていれば実装によっては同じように利用できるかもしれません（できないかもしれません）

<br />

---

# まとめ

import文含めたら4行でした...

今度[プラチナサーチャー](https://github.com/monochromegane/the_platinum_searcher)でも導入してみようと思います。

よければご利用ください。

