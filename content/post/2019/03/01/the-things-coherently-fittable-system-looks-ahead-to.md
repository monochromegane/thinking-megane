+++
date = "2019-03-01T16:47:54+09:00"
title = "なめらかなシステムの見据えるもの。個人的考察"
tags = [ "情報要求" ]
+++

ペパボ研究所ではシステムの利用や運用における様々な障壁を取り除いた、なめらかな利用や運用を目指して研究を行っている。
本稿では、なぜなめらかでなければならないかの問いを端緒とし、定義の解釈を試みることで、なめらかなシステムの見据える未来について考察する。
考察を通して、なめらかなシステムの見据えるシステム観を再認識し、システム設計の道標にするとともにそれらの普及した場合への対処としてのなめらかなシステムの有用性を確認する。

# なめらかなシステムを解釈する

## なめらかなシステムを構成する要件

ペパボ研究所では、システムの利用や運用における様々な障壁（ゴツゴツ）を取り除き、利用の快適さや運用における生産性の向上に繋げるため、以下の「なめらかなシステム」を提案、実現に向けた取り組みを進めている。

```
「なめらかなシステム」とは、情報システムのことをいうのみならず、互いに影響を及ぼし合う継続的な関係にある利用者（ユーザーおよび開発運用者）と情報システムとからなる総体としてのシステムであり、以下の要件を満たします：

1. 利用者と情報システムとが継続的な関係を取り持つ過程において、利用者それぞれに固有のコンテキストを見出したり、新たなコンテキストを創出したりできること
2. 要件1.を、利用者による明示的な操作を課すことなく実現できること
3. 要件1.および2.によって得られたコンテキストに基づき、情報システムが利用者に対して最適なサービスを自動的に提供できること

ref: https://rand.pepabo.com/
```

すなわち、システム総体における要素の状態、状況、条件といった「文脈」が暗黙的に認識され、文脈ごとに最適な振る舞いが提供されるシステムである。
ここで、なめらかなシステムは上記の要件をすべて満たす必要があることに注意したい。
特定指標の予測や限定的な自動化はなめらかなシステムを構成する基礎技術ではあるがそれ単体ではなめらか足り得ない。

## なぜなめらかでなければならないか

なめらかなシステムを実現するにあたっては、文脈をシステムで利用可能な指標に落とし込み、これを暗黙的に獲得し、この文脈指標に適応した振る舞いを定義しなければならない。
このように、なめらかなシステムは利用者や運用者への利益をもたらすが、同時に複雑性も持ち込む（システム操作に含まれない過去の経験や状況まで含めた文脈と指標の紐付けが困難であり、そのためのログ基盤、機械学習基盤、予測フィードバック機構の整備が大変なことは同意いただけるだろう）。

そのため、なぜなめらかでなければならないかを明確にし、その必然性から適用を検討すべきであると考える。
我々は[なめらかなシステムの論文化](http://tsys.jp/dicomo/2018/program/4B_abst.html)([論文と発表資料](https://rand.pepabo.com/article/2018/07/25/dicomo2018/))にあたり、従来の定義を洗練し、システムの利便を得るべき人間の存在をシステム総体のうちに組み込むことを明示した。
これにより、このような人間は、様々な文脈を持つこと、そして情報要求がコミュニケーションによって逐次変化していくことに着目し、情報システム側もこれに追従する仕組みが必要であると述べた。
すなわち、様々な文脈を持ち、変化し続ける対象としての人間の存在を以って、これに対応するためのなめらかなシステムの必要性を主張した。

一方で、なめらかなシステムでは、利用者やシステムコンポーネントなどは振る舞いを持って役割とし、本質的には等価にみなすことができると考える。
であれば、様々な文脈を持ち、変化し続ける対象としての人間の存在を解釈し直すことで、なめらかなシステムを適用すべき対象を定義することができるのではないか、というのが本稿の主旨である。

# 利用者を解釈する

## 個性を持つ存在として捉える

ここでは、様々な文脈を持ち、変化し続ける対象としての人間を「個性」という観点で汎用化を試みる。
そのために、利用者が様々な文脈を持つことと複雑性の関係について見直したい。
様々な文脈を持つことを何らかの方法でシステム的に取り扱える指標に変換できた場合、それは多次元の特徴量として表現されるだろう。
これらを以って振る舞いを定義する必要がある場合に、必ずしもすべての特徴量の組み合わせに対処するように定義しなければならないかといえばそうではない。
情報要求の満足度に対する各次元の寄与度は均等ではないはずで、一定のパターン数での対応で現実的には可能であるように思える。
実際、少なくとも現状のシステムは部分的には制御可能なパラメタのもと、ヒューリスティックな対処がまだ有効な範囲内で行われていることからも言えるだろう。
もちろん、様々な文脈としての特徴量の次元数が多いことは複雑さの増加には繋がる。
しかしながら、複雑であるとしてみなさなければならないのは対応パターン数ではなかろうか。

本稿では、このような対応パターン数を増加させる対象を「個性を持つ」と表現したい。
ここで、「個性を持つ」とは、ある面では同じクラスタに属しながらも同じ入力に対して異なる振る舞いを行うことだと定義する。
これには、人間はもちろんのこと、クラスとインスタンスの関係のように、異なるデータで学習した機械学習のモデルなどがあるだろう。
このような対象に対するコミュニケーションは傾向はあるとしても正解はなく、対象や文脈を仮定しながら試行を繰り返し歩み寄っていく必要があるはずだ。
そして、そのような歩み寄りの過程こそがなめらかなシステムの構成要件であると言える。

## 概念の獲得が個性をもたらす

集団における共通項として概念が獲得されることで概念を満たす内部実装が多様化し得るのではないか。
共通のインターフェースを持ちながらに、別の面では振る舞いに差が現れる。
人間は自身の感覚器からの情報を生で知覚するのではなく、（おそらく）多層の抽象化、概念化を通した結果として知覚する。
また、個々が理解している概念の表層として言葉を操り、（おそらく）同じような理解をしているであろう対象とコミュニケーションを行う。

これまで機械としてのシステムは、個性が生まれるように設計されたものがそのように振舞っていた。
しかしながら機械としてのシステムが互いに共通インターフェースである概念や定義を獲得し、それを満たす実装や解釈を行うようになれば個性の爆発が起きるのではないかと思う。

## わからないものとして扱う

ここではなめらかなシステムの実現にあたっての実装寄りのアプローチについて検討する。
個性を持つシステム要素はその性質上、相互にわかり合うことはできないため、なめらかなシステムのようなアプローチを通して歩み寄らなければならない。
このような「わからない状況」をシステムとして扱うためには、単体の手法ではなく状況に応じたいくつかの方法論の組み合わせが有効であろう。
まず入力の大局や傾向については回帰や機械学習などからなる予測的なアプローチをとり、予想外の入力変化に対しては例えばフィードバック制御などの脊髄反射的な対応が求められる。
次に、入力に応じたパターンの選定では、なめらかなシステムが前提とする対象において、無限のパターンが想定されることから、実運用環境での最適化が必要である。
具体的には多腕バンディット問題として取り扱ってもよいだろうし、進化的、遺伝的アルゴリズムを用いた解法も選択肢に入るだろう。
このパターン選定のための最適化アプローチは多くの試行を要するが、対象をシステム要素とみなす場合はその労苦は考慮不要であるし、
人間を対象とする場合であってもシステム利用者全体の操作を束ねたものを利用すればよいだろう。

# なめらかなシステム再考

ここまでの考察から、なめらかなシステムは個性を持つ対象とのコミュニケーションが発生する状況において求められ、これに継続的に適応可能なアーキテクチャのことを指すと考えられる。
例えば、推薦システムは、人間を対象とし提示した推薦結果に対する反応が異なることことから、なめらかなシステムの適用先の事例として考えやすいであろう。
では、なめらかなオートスケーリングはありうるか。
特定サービスの単一メトリクスに基づいてルールベースの増減するような場合、例え入力に多様性があっても、システムとしては決められた挙動を行うだけであり、動的制御であってもなめらかではないことは自明である。
一方で、オートスケーリングの指標において、より抽象的な概念である、キツい、ダルい、調子良いのような言葉を以って、いい感じに振る舞うことができるとすれば、そこには解釈の余地が生まれ、入力への対処の自由度、すなわち個性を持った対応が必要になる。
また、非常に複雑なシステムにおいて増減自体が他へ与える影響が読めないような場合にいい感じに落とし所を見つけなければならないのであれば、そのような状況も個性を前提としたものであろう。
このような対応パターンの増加に対するアプローチは先に述べた通りであるが、概念の創出とパターンに対する最適化は相互に洗練していくことになるだろう。

# まとめ

本稿では、なぜなめらかでなければならないかを問いの発端とし、システムの利便を得るべき人間について個性の観点から再解釈することで、なめらかなシステムの見据える未来を明確化した。
なめらかなシステムについての考察は、完全にこれを実現した突出したシステムが突然表出するわけではなく、システム総体に含まれる要素が相互に徐々に複雑化していく過程で、これに対応する必然性から最適化が行われ、
さらにその複雑性が、個性を育てるというような、今後のシステムの進化そのものを想起させる。
であれば、システム設計において進化の方向性としてこれを意識することで、従前の単なる自動化や動的制御の先を見据えた発想になるのではないか、その環境においてどのように対応すべきかを考える道標となるのではないか。
本考察が、その一助となれば幸いである。
