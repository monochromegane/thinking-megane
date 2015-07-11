---
title: "ComposerとPackagistでPHPライブラリを開発、テスト、公開する(1/2)"
date: 2013-06-02
comments: true
tags: [ "php", "composer", "packagist" ]
---

[PHP: The Right Way](http://ja.phptherightway.com/)を読んで、そのコンセプトに共感したので、今回、ComposerとPackagistでライブラリを開発、テスト、公開する手順をまとめておこうと思います。

# Composerって

[Composer](http://getcomposer.org/)は、PHPのライブラリ依存管理ツールです。RubyのBundlerのように依存するライブラリを設定ファイルに書いてコマンドたたけばライブラリのインストールと利用準備が整います。

![logo-composer-transparent](/images/2013/06/logo-composer-transparent.png)

また、Composer用のリポジトリだけでなく、PEARやGithubのリポジトリもComposer経由で取得できるので、とても便利です。


# 基本的な使い方

## インストール

プロジェクトのカレントディレクトリに`composer.phar`をダウンロードする。これだけ。

```sh
$ cd MyProject
$ curl -s http://getcomposer.org/installer | php
```


## 使いたいライブラリをcomposer.jsonに書く

カレントディレクトリにcomposer.jsonというファイルをつくって必要なライブラリを記述します。

`composer.json`
```json
{
    "require": {
        "monochromegane/query-builder": "dev-master"
    }
}
```


## ライブラリをダウンロードする

```sh
$ php composer.phar install
```

カレントディレクトリにcomposer.lockとvendorというディレクトリができていれば成功。
ライブラリを使うには、使う側でvendor/autoload.phpを読み込むだけでOKです。
かんたん、かんたん。

```php
require_once("vendor/autoload.php");
```

<hr />

# Composerに対応したライブラリをつくる

では、実際にComposerに対応したライブラリを開発していきましょう。
サンプルとして、今回公開した、[QueryBuilder](https://github.com/monochromegane/QueryBuilder)を使います。必要に応じてGithubの資産も参考にしてください。

## プロジェクトの準備

```sh
$ mkdir MyLibrary
$ cd MyLibrary
$ curl -s http://getcomposer.org/installer | php
```

## ライブラリの情報を定義する

プロジェクトのトップディレクトリにcomposer.jsonをつくります。

`composer.json`
```json
{
    "name": "monochromegane/query-builder",
    "description": "Simple query builder",
    "keywords": ["query", "builder", "pdo"],
    "type": "library",
    "license": "MIT",
    "homepage": "https://github.com/monochromegane/QueryBuilder",
    "authors": [
      {
        "name": "Your account",
        "email": "Your email"
      }
    ],
    "require-dev": {
        "phpunit/phpunit": "3.7.*"
    },
    "autoload": {
      "psr-0": {
        "Monochromegane\\QueryBuilder": "src/"
      }
    }
}
```

* name
  * 重複回避のため、必ずベンダー名を先頭につけてください。ベンダー名はGithubのアカウント名がよさそうです。

* keywords
  * 後述のライブラリ公開サイトでの検索キーワードとして利用されます。複数指定可。

* require-dev
  * 開発環境でのみ使われるライブラリを定義します。テストは必要なのでここは常に書いておきましょう。

* autoload
  * これからつくるライブラリがどのディレクトリにあって、名前空間がなんなのかを記載します。名前空間にもベンダー名をつけておきましょう。ちなみにここがないとうまくautoloadされないとおもいます。


## ライブラリをつくる

### ディレクトリ構成

`autoload`の項で`ベンダー名\\ライブラリ名: "ライブラリ格納先"`と定義している場合、以下のようなディレクトリ構成をとります。

`src/Monochromegane/QueryBuilder`(ライブラリ格納先/ベンダー名/ライブラリ名)


### ファイル配置と名前空間

上記で作成したディレクトリにライブラリ資産を作成します。

`src/Monochromegane/QueryBuilder/Query.php`(ライブラリ格納先/ベンダー名/ライブラリ名/ファイル名)

```
<?php
namespace Monochromegane\QueryBuilder;
class Query
{
}
```
ファイルの先頭に名前空間を定義します。名前空間はディレクトリ構成をバックスラッシュで区切った記述になります。


## テストする

テストないライブラリとか使う気がしないと思うのでテストを書きます。

### PHPUnitのインストール

`require-dev`項にphpunitを記載していれば以下のコマンドでテスト環境が作成されます。

```sh
$ php composer.phar install --dev
```

vendor/bin/phpunitが格納されていればOKです。

### phpunit.xmlを作成する

phpunit.xmlを作成しておけば、テストの実行が楽になります。

`tests/phpunit.xml`

```xml
<phpunit backupGlobals="false"
         backupStaticAttributes="false"
         bootstrap="../vendor/autoload.php"
         cacheTokens="true"
         colors="true"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         processIsolation="false"
         stopOnFailure="false"
         syntaxCheck="false"
         verbose="false">
    <filter>
        <whitelist addUncoveredFilesFromWhitelist="false">
            <directory>../src/Monochromegane/</directory>
        </whitelist>
    </filter>
    <testsuite name="Monochromegane\QueryBuilder Test Suite">
        <directory>.</directory>
    </testsuite>
</phpunit>
```

### テストを作成する

`tests/ベンダー名/ライブラリ名/テストファイル名`のような構成でテストを作成します。

```php
<?php
namespace Monochromegane\QueryBuilder\Tests;

class QueryTest extends \PHPUnit_Framework_TestCase
{
}
```
ここにも名前空間を適用しておきましょう。

### テストを実行する

```sh
$ cd tests
$ ../vendor/bin/phpunit
```

あとはライブラリとテストを整備していけばOKです。
長くなってきたので公開編は、別エントリにわけたいと思います。

[ComposerとPackagistでPHPライブラリを開発、テスト、公開する(2/2)](http://blog.monochromegane.com/blog/2013/06/02/development-php-composer-library-at-packagist-2/)

PHP: The Right Way !


