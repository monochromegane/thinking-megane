+++
date = "2020-05-31T12:16:27+09:00"
title = "Go言語でシミュレーション用のシンプルなフレームワークStageをつくった"
tags = [ "golang" ]
+++


時系列に対するコンピュータシミュレーションを開発する機会が増えてきたので、共通する処理の流れをフレームワーク化した。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fstage" title="monochromegane/stage" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/stage"&gt;monochromegane/stage&lt;/a&gt;</iframe>

## コンピュータシミュレーション

状況に応じたクリック数の最大化や変化点検出のような、システムの適応的な振る舞いを検証するために、時系列に対するコンピュータシミュレーションを行うことがある。
また、実環境で発生するランダムな誤差を表現する場合、乱数を用いたシミュレーション技法であるモンテカルロ法も利用することになる。

このようなシミュレーションのプログラムでは、変化する時系列を入力に、振る舞いのシミュレーション結果を出力するだけではなく、乱数によって発生する誤差を均すためにシミュレーションを数百〜数千回繰り返す必要がある。

## Stage

Stageは、これらの一連の流れの実行とこれに伴う煩雑な処理（シミュレーションの並列化、乱数の管理、進捗の監視、ログ出力）を開発者から隠蔽する。
開発者は、時系列や振る舞い、ログフォーマットをGo言語で記述するだけで良い。

Stageで行っているメイン処理の「疑似コード」は以下の通りシンプルである。

```go
for i := 0; i < iter; i++ {
    sem <- struct{}{}

    go func() {
        scenario := NewScenarioFn(rnd.Int63())
        actor := NewActorFn(rnd.Int63())

        for scenario.Scan() {
            action, _ := actor.Act(scenario.Line())
            w.Write(action.String())
        }
        <-sem
    }()
}
```

このフレームワークでは、メタファーとして劇場(Stage)を採用した。
劇場は、劇を開催する。
その劇は台本(Scenario)があり、演者(Actor)は台本の1行づつのセリフ(Line)に沿って演技を行い、結果(Action)を出力する、といった形だ。
そして劇は何度も繰り返される。

開発者は、時系列や振る舞いをそれぞれ、ScenarioとActorのinterfaceを実装によって記述する。
また、ログフォーマットもAction interfaceで指定するだけで良い。

```go
type Scenario interface {
	Scan() bool
	Line() Line
}

type Actor interface {
	Act(Line) (Action, error)
}

type Action interface {
	String() string
}
```

あとは実装したScenarioとActorを生成する関数を指定して必要な回数実行。

```go
dir := "log"                    // Directory for output
concurrency := runtime.NumCPU() // Number of concurrency for scenario
seed := 1                       // Seed for random
iter := 3                       // Number of iteration

s := stage.New(dir, concurrency, seed)
s.Run(iter, NewActorFn, NewScenarioFn, stage.NoOpeCallbackFn)
```

結果は、指定したログディレクトに出力され、実行日時や利用した乱数シードも確認することができる。

```sh
log
└── 20200530184234-1 # Timestamp-Seed
    ├── iter_00-a_7947919477105006377-s_5355116748216652230.log # Iteration log files
    ├── iter_01-a_4846631296614585111-s_2007235010091403794.log #   with seed for actor(a)
    └── iter_02-a_0610076349056253918-s_3540139325796113853.log #   and  seed for scenario(s)
```

なお、出力後の解析、グラフ出力については、Stageでは取り扱わない。
任意のフォーマットで出力できるのでお好みの言語やツールで操作して欲しい。

各インターフェースの具体的な実装方法については[README](https://github.com/monochromegane/stage/blob/master/README.md)を参照のこと。

## 複数のシナリオを持つシミュレーション例

例えば、以下の青線のような変化の仕方が異なるシナリオを切り替えて、橙の線のような振る舞いの違いをシミュレーションすることができる。

| Abrupt changes | Gradual changes |
| -------------- | --------------- |
| ![example_adwin_abrupt](https://user-images.githubusercontent.com/1845486/83343338-f8bcbe00-a333-11ea-983b-81ba9a6f8b08.png)               | ![example_adwin_gradual](https://user-images.githubusercontent.com/1845486/83343344-10944200-a334-11ea-9364-b4d4cedcdff6.png)                |

- [複数シナリオの実装例](https://github.com/monochromegane/stage/tree/master/_examples/adwin)

## 長時間かかるシミュレーション例

また、シミュレーションに時間がかかる場合は、各シナリオの完了時に呼ばれるCallback関数を指定することで、進捗の監視もできる。
お好みのProgress barライブラリを使えば進捗バーも表示できる。

```sh
$ go run _examples/progress/main.go
180 / 200 [---------------------------->___] 90.00% 40 p/s
```

- [プログレスバーの実装例](https://github.com/monochromegane/stage/tree/master/_examples/progress)

# まとめ

普段よりアルゴリズムの理解を深めるため手に馴染んだGo言語で実装を試みるので、必然的にシミュレーションもGoで書くことが多くなったことからGo言語での実装となった。
完全に自分用途でつくったフレームワークではあるが、各シミュレーションにおいてシナリオとアクターという要素のみを意識すればよくなり実装の効率が格段に上がりコードの見通しもよくなった。
また、Go言語を使うことで並列化が容易に実装できシンプルなフレームワークでありながら十分に安定して高速化を達成できていると思う。
付加的な利点として、ログ構成などが統一されたことで後段の解析やグラフ化のスクリプトも共通化が進んでおり、個人的に満足度が高いものができたと思う。
