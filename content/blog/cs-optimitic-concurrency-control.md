---
title: "乐观锁和悲观锁"
date: 2022-10-02T14:05:43+08:00
publishDate: 2022-10-02T14:05:43+08:00
draft: false
tags:
- concurrency
---

# 乐观锁和悲观锁

首先, 乐观锁和悲观锁和本身并不是一种具体锁.

而是一种编程的并发控制思想. 
原名应该叫做乐观并发控制(Optimistic concurrency control) 简称 OCC 和
悲观并发控制(Pessimistic Concurrency Control) 简称 PCC

## 什么是锁

[维基百科对锁的定义](https://en.wikipedia.org/wiki/Lock_(computer_science))
>In computer science, a lock or mutex (from mutual exclusion) is a synchronization primitive: a mechanism that enforces limits on access to a resource when there are many threads of execution. A lock is designed to enforce a mutual exclusion concurrency control policy, and with a variety of possible methods there exists multiple unique implementations for different applications. 

简单表述一下: 锁是一个同步原语, 是一种控制访问资源的线程的手段


## 乐观锁

乐观锁是对于要锁定的的访问资源或变量, 持有乐观的态度 -- 即在自己访问该变量的时候, 
不会有其他线程来访问该变量. 

主要思想是在写入数据的时候, 对比一下, 当前变量的值是不是与自己取出来的时候是一致,
如果一致即表示着 **数据没有被其他线程修改过**

有两种具体的策略

- 版本号
- CAS

### 版本号

在每一次对加锁数据进行修改时候的, 对版本号进行增加操作. 当回写的数据时候判断版本号
是否一致. 

如果保持一致, 才会继续进行操作.

### CAS

利用CPU硬件层面支持 -- 比较和写入两步为原子性. 直接对当前值进行判断, 是与取出的数
据一致. 一致才继续进行操作.

利用CAS, 自增完成数字自增的[伪代码](https://en.wikipedia.org/wiki/Compare-and-swap#Example%20application:%20atomic%20adder)
```
function add(p: pointer to int, a: int) returns int
    done ← false

    while not done
        value ← *p  // Even this operation doesn't need to be atomic.
        done ← cas(p, value, value + a)

    return value + a
```
如果一直失败的话, cpu就会保持自旋 -- 对cpu算力消耗较大, 直至成功.


### ABA 问题

在乐观锁中, 如果值没有变化, 它的背后含义代表该值没有对其他线程修改过. 

但是存在着这种情况. 

1. 线程1, 取值 A
2. 线程2, 取值 A
3. 线程2, 修改 B 值 -- 成功
4. 线程2, 取值 B, 再修改成 A -- 成功
5. 线程1, 对比 A值, 一致

修改的对象值已经被其他对象修改过, 但又被修改成旧的值. 对于 ABA 问题有没有危害,要
看具体的业务场景

如果使用版本号, 每一次修改值, 都增加版本号, 就可以避免该问题.

## 悲观锁

悲观锁, 认为自己取值之后, 一定会有其他线程过来修改自己取值的对象.
采取保守策略 -- 直接对该数据进行锁定.

按对数据的锁定类型, 可以分成两种锁: 

- 互斥锁
- 读写锁

### 互斥锁

对数据锁定期间, 不允许其他线程的访问 -- 读取也不允许. 其他线程只能等待当前的线程
执行完毕

常见的即是各种语言自带的互斥锁. 

### 读写锁

数据锁定期间, 其他线程可以读取数据, 但是不能写入数据. 

常见的也是各种语言的读写锁.


## 参考
- [乐观锁、悲观锁，这一篇就够了！](https://segmentfault.com/a/1190000016611415)
- [ABA问题](https://en.wikipedia.org/wiki/ABA_problem)
- [锁的定义](https://en.wikipedia.org/wiki/Lock_(computer_science))
