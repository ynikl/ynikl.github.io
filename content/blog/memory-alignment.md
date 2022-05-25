---
title: "Memory Alignment"
date: 2022-04-19T10:41:09+08:00
draft: true
tags:
- go
- os
---

# 内存对齐

## 为什么需要内存对齐

## go 中的结构体是怎么对齐的?

go 中内存对齐的大小: 4或8bytes(对应32或64位计算机)

如果结构体最长的数据字段为64位, 则对齐的大小为8bytes, 或者则为4bytes


``` go

```

## 工具介绍

## 参考连接:
[go结构体的内存对齐优化](https://itnext.io/structure-size-optimization-in-golang-alignment-padding-more-effective-memory-layout-linters-fffdcba27c61)
