---
title: "Golang 是否需要为每个请求 New 一个 Client"
date: 2022-12-19T15:35:53+08:00
publishDate: 2022-12-19T15:35:53+08:00
draft: false
tags:
- golang
---

## 背景

在改动旧代码的时候把, 一个使用全局 `http.Client` 的代码弄成了每一个请求会新 New
一个 `http.Client` 导致下游的 nginx 的连接数暴涨.

## 问题

处理多个请求的时候, 是否需要为每个请求 New 一个 Client

## 探索

在 StackOverflow 发现的相关答案

[How to release http.Client in Go?](https://stackoverflow.com/a/36688970/9992963)

给的答案是建议复用 `Client`

> The Client's Transport typically has internal state (cached TCP connections), so Clients should be reused instead of created as needed. Clients are safe for concurrent use by multiple goroutines.

http.Client 的结构体

```
type Client struct {

	Transport RoundTripper

	CheckRedirect func(req *Request, via []*Request) error

	Jar CookieJar

	Timeout time.Duration
}
```

在 `RoundTripper` 中实现了连接复用的逻辑

```
type RoundTripper interface {
	RoundTrip(*Request) (*Response, error)
}
```

中定义了 `RoundTrip` 方法, 提供客户端请求的时候调用.

[调用地址](https://cs.opensource.google/go/go/+/master:src/net/http/transport.go;drc=0b2ad1d815ea8967c49b32d848b2992d0c588d88;bpv=1;bpt=1;l=512?q=%2Fnet%2Fhttp%2Ftransport.go&ss=go%2Fgo)
 
[查看一下 Golang Transport 的基本实现](https://cs.opensource.google/go/go/+/master:src/net/http/transport.go;drc=0b2ad1d815ea8967c49b32d848b2992d0c588d88;l=95?q=%2Fnet%2Fhttp%2Ftransport.go&ss=go%2Fgo)

``` 
type Transport struct {
	idleMu       sync.Mutex
	closeIdle    bool                                // user has requested to close all idle conns
	idleConn     map[connectMethodKey][]*persistConn // most recently used at end
	idleConnWait map[connectMethodKey]wantConnQueue  // waiting getConns
	idleLRU      connLRU

	connsPerHostMu   sync.Mutex
	connsPerHost     map[connectMethodKey]int
	connsPerHostWait map[connectMethodKey]wantConnQueue // waiting getConns
	
	// 还有其他字段略
}
```

结构体中间有很多连接存储相关的字段.

在 http 请求调用 Transport 中间有一个关键方法 [getConn](https://cs.opensource.google/go/go/+/master:src/net/http/transport.go;bpv=1;bpt=1;l=1338) 获取一个连接 

方法声明一个想要的连接地址, [wantConn](https://cs.opensource.google/go/go/+/master:src/net/http/transport.go;drc=1b2ad1d815ea8967c49b32d848b2992d0c588d88;l=1194) 推入到 [queueForDial](https://cs.opensource.google/go/go/+/master:src/net/http/transport.go;drc=0b2ad1d815ea8967c49b32d848b2992d0c588d88;bpv=0;bpt=1;l=1422)

[QueueForDial 方法](https://cs.opensource.google/go/go/+/master:src/net/http/transport.go;drc=0b2ad1d815ea8967c49b32d848b2992d0c588d88;bpv=1;bpt=1;l=1422)会判断时候`connsPerHost` 中间是否有当前的请求的缓存连接

- 如果有直接拿来重复使用
- 如果没有, 就需要重新进行拨号

```
	w.beforeDial()
	if t.MaxConnsPerHost <= 0 {
		go t.dialConnFor(w)
		return
	}

	t.connsPerHostMu.Lock()
	defer t.connsPerHostMu.Unlock()

	if n := t.connsPerHost[w.key]; n < t.MaxConnsPerHost {
		if t.connsPerHost == nil {
			t.connsPerHost = make(map[connectMethodKey]int)
		}
		t.connsPerHost[w.key] = n + 1
		go t.dialConnFor(w)
		return
	}

	if t.connsPerHostWait == nil {
		t.connsPerHostWait = make(map[connectMethodKey]wantConnQueue)
	}
	q := t.connsPerHostWait[w.key]
	q.cleanFront()
	q.pushBack(w)
	t.connsPerHostWait[w.key] = q
```

## 结论 

重复使用 http.Client 可以达到 TCP 连接复用的效果
