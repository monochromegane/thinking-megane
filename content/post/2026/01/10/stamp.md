+++
date = "2026-01-10T14:17:46+09:00"
title = "プロジェクトのスケルトンをテンプレートから展開するstampというCLIツールを作った"
tags = [ "golang" ]
+++

CLIツールを多く作っていると、最初のコミットまでに機械的に準備する自分なりの一式（利用するコマンドラインパーサーによる初期実装、GitHub Actionの定義、Makefile、ライセンスファイル等々）ができてくるかと思います。
このような一式を予め用意しておくことが、普段通りの開発作法に従ってスムーズに開発を始めるための土台となります。
また、最近では、コーディングエージェントへ依頼する際に、このような一式を揃えて実装時の補助線としてもらう方が、認識が揃うことも多いように感じています。

自分の場合、この一式の準備には直近のリポジトリからコピーしてくることが多いですが、細々とした置換などが面倒でした。
秘蔵スクリプトを持っている方も多いのではないでしょうか。

このようなニーズのため、プロジェクトのスケルトンを管理するツールが開発されています（[copier](https://github.com/copier-org/copier)などがあるようです）。
一方で、これらのツールはテンプレートの適用後の継続的な同期まで面倒を見るために、プロジェクト開始時に一度展開できれば良いという自分のユースケースに対しては機能的に持て余してしまう印象がありました。

そこで、自分用に[stamp](https://github.com/monochromegane/stamp)と言うCLIツールを作りました。

機能は単純で、予め用意しておいた、テンプレート一式（ツールがスタンプなのでシートと呼んでいます）をディレクトリ構造のままコピーしてくるだけです。
各ファイルはGo言語のテンプレートとして処理されるため、`{{.name}}`のような記述に対して、動的な値を設定できます。

~~~sh
# my-appシートにhello.txtを登録（拡張子.stampでテンプレートとして解釈）
$ echo "Hello {{.name}}!" > "$(stamp config-dir)/sheets/my-app/hello.txt.stamp"
~~~

~~~sh
# my-appシートを適用
$ cd /path/to/workspace
$ stamp -s my-app name=alice
~~~

~~~sh
# 結果を確認
$ ls
hello.txt
$ cat hello.txt
Hello alice!
~~~

[chezmoi](https://www.chezmoi.io/)をご存知の方は、これを都度任意のディレクトリへapplyできるものと考えてもらうと想像しやすいかもしれません。

## 使用例

自身の具体的な利用ケースとして、Go言語のCLIのリポジトリを作成するシートを用意しています（というかこのために作った）。
拡張子.stampのファイルに含まれるモジュール名などを作成したいCLIツールの名称等に合わせてコピー時に設定することができます。

~~~sh
$ tree $(stamp config-dir)
~/.config/stamp/
├── sheets
│   └── go-cli            # go-cliというシート名で配下のディレクトリ構造を管理
│       ├── bootstrap.sh  # go mod tidyやgitのinitial commitなどを行う（後述）
│       ├── cmd           # 以下はCLIツールのスケルトンやMakefile、LICENSEなど
│       │   ├── cli_test.go
│       │   ├── cli.go.stamp
│       │   └── version.go
│       ├── go.mod.stamp
│       ├── internal
│       │   └── greet
│       │       ├── greet_test.go
│       │       └── greet.go
│       ├── LICENSE.stamp
│       ├── main.go.stamp
│       ├── Makefile.stamp
│       └── README.md.stamp
└── stamp.yaml            # ユーザー名などのデフォルト値を管理（後述）
~~~


以下のように実行すると、テストが通る状態でプロジェクトを開始できます。また、GitHubにpushすれば[tagpr](https://github.com/Songmu/tagpr)を用いたリリースの設定まで完了した状態になります。

~~~sh
APPNAME=sample; \
mkdir "$APPNAME" \
  && cd "$APPNAME" \
  && stamp -s go-cli appname="$APPNAME" \
  && sh bootstrap.sh
~~~

興味がある方はこちらを参照ください。シートをさらにchezmoiで管理していますが内容はそのまま確認できます。
- [Go言語のCLIのリポジトリを作成するテンプレート一式（シート）](https://github.com/monochromegane/dotfiles/commit/56ccfd6dc17e53df25e027b86ba142ba29fcab4a)

## 便利な機能

stampの基本機能はこれだけですが、利用にあたり少し便利な機能ももう少し作っているので軽く紹介します。

### シート登録の支援

既存のディレクトリやファイルをシートとして登録するための`collect`サブコマンドを用意しています。

~~~sh
# カレントディレクトリを再帰的にテンプレートとして（.stamp拡張子をつけて）
# sampleシートとして登録
$ stamp collect -s sample -t .
~~~

### デフォルト値の指定

`$(stamp config-dir)/stamp.yaml`にシートのテンプレート展開時にデフォルトで指定する値をキーごとに設定しておくことができます。
コマンド実行時に指定があればそちらが優先されます。

~~~sh
year: 2026
username: monochromegane
~~~

### パラメータ不足チェック機能

事前にテンプレートに必要なパラメータの値を覚えておいたり覗いて回って思い出すのは時間がかかります。
stampでは、必要なパラメータをうろ覚えのまま実行しても、不足があれば適用前に教えてくれます。どのファイルに何のパラメータが必要かが分かるので、便利です。

例えば、先ほどのgo-cliでは、stamp.yamlで定義しているデフォルトの値の他に、必ずappnameと言うキーに対して値を与えなければなりませんが、これをせずに実行するとこういうエラーがでます。

~~~sh
# stamp -s go-cli appname=monochromeganeとして呼ぶべき
$ stamp -s go-cli
Error: stamp failed: missing required template variables:

  - appname
    used in:
      - .gitignore.stamp
      - LICENSE.stamp
      - Makefile.stamp
      - README.md.stamp
      - cmd/cli.go.stamp
      - go.mod.stamp
      - main.go.stamp

Provide missing variables using:
  - Command line: stamp -s <sheet> -d <dest> appname=<value>
  - Config file: Create stamp.yaml in sheet or config directory
~~~

### その他

将来シートを増やした際の共通シートが欲しくなることを想定して、複数シートを同時に指定できる機能なども設けています。

なお、機能をシンプルに保ちたいため、適用後に実施したいスクリプトなどを実行する機能は持たせないことにしました。
ひとまずは、シート中に以下のような`bootstrap.sh`などを用意し、stampコマンド実行後に続けて実行する運用にしています。

~~~sh
#!/usr/bin/env sh
set -e

go mod tidy
git init
git add . ":!./$0"
git commit -m "Add project skeleton"
make test

rm -- "$0"
~~~

## おわりに

今回のツール実装はClaude Codeに任せ、コマンドラインとしての使い心地の観点からコメントしながら進めました。
ただし、出発点は上述のシート相当のプロジェクト構成を与えた上で実装を開始してもらったため、実装の方向性については想定した範囲に収まりやすかったかなと感じています。

CLIツール開発の着手時の迅速化のみならず、コーディングエージェントへの実装補助線を与えるという点でも役立つツールかなと思いますので、よければ使ってみてください。

