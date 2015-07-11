---
layout: post
title: "すべてが╭( ･ㅂ･)و ̑̑ ｸﾞｯ ! になる - Jenkinsビルドパイプライン結果をプルリクエストに表示する"
date: 2014-11-09
comments: true
categories: jenkins github
---

Jenkinsのジョブ結果をプルリクエストに表示するときは[GitHub pull request builder plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+pull+request+builder+plugin)を使ってますが、単体のジョブでしか利用できなかったので、複数ジョブ（ビルドパイプライン構成）のときに結果を表示する方法をまとめておきます。

<br />
---

# ビルドパイプラインの構成

今回のビルドパイプラインはこんな感じを想定しています。

![jenkins-build-pipeline](/images/2014/11/jenkins-build-pipeline.png)

プルリクエストへの更新やコメントをトリガーにジョブが起動し、単体テスト、回帰テストを実行します。
あわせて、実行中、失敗、成功の状態がプルリクエストに表示されます。

**成功したらプルリクエストに ╭( ･ㅂ･)و ̑̑ ｸﾞｯ ! って出ます。ｸﾞｯ ! って。**

これを満たす以下のジョブをつくっていきます。

1. ジョブの結果をプルリクエストに表示する
1. 複数のジョブをビルドパイプラインとして実行する
1. GitHub pull request builder plugin をトリガーとして利用する

<br />
---

# 1. ジョブの結果をプルリクエストに表示する

ジョブの結果をプルリクエストに表示するには[コミットのステータスを操作するAPI](https://developer.github.com/v3/repos/statuses/#create-a-status)を使います。

このAPIは、コミットの状態や表示するメッセージ、URLをパラメータとして渡すことができるので、これをラップするようなJenkinsのジョブをつくって他のジョブから呼べるようにしておきます。


## 設定

ここのポイントは`ビルドのパラメータ化`を使って、ジョブがパラメタを受け取れるようにしておくことです。
これによりジョブの汎用性が高まります。

### プロジェクト名

change-github-commit-status

### ビルドのパラメータ化

#### revision

`テキスト`として追加。対象となるコミット番号を指定します。

#### target_url

`テキスト`として追加。ジョブの結果を確認するためのJenkinsのジョブステータスURLを指定します。

#### state

`選択`として追加。ジョブの結果を指定します。

選択値は以下のとおり

- pending
- success
- failure

### ビルド

以下の内容を`シェルの実行`として追加

```sh
case "${state}" in
  "pending" ) description="|ω・)ﾐﾃﾏｽﾖ" ;;
  "failure" ) description="_(┐「ε:)_ｽﾞｺｰ" ;;
  "success" ) description="╭( ･ㅂ･)و ̑̑ ｸﾞｯ !" ;;
  *) description="";;
esac

curl -X POST -H "Authorization: token ACCESS_TOKEN" \
  https://api.github.com/repos/:owner/:repo/statuses/${revision} -d "{ \
  \"state\": \"${state}\", \
  \"target_url\": \"${target_url}\", \
  \"description\": \"${description}\" \
}"
```

---

このジョブは他のジョブから呼ばれることを想定していますが、`パラメータ付きビルド`でパラメタを指定することで
単体で実行可能です。

GitHubのプルリクエストに以下のように表示されれば実行成功です。

![github-commit-statuses](/images/2014/11/commit-statuses.png)


<br />
---

# 2. 複数のジョブをビルドパイプラインとして実行する

Jenkinsにはジョブを続けて実行するためのビルドパイプラインという機能があります。

ひとつの大きなジョブにするよりも小さなジョブを組み合わせるほうが柔軟にジョブ構成を組み立てることができます。

今回は以下の構成をつくります。

1. 単体テストを実行する`unit-test`
    1. 開始時に、コミットの状態を`実行中(Pending)`にする
    1. 単体テストが失敗したら、コミットの状態を`失敗(Failure)`にする
    1. 単体テストが成功したら、下流のジョブ(回帰テスト)を開始する
1. 回帰テストを実行する`regression-test`
    1. 回帰テストが失敗したら、コミットの状態を`失敗(Failure)`にする
    1. 回帰テストが成功したら、コミットの状態を`成功(Success)`にする

## パラメタを下流プロジェクトに渡す

ビルドパイプラインでは対象とするGitのコミット番号やジョブの状態をパラメタとして下流のジョブに渡すことでジョブを連動させることができます。

今回は、こういった用途に便利な[Parameterized Trigger Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Parameterized+Trigger+Plugin)をJenkinsのプラグインとして追加しておきます。

## 単体テストジョブの設定

### プロジェクト名

unit-test

### ソースコード管理

`Git`を選択

`Repository URL`は対象とするリポジトリのURLを指定、`Branches to build`は対象とするブランチ名（空白だと全てのブランチが対象）を指定。

### ビルド・トリガ

`SCMをポーリング`を選択

`スケジュール`には以下のようにポーリングしたい間隔をCRON形式で指定

```
H/15 * * * *
```

### ビルド

ここでは、単体テストを実行するためのジョブ（シェルの実行）などに加え、
開始時に、コミットの状態を`実行中(Pending)`にするためのジョブを追加します。

ビルド手順の追加から`Trigger/call builds on other projects`を選択します。

追加したビルドに以下の設定を行います。

#### Projects to builds

change-github-commit-status

#### Block until the triggered projects finish their builds

チェック不要。Pendingに更新するのを待たずに単体テストを開始します。

#### Add Parameters

`Predefined parameters`を選択します。

Parametersには以下を定義してください。

```sh
revision=${GIT_COMMIT}
target_url=${BUILD_URL}
state=pending
```

これで、現在のコミット番号とJenkinsのジョブURL、実行中ステータスを`github-change-commit-status`ジョブに渡して起動することができます。

### ビルド後の処理（失敗時のコミット状態の更新）

まず、失敗時にコミットの状態を`失敗(Failure)`にするビルドを追加します。

ビルド後の処理の追加から`Trigger/call builds on other projects`を選択します。

#### Projects to builds

change-github-commit-status

#### Trigger when build is

`Unstable or Failed but not stable`を選択します。

これでジョブ失敗時にのみこの処理が実行されるようになります。

#### Add Parameters

`Predefined parameters`を選択します。

Parametersには以下を定義してください。

```sh
revision=${GIT_COMMIT}
target_url=${BUILD_URL}
state=failure
```

### ビルド後の処理（成功時の下流プロジェクトの実行）

次に、成功時に下流プロジェクトを実行するビルドを追加します。

先ほど追加した`Trigger/call builds on other projects`の下部にある`Add Trigger...`でトリガーを追加します。

#### Projects to builds

regression-test

#### Trigger when build is

`Stable`を選択します。

これでジョブ成功時にのみこの処理が実行されるようになります。

#### Add Parameters

`Pass-through Git Commit that was built`を選択します。

これで、下流プロジェクトに対象のコミット番号が引き継がれます。

## 回帰テストジョブの設定

### プロジェクト名

regression-test

### ソースコード管理

`Git`を選択

`Repository URL`は対象とするリポジトリのURLを指定、`Branches to build`は対象とするブランチ名（空白だと全てのブランチが対象）を指定。

### ビルド・トリガ

上流から起動されるジョブなので特に指定は不要。

### ビルド

回帰テストを実行するためのジョブ（シェルの実行）を追加します。

### ビルド後の処理（実行結果に基づくコミット状態の更新）

実行結果に基づくコミット状態の更新を行うため、`Trigger/call builds on other projects`を選択します。

設定内容は、単体テストのビルド後の処理でコミット状態を更新するトリガーを追加したときと同じです。

`Stable`の場合に、stateを`success`で、
`Unstable or Failed but not stable`の場合に、stateを`failure`で指定すればOKです。

<br />
---

# 3. GitHub pull request builder plugin をトリガーとして利用する

上で述べた構成でビルドパイプラインの結果をプルリクエストに表示することができるようになりました。

しかし、現在のトリガーとしているSCMのポーリングでは、GitHub pull request builder pluginの
ようなプルリクエストへのコメント(test this please)によるジョブ実行やアクセス制御は行えません。

GitHub pull request builder plugin はビルドパイプラインには適用できませんが、
その機能をトリガーとしてだけ利用するジョブにすることで先程のビルドパイプラインに組み込めます。

## システムの設定

GitHub Pull Request Builder をプラグインとしてインストールします。

このプラグインは `Jenkinsの管理` > `システムの設定` > `GitHub Pull Request Builder`から
設定が必要です。

### GitHub server api URL

`https://api.github.com`

### Access Token

GitHubのアクセストークンを指定。


## トリガーとなるジョブの作成

### プロジェクト名

pull-request-trigger

### GitHub project

対象となるGitHubリポジトリのURL(`http://github.com/:owner/:repo`)

### ソースコード管理

`Git`を選択

`Repository URL`は対象とするリポジトリのURLを指定。

`Branches to build`は`${sha1}`を指定。

`高度な設定` > `Refspec` に以下を入力。

```
+refs/pull/*:refs/remotes/origin/pr/*
```

### ビルド・トリガ

`GitHub Pull Request Builder`にチェック

### ビルド後の処理

下流のビルドを追加します。

ビルド後の処理の追加から`Trigger/call builds on other projects`を選択します。

#### Projects to builds

unit-test

#### Trigger when build is

`Complete(always trigger)`を選択します。

これで常にこの処理が実行されるようになります。

#### Add Parameters

`Predefined parameters`を選択します。

Parametersには以下を定義してください。

```sh
revision=${ghprbActualCommit}
```

通常、下流プロジェクトにコミット番号を引き継ぐには、`Pass-through Git Commit that was built`を選択しますが、GitHub pull request builderではうまくいきません。

これはmergeのコンフリクトを検証するため、内部でmerge commitを生成するためです。
下流プロジェクトにはこのmerge commitされた番号が引き継がれ、そんなコミット番号知らないぞ、となるわけです。

そこで、元のコミット番号が保持されている`$ghprbActualCommit`をビルドパラメタとして渡すようにします。

## 下流プロジェクトの設定変更

下流プロジェクトとなる`unit-test`からトリガーを除外し、パラメタを受け付けるようにします。

### ビルドのパラメータ化

#### revision

`テキスト`として追加。対象となるコミット番号を指定します。

### ソースコード管理

`Branches to build`でパラメタで受け取った`${revision}`を使うようにします。

### ビルド・トリガ

上流から起動されるため、`SCMのポーリング`のチェックを外します。

* 実際はトリガとなるGitHub Pull Request Builder がコミット状態を一度成功にしますが、下流プロジェクトで状態を上書きするという動作をします。

<br />
---

設定は以上です。Jenkinsの設定を書き出していったら想定以上に長くなってしまいました...

Jenkinsは単体ジョブで使うことが多かったのですが、Parameterized Trigger Plugin があれば汎用的なジョブをつくってビルドパイプラインを構成することで可能性が広がりそうです。

すべてのプルリクエストを ╭( ･ㅂ･)و ̑̑ ｸﾞｯ ! にしよう。

