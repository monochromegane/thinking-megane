---
layout: post
title: "CSVテキストをMarkdown形式のテーブルに変換するツールをつくった"
date: 2015-05-17
comments: true
categories: golang markdown table csv automator
---

Markdown形式のテーブル記法をいつまでたってもうろ覚えで、毎回検索したり、変換サイトにアクセスするのが面倒だったのでツールをつくりました。**嘘です。Go書きたかっただけです。**

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fmdt" title="monochromegane/mdt" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/mdt"&gt;monochromegane/mdt&lt;/a&gt;</iframe>

<br />

## 使い方

CSVテキストを標準入力から受け取って整形済みのMarkdown形式テーブルを出力します。

```sh
$ mdt < hoge.csv
| headerA | headerB                |
| ------- | ---------------------- |
| short   | very very long content |
```

<br />
## 連携

コマンドラインツールなので、`pbpaste`や`Automator`と組み合わせて好きなように使えます。

以下は、`Automator`のサービスとして登録した`mdt`をGitHubのIssue作成時に呼び出している様子です。

![mdt](https://cloud.githubusercontent.com/assets/1845486/7668803/cc0a9178-fc87-11e4-9d0e-9fd32ea3c1fc.gif)

便利っぽい。

<br />

## 機能

### CSV/TSVに対応

タブ区切りにも対応しているのでエクセル表をコピーして

```sh
$ pbpaste | mdt
```

のようにしてMarkdown形式テーブルに変換できます。

### 整形

カラムの幅を自動で整えます。日本語文字が入っても大丈夫です。

### ヘッダでのalign定義

```
headerA:, :headerB:
content, content
```

のようなCSVから

```
| headerA | headerB |
| -------:|:-------:|
| content | content |
```

のようなalign定義されたテーブルを出力します。

### 繰り返し

Markdown形式のテーブルを編集するときもCSV形式で編集できるように、テーブルとCSVの混在した状態でも繰り返しコマンドを適用できます。もちろん毎回整形しなおします。

```
| headerA | headerB |
| -------:|:-------:|
| content | content |
next content, next content
```

のように行を追加した状態から以下の出力を得ることができます。

```
| headerA      | headerB      |
| ------------:|:------------:|
| content      | content      |
| next content | next content |
```

[Examples](https://godoc.org/github.com/monochromegane/mdt#pkg-examples)もあわせてどうぞ

<br />

## インストール

```sh
$ go get github.com/monochromegane/mdt/...
```

です。

<br />

## Automator

冒頭のGitHubのPR/Issueや他のアプリケーションから使うために`mdt`をAutomatorに登録する手順です。

### 設定

Automatorから新規サービスとして登録します。

アクションは`ライブラリ` > `Run Shell Script`を選んでください。

![mdt\_with\_automator](https://cloud.githubusercontent.com/assets/1845486/7668851/5d833f84-fc8c-11e4-8787-aa39ce6ab300.png)

このようにサービスとして登録することで

- 選択したテキストを標準入力としてmdtに渡す
- 選択したテキストを結果で置き換え

する動作を全てのアプリケーションから呼び出すことができます。

### ショートカットキー

このままだと`メニュー`の`サービス`から呼び出す必要があるので、ショートカットキーを定義します。

`システム環境設定` > `キーボード` > `ショートカット`タブ > `サービス` を選択し、登録したサービスにショートカットキーを割り当てます。

<br />
---

# まとめ

Automatorほとんど使ったことなかったのですが、コマンドラインツールと組み合わせてアプリケーション横断で同じ機能を使うことができるようになるのはいいなあと思いました。
ブラウザ上でPR/Issue書いているときに整形するためエディタに切り替えなくてよいのは便利です。

それから、このツールを作ることでMarkdown形式のテーブル記法をやっと覚えることができました。Go言語最高ですね。

