---
title: "Go言語でつくったツールをHomebrewで配布する"
date: 2014-05-19
comments: true
tags: [ "homebrew", "golang" ]
---


先日、Go言語でつくった高速検索ツール([the_platinum_searcher](http://blog.monochromegane.com/blog/2014/01/16/the-platinum-searcher/))をHomebrewで配布できるようにしました。  
意外とGo言語製ツールをHomebrew対応させる情報がなかったので、配布までの手順をまとめておきます。


---

# Homebrewに対応させる

[Homebrew](http://brew.sh/)はMacで利用できるパッケージマネージャーです。

Homebrewでツールやパッケージを配布するにはそれらに関する情報やインストール方法を定義した`Formula`(製法)ファイルが必要です。  
また、Formulaを登録するリポジトリも必要です。Homebrewには公式リポジトリがありますが、`tap`コマンドを使うことで独自リポジトリをFormula取得先として追加することができます。

今回は、独自リポジトリでFormulaを公開します。


## Formulaファイルの作成

Formulaファイルを格納するディレクトリは独自リポジトリの命名規則に沿って`homebrew-`を先頭につけておきます。

```sh
$ mkdir homebrew-リポジトリ名
$ cd !$
$ vi パッケージ名.rb
```

以下、the_platinum_searcher(pt)のFormulaを参考に解説を行います。

`monochromegane/homebrew-pt/pt.rb`

```ruby
require 'formula'

HOMEBREW_PT_VERSION='1.6.2'
class Pt < Formula
  homepage 'https://github.com/monochromegane/the_platinum_searcher'
  url 'https://github.com/monochromegane/the_platinum_searcher.git', :tag => "v#{HOMEBREW_PT_VERSION}"
  version HOMEBREW_PT_VERSION
  head 'https://github.com/monochromegane/the_platinum_searcher.git', :branch => 'master'

  depends_on 'go' => :build
  depends_on 'hg' => :build

  def install
    ENV['GOPATH'] = buildpath
    system 'go', 'get', 'github.com/shiena/ansicolor'
    system 'go', 'get', 'github.com/monochromegane/terminal'
    system 'go', 'get', 'github.com/jessevdk/go-flags'
    system 'go', 'get', 'code.google.com/p/go.text/transform'
    mkdir_p buildpath/'src/github.com/monochromegane/the_platinum_searcher'
    ln_s buildpath/'search', buildpath/'src/github.com/monochromegane/the_platinum_searcher/.'
    system 'go', 'build', '-o', 'pt'
    bin.install 'pt'
  end
end
```

#### url

urlにはツールのリポジトリを指定します。`:tag`オプションをつけることで指定のリリースタグ時点のものを取得させることができます。
---

#### version

Formulaのバージョンを指定します。上記のようにツールのバージョンとあわせておくとよいです。
---

#### head

`brew install --HEAD`したときに取得するリポジトリとブランチ。この場合、最新のmasterを取得します。
---

#### depends_on

ビルドに必要なパッケージを指定します。  
`go`は必須。`hg`は自分のツールが利用するパッケージが`code.google.com`にあれば必要になります。
---

#### ENV['GOPATH']

`ENV['GOPATH']`を`buildpath`にするのがお作法のようです。
---

#### go get

`system`をつかってツールで必要なGoのパッケージを取得しておきます。  
ここで取得したパッケージは`$GOPATH`(=`buildpath`)に配置されます。
---

#### mkdir_p, ln_s

ツール内でGoのパッケージを分ける構成にしている場合、`GOPATH/src`を探してしまい、パッケージが見つからないエラーになってしまいます。  
分けたパッケージを`go get`で取得してしまうとmasterを取得してしまうので、チェックアウトしたリポジトリの資産を使ってもらうにはGOPATH/src配下にシンボリックリンクをはる必要がありました。

もっとよいやり方があれば、教えてください...
---

#### go build

ビルドします。ここでは`-o`でビルド成果物のファイル名を指定しています。
---

#### bin.install

ビルド成果物をパスの通ったところに配置します。  
具体的には`/usr/local/Cellar/パッケージ名/バージョン/bin`配下にビルド成果物を配置して`/usr/local/bin/ビルド成果物名`にシンボリックリンクをはってくれます。

## Formulaの公開

Formulaができたら、上記資産をGithubに同名のリポジトリをつくってpushしておきます。

## インストール

### 独自リポジトリをHomebrewに追加する

以下のコマンドでHomebrewに独自リポジトリを追加します。  

```sh
$ brew tap ユーザ/リポジトリ
```

`ユーザ`は自分のGithubのアカウント名、`リポジトリ`はGithubのリポジトリ名から`homebrew-`を外したものを指定します。

`/usr/local/Library/Taps/`配下に`ユーザ/リポジトリ`のディレクトリができていれば追加成功です。


### インストール

独自リポジトリ内のFormulaに定義された内容でインストールします。

```sh
$ brew install パッケージ名
```

---

つまり、the_platinum_searcherの場合、ユーザがインストールのために行う作業は以下だけです。

```sh
$ brew tap monochromegane/pt # 独自リポジトリを追加
$ brew install pt            # インストール
```


## アップデート

ツールのバージョンが上がったら、リリースタグをつけて、Formulaのバージョンも同じように上げておきます。

その後は

```sh
$ brew update               # 最新のFormulaを取得
$ brew upgrade パッケージ名 # アップグレード
```

で、ツールの更新が行えます。

---

# Homebrewで配布する利点など

the_platinum_searcherでは、`go get`や`go install`を使ったインストールだとGo開発者ではないひとに敷居が高いため、drone.ioのartifacts機能を使ってビルド成果物をダウンロードしてもらうようにしていました。

ただ、これでもダウンロードしてパス通したり、バージョンアップなどでいくつか不便もあったので、そのあたりをHomebrewに乗っかることで解決できました。

また、ローカルビルドによりCGOが有効にできるので（drone.ioだとCGOが使えない）、CGO有無による予想しない挙動に悩まされないのももうれしい点です。

Go言語でつくったツールを配布する手段としてHomebrew、検討してみてはどうでしょうか。

