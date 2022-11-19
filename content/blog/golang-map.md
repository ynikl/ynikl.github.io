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

```
type hmap struct {

   // map中存入元素的个数， golang中调用len(map)的时候直接返回该字段
   count     int
   // 状态标记位，通过与定义的枚举值进行&操作可以判断当前是否处于这种状态
   flags     uint8
   B         uint8  // 2^B 表示bucket的数量， B 表示取hash后多少位来做bucket的分组
   noverflow uint16 // overflow bucket 的数量的近似数
   hash0     uint32 // hash seed （hash 种子） 一般是一个素数

   buckets    unsafe.Pointer // 共有2^B个 bucket ，但是如果没有元素存入，这个字段可能为nil
   oldbuckets unsafe.Pointer // 在扩容期间，将旧的bucket数组放在这里， 新buckets会是这个的两倍大
   nevacuate  uintptr        // 表示已经完成扩容迁移的bucket的指针， 地址小于当前指针的bucket已经迁移完成

   extra *mapextra // optional fields
}
```

## 初始化

map 是一个有"包含内容"的数据结构, 使用之前需要提前初始化, 即调用`make`

真正是调用源码是 [runtime.makemap](https://cs.opensource.google/go/go/+/master:src/runtime/map.go;l=283;bpv=1;bpt=1?q=makemap&ss=go%2Fgo)

## 获取数据


## 删除数据

``` go
func mapdelete_fast64(t *maptype, h *hmap, key uint64) {
	if h == nil || h.count == 0 {
		return
	}
	if h.flags&hashWriting != 0 {
		fatal("concurrent map writes")
	}

	hash := t.hasher(noescape(unsafe.Pointer(&key)), uintptr(h.hash0))

	// Set hashWriting after calling t.hasher for consistency with mapdelete
	h.flags ^= hashWriting

	bucket := hash & bucketMask(h.B)
	if h.growing() {
		growWork_fast64(t, h, bucket)
	}
	b := (*bmap)(add(h.buckets, bucket*uintptr(t.bucketsize)))
	bOrig := b
search:
	for ; b != nil; b = b.overflow(t) {
		for i, k := uintptr(0), b.keys(); i < bucketCnt; i, k = i+1, add(k, 8) {
			if key != *(*uint64)(k) || isEmpty(b.tophash[i]) {
				continue
			}
			// Only clear key if there are pointers in it.
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
			if t.elem.ptrdata != 0 {
				memclrHasPointers(e, t.elem.size)
			} else {
				memclrNoHeapPointers(e, t.elem.size)
			}
			b.tophash[i] = emptyOne
			// If the bucket now ends in a bunch of emptyOne states,
			// change those to emptyRest states.
			if i == bucketCnt-1 {
				if b.overflow(t) != nil && b.overflow(t).tophash[0] != emptyRest {
					goto notLast
				}
			} else {
				if b.tophash[i+1] != emptyRest {
					goto notLast
				}
			}
			for {
				b.tophash[i] = emptyRest
				if i == 0 {
					if b == bOrig {
						break // beginning of initial bucket, we're done.
					}
					// Find previous bucket, continue at its last entry.
					c := b
					for b = bOrig; b.overflow(t) != c; b = b.overflow(t) {
					}
					i = bucketCnt - 1
				} else {
					i--
				}
				if b.tophash[i] != emptyOne {
					break
				}
			}
		notLast:
			h.count--
			// Reset the hash seed to make it more difficult for attackers to
			// repeatedly trigger hash collisions. See issue 25237.
			if h.count == 0 {
				h.hash0 = fastrand()
			}
			break search
		}
	}

	if h.flags&hashWriting == 0 {
		fatal("concurrent map writes")
	}
	h.flags &^= hashWriting
}
```

## 重新扩容

## Sync map 的基本使用

## 参考
[Golang 中 map 探究](https://mp.weixin.qq.com/s/UT8tydajjOUJkfc-Brcblw)
