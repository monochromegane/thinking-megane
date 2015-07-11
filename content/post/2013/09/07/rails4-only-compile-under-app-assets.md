---
title: "Rails4でGemの資産がAssets Precomplieに含まれないときは"
date: 2013-09-07
comments: true
tags: [ "ruby", "rails", "assets", "precompile", "pipeline", "flatui-rails" ]
---


Rails4プロジェクトでGem内の画像ファイルやフォントファイルといった資産がprecompile対象に含まれない原因と対策をまとめておきます。

原因は、assets precompileのデフォルトの対象変更です。対策としては以下の方法をとる必要があります。

- assets precompile に含まれるよう資産を移動する
- assets precompile に含めたい資産を明示する

<br />
<hr />

# 原因

## precompile対象ディレクトリの変更

Rails4では、パスにapp/assetsを含む資産のみをデフォルトのprecompile対象とするように変更されました。

[rails/rails - Only compile non-js/css under app/assets by default](https://github.com/rails/rails/pull/7968)

precompileのパスに置かれているREADME.mdのような"loose"なファイルを対象から外すこと、`app/`以下をオートロードするRailsの方針と合わせることなどが理由として挙げられています。(※1)


<br />

## 対象ディレクトリはどうなったか

このプルリクでは以下のような修正が取り込まれています。

`railties/lib/rails/application/configuration.rb`

```ruby
@assets.precompile = [ Proc.new { |path, fn| fn =~ /app\/assets/ && !%w(.js .css).include?(File.extname(path)) },
                     /(?:\/|\\|\A)application\.(css|js)$/ ]
```

これにより、`Rails.application.config.assets.paths`に定義されているディレクトリのうち、`app/assets`というパスを含む、js, css 以外の資産とapplication.(js|css)が対象となり、`lib/assets`や`vendor/assets`にある画像やフォントといった資産はprecompileの対象外となりました。

もし、プロジェクトで利用しているGemが上記のディレクトリに js, css 以外のファイル（imageなど）をおいていた場合、本番配布時に頭を抱えることになります(※2)。

<br />
<hr />

# 対策

## assets precompile に含まれるよう資産を移動する

これがベストな解決案だと思います。

`lib/assets`や`vendor/assets`を`app/assets`に移動することで、Railsは配下の資産をprecompile対象として認識してくれます。

手元で試したところ、Rails3でも問題なく動作しました。


** 使っているGemが対応していなかったので こんな感じでPullRequest を出しました **

[pkurek/flatui-rails - Move assets to app/assets for Rails 4](https://github.com/pkurek/flatui-rails/pull/27)


<br />

## assets precompile に含めたい資産を明示する

Gemを触りたくない場合の対策です。

`config/application.rb`に以下のような記述を追加します。

```ruby
config.assets.precompile += %w(*.png *.jpg *.jpeg *.gif)
```

読み込ませたい資産にあわせて配列内の拡張子を変更してください。


<br />
<hr />

`Rails.application.config.assets.paths`に含まれているのにPrecomplieされない点で大いにハマりましたが、GithubのPullRequestやIssueで経緯や理由を把握できました。

Githubほんと便利。

日本語の情報がまだ少ないのでまとめてみました。お役に立てばうれしいです。


<br />
<hr />

- `※1`. Issue内容を見るとこの変更に困惑している意見も一部ありますが、DHHは、`This makes sense to me :+1:`な感じ
- `※2`. 抱えました。



