---
title: "Golang error 总结"
date: 2023-03-20T22:27:13+08:00
publishDate: 2023-03-20T22:27:13+08:00
draft: true
tags:
- golang
---


panic 和 error 的区别

error 错误信息的传递方式

1. 直接返回
2. wrap
3. 使用特定结构体

error 错误检查方式

1. 直接判断是否nil
2. 使用 `error.As, Is`
3. interface 的断言

error 的处理误区

1. 重复多次处理同一个错误
2. 明确是否需要忽略某一个调用函数的错误
3. 遗漏了 defer 中调用的函数的错误


## 参考

- [官方文档](https://pkg.go.dev/errors#pkg-overview)
- [Golang 100 Mistake](https://book.douban.com/subject/36084407/)


