+++
date = "2020-01-30T20:37:43+09:00"
title = "『強化学習アルゴリズム入門』第1章〜第2章の学習メモ"
tags = [ "強化学習" ]
+++

書籍『強化学習アルゴリズム入門』について第1章〜第2章を読んでの学習メモをまとめる。

強化学習は一連の行動によって得られる報酬を最大化する行動基準を学習する。
そのために、「一連の行動」を状態の遷移と捉えて、その状態に対する価値（状態価値）を求める。
また、各状態には取りうる複数の行動があると考え、その状態の価値を、これらの行動ごとの価値（行動状態価値）から求める。

## 強化学習アルゴリズムごとの価値関数

はじめに、第1章〜第2章までで学ぶ各強化学習アルゴリズムの状態価値関数と行動状態価値関数をまとめておく。

||状態価値関数 $V(S\_t)$|行動状態価値関数 $Q(S\_t,a\_t)$|
|---|---|---|
|動的計画法|$V(S\_t)=\sum\_{a} \pi (a \mid S\_t) Q(S\_t, a\_t) \tag{1}\label{1}$|$Q(S\_t,a\_t)=r\_{t+1}+\gamma V(S\_{t+1}) \tag{2}\label{2}$|
|モンテカルロ法|$V(S\_t)=\sum\_a Q(S\_t,a\_t) \frac{N\_a}{N} \tag{3}\label{3}$ $V(S\_t)=max\\{Q(S\_t,a\_t)\\} \tag{4}\label{4}$|$Q(S\_t,a\_t)=\frac{1}{m}\sum\_{i=1}^{m} G^i(S\_t,a\_t) \tag{5}\label{5}$ $Q(S\_t,a\_t) \gets Q(S\_t,a\_t)+\alpha[G(S\_t,a\_t)-Q(S\_t,a\_t)] \tag{6}\label{6}$ where $G(S\_t,a\_t)=r\_{t+1}+\gamma G(S\_{t+1},a\_{t+1}) \tag{7}\label{7}$|
|TD(0)法|同上|$Q(S\_t,a\_t) \gets Q(S\_t,a\_t)+\alpha[\\{r\_{t+1}+\gamma(Q(S\_{t+1},a\_{t+1}))\\}-Q(S\_t,a\_t)] \tag{8}\label{8}$|

以下、各強化学習アルゴリズムについて捕捉する。

## 動的計画法

確率型ベルマン方程式\eqref{1}に従い行動状態価値を更新する手法。
ここで$\pi (a \mid S\_t)$は行動確率であり、方策$\pi$に従い状態$S$において行動$a$を選択する確率を示す。
また、$Q(S\_t, a\_t)$は行動状態価値関数であり\eqref{2}によって求める。

行動状態価値関数\eqref{2}は状態$S$において行動$a$によって遷移する状態$S$で得られる報酬$r$とその状態の価値$V$の和によって求められる。
ここで$\gamma$は価値の割引率である。

書籍ではこの価値の求め方を「将来に対する平均（$X\_{t+1}$ から$X\_N$ までの平均$\bar{X}$）の逐次的な計算」との類似性によって説明している。
\\[
\bar{X\_t}=\bar{X}\_{t+1}+\frac{1}{N-t}(X\_{t+1}-\bar{X}\_{t+1})\\
=(\frac{1}{N-t})X\_{t+1}+(\frac{N-t-1}{N-t})\bar{X}\_{t+1}
\\]

動的計画法における学習プロセスは以下のようになる。

### 方策反復法

```
1. 状態価値が収束するまで以下のステップを繰り返す
2. 各状態において方策(行動ごとの行動状態価値Qによるepsilon-greedyなど)に従い行動確率を求める
3. 各状態において行動ごとの行動状態価値を遷移後の状態の報酬と状態価値から求める
4. 2と3よりその状態の状態価値を更新する
5. 全ての状態で状態価値の更新が終わったら次の学習イテレーションへ
```

### 価値反復法

$\epsilon$-Greedyの探索率$\epsilon=0$の場合は方策に依存しない価値反復法となる。

```
1. 状態価値が収束するまで以下のステップを繰り返す
2. 各状態において行動ごとの行動状態価値を遷移後の状態の報酬と状態価値から求める
3. 2のうち最大の値で状態の状態価値を更新する
4. 全ての状態で状態価値の更新が終わったら次の学習イテレーションへ
```

## モンテカルロ法

動的計画法は環境情報が既知の場合に使えるが未知の場合、環境も含めて手探りで行動基準を学習しなければならない。
そのような場合はモンテカルロ法を利用する。

モンテカルロ法の価値の学習には総報酬$G$\eqref{7}を用いる。
総報酬は一連の行動が終わり得た報酬$r$から遡って状態（と行動）に対して価値を算出する。
書籍ではこの価値の求め方も「将来に対する平均」で表現している（tの値をt+1から求める）。

なお、行動状態価値は\eqref{5}により試行数$m$での平均とする（学習ステップにおいて\eqref{3}\eqref{4}\eqref{6}は使わない）。

モンテカルロ法における学習プロセスは以下のようになる。

```
1. 任意の試行回数だけ以下のステップを繰り返す
2. 最初の状態から報酬rを得るまで以下のステップを繰り返す
3. 現在の状態において方策(行動ごとの行動状態価値Qによるepsilon-greedyなど)に従い行動aを決定する
4. 次の状態へ遷移
5. 報酬を得たら行動を遡り各状態（と行動）の総報酬Gを求め、試行回数での平均によって行動状態価値Qを更新する
```

## TD(0)法

モンテカルロ法は総報酬$G$を求めるために必ず一連の行動を終える必要がある。
そこで、逐次的に行動状態価値を求めることでモンテカルロの探索行為を効率的に行おうとするTD(0)法がある。

TD(0)法では、行動状態価値$Q$を\eqref{8}によって求める。
これは、モンテカルロ法の行動状態価値関数\eqref{5}を逐次計算表現にした\eqref{6}の$G(S\_t,a\_t)$を動的計画法の行動状態価値関数\eqref{2}で置き換えたものと考えることができる。
つまり、総報酬$G$を$t+1$の報酬と状態価値で近似するものである。

TD(0)法における学習プロセスは以下のようになる。

### SARSA （方策反復法）

```
1. 任意の試行回数だけ以下のステップを繰り返す
2. 最初の状態から報酬rを得るまで以下のステップを繰り返す
3. (S) 現在の状態において方策(行動ごとの行動状態価値Qによるepsilon-greedyなど)に従い行動aを決定、Q(S_t,a_t)を得る
4. (A) 行動aに従い状態を遷移
5. (R) 遷移後の状態で報酬rを得る（得られない時もある）
6. (S) 遷移後の状態において方策に従い行動aを決定、Q(S_t+1,a_t+1)を得る
7. (A) 行動aに従い状態を遷移
8. 5の報酬rと3と6の行動状態価値QでQ(S_t,a_t)を更新
```

### Q学習 （価値反復法）

$Q(S\_{t+1},a\_{t+1})$を$max\\{Q(S\_{t+1},a\_{t+1})\\}$で求めることで方策に非依存とする。

```
1. 任意の試行回数だけ以下のステップを繰り返す
2. 最初の状態から報酬rを得るまで以下のステップを繰り返す
3. 現在の状態においてランダムに行動aを決定、Q(S_t,a_t)を得る
4. 行動aに従い状態を遷移
5. 遷移後の状態で報酬rを得る（得られない時もある）
6. 遷移後の状態においてランダムに行動aを決定、Q(S_t+1,a_t+1)は全行動状態価値のうち最大の値とする
7. 行動aに従い状態を遷移
8. 5の報酬rと3と6の行動状態価値QでQ(S_t,a_t)を更新
```

# 学習書籍

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="https://rcm-fe.amazon-adsystem.com/e/cm?ref=qf_sp_asin_til&t=monochromeg03-22&m=amazon&o=9&p=8&l=as1&IS2=1&detail=1&asins=427422371X&linkId=e3e609dd57189d2dd910abc95a46bbd0&bc1=ffffff&lt1=_blank&fc1=333333&lc1=0066c0&bg1=ffffff&f=ifr">
    </iframe>
