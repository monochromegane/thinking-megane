---
layout: post
title: "ComposerとPackagistでPHPライブラリを開発、テスト、公開する(2/2)"
date: 2013-06-02
comments: true
categories: php composer packagist
---

今回は、前回[ComposerとPackagistでPHPライブラリを開発、テスト、公開する(1/2)](http://blog.monochromegane.com/blog/2013/06/02/development-php-composer-library-at-packagist-1/)で作成したライブラリをPackagistに公開します。

# Packagistって

[Packagist](https://packagist.org/)はComposerのメインリポジトリとなるサイトです。

![Packagist](/images/2013/06/01_packagist_submit_package.png)

ここで公開しておくことでcompser.jsonにリポジトリの指定をしなくてもライブラリを取得できるようになります。

<br />
<hr />

# ライブラリのGit管理

Packagistでの公開する場合は、ライブラリをGit管理しましょう。

## .gitignoreの準備

以下の内容で`.gitignore`を作成しておきます。

`.gitignore`

```
vendor/
composer.lock
composer.phar
```

<br />

## 資産のコミットとタグ付け

ライブラリが公開できるようになったら、資産をコミットします。
このとき、タグも一緒につけてください。このタグがバージョン番号としてPackagistで使われます。

```console
$ git tag 1.0.0
```

タグ名(バージョン番号)はx.y.zの形式を使うようです。

<br />

## Githubへの公開

タグ付けまで終わったら、Githubにリポジトリを作成して、プッシュします。
このとき、通常のプッシュに加えてタグのプッシュも行います。

ちなみに後述のServiceHookの仕組みを使う場合、タグがプッシュされたタイミングでPackagist側が更新されます。

```console
$ git push origin master
$ git push origin タグ名
```

<br />
<hr />

# Packagistへの公開

いよいよ、Packagistに公開です。

## composer.jsonをチェック

公開情報に問題がないか以下のコマンドで検証します。

```console
$ php composer.phar validate
./composer.json is valid
```

上記のような結果が出ればOK。

<br />

## アカウント取得

まずは[Packagist](https://packagist.org/)にアクセスしてアカウント取得を行なってください。
特に難しいことはないはずです。

<br />

## ライブラリの登録

ログイン後、画面上部に`Submit Package`ボタンがあるので押下します。

![Submit Package](/images/2013/06/01_packagist_submit_package.png)


画面下部の`Repository URL`欄に先ほどGithubで公開したリポジトリのURLを入力します。
ライブラリ名が重複してなければSubmitボタンが出るので押下します。

![Check Package](/images/2013/06/02_packagist_submit_package.png)

パッケージの公開が完了です！

![Package Not Auto-update](/images/2013/06/03_packagist_not_autoupdate.png)

<br />
<hr />

# 自動更新を設定する

上記の手順までで公開はできていますが、Github側に下記手順を行なっておくことで、ライブラリの更新が自動的にPackagistに反映されるようになります。

## ServiceHookの設定

公開したリポジトリで`Settings`を押下します。

![Settings](/images/2013/06/04_github_repository_settings.png)

再度メニューから`ServiceHook`を選択、一覧から`Packagist`を選びます。

![ServiceHook](/images/2013/06/05_github_service_hook.png)

必要な情報を入力します

- User: Packagistのユーザ名
- Token: Packagistの[Your API Token](https://packagist.org/profile/)で取得
- Domain: `http://packagist.org`で問題無さそう
- Active: チェックをいれる

![Update Settings](/images/2013/06/06_github_service_hook.png)

<br />

## 接続テスト

設定が完了したら、もう一度PackagistのServiceHook設定ページを表示します。

`Test Hook`ボタンが表示されているので押下します。

![Test Hook](/images/2013/06/07_github_test_hook.png)

Packagist側で`Not Auto-update`表記が消えていれば接続テスト成功です。

<br />
<hr />

# 公開したライブラリを使う

公開したライブラリを使うには、通常、プロジェクトのcomposer.jsonに以下の記述後、`php compser.phar install`を行いますが、

```json
{
  "require": {
    "monochromegane/query-builder": "dev-master"
  }
}
```

`php composer.phar require monochromegane/query-builder:dev-master`コマンドを使うことで設定ファイルへの追記、インストールを一度にやってくれます。

<br />
<hr />
ちょっと長かったですが、慣れてしまえば気にならない手順だと思います。

ComposerとPackagistを使うことでPHPのライブラリ管理がとてもやりやすくなるので、どんどん使っていきましょうー。

PHP: The Right Way！


