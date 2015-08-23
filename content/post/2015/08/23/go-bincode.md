+++
date = "2015-08-23T01:24:07+09:00"
title = "野良バイナリになっても大丈夫。Goのバイナリにソースコードを添付するツールをつくった。"
tags = [ "golang", "go-bincode" ]
+++

サーバーにあるGoのバイナリのバージョン管理は気を使います。どのバージョンで動いているのかわからなくならないよう、バージョンやリビジョンを埋め込む方法がありますが、そもそもリポジトリもわからないバイナリがあったら...そんなときに備えて、ソースコード自体をバイナリに埋め込んでおくというツールをつくってみました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fgo-bincode" title="monochromegane/go-bincode" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/go-bincode"&gt;monochromegane/go-bincode&lt;/a&gt;</iframe>

`go-bincode`は内部で`go-bindata`を使い、goのソースコードをバイナリに埋め込みます。また、埋め込んだソースコードを参照/リストアするための各種オプションを提供します。

## 利用方法

以下、様子です。

![go-bincode](https://cloud.githubusercontent.com/assets/1845486/9424618/4ada7c6a-492d-11e5-9958-73bc329ac35c.gif)

使い方は、ソースコードを埋め込みたい対象のリポジトリで go-bincode コマンドを叩くだけです。必要に応じて`pkg`オプションでパッケージ名を指定してください。

`go-bincode.go`と`go-bincoder.go`が生成され、ソースコードの参照/リストアを行うためのオプションが使えるようになります。

あとはいつも通りビルドしたバイナリを配布すればOKです。

### オプション

- **list-code**: ソースコード一覧を表示します
- **show-code**: 指定したソースコードの内容を表示します
- **restore-code**: ソースコードをリストアします

## 生成したコードの依存関係

go-bincodeを使って生成されるコードはコードのアーカイブと参照/リストア用のオプションを提供するだけです。
生成後、あなたのコードを修正する必要はありません。

アーカイブされるコードには生成したコードは含まれませんが、上記の通り依存関係が存在しないため、リストア後のコードも問題なくビルドできます。

## インストール

go-bindataを内部で使っているのであわせてインストールしてください。

```sh
$ go get -u github.com/jteeuwen/go-bindata/... # Requirement
$ go get -u github.com/monochromegane/go-bincode/...
```

# まとめ

CI、バージョン管理、デプロイ自動化や環境のコード化が一般になってきた時代なので、バイナリとリポジトリとの関連も分からない野良バイナリになるというのは正直なかなか考えにくいとは思いますが、いざという時に判断材料が多いのに越したことはないと思います。

念には念を入れ、数年後の「Goあるある」でバイナリのコードがわからない〜という話が出ないよう、よければ使ってみてください。

