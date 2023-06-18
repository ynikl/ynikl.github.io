---
title: "Golang Env Proxy"
date: 2023-05-30T18:52:20+08:00
publishDate: 2023-05-30T18:52:20+08:00
draft: true
tags:
- golang
- thoughts
---

## GOPROXY 环境变量简介

存在的意义: 

在使用 GOPROXY 之前, 拉取 golang 的包都是直接从源代码拉取编译, 会存在两个问题

- 内容会被变更
- 包被移除拉取不到

