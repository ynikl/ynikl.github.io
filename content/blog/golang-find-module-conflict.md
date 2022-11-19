---
title: "Golang 解决依赖包版本冲突"
date: 2022-10-31T08:26:33+08:00
publishDate: 2022-10-31T08:26:33+08:00
draft: false
tags:
- golang
---

遇到了 grpc 不遵循语义版本, 导致[不同版本包之间的冲突](https://github.com/weaveworks/common/issues/239). 

更新了目标的版本模块之后, 编译一下就发现原先项目引用的 gozero 框架报错了. 搜索一下
相关的关键词,就可以定位到问题是 grpc 搞的鬼.

再找到对应的兼容版本, 升级到对应的版本就可以了.

## go 依赖版本选择

[golang 的最小版本选择]{{< ref "/blog/golang-minimal-version-selection.md" >}}

大体意思:

会选择当前编译需要依赖包的最高版本(使用语义化版本)

## 寻找依赖的原因

[go mod why](https://go.dev/ref/mod#go-mod-why)

寻找自己项目引用某个包的 **最短引用路径**, 导致会引用目标包的

```
go mod why google.golang.org/grpc
```

输出目标包的引用依赖层级
```
❯ go mod why google.golang.org/grpc

# google.golang.org/grpc
hello/world/test
git.test.cn/company-open/rpc-pkgs
google.golang.org/grpc
```


[go mod graph](https://go.dev/ref/mod#go-mod-graph)

可以打印出, 模块的依赖图
```
example.com/main example.com/a@v1.1.0
example.com/main example.com/b@v1.2.0
example.com/a@v1.1.0 example.com/b@v1.1.1
example.com/a@v1.1.0 example.com/c@v1.3.0
example.com/b@v1.1.0 example.com/c@v1.1.0
example.com/b@v1.2.0 example.com/c@v1.2.0
```

