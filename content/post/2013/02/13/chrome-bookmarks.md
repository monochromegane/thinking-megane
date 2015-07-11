---
layout: post
title: "Unite.vimでChromeのブックマークを表示するスクリプトを作った"
date: 2013-02-13
comments: true
categories: vim
---

Unite.vimでChromeのブックマークを表示するスクリプトを作りました。(Mac限定)

[monochromegane/unite-script](https://github.com/monochromegane/unite-script) /examples/chrome_bookmarks.scpt
<br />


## Unite.vimとUnite-script
---

### Unite.vim
- 「候補群を絞り込んでアクションを実行する」インターフェースを実現するVimプラグイン
- [Shougo/unite.vim](https://github.com/Shougo/unite.vim)

### Unite-script
- 通常VimScriptで書かなくてはいけないUnite.vimの候補群（source）を好きなスクリプト言語で書くためのsource
- [hakobe/unite-script](https://github.com/hakobe/unite-script)

## ChromeのブックマークをUnite.vimで表示する
---

Unite.vimのsourceとしてchrome_bookmarksスクリプトを作成しました。
Unite-scriptから呼び出して使うことでUnite.vim上で以下のことができるようになります。

![chrome_bookmarks.scpt]({{ root_url }}/images/2013/02/unite_script_chrome_bookmarks.png)

- Unite.vimにChromeのブックマークを表示
- ブックマークを検索
- 選択したブックマークをブラウザで開く


**Mac Only!**
> chrome_bookmarksスクリプトはAppleScriptで書かれており、Macでのみ動作します。
<br />


## chrome_bookmarksスクリプトのインストール
---

### 事前準備

chrome_bookmarksの動作にはUnite.vimとUnite-scriptが必要です。
上述のURLから入手、設定をしておいてください。

**Unite-scriptのエラーについて**
>2013/02/13時点でUnite.vimのAPI変更にUnite-scriptが追随していないため実行時にs:bufferが未定義である旨のエラーが発生します。
>プルリクエスト等も取り込まれていないようなので、Forkして修正しました。
修正にあわせ、chrome_bookmarksスクリプトをexamplesフォルダに格納したので、以下の手順でインストールしてください。

### インストール

vimrcに`Bundle "monochromegane/unite-script"`を記載し、`:BundleInstall`でインストールしてください。

<br />


## chrome_bookmarksスクリプトの使い方
---

#### Chromeのブックマーク一覧を表示
`:Unite script:osascript:~/.vim/bundle/unite-script/examples/chrome_bookmarks.scpt`

#### ブックマークの検索
通常のUnite.vimによる検索が行えます。フォルダ、ブックマーク名に含まれる文字列が対象となります。

#### ブラウザで開く
ブックマークを選択し、Enterにより、ブラウザで開くことができます。

#### キーバインドを設定
`nnoremap <slient> your_keybind :<C-u>Unite script:osascript:~/.vim/bundle/unite-script/examples/chrome_bookmarks.scpt`

<br />
 
## まとめ
---

2013年はVimの年にするぞ！とvimrcを整備し始めました。
Unite.vimとVimfilerプラグインを使うようになって、コーディング作業の大部分がVimのなかで完結するようになり、もはや手放せないプラグインになってます。

ブラウザ操作もUnite.vimでできればうれしいと思い、手始めにブックマークを操作するスクリプトを作ってみました。

今後は、Google検索などもVimからできるようにスクリプトを増やしていこうと思います。

ではでは！

















