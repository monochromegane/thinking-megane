---
title: "今日のコードリーティング: tsenart/vegeta"
date: 2015-05-01
comments: true
tags: [ "code-reading", "golang" ]
---

[HTTP load testing tool and library. It's over 9000! - tsenart/vegeta](https://github.com/tsenart/vegeta)

## 概要

[前回](http://blog.monochromegane.com/blog/2015/04/30/code-reading-boom/)読んだ `rakyll/boom`と同じ負荷測定ツール。

```
$ echo "GET http://google.com/" | vegeta attack -duration=2s | vegeta report
```


## バージョン

2015/4/30時点の**master** (3739d8a2090cd61f4fe018e00adde281d824d3c6)


## コード

vegeta attack 処理を中心に必要なところだけを抜粋

### サブコマンドと引数

サブコマンド(attack, report...)ごとのコマンド構造体を定義。

```go
type command struct {
	fs *flag.FlagSet	     // コマンドごとのオプション
	fn func(args []string) error // コマンドごとの起点となるfunction
}
```

標準の`flag.Parse`ではなくて`flag.FlagSet.Parse`を使うことでParse対象に任意の引数を渡すことができるのでサブコマンドに必要な引数だけを渡している。便利。

### (\*Attacker) Attack

並行数分のworkerの立ち上げと時間あたりのリクエストの配分を制御しているところ。
処理に時間がかかるので戻り値として結果のchannelを返す。

```go
func (a *Attacker) Attack(tr Targeter, rate uint64, du time.Duration) <-chan *Result {
	workers := &sync.WaitGroup{}
	results := make(chan *Result)
	ticks := make(chan time.Time)
	for i := uint64(0); i < a.workers; i++ {
		go a.attack(tr, workers, ticks, results)
	}

	go func() {
		for began, done := time.Now(), uint64(0); done < hits; done++ {
			select {
			case ticks <- max(next, now):
				return
			}
		}
	}()

	return results
}
```

workerとして実際にリクエストを行う処理。time.Tick相当のタイマーでリクエストを行う。

```go
func (a *Attacker) attack(tr Targeter, workers *sync.WaitGroup, ticks <-chan time.Time, results chan<- *Result) {
	workers.Add(1)
	defer workers.Done()
	for tm := range ticks {
		results <- a.hit(tr, tm)
	}
}
```

このあたりもう少し改善の余地あると思った。

workerはタイマーを意識する必要はなくて単純にリクエストのjobキューを順番に処理していくだけにして、jobキューに投入する量をタイマーで制御するようにしたほうが見通しがよくなりそう。

### attack

上記Attackの結果を受信して、`gob.NewEncoder`にシリアライズして出力する。

```go
// gobパッケージを使ったエンコーダー
enc := gob.NewEncoder(out)

// 結果を受信
for {
	select {
	// Attackの結果を受信
	case r, ok := <-res:
		// 結果をシリアライズして出力
		if err = enc.Encode(r); err != nil {
			return err
		}
	}
}
```

[gob](http://golang.org/pkg/encoding/gob/)はGoで使えるシリアライザ。送信側と受信側でフィールド名と型があっていればデシリアライズしてくれる。`vegeta attack | vegeta report` 間でシリアライズしたデータをやりとりしている。


## 雑感

- `flag.FlagSet`を使ったサブコマンドのオプション実装は参考になった
- 実装は`rakyll/boom`のほうが洗練されてると思う
- 負荷測定は長くかかる処理なので`signal.Notify(sig, os.Interrupt)`を使って明示的に終了処理を呼び出すようにしているのはよいと思った
- `gob`知らなかったのでRPC的な処理をやるときの候補に入れておく

