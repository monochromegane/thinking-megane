---
layout: post
title: "agとUnite.vimで快適高速grep環境を手に入れる"
date: 2013-09-18 22:36
comments: true
categories: ag ack unite.vim grep the-silver-searcher
---

今までVim内のgrepにはUnite.vimを使っていたんですが、ファイル数が多いときに遅く感じることがあったので、前回導入した [ag(The Silver Searcher)](https://github.com/ggreer/the_silver_searcher)と組み合わせて快適高速grep環境をつくりました。

<br />
<hr />

# The Silver Searcher と Unite.vim

The Silver Searcherは、grepやackより高速な検索が売りのパターン検索を行うコマンドです。

また、Unite.vimは、様々なデータソースを共通のインターフェースで操作できるプラグインです。

ディレクトリのファイル一覧や、バッファ一覧などを同じインターフェースで操作できるので使いはじめると手放せなくなるプラグインです。

The Silver Searcherについてはこの辺が分かりやすいと思います。

- [ackを捨てて、より高速なag(The Silver Searcher)に切り替えた](http://blog.glidenote.com/blog/2013/02/28/the-silver-searcher-better-than-ack/)


<br />
<hr />

# インストール

## The Silver Searcher のインストール

Macの場合、homebrewで提供されてます。

```console
$ brew install the_silver_searcher
```

<br />
## Unite.vim のインストール

BundleやNeoBundleによるインストールがおすすめです。

Bundleの場合`~/.vimrc`に以下を記述してノーマルモードで`:BundleInstall`を実行、インストールします。

```
Bundle "Shougo/unite.vim"
Bundle "git://github.com/Shougo/vimproc" 
```

インストール後、vimprocをコンパイルします。

```console
$ cd ~/.vim/bundle/vimproc
$ make # Macの場合
```

makeファイルはOSごとに違うので[公式のREADME](https://github.com/Shougo/vimproc.vim)を確認してください。

<br />
<hr />

# 設定

`~/.vimrc`に以下を記述します。

```
" insert modeで開始
let g:unite_enable_start_insert = 1

" 大文字小文字を区別しない
let g:unite_enable_ignore_case = 1
let g:unite_enable_smart_case = 1

" grep検索
nnoremap <silent> ,g  :<C-u>Unite grep:. -buffer-name=search-buffer<CR>

" カーソル位置の単語をgrep検索
nnoremap <silent> ,cg :<C-u>Unite grep:. -buffer-name=search-buffer<CR><C-R><C-W>

" grep検索結果の再呼出
nnoremap <silent> ,r  :<C-u>UniteResume search-buffer<CR>

" unite grep に ag(The Silver Searcher) を使う
if executable('ag')
  let g:unite_source_grep_command = 'ag'
  let g:unite_source_grep_default_opts = '--nogroup --nocolor --column'
  let g:unite_source_grep_recursive_opt = ''
endif
```

<br />
<hr />

# 使い方

## カレントディレクトリ以下をパターン検索

ノーマルモードで`,g`を入力するとVimのコマンドラインに

```console
Pattern:  
```

と表示されるので、検索したいパターンを入力、Enterでカレントディレクトリ以下のファイルに対して再帰的に検索が開始されます。

![Unite Grep]({{ root_url }}/images/2013/09/unite-grep.png) 

こんな感じで候補が表示されるので、一番上のプロンプトで絞込語句を入力したり、`Ctrl+p` or `Ctrl+n`でカーソルを移動させることができます。

Enterで対象のファイルが新しいバッファで開きます。

<br />
## カーソル位置の単語でパターン検索

開いているファイル内に検索したい語句がある場合は、カーソルをそこまで持っていき、`,cg`でOKです。

さきほどの`Pattern:`のところにカーソル位置の単語が入力された状態になります。
あとの使い方は同じです。

<br />
## 検索結果の再呼び出し

候補を選択したあと、再度パターン検索の結果を表示したいときは、`,r`を入力します。

`,g`や`,cg`で検索した結果は`-buffer-name=search-buffer`オプションによりバッファに保存しており、このバッファを`UniteResume`で再利用することで再度結果を呼び出すことができます。

別の検索を実行するとバッファは上書きされます。

<br />
<hr />
# Tipsなど

## パターン検索の起点ディレクトリを指定する

`,g`や`,cg`で呼び出すUnite grepはプロジェクトのトップディレクトリで使うことを想定おり、カレントディレクトリ配下をデフォルトで指定するようにしています。

もしパターン検索の起点ディレクトリを毎回指定するときは、`Unite grep:.`の`:.`を削除してください。

`Pattern`プロンプトの前に、`Target:`が表示されるようになり、ここで起点ディレクトリを指定できるようになります。

<br />
## EUC-JP/Shift-JISのファイルがパターン検索にひっかからない

The Silver SearcherはEUC-JP/Shift-JISのエンコードがされたファイルをバイナリと見なして検索対象から除外します。
もしそれらのエンコードのファイルを利用する場合は、EUC-JP/Shift-JISのファイルも検索対象とする修正版を提供しているのでこちらを利用してみてください。

詳細はこちら

- [ag(The Silver Searcher)でEUC-JP/Shift-JISのファイルも検索できるようにしてみた](http://blog.monochromegane.com/blog/2013/09/15/the-silver-searcher-detects-japanese-char-set/)

homebrewならインストールは簡単です。

```console
brew install https://gist.github.com/morygonzalez/6588887/raw/b09a904e7ca9dd09abfef88b0e0e98a50a206d3b/the_legacy_searcher.rb
```

既存のコマンドは`brew uninstall the_silver_searcher`でアンインストールしておいてください。

<br />
<hr />

Unite.vimは標準でもgrepが同梱されているのですが、agと組み合わせることでより高速な環境を手にいれることができます。
Unite.vimは他にもたくさんのことができるのでプラグインを探してみてください。

ちなみに自分のdotfileはこちらに公開しています。参考になればー。

[GitHub: monochromegane/dotfiles](https://github.com/monochromegane/dotfiles)

