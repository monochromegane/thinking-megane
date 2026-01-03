+++
date = "2026-01-03T10:21:55+09:00"
title = "dotfilesを5年ぶりに整備した"
tags = [ "" ]
+++

なんとdotfilesも5年更新していなかった。
なにかと気忙しいのではあるが、木こりのジレンマに陥らないよう再起動していきたい。

さて、昨年末にMacbook Proを新調してまっさらな開発環境が手に入ったのに合わせてdotfilesを見直した。

- [monochromegane/dotfiles](https://github.com/monochromegane/dotfiles)

大きくは以下。

- dotfiles管理を[chezmoi](https://www.chezmoi.io/)に変更
- 開発ツール管理を[mise](https://mise.jdx.dev/)に変更
- ターミナルエミュレーターを[Ghostty](https://ghostty.org/)に変更

以下、それぞれについて簡単にメモをしておく。

## chezmoi

各ツールの設定などは大幅に変えていないのだが、dotfiles管理には[chezmoi](https://www.chezmoi.io/)を導入した。
これまで自作インストールスクリプトは、メンテナンスできていなかったし、設定ファイルの変更後のリポジトリへの反映ルールのようなものがなく反映漏れなど生じていたので、これを機にchezmoiの流儀に任せることにした。

### 設定ファイル

既存dotfilesがある場合、ファイルを`chezumoi add`していくと、chezmoiの命名規則に沿ったファイル名でリポジトリに加えていくことができる。
命名規則はホームディレクトリ以下のディレクトリもしくはファイル名のそれぞれについて`.`を`dot_`に置き換えたものになる。
例えば、`~/.config/zsh/.zshrc`であれば`dot_config/zsh/dot_zshrc`として管理する。


自分の場合は、このタイミングでXDG Base Directoryに合わせるようにもしたかったのと新しいPCで`~`が綺麗だったのもあり、chezmoi側（`chezmoi cd`で移動できるリポジトリのclone先。`~/.local/share/chezmoi`）で上記の命名に合わせて変更した上で、`chezmoi apply`をすることで反映する方法を取った。

```sh
$ chezmoi init git@github.com:monochromegane/dotfiles.git
$ chezmoi cd
$ mv zshrc dot_config/zsh/dot_zshrc
(snip)
$ chezmoi apply
```

今もリポジトリへの反映を忘れないよう基本はchezmoi側で修正してapplyをする運用にしている。

なお、dotfiles内の実行権限が必要なファイルについてはファイルに属性付与ではなく、`bin/executable_git_diff_wrapper`のように命名規則で制御する必要がある。

### インストールスクリプト

chezmoiではインストールスクリプトも管理できる。
dotfilesリポジトリの直下におくこともできるが`.chezmoiscripts`という名前のディレクトリを作ると複数スクリプトがあっても整理されるのでこの構成を採用した。

管理用ツールを導入するからには動作確認の必要な自作スクリプトの役割は最低限にしたかったので、パッケージ管理ツールのインストールのみに抑え、各パッケージのインストールなどはパッケージマネージャーに任せるようにしている。

具体的には、ツールや開発環境のパッケージ管理のためのHomebrew、miseと、Vim用のミニマルなパッケージ管理である[minpac](https://github.com/k-takata/minpac)だけをインストールしている。

```sh
$ chezmoi cd
$ ls .chezmoiscripts
run_once_after_01_install-brew.sh   run_once_after_03_install-mise.sh
run_once_after_02_install-minpac.sh
```

上記のような命名にすることで、apply後に一度だけスクリプトが順番に実行される。

## mise

開発環境やツール類はHomebrewで雑多に入れていたのと、言語ランタイムの`*env`系が増えてしまっていたのを整理すべく、[mise](https://mise.jdx.dev/)に寄せることにした。

"The front-end to your dev env"というコンセプト通り、開発環境構築に関連する多くのことができるのだが、ひとまずは、グローバルに各種言語ランタイムといくつかのツールを入れるようにしている。
ディレクトリごとに`mise.toml`を作って利用するツールを使い分けられるのはとても良さそう。

例えば、このブログで使っている[hugo](https://gohugo.io/)コマンドなどはブログ用のリポジトリでだけ使えれば良いので`mise use go:github.com/gohugoio/hugo@latest`とすれば、そのディレクトリ配下では`~/.local/share/mise/installs/go-github-com-gohugoio-hugo/0.154.2/bin/hugo`がPATH環境変数に追加されてコマンドが認識される状態になる。

なお、caskが便利なのもあり、しばらくはBrewfileによるHomebrewと併用していくことになると思う。

## Ghostty

長年iTerm2を使っていて特に不満はなかったが、chezmoi導入にあたって設定をファイルで管理したくなり、Ghosttyを使ってみることにした。

"Zero Configuration Philosophy"を謳っているだけあり、以下の設定で十分、動作も軽快で特に問題なく利用できている。念の為iTerm2もまだ入れているがこのまま乗り換えで良さそう。


```txt
command = /opt/homebrew/bin/zsh -l
mouse-hide-while-typing = true

# Theme
theme = Dracula+
background = #0f1015

# Title bar
macos-titlebar-proxy-icon = hidden

# Font
font-family = Moralerspace Neon
font-thicken = true
font-thicken-strength = 1
font-feature = -dlig
font-size = 15

# Cursor
shell-integration-features = no-cursor
cursor-style = block
cursor-style-blink = false

# TERM
term = "xterm-256color"
```

### フォントとテーマ

せっかくなので、フォントやテーマも変更してみた。

Ghosttyの標準だと日本語表示が微妙だったので、フォントとして[Moralerspace](https://github.com/yuru7/moralerspace)を導入し、`Neon`を設定し、上記設定にあるように太め大きめにしている。

また、テーマには、ダーク系のテーマでVim側ともマッチするものとして、`Dracula+`テーマを採用した。
[Dracula Theme for Ghostty](https://draculatheme.com/ghostty)としてもあるが、標準で選択できるようになっている。
Vim側も[Dracula Theme for Vim](https://draculatheme.com/vim)として公開されているので、minpackで以下のように設定を追加した。

```vimrc
call minpac#add('dracula/vim', {'name': 'dracula'})
```

テーマそのままだと背景がグレー寄りで個人的にはややぼやけたように見えてしまうので、Ghostty側で`background`を設定し、以下のようにctermbgをNoneにすることでGhostty側の背景色と同じにして統一感を出している。


```vimrc
"colorschemeを設定する
function! s:customize_dracula_bg() abort
  highlight Normal       ctermbg=None
  highlight CursorLine   ctermbg=None cterm=underline
endfunction

augroup DraculaCustomization
  autocmd!
  autocmd ColorScheme dracula call s:customize_dracula_bg()
augroup END

colorscheme dracula
```

GhosttyでDraculaで、背景色`#0f1015`の青みがかった夜空みたいな色でコンセプト的にも統一できてなかなか気に入っている。

## その他

いわゆるコーディングエージェント系の設定も見直していってるが長くなるのでまた次回とする。

## 後記的なもの

昨年、ブログを一本も書いていないことに気づいたのが今日。
年始に『[「書くこと」の哲学　ことばの再履修](https://www.kodansha.co.jp/book/products/0000415692)』を読みながら、今回の記事のような、やったことや事実ベースの「書きえる」ことすら書かずして、何を書けようという気持ちからブログ再開してみたのであるが、ただ書き出すことの爽快さを思い出せてとても良かった。
気負わずに再開していきたい。

