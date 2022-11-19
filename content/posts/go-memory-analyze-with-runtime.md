---
title: "go 使用 runtime 包进行内存占用分析"
date: 2022-11-18T19:27:35+08:00
publishDate: 2022-11-18T19:27:35+08:00
draft: false
tags:
- golang
---

### 使用场景

写个demo, 想查看一下程序内部的内存占用情况.

### 使用方法

主角 runtime 包

- 对象 `MemStats`
- 方法 `ReadMemStats`

demo 展示

``` go
// PrintMemUsage outputs the current, total and OS memory being used. As well as the number 
// of garage collection cycles completed.
func PrintMemUsage() {
	bToMb := func(b uint64) uint64 {
		return b / 1024 / 1024
	}
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	// For info on each, see: https://golang.org/pkg/runtime/#MemStats
	fmt.Printf("Alloc = %v MiB", bToMb(m.Alloc))
	fmt.Printf("\tTotalAlloc = %v MiB", bToMb(m.TotalAlloc))
	fmt.Printf("\tSys = %v MiB", bToMb(m.Sys))
	fmt.Printf("\tNumGC = %v\n", m.NumGC)
}
```
[代码出处](https://gist.github.com/j33ty/79e8b736141be19687f565ea4c6f4226)

### 解释

#### ReadMemStats

`runtime.ReadMemStats` 方法会读取到内存分配器的当前时刻最新的内存分配数据, 并将
其填充到传入参数的`MemStats` 的变量中. 

可以充当一个内存快照, 用于进行对比.


#### MemStats 字段

``` go
type MemStats struct {

	// 当前堆上对象的内存分配大小, 同HeapAlloc字段, 单位 bytes
	Alloc uint64

	// 历史总的累计分配内存大小
	TotalAlloc uint64

	// 从操作系统分配的内存大小
	Sys uint64

	// 记录指针索引性能, go 语言内部使用
	Lookups uint64

	// 堆上分配的对象数量
	Mallocs uint64

	// 堆上剩余的内存大小
	Frees uint64

	HeapAlloc uint64

	// 从操作系统分配的 堆 内存大小
	HeapSys uint64

	// 未使用的空闲内存分片大小 spans
	HeapIdle uint64

	// 使用中的内存分片大小
	HeapInuse uint64

	// 回退的内存大小
	HeapReleased uint64

	// 堆上分配的对象数量
	HeapObjects uint64

	// 栈上使用的内存片大小
	StackInuse uint64

	// 从操作系统分配的栈的内存大小
	StackSys uint64

	// MSpanInuse is bytes of allocated mspan structures.
	MSpanInuse uint64

	// MSpanSys is bytes of memory obtained from the OS for mspan
	// structures.
	MSpanSys uint64

	// MCacheInuse is bytes of allocated mcache structures.
	MCacheInuse uint64

	// MCacheSys is bytes of memory obtained from the OS for
	// mcache structures.
	MCacheSys uint64

	// BuckHashSys is bytes of memory in profiling bucket hash tables.
	BuckHashSys uint64

	// GCSys is bytes of memory in garbage collection metadata.
	GCSys uint64

	// OtherSys is bytes of memory in miscellaneous off-heap
	// runtime allocations.
	OtherSys uint64

	// 在多大的堆内存时, 触发GC
	NextGC uint64

	// 上次GC 时间
	LastGC uint64

	// PauseTotalNs is the cumulative nanoseconds in GC
	// stop-the-world pauses since the program started.
	//
	// During a stop-the-world pause, all goroutines are paused
	// and only the garbage collector can run.
	PauseTotalNs uint64

	// PauseNs is a circular buffer of recent GC stop-the-world
	// pause times in nanoseconds.
	//
	// The most recent pause is at PauseNs[(NumGC+255)%256]. In
	// general, PauseNs[N%256] records the time paused in the most
	// recent N%256th GC cycle. There may be multiple pauses per
	// GC cycle; this is the sum of all pauses during a cycle.
	PauseNs [256]uint64

	// PauseEnd is a circular buffer of recent GC pause end times,
	// as nanoseconds since 1970 (the UNIX epoch).
	//
	// This buffer is filled the same way as PauseNs. There may be
	// multiple pauses per GC cycle; this records the end of the
	// last pause in a cycle.
	PauseEnd [256]uint64

	// GC 次数
	NumGC uint32

	// 手动调用 GC 的次数
	NumForcedGC uint32

	// GC 使用的 CPU 时间
	GCCPUFraction float64

	// 可以GC,一直是true
	EnableGC bool

	// BySize reports per-size class allocation statistics.
	//
	// BySize[N] gives statistics for allocations of size S where
	// BySize[N-1].Size < S ≤ BySize[N].Size.
	//
	// This does not report allocations larger than BySize[60].Size.
	BySize [61]struct {
		// Size is the maximum byte size of an object in this
		// size class.
		Size uint32

		// Mallocs is the cumulative count of heap objects
		// allocated in this size class. The cumulative bytes
		// of allocation is Size*Mallocs. The number of live
		// objects in this size class is Mallocs - Frees.
		Mallocs uint64

		// Frees is the cumulative count of heap objects freed
		// in this size class.
		Frees uint64
	}
}

```

[源码出处](https://pkg.go.dev/runtime#MemStats)


### 我用于分析 map 的 delete 操作占用内存

``` go
package main

import (
	"fmt"
	"runtime"
)

func main() {

	m := make(map[string][]int)
	m["a"] = make([]int, 1*1024*1024*1024)
	PrintMemUsage()
	delete(m, "a")
	runtime.GC()
	PrintMemUsage()

	fmt.Println("make storage value")

	mint := make(map[int]int, 1<<30)
	for i := 0; i < 1*1024*2; i++ {
		mint[i] = 1024
	}
	PrintMemUsage()
	runtime.GC()
	PrintMemUsage()

	// 注意: 需要引用, 避免被提前回收
	fmt.Println(len(m), len(mint))

	// go run main.go
	// Outpu:
	//
	// Alloc = 8192 MiB	TotalAlloc = 8192 MiB	Sys = 8464 MiB	NumGC = 0
	// Alloc = 0 MiB	TotalAlloc = 8192 MiB	Sys = 8465 MiB	NumGC = 2
	// make storage value
	// Alloc = 39168 MiB	TotalAlloc = 47360 MiB	Sys = 48898 MiB	NumGC = 3
	// Alloc = 39168 MiB	TotalAlloc = 47360 MiB	Sys = 48898 MiB	NumGC = 4
	// 0 2048
}

func PrintMemUsage() {
	bToMb := func(b uint64) uint64 {
		return b / 1024 / 1024
	}
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	// For info on each, see: https://golang.org/pkg/runtime/#MemStats
	fmt.Printf("Alloc = %v MiB", bToMb(m.Alloc))
	fmt.Printf("\tTotalAlloc = %v MiB", bToMb(m.TotalAlloc))
	fmt.Printf("\tSys = %v MiB", bToMb(m.Sys))
	fmt.Printf("\tNumGC = %v\n", m.NumGC)
}
```

分析结果得出, 如果在 map 中存储的 value, 如果是引用值的话, 占用的内存是被 GC 回收
的. 但是, 如果是值类型如简单的`int`是不会被回收的.
