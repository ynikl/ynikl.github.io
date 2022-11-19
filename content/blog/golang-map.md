---
title: "Golang Map 介绍"
date: 2022-08-13T14:14:30+08:00
publishDate: 2022-08-13T14:14:30+08:00
draft: true
tags:
- golang
---

本想写一篇关于 golang 中 map 底层的文章， 但是发现已经了相当不错的文章 -- 
[字节跳动技术团队 - Golang 中 map 探究](https://mp.weixin.qq.com/s/UT8tydajjOUJkfc-Brcblw) 
这里只补充一下，缺少的 map 的删除操作

## 内部数据结构

## 初始化

map 是一个有"包含内容"的数据结构, 使用之前需要提前初始化, 即调用`make`

真正是调用源码是 [runtime.makemap](https://cs.opensource.google/go/go/+/master:src/runtime/map.go;l=283;bpv=1;bpt=1?q=makemap&ss=go%2Fgo)
## 获取数据


## 删除

[源码地址](https://cs.opensource.google/go/go/+/master:src/runtime/map_fast64.go;drc=3e5c2c155645ebaed62e4481430c455045b0fff5;bpv=1;bpt=1;l=273?q=mapdelete_fast64&ss=go%2Fgo)

删除的关键代码
``` go 

			// Only clear key if there are pointers in it.
			// # 当 Key 是指针类型的时候会去清空指针
			if t.key.ptrdata != 0 {
				if goarch.PtrSize == 8 {
					*(*unsafe.Pointer)(k) = nil
				} else {
					// There are three ways to squeeze at one ore more 32 bit pointers into 64 bits.
					// Just call memclrHasPointers instead of trying to handle all cases here.
					memclrHasPointers(k, 8)
				}
			}
			
			e := add(unsafe.Pointer(b), dataOffset+bucketCnt*8+i*uintptr(t.elemsize))
			// # 当 Value 为指针类型的时候, 指针为空, 解除引用 -> GC
			if t.elem.ptrdata != 0 {
				memclrHasPointers(e, t.elem.size)
			} else {
				memclrNoHeapPointers(e, t.elem.size)
			}
		
			// # 讲 hash 值标记为空
			b.tophash[i] = emptyOne
```

上述删除代码操作现象

- 当`map`的`value`类型中包含引用类型, 删除对应的`key`之后, 经过GC就会释放占用的内存
- 当`map`的`value` 类型不包含引用类型, 删除对应的`key`之后, GC无法释放类型

可以查看我自己的实验结果 {{}}

{{< ref "/blog/golang-memory-analyze-with-runtime.md" }}

## 扩容

## 参考
[Golang 中 map 探究](https://mp.weixin.qq.com/s/UT8tydajjOUJkfc-Brcblw)
