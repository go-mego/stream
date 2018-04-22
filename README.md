# Stream [![GoDoc](https://godoc.org/github.com/go-mego/stream?status.svg)](https://godoc.org/github.com/go-mego/stream)

Stream 是一個串流套件，能夠讓你方便地將字串、二進制資料不斷地寫入客戶端的回應中。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)
    * [寫入緩衝區](#寫入緩衝區)
    * [緩衝區沖刷](#緩衝區沖刷)
        * [沖刷與取得](#沖刷與取得)
        * [全部清空](#全部清空)
        * [取得與清空](#取得與清空)
        * [取得長度](#取得長度)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/stream
```

# 使用方式

串流中介軟體可以用作全域中介軟體，將 `stream.New()` 傳入 Mego 引擎的 `Use` 即可。

```go
package main

import (
	"github.com/go-mego/mego"
	"github.com/go-mego/stream"
)

func main() {
	m := mego.New()
	// 將串流中介軟體作為為全域中介軟體。
	m.Use(stream.New())
	m.Run()
}
```

串流中介軟體也能夠僅用於單個路由上。

```go
func main() {
	m := mego.New()
	// 在單個路由中套用串流中介軟體，並使用 `*stream.Stream`
	m.GET("/", stream.New(), func(s *stream.Stream) {
		// ...
	})
	m.Run()
}
```

## 寫入緩衝區

以 `Write` 函式來將資料寫入串流緩衝區供稍後發送。

```go
func main() {
	m := mego.New()
	m.GET("/", stream.New(), func(s *stream.Stream) {
		// 將目前時間寫入串流緩衝區中。
		s.Write(fmt.Sprintf("%v\n", time.Now()))
	})
	m.Run()
}
```

預設的 `Write` 能夠讓你將 `string` 型態的資料寫入串流緩衝區中，但還有其他函式（如：`WriteBytes`）能夠讓你寫入其他形態的內容至緩衝區。

```go
func main() {
	m := mego.New()
	m.GET("/", stream.New(), func(s *stream.Stream) {
		// 將位元組資料寫入串流串流緩衝區。
		s.WriteBytes([]byte("早安，我的朋友！"))
		// 將正整數寫入串流緩衝區。
		// s.WriteInt64(), s.WriteUint() ...
		s.WriteInt(12345)
	})
	m.Run()
}
```

## 緩衝區沖刷

緩衝區（Buffer）是一個送出資料前的資料暫存區，當寫入串流時，其實事是先寫入緩衝區並等候沖刷才會一口氣將資料全數送至客戶端。如果沒有進行沖刷，資料則會繼續累積在緩衝區。

透過 `Flush` 將寫入緩衝區的資料沖刷至客戶端，並清空緩衝區。

```go
func main() {
	m := mego.New()
	m.GET("/", stream.New(), func(s *stream.Stream) {
		// 在此路由不斷地執行下列程式。
		for {
			// 不斷地將目前時間寫入串流緩衝區中。
			s.Write(fmt.Sprintf("%v\n", time.Now()))
			// 接著沖刷，將緩衝區的內容寫入串流中。
			s.Flush()
			// 接著等待一秒，重複執行上列函式。
			<-time.After(1 * time.Second)
		}
	})
	m.Run()
}
```

### 沖刷與取得

透過 `GetAndFlush` 可以取得緩衝區的內容並且自動沖刷到客戶端。

```go
func main() {
	m := mego.New()
	m.GET("/", stream.New(), func(s *stream.Stream) {
		// 將目前時間寫入串流緩衝區中。
		s.Write(fmt.Sprintf("%v\n", time.Now()))
		// 取得緩衝區資料並且沖刷至客戶端。
		fmt.Println(s.GetAndFlush()) // 結果：2018-04-21 15:37:42 +0000 UTC m=+0.000000001
	})
	m.Run()
}
```

### 全部清空

如果緩衝區的內容和預期不一樣，可以在沖刷至客戶端前以 `Clean` 來清空。

```go
func main() {
	m := mego.New()
	m.GET("/", stream.New(), func(s *stream.Stream) {
		// 將目前時間寫入串流緩衝區中。
		s.Write(fmt.Sprintf("%v\n", time.Now()))
		// 清空緩衝區資料。
		s.Clean()
		// 此時沖刷至客戶端並不會有任何資料。
		s.Flush()
	})
	m.Run()
}
```

### 取得與清空

可以透過 `GetAndClean` 取得緩衝區的內容並自動清空緩衝區資料，這並不會將資料沖刷到客戶端。

```go
func main() {
	m := mego.New()
	m.GET("/", stream.New(), func(s *stream.Stream) {
		// 將目前時間寫入串流緩衝區中。
		s.Write(fmt.Sprintf("%v\n", time.Now()))
		// 清空並取得且顯示緩衝區資料。
		fmt.Println(s.GetAndClean()) // 結果：2018-04-21 15:37:42 +0000 UTC m=+0.000000001
		// 此時沖刷至客戶端並不會有任何資料。
		s.Flush()
	})
	m.Run()
}
```

### 取得長度

透過 `Length` 可以取得目前緩衝區的長度。

```go
func main() {
	m := mego.New()
	m.GET("/", stream.New(), func(s *stream.Stream) {
		// 將目前時間寫入串流緩衝區中。
		s.Write(fmt.Sprintf("%v\n", time.Now()))
		// 取得緩衝區的長度。
		fmt.Println(s.Length()) // 結果：45
	})
	m.Run()
}
```