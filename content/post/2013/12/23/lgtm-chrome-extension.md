---
title: "LGTMでめでたさを伝えるChrome拡張をつくった"
date: 2013-12-23
comments: true
tags: lgtm chrome extension 拡張 github
---

先日、[高速にドッグフードを食べる方法](https://speakerdeck.com/hitode909/gao-su-nidotuguhudowoshi-berufang-fa)を見て、LGTMと[LGTM.in](http://www.lgtm.in/)というサービスを知りました。

LGTMは、**"looks good to me."**の略で、GitHubのプルリクに対するOKコメント、LGTM.inは、そのコメントにノリのいい画像を添えて`spice up`しようというサービスです。コードレビューの終わりに**めでたさを伝え**てもらえれば、字面だけのLGTMよりもうれしいし、そういう小さな承認はお互いに大事。

ということで、LGTMでめでたさを伝えるChrome拡張をつくりました。

# LGTM Chrome拡張

インストールはこちらから。 [Chrome ウェブストア/LGTM](https://chrome.google.com/webstore/detail/lgtm/oeacdmeoegfagkmiecjjikpfgebmalof?hl=ja&gl=JP)

- 拡張を起動するとLGTM.inから取得したランダム画像を3件表示します。
- 画像が気に入らないときは`More LGTM`ボタンをクリックして画像を再取得できます。

![LGTM](/images/2013/12/LGTM_screenshot.png)

<br />

## 使い方

- 使用したいLGTMの画像をクリックするとURLをクリップボードにコピーします。
- GitHubのプルリクページを開いていれば、選択した画像のURLをマークダウン形式でコメント欄に入力します。

![LGTM](/images/2013/12/LGTM_comment.png)

<br />

## その他

リポジトリはこちら。 [monochromegane/LGTM](https://github.com/monochromegane/LGTM)

<br />
<hr />

LGTM.inはよいコンセプトだと思うのですが、プルリクのコメント書いているときに別のサイトを開いたり、提供されるBookmarkletがプレビューしづらかったりしてちょっと使いにくい。
こういう取り組みはめんどくさくなると途端にやらなくなるので、やりやすい環境を準備してみました。

冒頭にも書きましたが`めでたさを伝える`っていいことだと思うので、このChrome拡張でLGTM使ってみてはどうでしょうか。


