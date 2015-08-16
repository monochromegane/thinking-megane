+++
date = "2015-08-16T15:05:04+09:00"
title = "Goのデプロイを「もっと」簡単にする。ビルドプロキシCargo。"
tags = [ "golang", "cargo", "docker" ]
+++


Goアプリのデプロイはバイナリをひとつ配布して完了なのでとても楽なのですが、バイナリ自体をどこで管理するかについては意外と頭を悩ませることになります。
Goを使うにあたって、コードさえあればあとはバイナリも簡単に配布できる状態というのが望ましいと思い、仕組みを作ってみました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fcargo" title="monochromegane/cargo" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/cargo"&gt;monochromegane/cargo&lt;/a&gt;</iframe>

CargoはGoアプリのビルドと成果物ダウンロード用のAPIを提供するビルドプロキシです。

リクエストのURLで対象リポジトリ、GOOS、GOARCH、バージョンを指定します。
ビルドはDockerコンテナを用いて行い、バックエンドストレージに成果物が保存され、ビルドリクエストと同じURLでダウンロードを行うことができます。

Docker Hub の Automated Builds のGo版と考えてもらうとイメージしやすいかと思います。
Cargoは、特にGoの開発環境をつくりたくないサーバー上へのデプロイを簡単にすることを目的としています。

## クイックスタート

1. Cargoサーバーを起動します
2. ビルド用のエンドポイントにアクセスします

```sh
$ curl -X POST http://cargo-server/{remote}/{owner}/{repo}/{GOOS}/{GOARCH}/{version}
# e.g. http://cargo-server/github.com/monochromegane/cargo/linux/amd64/v0.0.1
```

しばらく経つとビルドが完了しているので、ダウンロード用のエンドポイントにアクセスします。

```sh
$ curl -OJL http://cargo-server/{remote}/{owner}/{repo}/{GOOS}/{GOARCH}/{version}
```

[こちら](http://cargo.monochromegane.com)にデモ用のAPIサーバーを用意しているので使ってみてください。

## Cargoの仕組み

![cargo\_overview](https://cloud.githubusercontent.com/assets/1845486/9277791/5c7020a2-42e7-11e5-8dfc-d109f2f11161.jpg)

基本的な仕組みは前述のとおりです。
リクエストURLに応じたリポジトリをDockerコンテナでビルドし、ダウンロードできるようにバックエンドのストレージに保存しています。

ビルドとダウンロードを一元管理することで

- リポジトリはコードを管理するだけでよい
- デプロイ元のPCやデプロイ先のサーバーでビルドせずにダウンロードするだけでよい

といったメリットがあります。

## カスタムビルド

CargoはGoアプリのビルドに`make`を使います。
単純なGoアプリの場合、Cargoがデフォルトで用意するビルド手順で問題ないですが、Godepsやgo-bindata、go generateなどを使う場合、リポジトリにMakefileを準備してください。

なお、デフォルトのMakefileは以下のとおりです。

```mak
build:
	go get -d ./...
	go build
```

## エンドポイント

紹介した、ビルド、ダウンロードの他にビルドログを確認するためのエンドポイントがあります。
詳しくは[README](https://github.com/monochromegane/cargo/blob/master/README.md)を参照ください。

# まとめ

とりあえずざっと作った感じで、まだプロトタイプに近いので環境つくる手順も整備できていない、かつスケールしやすい構成とはとても言えない状態ですが、

CIサービスの設定 + GitHub Releaseへの紐付け + デプロイレシピの修正をしながら、

「オレは！このバイナリを！ひとつ！サーバーに！置きたいだけなの！！」

そう思ったことがあるひとは使ってみてもよいかもしれません。
お盆休み明けから社内サーバーに入れてみて少しずつブラッシュアップしていきます。

