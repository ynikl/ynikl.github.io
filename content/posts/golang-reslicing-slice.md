---
title: "Go中slice[i:j:k]第三个参数是做什么的"
date: 2022-06-09T23:52:30+08:00
publishDate: 2022-06-09T23:52:30+08:00
draft: false
tags:
- golang
- qa
---

今天, 突然被同事卷到了, 被问到 golang 中 slice 的三个参数是干嘛的? 我突然一时间忘记了, golang 的重切片居然是可以接受第三个参数的, 枉费我已经了写了快两年的 go 了. 赶紧 google 一下, 并总结备忘. 

## 简单介绍 slice 的数据结构

首先, 介绍一下 golang 中切片的结构体: 

```
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

有三个字段:

- array 是切片所指向的底层数组数据
- len 就是切片的长度
- cap 即容量, 很明显
 
[源码地址](https://cs.opensource.google/go/go/+/master:src/runtime/slice.go;l=15?q=slice&ss=go%2Fgo)


## 简单版重切片 `a[low:high]`

- 接受切片中的开始下标和结束下标, "左闭右开原则" 即重的切片数据会包含 low 下标的值, 但没有 high 下标的值
- 新切片的 容量(cap) 即为开始下标到原 slice 数据容量结束, 即"cap(a) - low"
- low 参数可以省略 默认从 0 下标开始

```
a := [10]int{}
oldSlice := a[:5]
newSlice := a[2:4]
fmt.Printf("b: len %d, cap %d, c: len %d, c:cap %d", len(b), cap(b), len(c), cap(c))

输出:

b: len 5, cap 10, c: len 2, c:cap 8

```

底层分配情况如下:

```
底层数组       : [0 0 0 0 0 0 0 0 0 0]

旧切片容量: 10 : [0 0 0 0 0 0 0 0 0 0]
旧切片长度: 5  : [0 0 0 0 0]

新切片容量: 8  :     [0 0 0 0 0 0 0 0]
新切片长度: 2  :     [0 0]
```

## 完整版重切片 `a[low:high:max]`

完整版是为了补充简单版, 新切片会默认拥有从开始下标后的所有底层数组容量. 即新数组拥有修改全部数据底层能力 (**重切片出来的不同切片, 在没有 append 操作触发重新分配底层数组的前提下, 指向的都是同一个数据, 修改数据相互可见**)

- 增加了 max 参数, 表示新切片可以获取到最大的原切片的容量大小. 

```
a := [10]int{}
oldSlice := a[:5]
newSlice := a[2:4:6]
fmt.Printf("b: len %d, cap %d, c: len %d, c:cap %d", len(b), cap(b), len(c), cap(c))

输出:

b: len 5, cap 10, c: len 2, c:cap 4
```

底层分配情况如下

```
底层数组       : [0 0 0 0 0 0 0 0 0 0]

旧切片容量:10  : [0 0 0 0 0 0 0 0 0 0] 
旧切片长度:5   : [0 0 0 0 0]

新切片容量:4   :     [0 0 0 0] 
新切片长度:2   :     [0 0]
```

但是, 如果 max 要求获取的容量大于旧数据容量. 可想而知, 那一定会 `panic`

``` go
a := [10]int{}
b := a[:4:4]
c := b[0:4:5]

输出:

panic: runtime error: slice bounds out of range [::5] with capacity 4
```

所以参数要求: `0 <= low <= high <= max <= cap(a)`

## 参考

- [stack overflow 问题1 简单版](https://stackoverflow.com/questions/27938177/golang-slice-slicing-a-slice-with-sliceabc) 
- [stack overflow 问题2 详细版](https://stackoverflow.com/questions/12768744/re-slicing-slices-in-golang/18911267#18911267) 
- [golang 官方文档](https://go.dev/ref/spec#Slice_expressions) 


