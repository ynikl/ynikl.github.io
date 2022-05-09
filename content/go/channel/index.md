---
title: "Channel"
date: 2022-04-02T10:09:22+08:00
draft: false
tags:
	- go
---

# Go 中 channel 中用法和实现总结

以下分析和源码都是基于 go1.17 版本

## channel 简介

Go 语言的基础类型之一, 用于在协程与协程之间传递数据 (**channel 数据的传输方式也是值传递, Go语言的数据传输只有值传递**)

> Do not communicate by sharing memory; instead, share memory by communicating.

channel 保证:

1. 数据的先入先出
2. 并发情况下的数据安全
3. 已经关闭的 channel 不可重开

## channel 的实现

channel 在内部实现的结构体为 `runtime.hchan`

1. 有一个环形链表, 暂存要传输的数据. 无 buffer 的channel 该队列长度为0, 所以不进行数据缓冲. 
2. 有一把互斥锁`mutex`, 在并发情况下, 保护自身数据结构的一致性
3. 有两个协程等待链表, 用于挂载因为**发送/接收**而阻塞在该 channel 上的协程

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

1. channel 传递的元素不能太大
2. 如果是空结构体或者无缓冲队列, 是不需要分配环形队列内存
3. 如果传递数据类型有内含指针, 需要将环形队列分配到堆上

内部实现函数`runtime.makechan`

``` go
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
		// Queue or element size is zero. # 无缓冲队列或者空结构体为传递值, 不需要额外分配队列内存
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

	return c
}
```

### 关闭

核心代码`runtime.closechan`更新自身数据结构中的关闭状态, 并 **唤醒阻塞在 channel 上的所有协程**. 被唤醒的协程(`sudog`)的 success 标识会被置为 false. 

被唤醒的 写操作的协程, 也会发生panic. ( "send on closed channel" )

自身操作会发生 panic 的情况
1. 未初始化 channel
2. 重复关闭 channel

``` go
func closechan(c *hchan) {
	if c == nil {
		// 未初始化的channel 会发生panic
		panic(plainError("close of nil channel"))
	}

	// # 开始关闭, 锁定之后数据都进不来了
	lock(&c.lock)
	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("close of closed channel"))
	}

	c.closed = 1

	var glist gList

	// release all readers # 唤醒所有因为读取数据阻塞的协程
	for {
		sg := c.recvq.dequeue()
		if sg == nil {
			break
		}
		if sg.elem != nil {
			typedmemclr(c.elemtype, sg.elem)
			sg.elem = nil
		}
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		// # [channel 唤醒协议总是设置params为sudog](https://github.com/golang/go/commit/30a68bfb806b5217932e280f5a5f521237e69077)
		gp.param = unsafe.Pointer(sg)
		sg.success = false
		if raceenabled {
			raceacquireg(gp, c.raceaddr())
		}
		glist.push(gp)
	}

	// release all writers (they will panic) # 唤醒所有因为写入数据阻塞的协程
	for {
		sg := c.sendq.dequeue()
		if sg == nil {
			break
		}
		sg.elem = nil
		if sg.releasetime != 0 {
			sg.releasetime = cputicks()
		}
		gp := sg.g
		gp.param = unsafe.Pointer(sg)
		sg.success = false
		if raceenabled {
			raceacquireg(gp, c.raceaddr())
		}
		glist.push(gp)
	}
	unlock(&c.lock)

	// Ready all Gs now that we've dropped the channel lock.
	for !glist.empty() {
		gp := glist.pop()
		gp.schedlink = 0
		// # 唤醒协程, 将协程加入调度
		goready(gp, 3)
	}
}
``` 


### 发送数据

**向已经关闭的 channel 发送数据会发生 panic**


数据流程: 

1. 检查是否已经初始化
2. 非阻塞写入数据, 检查数据是否已经满, 快速返回
3. 是否已经关闭
4. 检查 channel 中是否已经有等待获取数据而阻塞的协程,  如果有直接将数据发送给等待的协程.
5. channel 的 buffer 是否还有空间, 如果有将数据放置到 buffer 中, 返回
6. channel 的 buffer 已经满了, 根据是否为 select 操作, 判断是否需要将协程阻塞
7. 当协程阻塞之后,  在被唤醒之后需要再检查一次, channel 是否已经关闭.


``` go

// # block 的参数是由是否在 select 中, 由编译过程决定的; 只有在select语句中block = true
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	if c == nil {
		if !block {
			return false
		}
		// # 向未初始化的 channel 发送数据会永远阻塞
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	// # 带 select 的 channel 在数据已经满了情况直接返回
	if !block && c.closed == 0 && full(c) {
		return false
	}

	// 保护数据
	lock(&c.lock)

	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}

	// # 先从接受协程队列中获取阻塞的协程, 直接将数据发送给阻塞的协程
	if sg := c.recvq.dequeue(); sg != nil {
		// Found a waiting receiver. We pass the value we want to send
		// directly to the receiver, bypassing the channel buffer (if any).
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}

	// # channel 的 buffer 中还有剩余空间
	if c.qcount < c.dataqsiz {
		// Space is available in the channel buffer. Enqueue the element to send.
		qp := chanbuf(c, c.sendx)
		if raceenabled {
			racenotify(c, c.sendx, nil)
		}
		typedmemmove(c.elemtype, qp, ep)
		c.sendx++
		// # 环形队列, 当索引到最后从头开始
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		// # 增加当前 channel buffer 存储的数据个数
		c.qcount++
		unlock(&c.lock)
		return true
	}

	if !block {
		unlock(&c.lock)
		return false
	}

	// # 发送数据的协程阻塞在当前 channel
	// Block on the channel. Some receiver will complete our operation for us.
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// No stack splits between assigning elem and enqueuing mysg
	// on gp.waiting where copystack can find it.
	mysg.elem = ep
	mysg.waitlink = nil
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.waiting = mysg
	gp.param = nil
	c.sendq.enqueue(mysg)
	// Signal to anyone trying to shrink our stack that we're about
	// to park on a channel. The window between when this G's status
	// changes and when we set gp.activeStackChans is not safe for
	// stack shrinking.
	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
	// Ensure the value being sent is kept alive until the
	// receiver copies it out. The sudog has a pointer to the
	// stack object, but sudogs aren't considered as roots of the
	// stack tracer.
	KeepAlive(ep)

	// # 协程被唤醒了
	// someone woke us up.
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	closed := !mysg.success
	gp.param = nil
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	mysg.c = nil
	releaseSudog(mysg)
	// # 挂载在协程上的发送协程会 panic
	if closed {
		if c.closed == 0 {
			throw("chansend: spurious wakeup")
		}
		panic(plainError("send on closed channel"))
	}
	return true
}
```

### 接收数据

与发送数据一样, 同样带着是否阻塞的参数. 在编译时,由是否有 select 操作决定. 核心代码`runtime.chanrecv`

1. 不带 select 从未初始化的 channel 获取数据, 会永远阻塞
2. `runtime.chanrecv` 返回值中, 第一个返回值`selected`表示在,select 语句中, 该 case 是否会被选中执行

接收数据流程:
1. 检查是否已经初始化
2. 检查非阻塞获取数据下, 是否可以直接返回
3. 如果已经关闭的 channel 且没有已经没有缓冲数据, 返回数据类型的默认值. 
4. 检查有因发送数据阻塞在 channel 的协程, 如果没有 buffer, 直接从阻塞的协程中获取数据, 否则从 buffer 中获取数据数据, 将第一个阻塞的协程的数据放入 buffer 中.
5. 如果缓冲 buffer 有数据, 则从buffer 中获取数据.
6. 非阻塞操作, 直接返回. 否则协程进行阻塞.

注意事项:

**当 select 一个 已经关闭的 channel 的时候, 该 case 会被疯狂输出, 导致cpu使用率上升**

``` go
// chanrecv receives on channel c and writes the received data to ep.
// ep may be nil, in which case received data is ignored.
// If block == false and no elements are available, returns (false, false).
// Otherwise, if c is closed, zeros *ep and returns (true, false).
// Otherwise, fills in *ep with an element and returns (true, true).
// A non-nil ep must point to the heap or the caller's stack.
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	if c == nil {
		if !block {
		// select 情况下, selected = false, 不执行该 case
			return
		}
		// # 非 select 会永远阻塞
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	// # 不加锁检查, 带 select 接收操作, 如果 channel 未关闭, 且没有可以获取的数据直接返回
	// Fast path: check for failed non-blocking operation without acquiring the lock.
	if !block && empty(c) {
		// After observing that the channel is not ready for receiving, we observe whether the
		// channel is closed.
		//
		// Reordering of these checks could lead to incorrect behavior when racing with a close.
		// For example, if the channel was open and not empty, was closed, and then drained,
		// reordered reads could incorrectly indicate "open and empty". To prevent reordering,
		// we use atomic loads for both checks, and rely on emptying and closing to happen in
		// separate critical sections under the same lock.  This assumption fails when closing
		// an unbuffered channel with a blocked send, but that is an error condition anyway.
		if atomic.Load(&c.closed) == 0 {
			// Because a channel cannot be reopened, the later observation of the channel
			// being not closed implies that it was also not closed at the moment of the
			// first observation. We behave as if we observed the channel at that moment
			// and report that the receive cannot proceed.
			return
		}
		// The channel is irreversibly closed. Re-check whether the channel has any pending data
		// to receive, which could have arrived between the empty and closed checks above.
		// Sequential consistency is also required here, when racing with such a send.
		if empty(c) {
			// The channel is irreversibly closed and empty.
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			// select 会选择改 case 疯狂输出
			return true, false
		}
	}

	lock(&c.lock)

	if c.closed != 0 {
	// # 如果 channel 已经关闭, 且buffer 中已经没有数据了, 返回传输数据类型的默认值
		if c.qcount == 0 {
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			// # 在返回的时候有可能刚好有数据会进来, 所以需要进行加锁操作
			unlock(&c.lock)
			if ep != nil {
				typedmemclr(c.elemtype, ep)
			}
			return true, false
		}
		// The channel has been closed, but the channel's buffer have data.
	} else {
	
		// # 如果 channel 没有 buffer 会直接从阻塞中的发送数据协程中获取数据
		// # 如果 channel 有 buffer. 当可以获取到因为发送数据而阻塞的协程时, 代表缓冲的 buffer 已经满了. 所以, 将从 buffer 中获取数据, 并将获取到的第一个阻塞协程, 的数据放入 buffer 尾端. 保证先入先出
		// Just found waiting sender with not closed.
		if sg := c.sendq.dequeue(); sg != nil {
			// Found a waiting sender. If buffer is size 0, receive value
			// directly from sender. Otherwise, receive from head of queue
			// and add sender's value to the tail of the queue (both map to
			// the same buffer slot because the queue is full).
			recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
			return true, true
		}
	}

	// # 从 buffer 中获取数据
	if c.qcount > 0 {
		// Receive directly from queue
		qp := chanbuf(c, c.recvx)
		if raceenabled {
			racenotify(c, c.recvx, nil)
		}
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		typedmemclr(c.elemtype, qp)
		c.recvx++
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		c.qcount--
		unlock(&c.lock)
		return true, true
	}

	// # 非阻塞操作, 返回
	if !block {
		unlock(&c.lock)
		return false, false
	}

	// 将获取数据的协程阻塞
	// no sender available: block on this channel.
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// No stack splits between assigning elem and enqueuing mysg
	// on gp.waiting where copystack can find it.
	mysg.elem = ep
	mysg.waitlink = nil
	gp.waiting = mysg
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.param = nil
	c.recvq.enqueue(mysg)
	// Signal to anyone trying to shrink our stack that we're about
	// to park on a channel. The window between when this G's status
	// changes and when we set gp.activeStackChans is not safe for
	// stack shrinking.
	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)

	// someone woke us up
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	
	// 如果是因为 channel的关闭 操作唤醒的, success 值为 false
	success := mysg.success
	gp.param = nil
	mysg.c = nil
	releaseSudog(mysg)
	return true, success
}
```
## 用法总结

初始化:

1. 避免对未初始化 channel 的进行读写操作, 可能会造成阻塞
2. 在 select 语句中, 对已经关闭的 channel 可以赋予  `nil` 值, 避免 cpu 飙高


关闭协程:

1. 关闭协程的动作, 应该由数据写入方操作
2. channel 当参数传递时, 尽可能带上操作方向(读取/写入), 编译器会保证, 单向写入协程不允许关闭
3. 关闭的时候要确保所有的写入协程都已经操作完毕. 避免引起写入协程发生 panic


在 channel 中阻塞的协程, 唤醒条件

1. 到达协程数据操作的目标, 写入 / 读取数据
2. channel 关闭


## Referrences

1. [Go官方源码](https://cs.opensource.google/go/go/+/refs/tags/go1.17:src/runtime/chan.go)
2. [Share Memory By Communication](https://go.dev/blog/codelab-share)
