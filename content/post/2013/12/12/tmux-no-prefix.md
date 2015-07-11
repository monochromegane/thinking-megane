---
title: "tmuxのプレフィックスに疲れたらbind -nオプションを使おう"
date: 2013-12-12
comments: true
tags: [ "tmux" ]
---

ターミナルマルチプレクサ[tmux](http://tmux.sourceforge.net/)はとても便利ですが、操作の際にプレフィックスが必要で、ペインの切り替えなど頻繁に行うのはちょっと面倒です。

実はtmuxには、プレフィックスなしで操作できるステキすぎるオプションがあるので紹介しておきます。

<hr />

# プレフィックス不要ってすばらしい

## bind-key の -n オプション

`bind-key`には`-n`オプションがあり、これをつけることでプレフィックスを使わなくてもよくなります。

これで単独のキーにコマンドを割り当てることができます。


## 設定

通常のキーバインド設定に`-n`をつけるだけです。

`tmux.conf`
```
bind -n キー コマンド
```


## プレフィックス不要なペイン移動

自分はペインの移動を最もよく使うので以下のような設定をしています

```sh
# Shift + 上下左右でペインを移動できるようにする。
bind -n S-left select-pane -L
bind -n S-down select-pane -D
bind -n S-up select-pane -U
bind -n S-right select-pane -R
```

これでもプレフィックスが不要になるので充分便利だったのですが、矢印キーはホームポジションから遠いので最近はこんな設定を追加して、`Ctrl+o`でペインを順番に移動するようにしています。

```sh
# ペインの移動(ローテート)
bind -n C-o select-pane -t :.+
```

ペインの数が少ない場合はこちらのほうが結果的に速く移動できる気がします。

# Tipsなど

## マニュアル

このあたりを参考にまだまだ応用できそう。

- [コマンド](https://bytebucket.org/ns9tks/tmux-ja/wiki/tmux-ja.html#id5)
- [キーバインド](https://bytebucket.org/ns9tks/tmux-ja/wiki/tmux-ja.html#id11)


## 設定の確認

以下の設定を入れておけば、プレフィックス + r で設定が即時反映されます。

```sh
# 設定リロード
bind r source-file ~/.tmux.conf
```

キーバインドはコンソールから`list-keys`で確認できます。

```console
$ tmux list-keys

bind-key -n     C-o select-pane -t :.+
中略
bind-key -n    S-Up select-pane -U
bind-key -n  S-Down select-pane -D
bind-key -n  S-Left select-pane -L
bind-key -n S-Right select-pane -R
```


## 参考

自分のtmux.confも公開してます。コメントもなるべく入れているので分かりやすいと思うので参考にどうぞ。

[monochromegane/dotfiles - tmux.conf](https://github.com/monochromegane/dotfiles/blob/master/tmux.conf)

<hr />

プレフィックス一回の違いですが、エンジニアであれば日に何十、何百と打っていることになるキーが省略できるというのは、かなり気持ち良いものです。

プレフィックスがないので想定しないところでキーバインドが被ったりしますが、よく使う操作だけでも`-n`オプションをつけてみる価値はあると思います。どうぞお試しあれ。


