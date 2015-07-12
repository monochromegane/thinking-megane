+++
date = "2015-07-12T16:13:11+09:00"
title = "OctopressからHugoに移行した"
tags = [ "hugo", "golang", "octopress" ]
+++


前々からOctopressのpreview反映の遅さに厳しさを感じていたのでHugoに移行しました。手順などを簡単にまとめておきます。

# Hugo

Go製の静的サイトジェネレーターです。Octopressに比べて機能やテンプレートは少ないですが、起動やpreviewの反映の速さがそれを補って余りある感じです。

以下、移行の手順です。

## インストール

```sh
# Hugoをインストール
$ go get github.com/spf13/hugo
# テーマを全てダウンロード
$ git clone --recursive https://github.com/spf13/hugoThemes themes
```

## サイトの雛形を生成

```sh
$ hugo new site my_blog
$ cd my_blog
```

## 画像の移動

Octopressで管理していた`source/images`を`static/images`に移動します。

## コンテンツの移動/変換

Octopressで管理していた`source/_posts`を移動させます。URLを維持しつつ、エントリの日時を保持したかったのでディレクトリ構成を少し工夫しています。

`source/_posts/2015-07-05-dsn.markdown`を`content/post/2015/07/05/dsn.md`のようにします。（元の拡張子がmarkdownなのは気にしないでください）

とりあえず雑なスクリプト

```sh
#!/bin/sh

for f in $(ls $1/source/_posts/*.markdown)
do
  name=$(basename $f)
  y=$(echo $name | cut -d"-" -f 1)
  m=$(echo $name | cut -d"-" -f 2)
  d=$(echo $name | cut -d"-" -f 3)
  t=$(echo $name | cut -d"-" -f 4- | sed s/\.markdown$/\.md/)
  dest=content/post/$y/$m/$d
  echo copy $f to $dest/$t...
  mkdir -p $dest && cp $f $dest/$t
done
```

```sh
$ sh copy.sh YOUR_OCTOPRESS_ROOT
```

## ファイル内容の置換

### 記事ヘッダの日付フォーマット

Octopressの記事形式だと日付フォーマットが異なってうまく解釈されないので、置換します。

```sh
$ cd content/post
$ find . -type f -exec sed -i "" -e 's/date: \([0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}\).*$/date: \1/g' {} \;
```

その他、自分の環境固有のやつで何個か置換したけど割愛。

## Octopress形式のURLを引き継ぐ

config.tomlに以下を追記します。

```ini
[permalinks]
  post = "/blog/:year/:month/:day/:filename/"
```

以上で、移行は完了。簡単。


# 起動と動作確認

```sh
$ hugo server --theme=THEME_NAME --watch
```

テーマは`themes`ディレクトリ配下のテーマを指定します。テーマごとにconfig.tomlで個別の設定が必要でした。

# ブログを書く

`hugo new`でエントリの雛形を作ります。

```sh
$ hugo new post/`date '+%Y'`/`date '+%m'`/`date '+%d'`/NEW_POST_NAME.md
$ vi content/!$
```

# 記事の公開

GitHub Pagesで公開しています。Octopress時代は公開用のコマンドがありましたが、Hugoにはないので、Werckerを経由して記事を公開します。（public配下の資産を自分でpushしてもOKです）

基本は[本家のドキュメント](http://gohugo.io/tutorials/automated-deployments/)通りに進めてOKですが、wercker.yamlを少し追加修正しています。

## wercker.yml

本家のドキュメントだと`box`に`wercker/default`を指定するようになってますが、新しいDocker環境だと以下のようにエラーになるので `debian`を指定しています。

```
Failed step: setup environment - GET https://registry.hub.docker.com/v1/repositories/wercker/default/images returned 404
```

それにあわせて、gitのインストールステップを追加しました。

```sh
box: debian
build:
  steps:
    - script:
        name: install git
        code: |
          apt-get update
          apt-get -y install git
    - script:
        name: download theme
        code: |
          git clone YOUR_THEME_REPO themes/THEME_NAME
          rm -rf themes/THEME_NAME/.git
    - arjen/hugo-build:
        version: "0.14"
        theme: THEME_NAME
    - script:
        name: create old RSS file
        code: |
          cp public/index.xml public/atom.xml
deploy:
  steps:
    - script:
        name: install git
        code: |
          apt-get update
          apt-get -y install git
    - lukevivier/gh-pages@0.2.1:
        token: $GIT_TOKEN
        repo: YOUR_GITHUB_ACCOUNT/YOUR_GITHUB_PAGES_REPO
        domain: YOUR_BLOG_DOMAIN
        basedir: public
```

deployの`repo`オプションはブログの管理と、GitHub Pagesのリポジトリを分けるときに指定します。
同じリポジトリでやる場合は、gh-pagesブランチがGitHub Pagesとして使われるようです。

あとは、先ほどの本家マニュアルに沿ってやればOK。

記事を書き終わったらcommitしてpushすれば、werckerにより公開されます。

# デザイン

同僚の[@keita_kawamoto](https://twitter.com/keita_kawamoto) に頼んだところ、Hugoテーマの作り方をシュッとものにして、かっこいいデザインをあててくれました！！ありがとうございます！！

# その他、メモ

- テーマによっては古い`{{ .Site.BaseUrl }}`を使っているので警告が気になるなら`{{ .Site.BaseURL }}`に変更する
- Octopressだとシンタックスハイライトがついた状態で生成されるが、Hugoだと設定が必要だったので今回は highlight.js を使うようにした
 - go、vim script は追加でjsの読み込みが必要だった
 - `//cdnjs.cloudflare.com/ajax/libs/highlight.js/8.6/languages/go.min.js`
 - `//cdnjs.cloudflare.com/ajax/libs/highlight.js/8.6/languages/vim.min.js`
- 初回のOctopressからHugoで生成したコンテンツへの置き換えた時だけ、サイトで以下の500エラーが表示されるが30分ぐらいでなおる。
 - `GitHub Pages is temporarily down for maintenance.`
 - 上記表示されてももう一度アクセスすれば、そのページは表示されるようになるのでGitHub Pages側の反映待ちっぽい
- OctopressだとRSSはatom.xmlだったけど、index.xmlになるので、コピーして既存のパスを残してみた（RSS2.0とAtomなのでダメかも）
 - RSSを出力するにはlayoutのしたにrss.xmlを置く（テンプレートの内容は[本家ドキュメント](http://gohugo.io/templates/rss/)を参考に）
 - `{{ .RSSlink }}` は複数回呼び出すと空になる様子。（あまり追ってない）

---

# まとめ

即座にpreviewが反映されるようになってストレスがなくなりました。Hugo移行オススメです。

