---
title: "Channel"
date: 2022-04-02T10:09:22+08:00
draft: true
---

# 关于 Go 中 channel 中用法和实现总结

## channel 

## channel 的实现

channel 在内部实现的结构体为 `runtime.hchan`

1. 有一个环形链表, 暂存要传输的数据. 无 buffer 的channel 该队列长度为0, 所以不进行暂存数据. 
2. 有一把互斥锁`mutex`, 在并发情况下, 保护自身数据结构的一致性
3. 有两个协程等待链表, 用于挂载因为发送/接收而阻塞在该 channel 上的协程

``` go
type hchan struct {
	qcount   uint           // 当前 buffer 中有暂存着多少个数据
	dataqsiz uint           // 环形数据的buffer个数, 由 make 初始化的时候第二个参数容量决定的
	buf      unsafe.Pointer // 环形数据开始地址
	elemsize uint16         // channel 传输的元素大小, 用于计算内存大小
	closed   uint32         // channel 是否已经关闭 0未关闭, 非0关闭
	elemtype *_type         // element type # channel 元素的类型
	sendx    uint           // 环形链表中, 发送数据存储的下标
	recvx    uint           // 环形链表中, 接受数据获取数据的下标
	recvq    waitq          // 阻塞在该 channel 等待获取数据的 Groutine 列表
	sendq    waitq          // 阻塞在该 channel 等待写入数据的 Groutine 列表

	lock mutex // # 互斥锁 用于保护自身数据变更
}
```

### 初始化

内部实现函数`runtime.makechan`

``` go
// 以下函数被我精简过
func makechan(t *chantype, size int) *hchan {

	// compiler checks this but be safe.
	// 编译器会校验channel元素的大小, 小于64KB. 若大于64KB, 编译器会报错
	if elem.size >= 1<<16 {
		throw("makechan: invalid channel element type")
	}
	if hchanSize%maxAlign != 0 || elem.align > maxAlign {
		throw("makechan: bad alignment")
	}

	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	// Hchan does not contain pointers interesting for GC when elements stored in buf do not contain pointers.
	// buf points into the same allocation, elemtype is persistent.
	// SudoG's are referenced from their owning thread so they can't be collected.
	// TODO(dvyukov,rlh): Rethink when collector can move allocated objects.
	var c *hchan
	switch {
	case mem == 0:
		// Queue or element size is zero. # 无缓冲队列或者空结构体为传递值
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		// Race detector uses this location for synchronization.
		c.buf = c.raceaddr()
	case elem.ptrdata == 0: // # channel 元素不存在指针引导数据, 将环形数据分配在 hchan 后面
		// Elements do not contain pointers.
		// Allocate hchan and buf in one call.
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		// Elements contain pointers.
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}

	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)
	lockInit(&c.lock, lockRankHchan) // # locakRankHchan 锁的等级

	if debugChan {
		print("makechan: chan=", c, "; elemsize=", elem.size, "; dataqsiz=", size, "\n")
	}
	return c
}
```

### 发送数据

### 接收数据

### 关闭

## 用户总结


