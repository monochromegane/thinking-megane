---
title: "[Rails] Unite.vimでSettingsLogicの長いキー名を入力する(unite-yaml)"
date: 2013-11-10
comments: true
categories: unite.vim yaml rails settingslogic
---

Railsで定数を管理する場合、[SettingsLogic](https://github.com/binarylogic/settingslogic)を使っていますが、プロジェクトが大きくなるとYAMLのキーが多くなったり、階層が深くなったりして入力が手間になってきます。
先週、Typoして時間をムダにしたのでUnite.vimでSettingsLogicのキー名を入力するUniteSource、[unite-yaml](https://github.com/monochromegane/unite-yaml)を作ってみました。

# unite-yaml

unite-yamlを使うと、SettingsLogicで `Settings.somekey.subkey.subsubkey.subsubsubkey` のような長いキーをUniteの候補として選択、入力を行えます。

<br />

## インストール

インストールはBundle、またはNeoBundleで`monochromegane/unite-yaml`を指定してください。

Bundleの場合は、`.vimrc`に`Bundle "monochromegane/unite-yaml”`を定義して、` :BundleInstall`です。

<br />

## 使い方

### 1. YAMLファイルを選択する

以下のコマンドでカレントディレクトリ以下の`*.yml`ファイルが検索され、Uniteのウィンドウに表示されます。


```console
:Unite yaml-list
```

![unite yaml-list](/images/2013/11/unite-yaml-list.png) 

対象のYAMLファイルを選択し、`Enter`を押してください。  
unite-yamlは`ERB enabled YAML`もサポートしています。

### 2. 入力したいYAMLのキーを選択する

選択したYAMLファイルに定義されているキーの一覧がSettingsLogicのフォーマット(アクセサのキーチェーン)で表示されます。
また、キー名のあとには該当する値も表示されます。

以下のYAMLファイルを選択した場合

```ruby
# config/application.yml
defaults: &defaults
  cool:
    saweet: nested settings
  neat_setting: 24
  awesome_setting: <%= "Did you know 5 + 5 = #{5 + 5}?" %>

development:
  <<: *defaults
  neat_setting: 800

test:
  <<: *defaults

production:
  <<: *defaults
```

このように展開されます。

![unite yaml](/images/2013/11/unite-yaml.png) 

入力したいキー名を選択して、`Enter`を押すと、カーソル位置にキー名が入力されます（値は含まれません）


<br />

## キーバインド

以下のようなキーバインドをvimrcに定義しておくといいでしょう。

```vim
" yaml
let g:unite_yaml_prefix = "Settings."
nnoremap <silent> ,y  :<C-u>Unite yaml-list<CR>
nnoremap <silent> ,Y  :<C-u>UniteResume yaml-buffer<CR>
```
 
<br />

- yaml-listから開いたYAMLのキー一覧は`yaml-buffer`という名前のbufferで開かれているので上記のように`UniteResume`を使ってYAMLの再パースなしに再度呼び出すことができます。
- `g:unite_yaml_prefix`を定義すると、入力時のキー名の前に指定した語が追加されます。`Settings.`を指定することで入力の手間を省くことができます。

<br />
<hr />

SettingsLogicは便利ですが、キーの打ち間違いや定義されているキーを見比べるためにファイル間を行ったり来たりして時間をムダにしがちです。
Unite.vimを使っている人は、unite-yamlを導入してみてはどうでしょうか。


