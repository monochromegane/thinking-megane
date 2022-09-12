+++
date = "2022-09-11T12:00:00+09:00"
title = "カーネルリッジ回帰 入門"
tags = [ "regression" ]
+++

基底関数を用いた線形回帰モデルのように入力に非線形な変換を施してモデルの表現力の向上を図る手法では、以下の2つの問題が発生する。
1. 特徴ベクトルの次元数の増加に伴う計算量の増加
1. 予測に有用な基底関数の選定

カーネルリッジ回帰は、カーネル関数を用いて上記の課題を解決する。
このエントリは、このカーネルリッジ回帰を最初に学んだ際の内容を自分なりにまとめたものである。

以下では、はじめに、記法の整理を兼ねて基本的な線形回帰モデルを説明する。
次に、カーネル法を用いた線形回帰であるカーネルリッジ回帰の説明を通して上記の課題の解決アプローチを学ぶ。
最後に、カーネル法における計算量の課題を解決するためのアプローチである、Random Fourier Featuresも紹介する。

## 1. 線形回帰モデル

線形回帰モデルの問題設定と解法について述べる。
いま、$N$個の入力$X=(\boldsymbol{x}\_1,\ldots,\boldsymbol{x}\_N)^{\top} \in \mathbb{R}^{N \times D}$と出力$\boldsymbol{y}=(y\_1,\ldots,y\_N)^{\top} \in \mathbb{R}^{N}$が与えられている。
このとき、$y$は$\boldsymbol{x}$を入力とした線形回帰モデル$\hat{y} = \boldsymbol{w}^{\top}\boldsymbol{\phi}(\boldsymbol{x})$の出力として得られると仮定する。
ここで、$\boldsymbol{\phi}(\boldsymbol{x}) = (\phi\_1(\boldsymbol{x}),\ldots,\phi\_F(\boldsymbol{x}))^{\top} \in \mathbb{R}^{F}$は$F$個の基底関数$\phi: \mathbb{R}^{D} \rightarrow \mathbb{R}$からなる特徴ベクトル、$\boldsymbol{w} \in \mathbb{R}^{F}$はこれに対応する係数ベクトルである。
このとき、この係数ベクトル$\boldsymbol{w}$を推定することが本問題の目標である。

推定には最小二乗法を用いる。
なお、実用上は汎化性能の考慮から正則化を施すことが多いと思われるので、これを適用した**リッジ回帰**を行う。
すなわち、$\Phi = (\boldsymbol{\phi}(\boldsymbol{x}\_1),\ldots,\boldsymbol{\phi}(\boldsymbol{x}\_N))^{\top} \in \mathbb{R}^{N \times F}$としたとき、学習データに対する誤差の二乗和（に正則化項を加えたもの）$E(\boldsymbol{w}) = \\| \Phi \boldsymbol{w} - \boldsymbol{y} \\|^2 + \lambda \\| \boldsymbol{w} \\|^2$を最小化する$\boldsymbol{w}$を求める。

このために、まず$E(\boldsymbol{w})$を次のように整理する。

\begin{align}
\begin{split}
E(\boldsymbol{w}) &= (\Phi \boldsymbol{w} - \boldsymbol{y})^{\top}(\Phi \boldsymbol{w} - \boldsymbol{y}) + \lambda\boldsymbol{w}^{\top}\boldsymbol{w} \\\\
  &= ((\Phi \boldsymbol{w})^{\top} - \boldsymbol{y}^{\top})(\Phi \boldsymbol{w} - \boldsymbol{y})  + \lambda\boldsymbol{w}^{\top}\boldsymbol{w}\\\\
  &= ( \boldsymbol{w}^{\top}\Phi^{\top} - \boldsymbol{y}^{\top})(\Phi \boldsymbol{w} - \boldsymbol{y})  + \lambda\boldsymbol{w}^{\top}\boldsymbol{w}\\\\
  &= \boldsymbol{w}^{\top}\Phi^{\top}\Phi\boldsymbol{w} - \boldsymbol{w}^{\top}\Phi^{\top}\boldsymbol{y} - \boldsymbol{y}^{\top}\Phi\boldsymbol{w} + \boldsymbol{y}^{\top}\boldsymbol{y}  + \lambda\boldsymbol{w}^{\top}\boldsymbol{w}\\\\
  &= \boldsymbol{w}^{\top}\Phi^{\top}\Phi\boldsymbol{w} - \boldsymbol{w}^{\top}\Phi^{\top}\boldsymbol{y} - \boldsymbol{w}^{\top}\Phi^{\top}\boldsymbol{y} + \boldsymbol{y}^{\top}\boldsymbol{y}  + \lambda\boldsymbol{w}^{\top}\boldsymbol{w}\\\\
  &= \boldsymbol{w}^{\top}\Phi^{\top}\Phi\boldsymbol{w} - 2\boldsymbol{w}^{\top}\Phi^{\top}\boldsymbol{y} + \boldsymbol{y}^{\top}\boldsymbol{y}  + \lambda\boldsymbol{w}^{\top}\boldsymbol{w}.
\label{eq:ew}
\end{split}
\end{align}

整理には、行列の定理$(AB)^{\top}=B^{\top}A^{\top}$と、スカラーは転置しても値が同じであることを利用した（$\boldsymbol{y}^{\top}\Phi\boldsymbol{w} = (\boldsymbol{y}^{\top}\Phi\boldsymbol{w})^{\top} = \boldsymbol{w}^{\top}\Phi^{\top}\boldsymbol{y}$）。

次に、$E(\boldsymbol{w})$を最小化する$\boldsymbol{w}$を求めるために$\boldsymbol{w}$について偏微分して0とおく。

\begin{align}
\frac{\partial E(\boldsymbol{w})}{\partial \boldsymbol{w}} &= 2\Phi^{\top}\Phi\boldsymbol{w} -2\Phi^{\top}\boldsymbol{y} + 0 + 2\lambda\boldsymbol{w} = 0.
\end{align}

偏微分には、対称行列（$A=\Phi^{\top}\Phi=A^{\top}$）に対する二次形式（$\boldsymbol{w}^{\top}A\boldsymbol{w}$）の$\boldsymbol{w}$の偏微分が$(A + A^{\top})\boldsymbol{w} = 2A\boldsymbol{w}$であること、$\boldsymbol{w}^{\top}\boldsymbol{a}, \boldsymbol{a}=\Phi^{\top}\boldsymbol{y}$としたときの$\boldsymbol{w}$の偏微分が$\boldsymbol{a}$であることを利用した。

最後に、$\boldsymbol{w}$について整理する。

\begin{align}
\begin{split}
2\Phi^{\top}\Phi\boldsymbol{w} -2\Phi^{\top}\boldsymbol{y} + 2\lambda\boldsymbol{w} &= 0\\\\
2\Phi^{\top}\Phi\boldsymbol{w} + 2\lambda\boldsymbol{w} &= 2\Phi^{\top}\boldsymbol{y}\\\\
\Phi^{\top}\Phi\boldsymbol{w} + \lambda\boldsymbol{w} &= \Phi^{\top}\boldsymbol{y}\\\\
(\Phi^{\top}\Phi + \lambda I\_F)\boldsymbol{w} &= \Phi^{\top}\boldsymbol{y}\\\\
\boldsymbol{w} &= (\Phi^{\top}\Phi + \lambda I\_F)^{-1}\Phi^{\top}\boldsymbol{y}.
\label{eq:w}
\end{split}
\end{align}

ここでは、行列のサイズが$F \times F$の単位行列を$I_{F}$と表記した。

新しい入力$\boldsymbol{x}$に対する出力は推定した$\boldsymbol{w}$を用いて、$\hat{y} = \boldsymbol{w}^{\top}\boldsymbol{\phi}(\boldsymbol{x})$と予測できる。

## 2. カーネル法による線形回帰モデル

上述の線形回帰モデルを利用する際には、基底関数を増やし多くの特徴量の候補から予測に有用なものに重み付けできれば良いと考えられることから、特徴ベクトルの次元数$F$を増やすアプローチが検討される。
しかしながら、この場合、パラメータ$\boldsymbol{w}$の推定や出力$\hat{y}$の予測に必要な計算量も同時に増加してしまう。
そこで、カーネル法による線形回帰モデルでは、パラメータの次元数を$N$に抑えるようなアプローチをとる。
これは、学習データ数$N$が特徴ベクトルの次元数$F$よりも少ない場合に有効である。
また、線形回帰モデルにはもう一つ、予測に対して有用な基底関数の種類や数が明らかではないという課題がある。
これに対しカーネル法による線形回帰モデルは、カーネル関数を導入することで、基底関数の選定を省略することができる。

以下、上述のリッジ回帰にカーネル法を適用した**カーネルリッジ回帰**を説明する。

### 2.1. 線形回帰モデルの双対表現

パラメータの次元数を$N$に抑えるため、$F$次元の$\boldsymbol{w}$についての線形モデル$\hat{y} = \boldsymbol{w}^{\top}\boldsymbol{\phi}(\boldsymbol{x})$を、$N$個の学習データ$\Phi$に対応するパラメータ$\boldsymbol{\alpha} \in \mathbb{R}^{N}$で表現することを考える。

そのために、式\eqref{eq:w}の3行目以降の変形を以下のように進める。

\begin{align}
\begin{split}
\Phi^{\top}\Phi\boldsymbol{w} + \lambda\boldsymbol{w} &= \Phi^{\top}\boldsymbol{y}\\\\
\lambda\boldsymbol{w} &= - \Phi^{\top}\Phi\boldsymbol{w} + \Phi^{\top}\boldsymbol{y}\\\\
\lambda\boldsymbol{w} &= - \Phi^{\top}(\Phi\boldsymbol{w} - \boldsymbol{y})\\\\
\boldsymbol{w} &= - \frac{1}{\lambda} \Phi^{\top}(\Phi\boldsymbol{w} - \boldsymbol{y}).
\end{split}
\end{align}

ここで$\boldsymbol{\alpha} = - \frac{1}{\lambda}(\Phi\boldsymbol{w} - \boldsymbol{y})$とすると、$\boldsymbol{w} = \Phi^{\top}\boldsymbol{\alpha}$となる。
これにより、元の線形回帰モデルを$\hat{y} = (\Phi^{\top}\boldsymbol{\alpha})^{\top}\phi(\boldsymbol{x})$と、（右辺にも$\boldsymbol{w}$は含まれるが）見かけ上は$\boldsymbol{w}$を含まない形で表現できるようになった（**双対表現**）。

この双対表現の線形回帰モデルについて、最小二乗法を用いて$\boldsymbol{\alpha}$を推定する。
これは上述の$E(\boldsymbol{w})$を双対表現の線形回帰モデルで記述し、解を求めることと同等である。

このために、式\eqref{eq:ew}に$\boldsymbol{w} = \Phi^{\top}\boldsymbol{\alpha}$を代入し$\boldsymbol{\alpha}$についての関数$E(\boldsymbol{\alpha})$とした上で、以下のように整理する。

\begin{align}
\begin{split}
E(\boldsymbol{\alpha}) &= (\Phi^{\top}\boldsymbol{\alpha})^{\top}\Phi^{\top}\Phi(\Phi^{\top}\boldsymbol{\alpha}) - 2(\Phi^{\top}\boldsymbol{\alpha})^{\top}\Phi^{\top}\boldsymbol{y} + \boldsymbol{y}^{\top}\boldsymbol{y}  + \lambda(\Phi^{\top}\boldsymbol{\alpha})^{\top}(\Phi^{\top}\boldsymbol{\alpha})\\\\
                  &= \boldsymbol{\alpha}^{\top}\Phi\Phi^{\top}\Phi\Phi^{\top}\boldsymbol{\alpha} - 2\boldsymbol{\alpha}^{\top}\Phi\Phi^{\top}\boldsymbol{y} + \boldsymbol{y}^{\top}\boldsymbol{y}  + \lambda\boldsymbol{\alpha}^{\top}\Phi\Phi^{\top}\boldsymbol{\alpha}\\\\
                  &= \boldsymbol{\alpha}^{\top}KK\boldsymbol{\alpha} - 2\boldsymbol{\alpha}^{\top}K\boldsymbol{y} + \boldsymbol{y}^{\top}\boldsymbol{y}  + \lambda\boldsymbol{\alpha}^{\top}K\boldsymbol{\alpha}.
\end{split}
\end{align}

なお、
\\[
K = \Phi\Phi^{\top} =
\begin{pmatrix}
\boldsymbol{\phi}(\boldsymbol{x}\_1)^{\top}\boldsymbol{\phi}(\boldsymbol{x}\_1) & \ldots & \boldsymbol{\phi}(\boldsymbol{x}\_1)^{\top}\boldsymbol{\phi}(\boldsymbol{x}\_N) \\\\
\vdots & \ddots & \vdots \\\\
\boldsymbol{\phi}(\boldsymbol{x}\_N)^{\top}\boldsymbol{\phi}(\boldsymbol{x}\_1) & \ldots & \boldsymbol{\phi}(\boldsymbol{x}\_N)^{\top}\boldsymbol{\phi}(\boldsymbol{x}\_N)
\end{pmatrix} =
\begin{pmatrix}
k(\boldsymbol{x}\_1,\boldsymbol{x}\_1) & \ldots & k(\boldsymbol{x}\_1,\boldsymbol{x}\_N) \\\\
\vdots & \ddots & \vdots \\\\
k(\boldsymbol{x}\_N,\boldsymbol{x}\_1) & \ldots & k(\boldsymbol{x}\_N,\boldsymbol{x}\_N)
\end{pmatrix}
\in \mathbb{R}^{N \times N}
\\]
とした。
ここで、入力$\boldsymbol{p}$と$\boldsymbol{q}$を$\boldsymbol{\phi}$によって特徴ベクトルに変換し内積をとる操作である$k(\boldsymbol{p},\boldsymbol{q})$をカーネル関数と呼ぶ。
また、学習データ$X$に対する全ての組み合わせである$K$をグラム行列と呼ぶ。

次に、$E(\boldsymbol{\alpha})$を最小化する$\boldsymbol{\alpha}$を求めるために$\boldsymbol{\alpha}$について偏微分して0とおく。

\begin{align}
\begin{split}
\frac{\partial E(\boldsymbol{\alpha})}{\partial \boldsymbol{\alpha}} &= 2KK\boldsymbol{\alpha} - 2K\boldsymbol{y} + 0 + 2\lambda K\boldsymbol{\alpha}= 0.
\end{split}
\end{align}

最後に、$\boldsymbol{\alpha}$について整理する。

\begin{align}
\begin{split}
2KK\boldsymbol{\alpha} - 2K\boldsymbol{y} + 2\lambda K\boldsymbol{\alpha} &= 0\\\\
KK\boldsymbol{\alpha} - K\boldsymbol{y} + \lambda K\boldsymbol{\alpha} &= 0\\\\
KK\boldsymbol{\alpha} + \lambda K\boldsymbol{\alpha} &= K\boldsymbol{y}\\\\
K\boldsymbol{\alpha} + \lambda \boldsymbol{\alpha} &= \boldsymbol{y}\\\\
(K + \lambda I\_N)\boldsymbol{\alpha} &= \boldsymbol{y}\\\\
\boldsymbol{\alpha} &= (K + \lambda I\_N)^{-1}\boldsymbol{y}.
\label{eq:alpha}
\end{split}
\end{align}

新しい入力$\boldsymbol{x}$に対する出力は、推定した$\boldsymbol{\alpha}$を用いて以下のように予測できる。

\begin{align}
\begin{split}
\hat{y} &= (\Phi^{\top}\boldsymbol{\alpha})^{\top}\boldsymbol{\phi}(\boldsymbol{x}) \\\\
        &= \boldsymbol{\alpha}^{\top}\Phi\boldsymbol{\phi}(\boldsymbol{x}) \\\\
        &= \sum_{n=1}^{N} \alpha\_n \boldsymbol{\phi}(\boldsymbol{x}\_n)^{\top}\boldsymbol{\phi}(\boldsymbol{x}) \\\\
        &= \sum_{n=1}^{N} \alpha\_n k(\boldsymbol{x}\_n, \boldsymbol{x}).
\label{eq:yk}
\end{split}
\end{align}

すなわち、$F$次元の$\boldsymbol{w}$を用いず、学習データ数$N$を次元とする$\boldsymbol{\alpha}$で予測できるようになった。

### 2.2. カーネル関数の構成

ここまで、$F$個の基底関数$\phi$を用いた特徴ベクトルとしての$\boldsymbol{\phi}$同士の内積として、カーネル関数をボトムアップ的に定義した。

この場合、どのような基底関数を選定するかという線形回帰モデルのもう一つの課題が残る。
カーネル法を用いた線形回帰モデルでは、発想を逆転させ、カーネル関数をトップダウン的に先に直接定義することで、その内部の基底関数を陽に知らずにすませるというアプローチをとる。

ただし、この場合は定義したカーネル関数が正定値性を満たす必要がある。
すなわち、任意の$M$個の点から計算される、定義したカーネル関数による$M \times M$のグラム行列$K$の2次形式が常に非負（任意の$\boldsymbol{\mu} \in \mathbb{R}^M$に対して$\boldsymbol{\mu}^{\top}K\boldsymbol{\mu} \geq 0$）となる必要がある。

このような条件を満たすカーネル関数を直接定義できたならば、これはなんらかの特徴ベクトル同士の内積と考えることができる。
ありがたいことに、式\eqref{eq:yk}は基底関数$\phi$を使わず全てカーネル関数$k$で表現できているため、直接定義したカーネル関数があれば、基底関数の選定が不要となる（**カーネルトリック**）。

幸い、さまざまなカーネル関数が考案されているため、まずはそれらのカーネル関数を利用することとなる。
以下、多項式カーネルとガウスカーネルを紹介する。

#### 多項式カーネル

多項式カーネルは$k(\boldsymbol{p}, \boldsymbol{q}) = (\boldsymbol{p}^{\top}\boldsymbol{q} + c)^{m}$の形をとるカーネル関数である。
上述のカーネルリッジ回帰を利用するにあたってはどのような特徴ベクトルが利用されているのかを陽に知る必要はないが、例えば$m=2,\boldsymbol{p},\boldsymbol{q} \in \mathbb{R}^2$であれば次のような特徴ベクトルが対応することが分かる。

対応する特徴ベクトルを調べるために、多項式カーネルを特徴ベクトルの内積$\boldsymbol{\phi}(\boldsymbol{p})^{\top}\boldsymbol{\phi}(\boldsymbol{q})$に変形する。

\begin{align}
\begin{split}
k(\boldsymbol{p}, \boldsymbol{q}) &= (\boldsymbol{p}^{\top}\boldsymbol{q} + c)^{2} \\\\
  &= ((p\_1, p\_2)^{\top}(q\_1, q\_2) + c)^2 \\\\
  &= (p\_1q\_1 + p\_2q\_2 + c)^2 \\\\
  &= p\_1^2q\_1^2 + p\_2^2q\_2^2 + c^2 + 2p\_1q\_1p\_2q\_2 + 2cp\_2q\_2 + 2cp\_1q\_1\\\\
  &= p\_1^2q\_1^2 + p\_2^2q\_2^2 + c^2 + 2p\_1p\_2q\_1q\_2 + 2cp\_2q\_2 + 2cp\_1q\_1\\\\
  &= p\_1^2q\_1^2 + p\_2^2q\_2^2 + c^2 + \sqrt{2}p\_1p\_2\sqrt{2}q\_1q\_2 + \sqrt{2c}p\_2\sqrt{2c}q\_2 + \sqrt{2c}p\_1\sqrt{2c}q\_1\\\\
  &= (p\_1^2, p\_2^2, c, \sqrt{2}p\_1p\_2, \sqrt{2c}p\_2, \sqrt{2c}p\_1)^{\top}(q\_1^2, q\_2^2, c, \sqrt{2}q\_1q\_2, \sqrt{2c}q\_2, \sqrt{2c}q\_1).
\end{split}
\end{align}

つまり、今回の例では、特徴ベクトルが$\boldsymbol{\phi}(\boldsymbol{x}) = (x\_1^2, x\_2^2, c, \sqrt{2}x\_1x\_2, \sqrt{2c}x\_2, \sqrt{2c}x\_1)$として扱われていたことが分かる。
言葉を返せば、多項式カーネルを用いると、このような変換を行う6つの基底関数を明示的に選定して地道に内積を取った場合と同様の結果を得られる。

なお、多項式カーネルの$m=1,c=0$の場合、これを線形カーネルと呼び、この時の特徴ベクトルは入力と等しい（$\boldsymbol{\phi}(\boldsymbol{x}) = \boldsymbol{x}$）。

#### ガウスカーネル

ガウスカーネルは$k(\boldsymbol{p}, \boldsymbol{q}) = \exp(-\frac{\\| \boldsymbol{p} - \boldsymbol{q}\\|^2}{2\sigma^2})$の形をとるカーネル関数である。

以下では、ガウスカーネルにどのような特徴ベクトルが対応するかを確認する。
まず、ガウスカーネルを以下のように展開、変形する。

\begin{align}
\begin{split}
k(\boldsymbol{p}, \boldsymbol{q}) &= \exp(-\frac{\\| \boldsymbol{p} - \boldsymbol{q}\\|^2}{2\sigma^2}) \\\\
  &= \exp(-\frac{(\boldsymbol{p} - \boldsymbol{q})^{\top}(\boldsymbol{p} - \boldsymbol{q})}{2\sigma^2}) \\\\
  &= \exp(-\frac{(\boldsymbol{p}^{\top} - \boldsymbol{q}^{\top})(\boldsymbol{p} - \boldsymbol{q})}{2\sigma^2}) \\\\
  &= \exp(-\frac{\boldsymbol{p}^{\top}\boldsymbol{p} - \boldsymbol{q}^{\top}\boldsymbol{p} - \boldsymbol{p}^{\top}\boldsymbol{q} + \boldsymbol{q}^{\top}\boldsymbol{q}}{2\sigma^2}) \\\\
  &= \exp(-\frac{\boldsymbol{p}^{\top}\boldsymbol{p} - (\boldsymbol{q}^{\top}\boldsymbol{p})^{\top} - \boldsymbol{p}^{\top}\boldsymbol{q} + \boldsymbol{q}^{\top}\boldsymbol{q}}{2\sigma^2}) \\\\
  &= \exp(-\frac{\boldsymbol{p}^{\top}\boldsymbol{p} - \boldsymbol{p}^{\top}\boldsymbol{q} - \boldsymbol{p}^{\top}\boldsymbol{q} + \boldsymbol{q}^{\top}\boldsymbol{q}}{2\sigma^2}) \\\\
  &= \exp(-\frac{\boldsymbol{p}^{\top}\boldsymbol{p} - 2\boldsymbol{p}^{\top}\boldsymbol{q} + \boldsymbol{q}^{\top}\boldsymbol{q}}{2\sigma^2}) \\\\
  &= \exp(\frac{-\boldsymbol{p}^{\top}\boldsymbol{p}}{2\sigma^2} + \frac{\boldsymbol{p}^{\top}\boldsymbol{q}}{\sigma^2} + \frac{-\boldsymbol{q}^{\top}\boldsymbol{q}}{2\sigma^2}) \\\\
  &= \exp(\frac{-\boldsymbol{p}^{\top}\boldsymbol{p}}{2\sigma^2})\exp(\frac{\boldsymbol{p}^{\top}\boldsymbol{q}}{\sigma^2})\exp(\frac{-\boldsymbol{q}^{\top}\boldsymbol{q}}{2\sigma^2}) \\\\
  &= \rho\exp(\frac{\boldsymbol{p}^{\top}\boldsymbol{q}}{\sigma^2})\psi \\\\
  &= \rho\psi\exp(\frac{\boldsymbol{p}^{\top}\boldsymbol{q}}{\sigma^2}).
\end{split}
\end{align}

最後の2行では以降の展開を見やすくするため、$\rho=\exp(\frac{-\boldsymbol{p}^{\top}\boldsymbol{p}}{2\sigma^2}), \psi=\exp(\frac{-\boldsymbol{q}^{\top}\boldsymbol{q}}{2\sigma^2})$とおいた。

次に、$e^x$のマクローリン展開の公式（$e^x = \sum\_{i=0}^{\infty} \frac{x^i}{i!} = 1 + x + \frac{x^2}{2} + \frac{x^3}{6} + \cdots$）を使い、特徴ベクトルの内積$\boldsymbol{\phi}(\boldsymbol{p})^{\top}\boldsymbol{\phi}(\boldsymbol{q})$に変形する。

簡単のため$\sigma=1$とすると、次のような式が得られる。

\begin{align}
\begin{split}
\rho\psi\exp(\boldsymbol{p}^{\top}\boldsymbol{q}) &= \rho\psi \left(1 + (\boldsymbol{p}^{\top}\boldsymbol{q})^1 + \frac{(\boldsymbol{p}^{\top}\boldsymbol{q})^2}{2} + \frac{(\boldsymbol{p}^{\top}\boldsymbol{q})^3}{6} + \ldots \right).
\end{split}
\end{align}

これは$c=0$の多項式カーネルを次数$m$を増やしながら無限回足したものである。
よって例えば、$\boldsymbol{p},\boldsymbol{q} \in \mathbb{R}^2$では、次のような**無限次元**の特徴ベクトルが対応することが分かる。

\begin{align}
\begin{split}
\rho\psi\exp(\boldsymbol{p}^{\top}\boldsymbol{q}) &= \rho\psi \left(1 + (\boldsymbol{p}^{\top}\boldsymbol{q})^1 + \frac{(\boldsymbol{p}^{\top}\boldsymbol{q})^2}{2} + \frac{(\boldsymbol{p}^{\top}\boldsymbol{q})^3}{6} + \ldots \right) \\\\
  &= \rho\psi \left(1 + (p\_1q\_1 + p\_2q\_2) + \frac{p\_1^2q\_1^2 + p\_2^2q\_2^2 + \sqrt{2}p\_1p\_2\sqrt{2}q\_1q\_2}{2} + \frac{p\_1^3q\_1^3 + p\_2^3q\_2^3 + \sqrt{3}p\_1^2p\_2\sqrt{3}q\_1^2q\_2 + \sqrt{3}p\_1p\_2^2\sqrt{3}q\_1q\_2^2}{6} + \ldots \right) \\\\
  &= \rho\left(1, p\_1, p\_2, \frac{p\_1^2}{\sqrt{2}}, \frac{p\_2^2}{\sqrt{2}}, \frac{\sqrt{2}p\_1p\_2}{\sqrt{2}}, \frac{p\_1^3}{\sqrt{6}}, \frac{p\_2^3}{\sqrt{6}}, \frac{\sqrt{3}p\_1^2p\_2}{\sqrt{6}}, \frac{\sqrt{3}p\_1p\_2^2}{\sqrt{6}}, \ldots \right)^{\top}\psi\left(1, q\_1, q\_2, \frac{q\_1^2}{\sqrt{2}}, \frac{q\_2^2}{\sqrt{2}}, \frac{\sqrt{2}q\_1q\_2}{\sqrt{2}}, \frac{q\_1^3}{\sqrt{6}}, \frac{q\_2^3}{\sqrt{6}}, \frac{\sqrt{3}q\_1^2q\_2}{\sqrt{6}}, \frac{\sqrt{3}q\_1q\_2^2}{\sqrt{6}},\ldots \right) \\\\
  &= \rho\left(1, p\_1, p\_2, \frac{p\_1^2}{\sqrt{2}}, \frac{p\_2^2}{\sqrt{2}}, p\_1p\_2, \frac{p\_1^3}{\sqrt{6}}, \frac{p\_2^3}{\sqrt{6}}, \frac{p\_1^2p\_2}{\sqrt{2}}, \frac{p\_1p\_2^2}{\sqrt{2}}, \ldots \right)^{\top}\psi\left(1, q\_1, q\_2, \frac{q\_1^2}{\sqrt{2}}, \frac{q\_2^2}{\sqrt{2}}, q\_1q\_2, \frac{q\_1^3}{\sqrt{6}}, \frac{q\_2^3}{\sqrt{6}}, \frac{q\_1^2q\_2}{\sqrt{2}}, \frac{q\_1q\_2^2}{\sqrt{2}},\ldots \right).
\end{split}
\end{align}

ガウスカーネルでは、このような無限個の基底関数を用意して内積を地道に計算した結果が、$k(\boldsymbol{p}, \boldsymbol{q}) = \exp(-\frac{\\| \boldsymbol{p} - \boldsymbol{q}\\|^2}{2\sigma^2})$のみの計算から得られるとも考えられる。

## 3. Random Fourier Features（乱択化フーリエ特徴）

カーネルリッジ回帰による入力$\boldsymbol{x}$に対する予測は式\eqref{eq:alpha}\eqref{eq:yk}より、$\hat{y} = \sum_{n=1}^{N} \alpha\_n k(\boldsymbol{x}\_n, \boldsymbol{x}), \boldsymbol{\alpha} = (K + \lambda I\_N)^{-1}\boldsymbol{y}$となるのであった。
カーネルリッジ回帰では、$F \gg N$となるような学習データ数$N$に対し十分大きい$F$個の基底関数を用いる場合に、双対表現によってパラメータ数を$N$に抑えつつ、カーネル関数の導入によって$F$次元の特徴ベクトルの直接的な算出を回避できた。

しかしながら、カーネルリッジ回帰は、学習データ数$N$に対して計算量が指数的に増加してしまう課題がある。
これは、グラム行列$K$のサイズが$N \times N$であるため、$\boldsymbol{\alpha}$の計算にあたって、必要なカーネル関数の計算が$N^2$のオーダーで増加することからも分かる。
また、逆行列の計算も$K$のサイズに応じて計算量が増加する。
加えて、$\hat{y}$の計算にあたっても、新しい入力$\boldsymbol{x}$に対して$N$回のカーネル関数の計算が都度必要となってしまう。

**Random Fourier Features**は、ある確率分布からのサンプリング結果でカーネル関数とグラム行列を近似することで、上述の課題を解決する、高速化のための手法である。

この手法では、$\mathbb{R}^D$上のカーネル関数$k(\boldsymbol{p},\boldsymbol{q})$が$\boldsymbol{p}$と$\boldsymbol{q}$の差の形（$k(\boldsymbol{p}-\boldsymbol{q})$）で表せるとき、このカーネル関数が、ある確率密度関数$p(\boldsymbol{\omega})$のフーリエ変換で表せることを利用する。

\begin{align}
\begin{split}
k(\boldsymbol{p},\boldsymbol{q}) &= k(\boldsymbol{p} - \boldsymbol{q})\\\\
  &= \int_{\mathbb{R}^{D}} p(\boldsymbol{\omega}) \exp(i \boldsymbol{\omega}^{\top}(\boldsymbol{p} - \boldsymbol{q}))d\boldsymbol{\omega} \\\\
  &= \mathbb{E}\_{\boldsymbol{\omega}}[\exp(i \boldsymbol{\omega}^{\top}(\boldsymbol{p} - \boldsymbol{q}))] \\\\
  &= \mathbb{E}\_{\boldsymbol{\omega}}[\cos(\boldsymbol{\omega}^{\top}(\boldsymbol{p} - \boldsymbol{q}))] \\\\
  &= \mathbb{E}\_{\boldsymbol{\omega},b}[\sqrt{2}\cos(\boldsymbol{\omega}^{\top}\boldsymbol{p} + b) \cdot \sqrt{2}\cos(\boldsymbol{\omega}^{\top}\boldsymbol{q} + b)] \\\\
  &\approx \frac{1}{R} \sum_{r=1}^{R} \sqrt{2}\cos(\boldsymbol{\omega}\_r^{\top}\boldsymbol{p} + b\_r) \cdot \sqrt{2}\cos(\boldsymbol{\omega}\_r^{\top}\boldsymbol{q} + b\_r).
\end{split}
\end{align}

ここで、$\boldsymbol{\omega} \sim p(\boldsymbol{\omega}), b \sim \text{Uniform}(0, 2\pi)$である。
なお、$p(\boldsymbol{\omega})$は利用するカーネル関数によって異なるが、ガウスカーネルの場合、$\boldsymbol{w} = (w\_d)^{1 \leq d \leq D}, w\_d \sim \mathcal{N}(0, \sigma^2)$となる。

最後の行は、この確率分布の期待値（$=$カーネル関数の結果）を$R$個のサンプリング結果の平均で近似することを示している。

すなわち、各サンプルから求めた結果を、$z\_r(\boldsymbol{x}) = \cos(\boldsymbol{\omega}\_{r}^{\top}\boldsymbol{x} + b\_r)$、$R$個の$z\_r$を$\boldsymbol{z}(\boldsymbol{x}) = \sqrt{\frac{2}{R}}(z\_1(\boldsymbol{x}),\ldots,z\_R(\boldsymbol{x}))^{\top}$とおくと、カーネル関数$k$の近似$\hat{k}$が$\hat{k}(\boldsymbol{p},\boldsymbol{q}) = \boldsymbol{z}(\boldsymbol{p})^{\top}\boldsymbol{z}(\boldsymbol{q})$のように得られる。
また、学習データ$X$に対する$\boldsymbol{z}$の集合を$Z = (\boldsymbol{z}(\boldsymbol{x}\_1)^{\top}, \ldots, \boldsymbol{z}(\boldsymbol{x}\_N)^{\top})^{\top} \in \mathbb{R}^{N \times R}$とすると、$\hat{K} = ZZ^{\top}$の操作によって$N \times N$行列の各要素がカーネル関数$k$の近似$\hat{k}(\boldsymbol{x}\_i, \boldsymbol{x}\_j) = \boldsymbol{z}(\boldsymbol{x}\_i)^{\top}\boldsymbol{z}(\boldsymbol{x}\_j)$となるグラム行列$K$の近似$\hat{K}$を得られる。


よって、これらの近似を用いたカーネルリッジ回帰のパラメータ$\boldsymbol{\alpha}$の近似$\hat{\boldsymbol{\alpha}}$は、$\hat{\boldsymbol{\alpha}} = (\hat{K} + \lambda I\_{N})^{-1}\boldsymbol{y}$となる。
ただし、この手法では、$\hat{K} = ZZ^{\top}$であることを利用して、計算量を学習データ数$N$ではなくサンプリング数$R$のオーダーに変えることができる。
そのため、$R \ll N$であるならば、非常に効率よく予測が可能となる。

以下に、$\hat{K}$や$\hat{\boldsymbol{\alpha}}$を消去していく経過と合わせて、この手法における予測式を示す。

\begin{align}
\begin{split}
\hat{y} &= \sum_{n=1}^{N} \hat{\alpha}\_n \hat{k}(\boldsymbol{x}\_n, \boldsymbol{x}) \\\\
  &= \sum_{n=1}^{N} \hat{\alpha}\_n \hat{k}(\boldsymbol{x}, \boldsymbol{x}\_n) \\\\
  &= \sum_{n=1}^{N} \hat{\alpha}\_n \boldsymbol{z}(\boldsymbol{x})^{\top}\boldsymbol{z}(\boldsymbol{x}\_n) \\\\
  &= \boldsymbol{z}(\boldsymbol{x})^{\top} \sum_{n=1}^{N} \hat{\alpha}\_n \boldsymbol{z}(\boldsymbol{x}\_n) \\\\
  &= \boldsymbol{z}(\boldsymbol{x})^{\top} Z^{\top} \hat{\boldsymbol{\alpha}} \\\\
  &= \boldsymbol{z}(\boldsymbol{x})^{\top} Z^{\top} (\hat{K} + \lambda I\_{N})^{-1}\boldsymbol{y} \\\\
  &= \boldsymbol{z}(\boldsymbol{x})^{\top} Z^{\top} (ZZ^{\top} + \lambda I\_{N})^{-1}\boldsymbol{y} \\\\
  &= \boldsymbol{z}(\boldsymbol{x})^{\top} (Z^{\top}Z + \lambda I\_{R})^{-1}Z^{\top} \boldsymbol{y}.
\end{split}
\end{align}

よって、$A=(Z^{\top}Z + \lambda I\_{R}) \in \mathbb{R}^{R \times R}$、$\boldsymbol{b} = (Z^{\top} \boldsymbol{y}) \in \mathbb{R}^{R}$、$\boldsymbol{\beta} = A^{-1}\boldsymbol{b}$として、$\hat{y} = \boldsymbol{z}(\boldsymbol{x})^{\top} \boldsymbol{\beta}$と予測できる。
$Z$は$N \times R$であるものの、逆行列のサイズは$R \times R$となること、$\hat{y}$の計算も、新しい入力$\boldsymbol{x}$に対して$R$回の計算で済む$\boldsymbol{z}(\boldsymbol{x})$のみで良いことから、計算量が抑えられることが分かる。

なお、最後の行では逆行列の補題（$P(I\_N + QP)^{-1} = (I\_R +PQ)^{-1}P, P \in \mathbb{R}^{R \times N}, Q \in \mathbb{R}^{N \times R}$）を用いた。

## 参考

### 線形回帰モデル

- [正規方程式の導出と計算例](https://manabitimes.jp/math/1128)
- [スタンフォード　ベクトル・行列からはじめる最適化数学](https://www.kspub.co.jp/book/detail/5161968.html)
- [ガウス過程と機械学習](https://www.kspub.co.jp/book/detail/1529267.html)

### カーネル法による線形回帰モデル

- [線形な手法とカーネル法（回帰分析）](https://qiita.com/wsuzume/items/09a59036c8944fd563ff)
- [カーネルトリック](https://qiita.com/kilometer/items/66e6116cc661019ead59)
- [PRML第６章「カーネル法」](https://www.slideshare.net/KeisukeSugawara/slide0629)
- [PRML 6.1章 カーネル法と双対表現](https://www.slideshare.net/hagino_3000/prml-61-17081123)
- [カーネル多変量解析 非線形データ解析の新しい展開](https://www.iwanami.co.jp/book/b257891.html)
- [Understanding the Kernel Trick with fundamentals](https://towardsdatascience.com/truly-understanding-the-kernel-trick-1aeb11560769)
- [Radial basis function kernel](https://en.wikipedia.org/wiki/Radial_basis_function_kernel)

### Random Fourier Features

- [乱択化フーリエ特徴を用いたリッジ回帰](https://qiita.com/githug0906/items/448daec79fac2ffd82a0)
- [機械学習のためのカーネル100問 with Python](https://www.kyoritsu-pub.co.jp/book/b10003381.html)
- [Random Fourier Features](https://gregorygundersen.com/blog/2019/12/23/random-fourier-features/)
- [Rahimi, Ali, and Benjamin Recht. "Random features for large-scale kernel machines." Advances in neural information processing systems 20 (2007).](https://proceedings.neurips.cc/paper/2007/hash/013a006f03dbc5392effeb8f18fda755-Abstract.html)
