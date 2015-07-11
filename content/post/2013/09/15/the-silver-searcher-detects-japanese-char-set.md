---
title: "ag(The Silver Searcher)でEUC-JP/Shift-JISのファイルも検索できるようにしてみた"
date: 2013-09-15
comments: true
categories: ag the-silver-searcher euc-jp shift-jis ack
---

[ag(The Silver Searcher)](https://github.com/ggreer/the_silver_searcher)は、高速なパターン検索が行えるため、ackから切り替える人も多いと思いますが、日本語文字セット（EUC-JPやShift-JIS）のファイルがバイナリファイルとして判定されて検索対象からスキップされてしまうという問題があります。

- [nDiki: ag やめて ack に戻す](http://www.naney.org/diki/d/2013-07-17-The-Silver-Searcher.html)

業務ではまだEUC-JPやShift-JISのファイルを触る機会があるため、ag でEUC-JP/Shift-JISのファイルも検索できるようにしてみました。

<br />
<hr />


# インストール

```console
$ git clone https://github.com/monochromegane/the_silver_searcher
$ cd the_silver_searcher
$ git checkout detect-japanese-char-set
$ ./build.sh
$ make install
```

- ** 既にhomebrewなどでインストール済みの agコマンド がある場合はアンインストールしておいてください **
- ** コンパイル前に`automake`とか`pcre-devel`を適宜インストールおねがいします **

<br />
<hr />

# 利用方法

通常の ag コマンドと同様です。

```console
$ ag [options] PATTERN [PATH]
```

<br />
<hr />

# 修正点など

`src/util.c`の`is_binary`内に EUC-JPとShift-JISの判定を追加しています。

( なぜか10進数になっているのは既存のUTF-8を検出しているとこにあわせてるからです。16進数のほうがいいかな... )

```c
/* EUC-JP detection */
if (buf_c[i] == 142) {
    i++;
    if (buf_c[i] > 160 && buf_c[i] < 224) {
      continue;
    }
} else if (buf_c[i] > 160 && buf_c[i] < 255) {
  i++;
  if(buf_c[i] > 160 && buf_c[i] < 255) {
    continue;
  }
}
/* Shift-JIS detection */
if (buf_c[i] > 160 && buf_c[i] < 224) {
  continue;
} else if ((buf_c[i] > 128 && buf_c[i] < 160) || (buf_c[i] > 223 && buf_c[i] < 240)) {
  i++;
  if ((buf_c[i] > 63 && buf_c[i] < 127) || (buf_c[i] > 127 && buf_c[i] < 253)) {
    continue;
  }
}
```

性能に関しても特に劣化していないと思います。
不具合などあれば、プルリクください。

<br />
<hr />

# 今後

日本語圏に限定された修正なので、プルリクだしても本家側には取り込まれない気がする…

このForkしたブランチでも本家masterの修正に追随していく予定なので、EUC-JPやShift-JISの資産をさわる方々、しばらくはこちらを使ってみてください。

