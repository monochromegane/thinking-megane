---
title: "Go言語でag(The Silver Searcher)ライクな高速検索ツールをつくった。EUC-JP/Shift-JISも検索できマス。"
date: 2014-01-16
comments: true
tags: [ "ag", "ack", "grep", "the-silver-searcher", "unite.vim", "golang" ]
---

いまや高速パターン検索といえばag(The Silver Searcher)ですが、検索対象がUTF-8のテキストを前提としているため、EUC-JPやShift-JISといったファイルを検索するのに課題があります。

これまで、それらの日本語文字セットを検索できるようにするため、色々とagの改造、公開を行っていました。

- [ag(The Silver Searcher)でEUC-JP/Shift-JISのファイルも検索できるようにしてみた](/blog/2013/09/15/the-silver-searcher-detects-japanese-char-set/)
- [日本語圏特化型ag -白金- の配備が完了しました](/blog/2013/09/23/sg-spec/)

しかし、特定の国の文字コードだけに特化した修正というのをmasterに取り込んでもらうわけにもいかず、派生ブランチとして追随するのも、やはり面倒...

そこで年始にGo言語を触ったのをきっかけに、Go言語でパターン検索ツールをつくってみました。

せっかくなのでポストagを目指して、[プラチナサーチャー(The Platinum Searcher)](https://github.com/monochromegane/the_platinum_searcher)と名付けてます。

<hr />

# The Platinum Searcher

## 機能

- ag相当の検索速度
- プロジェクトのgitignore, hgignoreファイルによる検索対象の除外
- 日本語文字セット(UTF-8, EUC-JP, Shift-JIS)ファイルの検索
- 各OS(Mac OS X, Windows, Linux)向けのバイナリ提供

### ベンチマーク

`ack`, `ag`と比較した結果です。agと同じくらいの速度は出せていると思います。

```
ack go  6.24s user 1.06s system 99%  cpu 7.304 total
ag go   0.88s user 1.39s system 221% cpu 1.027 total
pt go   1.05s user 1.03s system 195% cpu 1.066 total
```

ディレクトリ検索、Grep処理、結果表示などをゴルーチンを使って並行化したりしていますが、今回のようなシンプルな実装でこれだけの速度を出せているGo言語、なかなかよさそうです。



## インストール

各OS向けにビルドしたものを提供しています。

以下のURLで提供しているバイナリファイルをパスの通ったところに置けばOKです。

- [Mac OS X(x86 64bit)](https://drone.io/github.com/monochromegane/the_platinum_searcher/files/artifacts/bin/darwin_amd64/pt)
- [Mac OS X(x86 32bit)](https://drone.io/github.com/monochromegane/the_platinum_searcher/files/artifacts/bin/darwin_i386/pt)
- [Windows(x86 64bit)](https://drone.io/github.com/monochromegane/the_platinum_searcher/files/artifacts/bin/windows_amd64/pt.exe)
- [Windows(x86 32bit)](https://drone.io/github.com/monochromegane/the_platinum_searcher/files/artifacts/bin/windows_i386/pt.exe)
- [Linux(x86 64bit)](https://drone.io/github.com/monochromegane/the_platinum_searcher/files/artifacts/bin/linux_amd64/pt)
- [Linux(x86 32bit)](https://drone.io/github.com/monochromegane/the_platinum_searcher/files/artifacts/bin/linux_i386/pt)


## 使い方

通常の ag コマンドと同様です。

```sh
$ pt [OPTIONS] PATTERN [PATH]
```

コマンド名は`pt`です。プラチナなので。


## Unite.vimとの連携

[agとUnite.vimで快適高速grep環境を手に入れる](/blog/2013/09/18/ag-and-unite/) で紹介したのと同じようにプラチナもUnite.vimと連携できます。

```vim
nnoremap <silent> ,g :<C-u>Unite grep:. -buffer-name=search-buffer<CR>
if executable('pt')
  let g:unite_source_grep_command = 'pt'
  let g:unite_source_grep_default_opts = '--nogroup --nocolor'
  let g:unite_source_grep_recursive_opt = ''
endif
```

<hr />

今後は、ポストag、プラチナの名に恥じないよう、agより速く検索できるようにしていきたいなと思ってます。

まだGo言語は触り始めたところなので、プルリクでのツッコミもらえたらうれしいです。

