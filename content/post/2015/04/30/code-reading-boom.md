---
title: "今日のコードリーディング: rakyll/boom"
date: 2015-04-30
comments: true
tags: code-reading golang
---

[HTTP(S) load generator, ApacheBench (ab) replacement, written in Go - rakyll/boom](https://github.com/rakyll/boom)

## 概要

Goで書かれたのApacheBenchライクな負荷測定ツール。Python製の[tarekziade/boom](https://github.com/tarekziade/boom)がオリジナル。

```
# 合計100リクエストを10並行で実行する
$ boom -n 100 -c 10 https://google.com
```

<br />

## バージョン

2015/04/29時点の**master** (372ea3b0e6cb084657d1db7eacccdfae929af5b9)

<br />

## コード

リクエスト実行部分を中心に必要なところだけ抜粋。

### (\*Boomer) Run

結果格納用のchannelをつくってrun()

```go
func (b *Boomer) Run() {
	b.results = make(chan *result, b.N)
	b.run()
	close(b.results)
}
```

### (\*Boomer) run

リクエストの配分を行うところ。N個のリクエストをC個のworkerで処理している。
NをCで割るんじゃなくてCをworker数として個別のworkerが処理できるだけやる。

```go
func (b *Boomer) run() {
	// 総リクエスト数をWaitGroupにAdd
	var wg sync.WaitGroup
	wg.Add(b.N)

	// 総リクエスト分のバッファを持ったchannelを作成
	jobs := make(chan *http.Request, b.N)
	for i := 0; i < b.C; i++ {
		// 並行数分だけworkerを起動
		go func() {
			b.worker(&wg, jobs)
		}()
	}

	// jobsのchannelに総リクエスト分のリクエストを送信
	for i := 0; i < b.N; i++ {
		if b.Qps > 0 {
			<-throttle
		}
		jobs <- b.Req.Request()
	}
	close(jobs)

	// 全てのリクエストが処理されるまで待つ
	wg.Wait()
}
```

WaitGroupを各処理で`Add(1)`せずに一度に`Add(N)`しているのはAdd内での排他による待ちを回避したかったからだと思う。
実行数が分かっているものは一度にAddしておくというのはよさそう。

### (\*Boomer) worker

jobsから受信して実際にリクエスト、結果をb.resultsに送信

```go
func (b *Boomer) worker(wg *sync.WaitGroup, ch chan *http.Request) {
	client := &http.Client{Transport: tr}
	for req := range ch {
		resp, err := client.Do(req)
		wg.Done()
		b.results <- &result{}
	}
}
```

<br />

## 雑感

- 必要な処理がコンパクトに実装され、非常に読みやすかった
- waitGroupを使ったworkerパターンなどgoroutineの基本的な使い方例として教えるときによさそう
- Add(N)については前述のとおり。
- time.Tickを使ったworkerへのchannel送信の制御(throttleの部分)は覚えておいてよさそう。


