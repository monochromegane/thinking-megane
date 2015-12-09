+++
date = "2015-12-08T20:23:40+09:00"
title = "V2! V2! Go言語製 高速検索ツールThe Platinum Searcherのv2をリリースしました"
tags = [ "golang", "ag", "pt", "the_platinum_searcher" ]
+++

**What a lovely day !!!**

---

本日、Go言語製 高速検索ツール The Platinum Searcher(pt) のバージョン2をリリースしました。今回は検索速度の向上に主軸を置き、旧バージョンと比較して5倍の高速化を実現しています。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fthe_platinum_searcher" title="monochromegane/the_platinum_searcher" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/the_platinum_searcher"&gt;monochromegane/the_platinum_searcher&lt;/a&gt;</iframe>

# v2

[約2年前にGo言語の勉強のためつくったThe Platinum Searcher(pt)](http://blog.monochromegane.com/blog/2014/01/16/the-platinum-searcher/) ですが、おかげ様でたくさんのPRをもらいながら随分と高機能になりました。反面、速度面についてはまだチューニングの余地を残した状態が続いていたため、今回のバージョンアップにあたって全面書き換えを行い高速化を図りました。

## Benchmark

まずはベンチマーク結果をご覧ください。

![pt_benchmark](/images/2015/12/pt_benchmark.png)

MacBook Pro (OS X 10.11.1 / CPU: 2.5GHz Core i5, Memory: 8GB) での実行結果。

---

[Linuxカーネルのソースコード](https://github.com/torvalds/linux) (約1.8GB) を `EXPORT_SYMBOL_GPL` という文字列で検索しています。
青がOSのキャッシュが効いた状態、赤が効いていない状態の実行時間です。

旧バージョン(`pt(old)`)と比較して、キャッシュなしで**2倍**、キャッシュありの状態では**5倍**の速度で検索できるようになりました。キャッシュが効いていない初回検索時の速度が向上したのは個人的にもうれしいところです。

## 実装について

速度改善に向けた実装の主な変更点は

- Find時の多重度を増やす
- Grep時のsyscall.Read回数を減らす
- Print時の不要なchannelをやめる
- Gitignoreのマッチングアルゴリズムの見直し

でした。ここについては [Advent Calendar 2015 - Go その2](http://qiita.com/advent-calendar/2015/go2)で担当の12/15にまとめようと思いますのでよければご覧ください。

# 使い方

使い方などはこれまでと一緒です。

## インストール

```sh
$ go get -u github.com/monochromegane/the_platinum_searcher/...
```

するか、Macの場合はHomebrewでもインストールできます。

```sh
$ brew tap monochromegane/pt
$ brew tap-pin monochromegane/pt
$ brew install pt
```

または[こちら](https://github.com/monochromegane/the_platinum_searcher/releases)から各OS用のバイナリをダウンロードしてください。対応OSはWindows、Linux、Macです。

## 検索

インストールした`pt`コマンドで検索を行います。対応している文字コードはUTF-8、EUC-JP、Shift-JISです。

```sh
# カレントディレクトリ以下を検索
$ pt PATTERN

# ディレクトリを指定して検索
$ pt PATTERN dir1 dir2
```

ディレクトリ内に.gitignoreがあれば記載されたパターンを検索対象から除外します。

## オプションと設定ファイル

ptには様々なオプションがありますが、これらを `~/.ptconfig.toml` として定義しておくことでデフォルトの挙動を変更することができるようになっています。

```ini
color = true
context = 3
ignore = ["dir1", "dir2"]
```

同名のオプションを実行時に指定した場合は、そちらが優先されます。

# まとめ

機能の豊富さと性能をバランスよく保ちながらプロダクトを成長させていくのはなかなか難しいですが面白いところです。The Platinum Searcherは色々思い入れのあるツールなので今後も大切に育てていきたいと思います。

今後はキャッシュが効いている状態での一層の性能向上と正規表現まわりの改善を進めていきたいなあと思っています。v2になって以前と違う動きをしているよ〜等ありましたらIssueやPRで知らせていただけるとうれしいです。

