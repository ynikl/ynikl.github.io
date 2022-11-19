---
title: "如何查看 golang 编译之后调用的源码方法"
date: 2022-09-18T21:37:55+08:00
publishDate: 2022-09-18T21:37:55+08:00
draft: false
tags:
- golang
---

在 golang 中查看源码是比较方便的. 可以直接到 [官方包文档](https://pkg.go.dev/)中直接查看文档和跳转到源码

但是, 当我们想看一些更加底层的实现方法时, 就需要知道编译器将对应的方法编译成
什么底层方法了. 

比如, 我知道一些`make(map[int]bool)`是怎么实现的.

这时候就需要一些方法了. 引用一下[鸟窝大佬的文章](https://colobu.com/2018/12/29/get-assembly-output-for-go-programs/)
总结一下三种方法:

- `go tool compile -N -l -S makemap.go`
- `go tool compile -N -l -S makemap.go`
- `go tool compile -N -l -S makemap.go`

`go tool compile` 产生的汇编代码

```
	0x001c 00028 (main.go:5)	FUNCDATA	$1, gclocals·PdpXzE88ETLbqQ9okAZ04w==(SB)
	0x001c 00028 (main.go:5)	FUNCDATA	$2, main.main.stkobj(SB)
	0x001c 00028 (main.go:6)	STP	(ZR, ZR), main..autotmp_4-48(SP)
	0x0020 00032 (main.go:6)	STP	(ZR, ZR), main..autotmp_4-32(SP)
	0x0024 00036 (main.go:6)	STP	(ZR, ZR), main..autotmp_4-16(SP)
	0x0028 00040 (main.go:6)	MOVD	$type.map[int]bool(SB), R0
	0x0030 00048 (main.go:6)	MOVD	$100, R1
	0x0034 00052 (main.go:6)	MOVD	$main..autotmp_4-48(SP), R2
	0x0038 00056 (main.go:6)	PCDATA	$1, ZR
	0x0038 00056 (main.go:6)	CALL	runtime.makemap(SB)
	0x003c 00060 (main.go:6)	MOVD	R0, main.mp-112(SP)
	0x0040 00064 (main.go:7)	MOVD	R0, R1
	0x0044 00068 (main.go:7)	MOVD	ZR, R2
	0x0048 00072 (main.go:7)	MOVD	$type.map[int]bool(SB), R0
```

`go tool objdump`产生的汇编代码

```
  main.go:6		0x10008a734		a907ffff		STP (ZR, ZR), 120(RSP)			
  main.go:6		0x10008a738		90000160		ADRP 180224(PC), R0			
  main.go:6		0x10008a73c		91100000		ADD $1024, R0, R0			
  main.go:6		0x10008a740		d2800c81		MOVD $100, R1				
  main.go:6		0x10008a744		910163e2		ADD $88, RSP, R2			
  main.go:6		0x10008a748		97fe0522		CALL runtime.makemap(SB)		
  main.go:6		0x10008a74c		f90023e0		MOVD R0, 64(RSP)			
```

`go build -gcflags -S` 产生的汇编代码

```
mp_10-32(SP)
        0x0024 00036 (/Users/ian/play/map/main.go:6)    STP     (ZR, ZR), main..autotmp_10-16(SP)
        0x0028 00040 (/Users/ian/play/map/main.go:6)    MOVD    $type.map[int]bool(SB), R0
        0x0030 00048 (/Users/ian/play/map/main.go:6)    MOVD    $100, R1
        0x0034 00052 (/Users/ian/play/map/main.go:6)    MOVD    $main..autotmp_10-48(SP), R2
        0x0038 00056 (/Users/ian/play/map/main.go:6)    PCDATA  $1, ZR
        0x0038 00056 (/Users/ian/play/map/main.go:6)    CALL    runtime.makemap(SB)
        0x003c 00060 (/Users/ian/play/map/main.go:6)    MOVD    R0, main.mp-72(SP)
```

大同小异, 根据源代码的行号(`mian.go:6`) 都可以从代码中看到,
调用`call` 了`makemap` 这个方法

我们在到[源码](https://cs.opensource.google/go/go/+/master:src/runtime/map.go;l=283?q=makemap&ss=go%2Fgo)中, 找到`makemap`方法, 就可以查看对应的源码了


## 参考

[得到Go程序的汇编代码的方法](https://colobu.com/2018/12/29/get-assembly-output-for-go-programs/)
