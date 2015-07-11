---
title: "日本語圏特化型ag -白金- の配備が完了しました"
date: 2013-09-23
comments: true
categories: ag the-silver-searcher grep sg unite.vim
---

かねてより開発を続けていたパターンマッチ検索sg (日本語圏特化型ag) の配備が完了したので、その経緯と仕様を以下に記す。

<br />
<hr />

# 開発経緯

パターンマッチ戦線に鳴り物入りで配備されたUTF8連合の誇るagだったが、極東戦線において、旧式日本語文字セットを散りばめ潜伏するEUC-JP/Shift-JIS軍の極東迷彩の前に、その索敵機能を充分に発揮できないことが判明するや一気に劣勢化。戦線から離脱しつつあった。

事態を重く見たUTF8連合極東支部開発主任は、UTF8ディテクション機能を応用したEUC-JP/Shift-JISディテクション機能を追加、日本語圏仕様の改良型を開発した。[[開発コード legacy-0.1]](https://github.com/monochromegane/the_silver_searcher/releases/tag/legacy-0.1)

戦線投入のため、同支局によりHomeBrew化が加えられるも、入出力機構が従来型のため活躍の範囲は限定された。

後に入出力機構にiconvを組み込み、旧式日本語文字セットの自動検出に対応した特化型が開発された。[[開発コード legacy-0.2]](https://github.com/monochromegane/the_silver_searcher/releases/tag/legacy-0.2)

またこの特化型への換装以降、agとの誤用を避けるため、白金（しろがね）と呼称を変更、開発コードもsgに変更されている。[[開発コード sg-0.16.4]](https://github.com/monochromegane/the_silver_searcher/releases/tag/sg-0.16.4)


<br />
<hr />

# 仕様

## 配備

[白金専用Fomula](https://github.com/monochromegane/homebrew-sg)からHomeBrewにて配備を行う。

```console
$ brew tap monochromegane/sg
$ brew install sg
```
<br />

## 実行

基本的に換装前と操作は同じである。

```console
$ sg [options] PATTERN [PATH]
```

<br />

## 自動検出

ASCIIのみ、UTF-8、EUC-JP、Shift-JISでエンコードされたファイルを自動で検出し、パターンマッチの対象とする。

上記に該当しない場合の挙動については、換装前と同じく、バイナリと見なし特定のオプションを渡さない限りパターンマッチ対象外となる。

<br />

## 入出力機構

実行時の利便性を高めるため、対象ファイルと入出力のエンコードの差異を吸収する。
対象ファイルから読みだした行のエンコードを`LANG`環境変数に設定されているエンコードに変換しながら検索する実装となっている。

これにより、UTF-8環境でEUC-JP/Shift-JISのファイルを検索した場合にも、日本語パターンで合致させること(入力)、結果をコンソール上に文字化けせず表示させること(出力)が可能となった。

<br />
<hr />

# 連携

Unite.vimとの連携により開発効率が向上する。
基本的な連携手順については以下を参照すること。

- [agとUnite.vimで快適高速grep環境を手に入れる](http://blog.monochromegane.com/blog/2013/09/18/ag-and-unite/)

なお、検索コマンドをagからsgに変更する必要がある。

```
" unite grep に sg(The Silver Searcher - Ver.Shirogane) を使う
if executable('sg')
  let g:unite_source_grep_command = 'sg'
  let g:unite_source_grep_default_opts = '--nogroup --nocolor --column'
  let g:unite_source_grep_recursive_opt = ''
endif
```
※オプションについては特に変更は不要。


<br />
<hr />


# 参考

- [ag(The Silver Searcher)](https://github.com/ggreer/the_silver_searcher)
- [ackを捨てて、より高速なag(The Silver Searcher)に切り替えた](http://blog.glidenote.com/blog/2013/02/28/the-silver-searcher-better-than-ack/)
- [nDiki: ag やめて ack に戻す](http://www.naney.org/diki/d/2013-07-17-The-Silver-Searcher.html)
- [ag(The Silver Searcher)でEUC-JP/Shift-JISのファイルも検索できるようにしてみた](http://blog.monochromegane.com/blog/2013/09/15/the-silver-searcher-detects-japanese-char-set/)

<br />
<hr />

極東迷彩と相対する部門での積極的な活用を期待します。



