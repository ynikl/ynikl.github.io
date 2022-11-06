---
title: "Golang Find Module Conflict"
date: 2022-10-31T08:26:33+08:00
publishDate: 2022-10-31T08:26:33+08:00
draft: true
tags:
- golang
- thoughts
---

遇到了 grpc 不遵循语义版本, 导致[不同版本包之间的冲突](https://github.com/weaveworks/common/issues/239). 

更新了目标的版本模块之后, 编译一下就发现原先项目引用的 gozero 框架报错了. 搜索一下
相关的关键词,就可以定位到问题是 grpc 搞的鬼.


## go 最小版本选择

[golang的最小版本选择](https://research.swtch.com/vgo-mvs): 

- 

## 寻找依赖的原因

寻找自己项目是因为引用了哪个包, 导致会引用目标包的

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

你就可以知道你的项目编译的时候, **为什么会需要编译到目标包**

