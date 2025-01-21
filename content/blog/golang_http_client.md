---
title: "Golang_http_client"
date: 2025-01-21T22:16:45+08:00
publishDate: 2025-01-21T22:16:45+08:00
draft: true
tags:
- golang
- thoughts
---

Golang Http

1. client 
2. transport
3. server

- client 修改重定向策略
-  Transport 修改代理，TLS, 连接
-  一次创建重复使用，避免连接重复创建

## transport

1. Proxy
2. DialContent

Dial / DialContent  用于 tcp 连接拨号，可以用于重写，构建预连接的 tcp 连接池.  Content 指的是带超时控制。  

[默认实现在 net.sysDialer](https://cs.opensource.google/go/go/+/refs/tags/go1.22.2:src/net/dial.go;l=455) 

resolveAddrList 提供连接解析

### 空闲连接数的配置

因为来自 gpt-4 的回答
> 评估HTTP客户端需要保持多少空闲连接的数量需要考虑以下因素：
>
1. **并发请求的数量**：如果你的应用经常需要并发发送多个HTTP请求，那么保持更多的空闲连接可能会有所帮助。这样可以避免每次请求都需要建立新的连接，从而节省了握手的时间。
>
2. **请求的频率**：如果你的应用发送请求的频率很高，那么保持更多的空闲连接也是有用的。如果请求的频率较低，那么保持大量的空闲连接可能会浪费资源，因为这些连接可能大部分时间都在空闲状态。
>
3. **服务的响应时间**：如果你的服务的响应时间较长，那么保持更多的空闲连接可能会有所帮助。这样可以确保即使有一些连接正在等待响应，也有其他的连接可以用来发送新的请求。
>
4. **资源限制**：你需要考虑你的系统的资源限制，如文件描述符的数量。每个打开的连接都需要一个文件描述符，因此如果你的系统的文件描述符的数量有限，那么你可能需要限制空闲连接的数量。
>
5. **网络稳定性**：如果网络连接经常中断，那么保持大量的空闲连接可能没有太大的帮助，因为这些连接可能会因为网络问题而被关闭。
>
在Go的`net/http`包中，你可以通过设置`Transport`的`MaxIdleConns`和`MaxIdleConnsPerHost`来控制空闲连接的数量。`MaxIdleConns`控制总的空闲连接的数量，而`MaxIdleConnsPerHost`控制每个主机的空闲连接的数量。
>
你可能需要通过观察和调整这些参数来找到最佳的设置。你可以通过监控系统的资源使用情况，以及HTTP客户端的性能（如请求的延迟和失败率）来帮助你进行调整。


## Shutdown

会关闭所有连接，但是 `ListenAndServer` 会立即结束并返回关闭错误，整个进程需要等待所有连接关闭才能退出。

[官方示例代码](https://pkg.go.dev/net/http#example-Server.Shutdown)
