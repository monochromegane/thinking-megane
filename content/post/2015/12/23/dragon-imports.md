+++
date = "2015-12-23T22:14:50+09:00"
title = "goimportsを高速化するdragon-importsコマンドをつくった"
tags = [ "golang", "goimports" ]
+++

`goimports`はコードのフォーマットに加えてインポート行の追加・削除を行ってくれる便利なコマンドですが、GOPATH配下に大量のリポジトリが存在するとインポートの解決に時間がかかるようになってしまいます。いつでも素早くインポートしたい！ということでdragon-importsというコマンドをつくってみました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fdragon-imports" title="monochromegane/dragon-imports" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/dragon-imports"&gt;monochromegane/dragon-imports&lt;/a&gt;</iframe>

## 使い方

インストールして`dragon-imports`コマンドを実行するだけです。

```sh
$ go get github.com/monochromegane/dragon-imports/...
$ dragon-imports
```

コマンド実行後、`goimports`コマンドが高速化されます。`GOPATH/src`に230リポジトリある環境での実行時間の差は以下の通りです。

---

**goimports の実行時間(秒)**

| before | after |
| ------:| -----:|
| 0.893  | 0.019 |

体感では全く結果を待つ必要がなくなりました！

## どうやって高速化するのか

### goimportsの動作

`goimports`は標準パッケージのシンボルとインポートパスのマッピングを内部に保持しているため、それらに対するインポートパスの解決は即座に行われます。

ここで解決できなかった場合は、`GOPATH/src`配下に対する探査が発生してしまうため、GOPATH配下に大量にリポジトリがある場合は、時間がかかるようになります（複数回インポートパスを追加する場合でも探査自体は一度しかしないようです）。

### dragon-importsはどう解決するか

`dragon-imports`は既存の`goimports`コマンドを上書きします。

`dragon-imports`は`GOPATH/src`配下にあるエクスポートされた関数や型を上述のマッピングに追加して、時間のかかる探査自体を発生させないようにします。具体的には

- 1. `GOROOT/api`配下の定義から標準パッケージのマッピングを取得
- 2. `GOPATH/src`配下のパッケージからエクスポートされた関数や型のマッピングを生成
- 3. 上記1.と2.をあわせて`goimports`のマッピング定義を上書き(zstdlib.go)
- 4. `goimports`を再度インストール(go install -a)

を行っています。

---

以上、GOPATH爆発問題をちょっと強気に解決するdragon-importsを紹介させていただきました。
現状使っている分には特に困っていないですが、やり方がやり方なだけにご利用は自己責任でお願いします。

それでも普段使うGOPATH配下のパッケージがストレスなくインポートされるのはやはり気持ちがよいですね。

### やっぱりコワいんですけど

安心してください、戻せますよ。

```sh
$ cd $GOPATH/src/golang.org/x/tools
$ git checkout imports/zstdlib.go
$ cd $GOPATH/src/golang.org/x/tools/cmd/goimports
$ go install -a .
```

## TODO

- パッケージの変更を検知してdragon-importsをバックグラウンドで実行する仕組み
- dragon-importsの対象から除外するオプション
- vim-goの`g:go_fmt_command = "goimports"`でもう少し速度向上を実感できるようにする

