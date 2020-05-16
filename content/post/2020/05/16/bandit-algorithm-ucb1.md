+++
date = "2020-05-16T16:22:39+09:00"
title = "多腕バンディット問題におけるUCB方策を理解する"
tags = [ "bandit" ]
+++

多腕バンディット問題における解法の一つであるUCB1方策では以下のスコアを各腕に対して求め、最大のものを選択する。

\\[
\overline{x}\_j+\sqrt{\frac{2\ln{n}}{n\_j}} \tag{1}\label{eq-1}
\\]

ここで、$\overline{x}\_j$は腕$j$に対して観測された平均報酬、$n\_j$は腕$j$が選択された回数、$n$は全体の試行回数である。
第2項は選択された回数$n\_j$が少ないほど大きくなることから、評価が不確かな腕ほど積極的に探索を行う（`Optimism in face of uncertainty`）。

本エントリでは、この第2項について上記の振る舞い以上の理解を進める。

# 確率不等式

\eqref{eq-1}式は確率不等式の一種であるヘフディングの不等式に基づいたものであるため、まず確率不等式を学ぶ。
確率不等式は、確率の分布に依らない確率の近似的な評価に利用される不等式である。
簡単には、稀に発生する事象についての確率の上限を求めることができる。

## マルコフの不等式 Markov's inequality

非負の確率変数$X\geq0$と任意の正数$a > 0$に対して

\\[
Pr \left[ X \geq a \right] \leq \frac{E[X]}{a} \tag{2}\label{eq-2}
\\]

が成り立つ。\eqref{eq-2}をマルコフの不等式と呼ぶ。

この不等式を用いることで、確率変数$X$の実現値が$a$以上になる確率の上限を、確率分布に依らず期待値$E[X]$から求めることができる（ただし$a \leq E[X]$だと実用的な値にならない）。

マルコフの不等式を用いて例えば「期待値が10の確率変数で100以上が出る確率は10%以下」（$Pr \left[ X \geq 100 \right] \leq \frac{10}{100} = 0.1$）などと求めることができる。

証明は

\begin{align}
E[X] &= \int\_0^\infty x f(x) dx \notag \\\\ &= \int\_0^a x f(x) dx + \int\_a^\infty x f(x) dx \notag \\\\ &\geq \int\_a^\infty x f(x) dx \notag \\\\ &\geq \int\_a^\infty a f(x) dx = a \int\_a^\infty f(x) dx = a Pr [X \geq a] \notag \\\\
\end{align}

となる。なお、2行目から3行目は、右辺第1項が$\geq 0$であるため。3行目から4行目は積分区間で最も小さい$a$を使うため。

## チェビシェフの不等式 Chebyshev’s inequality

確率変数$X$が期待値$E[X]=\mu$、分散$\sigma^2$を持つとき、任意の正数$a > 0$に対して

\\[
Pr \left[ \mid X - \mu \mid \geq a \sigma \right] \leq \frac{1}{a^2} \tag{3}\label{eq-3}
\\]

が成り立つ。\eqref{eq-3}をチェビシェフの不等式と呼ぶ。

この不等式を用いることで、確率変数$X$の実現値と期待値$\mu$の差が標準偏差$\sigma$の$a$倍以上になる確率の上限を、確率分布に依らず期待値$\mu$と分散$\sigma^2$から求めることができる。

証明は、マルコフの不等式に対して$X = (X-\mu)^2$と$a=a^2\sigma^2$とおいた時、

\\[
Pr[(X-\mu)^2 \geq a^2\sigma^2] \leq \frac{E[(X-\mu)^2]}{a^2\sigma^2} = \frac{\sigma^2}{a^2\sigma^2} = \frac{1}{a^2} \\\\ Pr[\mid X-\mu \mid \geq a\sigma] \leq \frac{1}{a^2}
\\]

となる。なお、少し工夫して$a=a^2$とすると

\\[
Pr[(X-\mu)^2 \geq a^2] \leq \frac{E[(X-\mu)^2]}{a^2} = \frac{\sigma^2}{a^2} \\\\ Pr[\mid X-\mu \mid \geq a] \leq \frac{\sigma^2}{a^2}
\\]

のように、確率変数$X$の実現値と期待値$\mu$の差を標準偏差$\sigma$を使わずに表現できる。この場合例えば「期待値が10で標準偏差が5の確率変数で100以上が出る確率は0.31%未満」（$Pr[X-10 \geq 90] \leq \frac{5^2}{90^2} = 0.00308...$）などと求めることができる。

## ヘフディングの不等式 Hoeffding’s inequality

独立な$n$個の確率変数$X[a,b]$において平均を$\overline{X}$、期待値を$E\left[\overline{X}\right]$とおいた時、任意の正数$u > 0$に対して

\\[
Pr \left[\overline{X}-E\left[\overline{X}\right] \geq u\right] \leq \exp\left(-\frac{2n^2u^2}{\sum\_{i=1}^{n}(b\_i-a\_i)^2}\right) \tag{4}\label{eq-4}
\\]

が成り立つ。\eqref{eq-4}をヘフディングの不等式と呼ぶ。

この不等式を用いることで、n個の確率変数$X$の平均と期待値の誤差が$u$以上になる（すなわち真の期待値を過大評価している）確率の上限を、確率分布に依らずに求めることができる。

同様に真の期待値を過小評価している確率の上限は、

\\[
Pr \left[-\overline{X}+E\left[\overline{X}\right] \geq u\right] \leq \exp\left(-\frac{2n^2u^2}{\sum\_{i=1}^{n}(b\_i-a\_i)^2}\right) \tag{5}\label{eq-5}
\\]

であり、真の期待値に対して$u$以上の誤差が生じる確率の上限は\eqref{eq-4}と\eqref{eq-5}を合わせた、

\\[
Pr \left[\mid \overline{X}-E\left[\overline{X}\right] \mid \geq u\right] = Pr \left[\overline{X}-E\left[\overline{X}\right] \geq u\right] + Pr \left[-\overline{X}+E\left[\overline{X}\right] \geq u\right] \tag{6}\label{eq-6}
\\]

となる。

また、確率変数$X[a,b]$で$a=0,b=1$の時、\eqref{eq-4}より

\\[
Pr \left[\overline{X}-E\left[\overline{X}\right] \geq u\right] \leq \exp\left(-\frac{2n^2u^2}{n}\right) = \exp\left(-2nu^2\right)\tag{7}\label{eq-7}
\\]

同じく\eqref{eq-5}より

\\[
Pr \left[-\overline{X}+E\left[\overline{X}\right] \geq u\right] \leq \exp\left(-\frac{2n^2u^2}{n}\right) = \exp\left(-2nu^2\right)\tag{8}\label{eq-8}
\\]

である。証明は論文に譲る。

# UCB1方策

UCB1方策では、ヘフディングの不等式\eqref{eq-8}を用いて、腕$j$の報酬の期待値の信頼区間の上側の上限である\eqref{eq-1}を求める。
\eqref{eq-1}を再掲する。

\\[
\overline{x}\_j+\sqrt{\frac{2\ln{n}}{n\_j}} \tag{1}
\\]

腕$j$の報酬の期待値を$x^\*\_j$とすると\eqref{eq-8}は

\\[
Pr \left[x^\*\_j \geq \overline{x}\_j + u\right] \leq \exp\left(-2n\_ju^2\right)\tag{9}\label{eq-9}
\\]

\eqref{eq-9}は真の期待値を過小評価している確率の上限であり、これを試行回数$n$が進むごとに小さくしたいと考える。そこで右辺を$n^{-m}=\frac{1}{n^m}$とすると、

\begin{align}
\exp\left(-2n\_ju^2\right) &= \frac{1}{n^m} \notag \\\\ -2n\_ju^2 &= \ln{\frac{1}{n^m}} \notag \\\\ &= \ln{1} - \ln{n^m} \notag \\\\ &= 0 - m\ln{n} \notag \\\\ 2n\_ju^2 &= m\ln{n} \notag \\\\ u^2 &= \frac{m\ln{n}}{2n\_j} \notag \\\\ u &= \sqrt{\frac{m\ln{n}}{2n\_j}} \notag
\end{align}

が得られる。理論限界より$n$回目の試行において$1/n$の確率で各腕が選択する必要があるとされており、これに従えば$m=1$だが、論文では$m=4$としている（すなわち$u = \sqrt{\frac{2\ln{n}}{n\_j}}$）。

これを\eqref{eq-9}に代入し

\\[
Pr \left[x^\*\_j \geq \overline{x}\_j + \sqrt{\frac{2\ln{n}}{n\_j}}\right] \leq \frac{1}{n^4}\tag{10}\label{eq-10}
\\]

よって$1/n^4$の上限において、\eqref{eq-1}のスコアを用いて報酬の期待値を最も低く見積もっているであろう腕を選択することができる。すなわち`Optimism in face of uncertainty`に従う。

# 参考

- [マルコフの不等式](https://ja.wikipedia.org/wiki/%E3%83%9E%E3%83%AB%E3%82%B3%E3%83%95%E3%81%AE%E4%B8%8D%E7%AD%89%E5%BC%8F)
- [チェビシェフの不等式](https://ja.wikipedia.org/wiki/%E3%83%81%E3%82%A7%E3%83%93%E3%82%B7%E3%82%A7%E3%83%95%E3%81%AE%E4%B8%8D%E7%AD%89%E5%BC%8F)
- [【大学数学】チェビシェフの不等式【確率統計】- 予備校のノリで学ぶ「大学の数学・物理」](https://www.youtube.com/watch?v=d-ugoDdXWrU)
- [確率論 (Probability Theory) 第 9 週](http://stat.inf.uec.ac.jp/lib/exe/fetch.php?media=prob:prob-9-note-and-quizes-20120614.pdf)
- [Hoeffding's inequality](https://en.wikipedia.org/wiki/Hoeffding%27s_inequality)
- [Hoeffding, Wassily (1963). "Probability inequalities for sums of bounded random variables". Journal of the American Statistical Association. 58 (301): 13–30.](http://repository.lib.ncsu.edu/bitstream/1840.4/2170/1/ISMS_1962_326.pdf)
- [Auer, Peter, Nicolo Cesa-Bianchi, and Paul Fischer. "Finite-time analysis of the multiarmed bandit problem." Machine learning 47.2-3 (2002): 235-256.](https://homes.di.unimi.it/~cesabian/Pubblicazioni/ml-02.pdf)
- [ヘフディングの不等式(Hoeffding's inequality)と諸々の確率の評価の不等式](https://ludu-vorton.hatenablog.com/entry/2019/06/06/073000)
- [バンディットアルゴリズムで最適な介入を見つける（基本編）](https://qiita.com/usaito/items/ad15394547bd5daf8937)
- [バンディットアルゴリズムの続き](https://research.miidas.jp/2020/02/%E3%83%90%E3%83%B3%E3%83%87%E3%82%A3%E3%83%83%E3%83%88%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0%E3%81%AE%E7%B6%9A%E3%81%8D/)
- [強化学習その1](https://www.slideshare.net/nishio/1-70974083)
- [バンディット問題の理論とアルゴリズム (機械学習プロフェッショナルシリーズ)](https://www.amazon.co.jp/dp/406152917X/)
