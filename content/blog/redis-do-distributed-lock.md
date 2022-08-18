---
title: "Redis 用于做分布式锁"
date: 2022-08-18T12:18:53+08:00
publishDate: 2022-08-18T12:18:53+08:00
draft: false
tags:
- redis
---

## 操作

## 演进

### 加锁后进程挂掉了

加锁成功之后, 进程挂掉了没有进行解锁操作. 
导致进入死锁状态.

引入 `expire` 设置超时时长, 自动释放 key
```
> setnx lock:key true 
> OK

> expire lock:key 5 

" ... do something critical ... 

> del lock:codehole
```

### 加锁动作 到 expire 之间挂掉了

redis 2.8 之后支持 `set` 拓展指令

```
> set lock:key true ex 5 nx
```

加锁互斥锁, 并同时设置超时时长

### 执行超时, 被其他进程获取到了锁

加锁之后, 本身进程执行时间超过了预先设置的 `expire` 的时间. 就会导致锁被提前释放.

解决方案:

1. 尽量不要用与锁住时间教长的任务, 尝试缩小锁定的"关键区域"
2. **续锁** (会使客户端负责化) : 起一个支线进程, 定期(在超时时间内) 检查锁和value(保证是自己的锁), 重新设置超时时间   


### 执行超时, 被其他进程获取到了锁之后, 超时进程误删其他进程的锁

> Redis 的分布式锁不能解决超时问题，如果在加锁和释放锁之间的逻辑执行的太长，以至 于超出了锁的超时限制，就会出现问题。因为这时候锁过期了，第二个线程重新持有了这把锁， 但是紧接着第一个线程执行完了业务逻辑，就把锁给释放了，第三个线程就会在第二个线程逻 辑执行完之间拿到了锁。
> -- Redis深度历险

解决方案: 给加锁的 key 设置随机数的value, 删除之前先匹配是否一致再删除

匹配和删除动作之间的原子性可以用 Lua 脚本保证
``` lua
# delifequals
if redis.call("get",KEYS[1]) == ARGV[1] then
	return redis.call("del",KEYS[1])
else
	return 0
end
```

### 可重入锁

利用线程的本地变量(Threadloacl), 维护锁的持有计数, 实现支持多次加锁, 多次解锁


## 参考
- *Redis 深度历险*
- [阿里二面：Redis分布式锁过期了但业务还没有执行完，怎么办](https://www.51cto.com/article/679902.html)



