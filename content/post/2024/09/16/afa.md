+++
date = "2024-09-16T10:48:05+09:00"
title = "ターミナルフレンドリーなAIコマンド、afaを作った"
tags = [ "" ]
+++

Go言語で、AIモデルに対する推論をコマンドラインで実行する[afa](https://github.com/monochromegane/afa)というツールを作りました。入出力としてテキストストリームを前提としており、パイプやリダイレクトを用いて他のコマンドと連携しやすいのが特徴です。

```sh
$ echo $ERROR_MESSAGE | afa new -p "What is happening?" /path/to/file1 /path/to/file2
```

与えるプロンプトやコンテキスト情報によってAIモデルが柔軟に振る舞いを変えてくれるので、これまでの個別ツールでは対応が難しかった用途における自動化にも有用でしょう。
また、テキストストリームを扱えるリッチなTUIのコマンド[afa-tui](https://github.com/monochromegane/afa-tui)も別途提供しているので、ターミナル上でのチャットも快適です。

## デモ

### リッチTUIによるチャット

快適なインタラクティブチャットのため、Markdownを装飾して描画しつつ、`less`コマンド相当の操作感のページャーと、プロンプトの入力欄を持つViewerと連携できます。

![Chat](/images/2024/09/chat.gif)

### Zsh Line Editor(ZLE)を用いたターミナル上でのコマンド提案

ZLEと連携すれば、コマンドライン上でのプロンプトをそのまま提案されたコマンドに置き換えて実行できます。

![Command Suggestions](/images/2024/09/command_suggestion.gif)

### Vim上でのコード提案

Vimと連携すれば、選択範囲のコードを提案されたものに置換できます。使い慣れたエディタであればdiff表示や差分の部分反映もお手のものですね。

![Code Suggestions](/images/2024/09/code_suggestion.gif)


## 機能一覧

- ターミナルフレンドリーなAIコマンドとして機能します。
- リッチなターミナルユーザーインターフェース（TUI）を持つチャットクライアントとして機能します。
- システムとユーザーのための文脈に応じたプロンプトをテンプレートを使用してサポートします。
- プロンプト、標準入力、およびファイルパスを文脈として受け付けます。
- セッションを管理し、`resume` サブコマンドを介して迅速に再開できるようにします。
- 安全にエスケープされたJSONオプションで構造化された出力をサポートし、他のコマンドとの統合を容易にします。
- コアアプリケーションはサードパーティライブラリに依存せずに独立して動作します。
- AIモデルとして`OpenAI`をサポートします（他のAIモデルのサポートは将来計画されています）。

なお、この文章は、英語で書かれたafaのREADMEのFeatures項目をコピーして、`pbpaste | afa new -script -p "翻訳して" | pbcopy`で貼り付けました。便利ですね。

## 基本的な使い方

Viewerを使用しないシンプルなチャットを起動する。

```sh
$ afa new
```

Viewerを使用してチャットを起動する。

```sh
$ afa new -V
```

コンテキスト情報を加えてチャットを起動する。コンテキスト情報には「標準入力」「プロンプトメッセージ(-pオプションによるもの)」、そして「ファイルパス」を与えることができる。
ただし、仕様上、標準入力を与えるとキーボードからの入力を継続できないため、インタラクティブなチャットはできない。この場合は、プロセス置換（`<()`）をファイルパスとして与えることを検討されたい。

```sh
$ echo $ERROR_MESSAGE | afa new -p "What is happening?" /path/to/file1 /path/to/file2
# Please be cautious; when standard input is provided, interactive mode is disabled.
# Consider using process substitution.
#=> afa new -p "What is happening?" /path/to/file1 /path/to/file2 <( echo $ERROR_MESSAGE )
```

直近のセッションを再開する。

```sh
$ afa resume
```

指定したセッションを再開する。なお、`afa list`コマンドでセッション一覧を表示できる。

```sh
# The command `afa list` displays past sessions.
$ afa source -l SESSION_NAME
```

ユーザープロンプトを指定する。ユーザープロンプトは、Go言語のテンプレート形式であり、テンプレート内ではプリセットの値としてコンテキスト情報に対応する、`Message`、`MessageStdin`、そして `Name`と`Content`をメンバとする`File`構造体のリストである`Files`を以下のように呼べる。これにより動的なプロンプトの生成が可能となる。

```sh
# `Message`, `MessageStdin`, and `Files` that include `File` with `Name` and `Content` members can be used in the template file.
$ echo "Please explain the following.\n{{ (index .Files 0).Content }}" > CONFIG_PATH/templates/user/explain.tmpl
afa -u explain /path/to/file
```

スクリプトモードを指定して起動する。コマンドライン実行に適したモードのオプション設定を一括で行う`-script`オプションを指定することで、他のコマンドとの連携が容易になる。具体的には`-I=false -H=false -S=false -V=false -L=false`が設定される。

```sh
pbpaste | afa new -script -p "Transrate this" | pbcopy
```

`OpenAI`の独自機能として提供される`Structured Output`用のスキーマを指定できる。これは指定したJSON構造体に沿うような出力を強制する機能である。例えば、余分な説明が不要なコマンド提案などに適している。
クオート文字のエスケープを担う`-Q`オプションと合わせて使うことで`jq`による整形が容易になるので活用されたい。

```sh
$ cat <<< EOS > CONFIG_PATH/schemas/command_suggestion.json
{
  "type": "object",
  "properties": {
    "suggested_command": {
      "type": "string"
    }
  },
  "additionalProperties": false,
  "required": [
    "suggested_command"
  ]
}
EOS

$ P="List Go files from the directory named 'internal' and print the first line of each file."
$ afa new -script -Q -j command_suggestion -p $P | jq '. | fromjson' | jq -r '.suggested_command'
#=> find internal -name '*.go' -exec head -n 1 {} \;
```

## 実践例

実践例を列挙します。
もし使いたい例があれば、GitHubのリポジトリの[README](https://github.com/monochromegane/afa?tab=readme-ov-file#practical-examples)にプロンプトや設定例と共に載せているので参考にしてみてください。

- Zsh Line Editor(ZLE)を用いたターミナル上でのコマンド提案
    - デモで紹介したものです
- Vim上でのコード提案
    - これもデモで紹介したものです
- ターミナルに表示されたエラーメッセージの分析
    - `tmux`の`capture-pane`を使って直近の実行結果を分析できます。複数行にわたるエラーメッセージのコピペの手間が省けて便利ですね
- GitHubのプルリクエストの自動生成
    - `git diff`と`git log`の情報を元にタイトルと要約を生成してプルリクエストを開きます。ZLEを想定しているので背景情報も踏まえて考えてもらえるようになっています。また、既存のテンプレートの上に要約を差し込む形にしているので、どのリポジトリでも使いやすいかなと思います。
- pecoを用いたセッションの絞り込みと読み込み
   - インタラクティブな絞り込みで素早く過去のセッションにアクセスできます。

## AFA

AFAは`AI for All`の略で、あらゆるコマンドラインツールに対して連携可能（$\forall x \in X \, \exists \mathrm{AI}(x)$、つまりコマンドラインツールの集合`X`の全ての要素`x`に対し、その結果を入力とする`AI(x)`な組み合わせが存在する）なAIコマンドになるといいなと思って付けた名前です。

現時点でもまだViewerの入力欄が不調になることがあったりWindowsでの動作確認ができていないなど課題はありますが、通常使う分には十分な品質になったと思うので公開しました。よく使うオプションセットをconfigに設定し、いくつかのユーザープロンプトを登録しておいて`alias af='afa new'`とするだけで、個人的には、調べ物やチャットを含め、ブラウザの利用頻度がかなり減ったことを実感しています。

Homebrewによるインストール`brew install monochromegane/tap/afa monochromegane/tap/afa-tui`も提供していますので、よければ使ってみてください。

将来的には、サーバー上での異常検知のような定式化の難易度が高いタスクでの適用も検討しています（サードパーティー製ライブラリへの非依存とViewerツールとの分離は、そのためでもあります）。AFAであれば、他のコマンドラインツールとの親和性を活かして、通知やロギングは従来の仕組みも活用できると考えています。適用例など共有してもらえたら嬉しいです。

