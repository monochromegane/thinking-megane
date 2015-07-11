---
layout: post
title: "Jenkinsのジョブ定義でGitHub AccessTokenを隠す"
date: 2014-11-11
comments: true
categories: jenkins github
---

Jenkinsのジョブ定義にGitHubのAccessTokenを直接書きたくないときに使えるプラグインがあったのでメモしておきます。

- [Mask Passwords Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Mask+Passwords+Plugin)

## 使い方

プラグインをインストールすると、各ジョブ設定の`ビルド環境`に`Mask passwords (and enable global passwords)`という項目が追加されます。

`Name`と`Password`の組み合わせを追加すると、ビルドスクリプトのなかで`${Name}`として利用することができます。

![jenkins-mask-access-token](/images/2014/11/jenkins-mask-access-token.png)

## AccessTokenを使う

[前回のエントリ](http://blog.monochromegane.com/blog/2014/11/09/jenkins-github-commit-status/)の例だと
`access_token`という名前でトークンを保存して、シェル内で以下のように使えます。

```sh
curl -X POST -H "Authorization: token ${access_token}" \
  https://api.github.com/repos/:owner/:repo/statuses/${revision} -d "{ \
  \"state\": \"${state}\", \
  \"target_url\": \"${target_url}\", \
  \"description\": \"${description}\" \
}"
```

実行結果もマスキングされるので安心です。

```
+ curl -X POST -H 'Authorization: token ********' https://api.github.com/repos/:owner/:repo/statuses/revision -d ...
```

<br />
---

その他、`Mask Passwords Plugin`は、ジョブごとのパスワード設定だけではなく、グローバルで使えるパスワードやマスキング対象を設定できるみたいです。
プラグインだと設定にパスワードフィールドが用意されていますが、自作のジョブでマスキングしたいときは使えるプラグインだと思います ╭( ･ㅂ･)و ̑̑ ｸﾞｯ !

