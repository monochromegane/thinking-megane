+++
date = "2016-08-06T12:07:22+09:00"
title = "プログラマのための数学勉強会@福岡#5 で「Goによる勾配降下法 -理論と実践-」を発表してきた"
tags = [ "golang", "機械学習" ]
+++

8/6に開催された[プログラマのための数学勉強会@福岡#5](http://maths4pg-fuk.connpass.com/event/34164/)で「Goによる勾配降下法 -理論と実践-」を発表してきました。

<div style="speakerdeck">
<script async class="speakerdeck-embed" data-id="7b6257ea0e4942bcb3d982246f8236a0" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>
</div>

今回は勾配降下法にフォーカスした内容となっています。機械学習というブラックボックスが実は誤差を最小化するものであり、そのために勾配降下法というアプローチがある、という基本でもあり、数式に抵抗があると最初につまづく箇所でもあります。

今回は数式と図解に加え、Go言語によるサンプル実装も添えることでプログラマへも理解しやすくなるように資料を作ってみました。

また、勾配降下法の手法だけではなく収束速度の改善や学習率の自動調整といった最適化の手法も紹介しているので、基本を理解している人もよければ御覧ください。

## サンプル実装

発表で使ったサンプル実装はこちらで公開しています。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fgradient_descent" title="monochromegane/gradient_descent" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/gradient_descent"&gt;monochromegane/gradient_descent&lt;/a&gt;</iframe>

正弦関数を元にしたトレーニングセットに対して多項式回帰を行うことができます。

このような感じで各種勾配降下法や最適化手法、ハイパーパラメーターの調整によってどのように学習が収束していくか、遊んでみてください。

```sh
$ go run cmd/gradient_descent/main.go -eta 0.075 -m 3 -epoch 40000 -algorithm sgd -momentum 0.9
```

こういうグラフが出力されます。

![gradient_descent](/images/2016/08/gradient_descent.png)

## プログラマのための数学勉強会@福岡

今回は発表に向けて自分で最適化手法について調べて実装して、検証してを繰り返すことで自分自身も理解が深まってよかったです。

発表内容も多様で、個人的には@hokutsさんの、意識の有無を統合情報量として数式に落としこむ統合情報理論の紹介が面白かったです。高度に複雑化した機械学習のモデルが意識を持つのか。色々妄想が進みそうです。

福岡で数学に興味ある方、お気軽に。発表者募集中とのことです。

