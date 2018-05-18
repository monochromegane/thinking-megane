+++
date = "2018-02-25T20:43:40+09:00"
tags = [ "golang" ]
title = "Go言語でオンライン外れ値検出エンジンSmartSifterを実装した"
+++

推薦システムにおける被推薦者の文脈把握に向けて確率的な手法での行動分析を検討している．そこで，[データマイニングによる異常検知](https://www.amazon.co.jp/gp/product/4320018826/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=monochromeg03-22&creative=1211&linkCode=as2&creativeASIN=4320018826&linkId=9ef342a0d75794d32bcc7a18b8641e4d)で紹介されていたオンライン外れ値検出エンジンである`SmartSifter`の理解を深めるため実装してみた．

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fsmartsifter" title="monochromegane/smartsifter" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/smartsifter"&gt;monochromegane/smartsifter&lt;/a&gt;</iframe>

# SmartSifter

> On-line Unsupervised Outlier Detection Using Finite Mixtures with Discounting Learning Algorithms. This method is proposed by Yamanishi, K., Takeuchi, J., Williams, G. et al. (2004)

> refs: http://cs.fit.edu/~pkc/id/related/yamanishi-kdd00.pdf

SmartSifterはデータの発生分布が時間とともに非定常に変化していくことを考慮した外れ値検出の手法である．離散値と連続値のベクトルを入力とし，離散値はヒストグラム型の確率密度関数で，連続値は混合ガウスモデルで表し，これをデータから推定する．実際にはオンライン対応のために忘却型学習アルゴリズムとしてそれぞれ拡張されたSDLEとSDEMと呼ばれるアルゴリズムが利用される．また，パラメトリックなモデルを用いるSDEMアルゴリズムはノンパラメトリックなSPDUアルゴリズムと交換することが可能である．外れ値のスコアリングには対数損失またはヘリンジャースコアを利用することができる．詳細は上記論文を参照されたい．

# 外れ値検出

[Old Faithful Geyser Data](http://www.stat.cmu.edu/~larry/all-of-statistics/=data/faithful.dat)として公開されている間欠泉に関するデータを用いてオンライン外れ値検出を試みる．

![smartsifter](https://user-images.githubusercontent.com/1845486/36640826-93e5ab9a-1a69-11e8-8672-7b59116528ad.gif)

x軸が`eruptions`で，y軸が`waiting`．各軸は標準化N(0,1)で標準化済みとする．このデータを順番に入力し，各入力でオンライン学習されたSmartSifterの外れ値スコアを全エリアで確認した．スコアが高い（=外れ値の確率が高い）ほど黄色に近付く．

入力が進むごとにデータの分布に大まかに沿うようにスコアリングされていることが確認できる．

# 離散値を含んだ外れ値検出

SmartSifterは離散値ベクトルを排反な集合に分類したセルごとに混合ガウスモデルを学習する．標準化後の間欠泉のデータからxが正か負,yが正か負かの4象限をセルとしてオンライン外れ値検出を試みる．

![smartsifter_2](https://user-images.githubusercontent.com/1845486/36641381-247afd9c-1a72-11e8-9fe2-2303e3f67b4a.gif)

各セルごとにスコアが異なることが見て取れる．実運用ではsource-IPaddressなど，各セルごとに連続値側の振る舞いが異なることが想定されるようなものが与えられるだろう．

# 実装

実装は論文に準じているが，$\overline{\mu}\_i^{(t)}$と$\overline{\Lambda}\_i^{(t)}$については論文中に初期値が与えられていなかったので，それぞれ$\mu\_i^{(t)}$と$\Lambda\_i^{(t)}$から求めた．また，$\Lambda\_i^{(0)}$は単位行列とした．

現状，Golangからは以下のように利用することができる．

```go
r := 0.1 // Discounting parameter.
alpha := 1.5 // Hyper parameter for continuous variables.
beta := 1.0 // Hyper parameter for categorical variables.
cellNum := 0 // Only continuous variables.
mixtureNum := 2 // Number of mixtures for GMM.
dim := 2 // Number of dimentions for GMM.

ss := smartsifter.NewSmartSifter(r, alpha, beta, cellNum, mixtureNum, dim)
logLoss := ss.Input(nil, []float64{0.1, 0.2}, true)
fmt.Println("Score using logLoss: %f\n", logLoss)
```

# まとめ

こういった確率モデルに従うようなアルゴリズムの勉強のため，今回はSmartSifterを実装した．実運用ではハイパーパラメータの調整が必要になってくると思われるが，今回のような単純なデータであれば想定した結果を得られることがわかった．

現状，ヘリンジャースコアやノンパラメトリックなアルゴリズム(SPDU)は実装していない．また，CLIも用意していないので今後必要になれば順次実装していく．

また，今後は独立モデルとしての外れ値検出だけでなく時系列モデルや行動モデルを仮定していく異常検出も試していきたい．

