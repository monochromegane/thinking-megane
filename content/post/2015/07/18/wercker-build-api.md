+++
date = "2015-07-18T11:07:01+09:00"
title = "werckerのtriggerBuild APIを使ってビルドパイプラインをつくる"
tags = [ "wercker", "hugo" ]
+++

先日、Hugoに移行した際、ブログ本体のリポジトリとテーマのリポジトリを分けるようにしました。ブログ本体のほうはweckerと連携しておりpushすることでGitHubPagesに公開されるようにしましたが、テーマ側を更新したときも同様のことができるようにAPIを使ってビルドパイプラインっぽいのをつくってみました。

# triggerBuild API

werckerが提供するビルドをキックするためのAPIです。

- [Triggering builds with the wercker API](http://blog.wercker.com/2015/06/23/Trigger-builds-with-the-api.html)
- [API Doc](http://devcenter.wercker.com/api/endpoints/builds.html#trigger-a-build)

[ここの手順](http://devcenter.wercker.com/api/getting-started/authentication.html)に従い、トークンを取得して利用します。

```sh
curl -H 'Content-Type: application/json' \
     -H "Authorization: Bearer <token>"  \
     -X POST -d '{"applicationId": "<applicationId>", "branch":"master", "message":"<your custom message>"}' \
     https://app.wercker.com/api/v3/builds
```

のようにして使います。

`applicationId`は、ビルドを実行したいアプリケーションのIDです。
自分の登録しているアプリケーションのURLにIDがついているのでそこで確認できます。

```
https://app.wercker.com/#applications/<applicationId>
```

# ビルドパイプライン

あとは特定のアプリケーションのビルドから、他のアプリケーションのビルドを呼ぶように設定するだけです。

## wercker.yml

自分の場合、テーマのリポジトリのほうに以下のようなwercker.ymlをおいています。

```sh
box: debian
build:
  steps:
    - script:
        name: echo
        code: |
          echo 'Update theme'
deploy:
  steps:
    - script:
      name: install curl
      code: |
        apt-get update
        apt-get -y install curl
    - script:
      name: trigger THINKING MEGANE build
      code: |
        curl -H 'Content-Type: application/json' -H "Authorization: Bearer $WERCKER_TOKEN" -X POST -d "{\"applicationId\": \"$WERCKER_APP_ID\", \"branch\":\"master\", \"message\":\"Update theme\"}" https://app.wercker.com/api/v3/builds
```

buildのstepはwerckerで必須のようなのでダミーを入れています。

（指定しないと `Failed step: setup environment - No 'build' pipeline definition in wercker.yml` エラーが出ます）

## wercker

wercker側では `Deploy targets` > `Custom Deploy` を選択し、`Auto deploy`にチェックを入れておきます。

wercker.ymlで指定した、WERCKER\_TOKENとWERCKER\_APP\_IDもここの `Deploy pipeline` > `Add new variable` で追加します。

少なくともWERCKER\_TOKENは`protected`にチェックを入れるのを忘れないようにしましょう。


## ビルド

あとは、テーマのリポジトリにpushするとブログのほうで設定したGitHub Pagesへの公開するビルドが実行されます。
ブログコンテンツ側のビルド設定は、[前回のエントリ](http://blog.monochromegane.com/blog/2015/07/12/octopress-to-hugo/)を参考にしてください。

---

ビルド起動がAPIでできるとビルドパイプラインだけでなく、いろいろ応用できそう。
werckerのAPIは最近充実してきているようなのでいいですね。

