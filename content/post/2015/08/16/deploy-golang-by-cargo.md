+++
date = "2015-08-16T15:05:04+09:00"
title = "Goのデプロイを「もっと」簡単にする。ビルドプロキシCargo。改めTorokko。"
tags = [ "golang", "torokko", "docker" ]
+++

Goアプリのデプロイはバイナリをひとつ配布して完了なのでとても楽なのですが、バイナリ自体をどこで管理するかについては意外と頭を悩ませることになります。
Goを使うにあたって、コードさえあればあとはバイナリも簡単に配布できる状態というのが望ましいと思い、仕組みを作ってみました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Ftorokko" title="monochromegane/torokko" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/torokko"&gt;monochromegane/torokko&lt;/a&gt;</iframe>

**2015/08/16追記: CargoはRustのパッケージマネージャーと名前がかぶってたのでTorokko(トロッコ)に改名しました！**

TorokkoはGoアプリのビルドと成果物ダウンロード用のAPIを提供するビルドプロキシです。

リクエストのURLで対象リポジトリ、GOOS、GOARCH、バージョンを指定します。
ビルドはDockerコンテナを用いて行い、バックエンドストレージに成果物が保存され、ビルドリクエストと同じURLでダウンロードを行うことができます。

Docker Hub の Automated Builds のGo版と考えてもらうとイメージしやすいかと思います。
Torokkoは、特にGoの開発環境をつくりたくないサーバー上へのデプロイを簡単にすることを目的としています。

## クイックスタート

1. Torokkoサーバーを起動します
2. ビルド用のエンドポイントにアクセスします

```sh
$ curl -X POST http://torokko-server/{remote}/{owner}/{repo}/{GOOS}/{GOARCH}/{version}
# e.g. http://torokko-server/github.com/monochromegane/torokko/linux/amd64/v0.0.1
```

しばらく経つとビルドが完了しているので、ダウンロード用のエンドポイントにアクセスします。

```sh
$ curl -OJL http://torokko-server/{remote}/{owner}/{repo}/{GOOS}/{GOARCH}/{version}
```

[こちら](http://torokko.monochromegane.com)にデモ用のAPIサーバーを用意しているので使ってみてください。

## Torokkoの仕組み

![torokko\_overview](https://cloud.githubusercontent.com/assets/1845486/9293092/eba8190c-4457-11e5-9176-19d9f7ac3363.jpg)

基本的な仕組みは前述のとおりです。
リクエストURLに応じたリポジトリをDockerコンテナでビルドし、ダウンロードできるようにバックエンドのストレージに保存しています。

ビルドとダウンロードを一元管理することで

- リポジトリはコードを管理するだけでよい
- デプロイ元のPCやデプロイ先のサーバーでビルドせずにダウンロードするだけでよい

といったメリットがあります。

## カスタムビルド

TorokkoはGoアプリのビルドに`make`を使います。
単純なGoアプリの場合、Torokkoがデフォルトで用意するビルド手順で問題ないですが、Godepsやgo-bindata、go generateなどを使う場合、リポジトリにMakefileを準備してください。

なお、デフォルトのMakefileは以下のとおりです。

```mak
build:
	go get -d ./...
	go build
```

## エンドポイント

紹介した、ビルド、ダウンロードの他にビルドログを確認するためのエンドポイントがあります。
詳しくは[README](https://github.com/monochromegane/torokko/blob/master/README.md)を参照ください。

# まとめ

とりあえずざっと作った感じで、まだプロトタイプに近いので環境つくる手順も整備できていない、かつスケールしやすい構成とはとても言えない状態ですが、

CIサービスの設定 + GitHub Releaseへの紐付け + デプロイレシピの修正をしながら、

「オレは！このバイナリを！ひとつ！サーバーに！置きたいだけなの！！」

そう思ったことがあるひとは使ってみてもよいかもしれません。
お盆休み明けから社内サーバーに入れてみて少しずつブラッシュアップしていきます。

