---
title: "Go-互斥锁的实现"
date: 2022-05-29T20:52:35+08:00
draft: false
tags:
- go
---

## Mutex 数据结构

``` go
type Mutex struct {
	state int32
	sema uint32
}
```

- Mutex 使用过之后是不可被拷贝的
- state 等于 0 值的时候才是无锁的状态
- sema 字段为信号量字段，通过该字段控制协程的阻塞和唤醒，具体实现在runtime 中。

Mutex 对象总共有三个公开方法

- Lock 尝试抢占互斥锁，如果已经被锁定，则调用协程进入阻塞
- TryLock
- Unlock 解除互斥锁， **解锁未锁定的互斥锁会发生panic**

Mutex 与协程无法关，允许一个协程锁定，另一个协程进行解锁。

Mutex 实现了一个 `sync.Locker` 接口, 该接口只有两个方法

- Lock
- Unlock

Mutex 锁有几种状态

- mutexLocked = 1 已经锁定 
- mutexWoken = 2  表示当前锁的等待队列，有协程正在活跃地获取锁，可以考虑不用释放信号量
- mutexStarving = 4 当前锁已经进入了饥饿状态

其他常量
- mutexWaiterShift = 3 统计的等待在`Mutex.state`字段等待数量。（前3位，用于表示锁的状态, 即 mutexLocked, mutexWoken, mutexStarving）
- starvationThresholdNs = 1e6 进入饥饿模式的阈值 1ms

## Mutex 锁的竞争方式

Mutex 锁有两种状态

- 正常模式 normal
- 饥饿模式 starvation

正常模式下，等待获取的锁的协程遵循先进先出的原则。

但是，当释放锁的时候如果有新协程进入获取锁代码的时候。因为新入协程本身已经运行在CPU上了，所以有抢占锁的优势。由于阻塞在等待队列上的协程，竞争不过新入协程。当等待时间操作 1ms，就会触发饥饿模式。

饥饿模式，进行互斥锁解锁的协程直接的将锁的所有权直接已经给等待队列的第一个协程。新进协程被禁止抢占互斥锁。

在转移所有权的时候，如果满足一下任意条件，则进入正常模式：

- 锁的等待者只剩最后一个
- 等待时间小于1ms

正常模式有利于更好的性能，饥饿模式则避免出现“饿死”情况。
 
## Mutex 的方法详解

### Lock

第一步，通过调用 atomic 的 CAS 操作，尝试加锁，如果成功加锁直接返回
```
	atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked)
```

如果第一步失败，则代表该锁已经被加过锁，锁定了。

```
func (m *Mutex) lockSlow() {
	// 当前协程的变量, 可以用于表示当前协程的状态
	// 用于统计锁的等待时长，是否进入饥饿模式
	var waitStartTime int64
	starving := false // 当前协程是否处于饥饿
	awoke := false // 是否处于唤醒
	iter := 0 // 统计自旋次数
	old := m.state
	for {
		// 进入自旋的状态条件， **已经锁定** 且非饥饿状态。
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			// 如果锁的状态 woken 字段未被标记， 将自身标记位唤醒，且将 Mutex 的 woken 位标记位1
			// 当协程自己进入获取锁的第一候选人
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				awoke = true
			}
			runtime_doSpin()
			// 控制自旋次数，大于4次之后不进入自旋状态
			iter++
			// 获取最新的状态
			old = m.state
			continue
		}
		// 有可能，自旋自后已经解锁或者只是单纯不能自旋限制了。下面尝试通过 CAS 竞争锁。
		// 新值用于设置新的状态
		new := old
		// 非饥饿状态才设置锁定
		if old&mutexStarving == 0 {
			new |= mutexLocked
		}
		if old&(mutexLocked|mutexStarving) != 0 {
			// 等待者加1
			new += 1 << mutexWaiterShift
		}
		// 当前协程是饥饿状态，尝试标记锁的新状态位饥饿状态。
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving
		}
		if awoke {
			// The goroutine has been woken from sleep,
			// so we need to reset the flag in either case.
			if new&mutexWoken == 0 {
				// Mutex 的唤醒位被抢走，出现不一致。协程变量的唤醒位应该与 Mutex 的唤醒位一致
				throw("sync: inconsistent mutex state")
			}
			// 标志 锁的唤醒位为0
			new &^= mutexWoken
		}
		// CAS 尝试, Mutex 状态没有被变更
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			// 非饥饿状态下，CAS 成功新的 new 字段中会有 mutexLocked 标记（在new下的第一个if）。当前协程获取到了互斥锁,
			if old&(mutexLocked|mutexStarving) == 0 {
				break // locked the mutex with CAS
			}
			// queueLifo 表示该协程是否为第一次获取锁。如果中间被唤醒过，这放在等待队列头部
			queueLifo := waitStartTime != 0
			if waitStartTime == 0 {
				waitStartTime = runtime_nanotime()
			}
			// 通过信号量，进入阻塞 
			runtime_SemacquireMutex(&m.sema, queueLifo, 1)
			// 进入饥饿模式
			starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
			old = m.state
			// 当前是饥饿模式
			if old&mutexStarving != 0 {
				// 如果当前锁的状态位饥饿模式，但是当前协程可以执行的当前行代码，代表当前协程已经被从阻塞中唤醒。
				// 饥饿状态 + 被唤醒 =》当前锁已经已经到当前协程上
				if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
					// 检查状态一致
					throw("sync: inconsistent mutex state")
				}
				// 由当协程来设置最新的锁定状态
				delta := int32(mutexLocked - 1<<mutexWaiterShift)
				// 判断是否需要退出饥饿模式
				if !starving || old>>mutexWaiterShift == 1 {
					delta -= mutexStarving
				}
				atomic.AddInt32(&m.state, delta)
				break
			}
			// 当前是正常模式，通过CAS自由竞争锁。
			awoke = true
			iter = 0 // 重置计数
		} else {
			// 再来
			old = m.state
		}
	}
}
```

是否可以进入自旋状态代码解释[源码地址](https://cs.opensource.google/go/go/+/master:src/runtime/proc.go;l=6175?q=sync_runtime_canSpin&ss=go%2Fgo)


### Unlock

第一步直接减去 mutexLocked 标志位常量，如果 new 等于0结束——简单（锁一次开一次，easy）。
```
new := atomic.AddInt32(&m.state, -mutexLocked)
```

如果 state 还不等于0（有协程等待，竞争锁）, 进入 unlockSlow
```
func (m *Mutex) unlockSlow(new int32) {
	// 上一步已经减过 locked 位， 在加上应该等于1，否则不是正常解锁（解锁未锁定的锁）。
	if (new+mutexLocked)&mutexLocked == 0 {
		fatal("sync: unlock of unlocked mutex")
	}
	
	// 正常模式
	if new&mutexStarving == 0 {
		old := new
		for {
			// If there are no waiters or a goroutine has already
			// been woken or grabbed the lock, no need to wake anyone.
			// In starvation mode ownership is directly handed off from unlocking
			// goroutine to the next waiter. We are not part of this chain,
			// since we did not observe mutexStarving when we unlocked the mutex above.
			// So get off the way.
			// 没有等待协程无需通过信号量唤醒
			// 1. 如果 mutexLocked 位为1，则代表锁已经被新入协程获取。
			// 2. mutexWoken 代表协程有协程正在活动，无需再释放信号量
			// 3. mutexStarving 锁的状态一直被抢占，才会导致当前位饥饿状态，无需在释放信号量
			if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
				return
			}
			// Grab the right to wake someone.
			new = (old - 1<<mutexWaiterShift) | mutexWoken
			if atomic.CompareAndSwapInt3(&m.state, old, new) {
				// 信号量唤醒，各自竞争
				runtime_Semrelease(&m.sema, false, 1)
				return
			}
			old = m.state
		}
	} else {
		// 饥饿模式，通过信号量直接移交当前CPU时间
		runtime_Semrelease(&m.sema, true, 1)
	}
}2
```

如何把锁移交给等待队列的协程？ 

使用 `Mutex.sema` 信号量实现锁转移


## 参考
[源代码地址sync.mutex.go](https://cs.opensource.google/go/go/+/refs/tags/go1.18:src/sync/mutex.go;bpv=1;bpt=1)
[包说明文档](https://pkg.go.dev/sync#Mutex)
