---
title: "Go Build Time Variable"
date: 2023-11-20T13:50:52+08:00
publishDate: 2023-11-20T13:50:52+08:00
draft: false
tags:
- golang
---

使用背景

一套程序代码，配置文件，需要同时在不同的云厂商运行。 

通过修改 jenkins, 的部署 pipeline , 让开发人员准确的获取到当前程序的运行环境
还能保持代码统一.

- ldflags 修改的变量, 无大小写限制, 小写变量也可修改编译过程进行修改
- 变量引用需要 fullpath. 例如 main 文件中的变量 `go build -ldflags="-X 'main/varName=xxx'`
- 子包，需要当前 mod 包名称开头。 可以查看第二篇文章

- [go build variable](https://belief-driven-design.com/build-time-variables-in-go-51439b26ef9/)
- https://programmingpercy.tech/blog/modify-variables-during-build/


发现更加简单的解决方案

`go build -tags targetCloud`

- https://pkg.go.dev/go/build#hdr-Build_Constraints
