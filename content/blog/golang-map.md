---
title: "Golang Map"
date: 2022-08-13T14:14:30+08:00
publishDate: 2022-08-13T14:14:30+08:00
draft: true
tags:
- golang
- thoughts
---

## 内部数据结构


## 初始化

map 是一个有"包含内容"的数据结构, 使用之前需要提前初始化, 即调用`make`

真正是调用源码是 [runtime.makemap](https://cs.opensource.google/go/go/+/master:src/runtime/map.go;l=283;bpv=1;bpt=1?q=makemap&ss=go%2Fgo)

## 获取数据

## 删除数据

## 重新扩容

## Sync map 的基本使用

## 参考
[Golang 中 map 探究](https://mp.weixin.qq.com/s/UT8tydajjOUJkfc-Brcblw)
