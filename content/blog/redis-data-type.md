---
title: "Redis 中的数据类型"
date: 2022-05-15T23:06:25+08:00
draft: true
tags:
- redis
- data-structures
---

# 五种基本数据结构

- [String](#String)
- [List](##List)
- [Hash](##Hash)
- [Set](##Set)
- [Zset](##Zset)

## String

- 最大长度 512MB
- redis 的 Key 也是 string 类型

Commands

- set
- get
- incr

incr 会将string 识别成为整数, **原子性** 增加

SDS 动态字符串

### Bitmaps

数据结构也是string，基于string衍生出的bit操作， 主要目的是为了节省空间

Commands
- setbit
- getbit
- bitops
- bitcount
- bitpos

场景
- 用户签到
- 用户活跃用户统计
- 用户是否在线

使用示例
``` 
# 登记用户上线记录
# 设置 2022年05月18日  用户id 1308 登录过网站
> setbit 20220518 1308 1

# 获取连续两条都有登录的用户集
> bitop AND login:2022051819 20220518 20220519

# 获取用户id
> bitpos login:2022051819 1

# 统计数量
> bitcount login:2022051819  

# 还是string类型
> typp bitmap:1
"string"

```

获取一个全部为1的key
```
# 设置100位为0， 一个字节8位， 此时已经已经分配 13byte， 104位
> setbit bitmap:all:0 100 0

# 获取到一个 104位 的 全1 位图 
> bitop NOT bitmap:all:1 bitmap:all:0
```

## List

链表实现

- 添加元素时是 O(1)

Commands
- rpush
- rpop
- lpush
- lpop
- lrange
- ltrim
- blpop 阻塞操作
- brpop
- llen

使用场景
- 记录用户访问网络
- 两个程序的“生产-消费者”模式

## Hash

方便用于表现"对象"

Commands
- hmset
- hmget
- hincrby

## Set

无序集合, 

与 List 的不同之处，

- Set 的值是唯一的
- 无序

Commands
- sadd
- smember
- sismember
- sinter
- sunionstore 
- scard (cardinality)

## Sorted Set / ZSet

每个元素都有附加带有 浮点数分数 

- 分数越大时，值越大，排序越靠后
- 分数相当，使用字符顺序

Commands
- zadd
- zrange
- zrank
- zremrangebyscore

# 底层实现

## SDS 字符串

``` c

/*
 * 保存字符串对象的结构
 */
struct sdshdr {
    
    // buf 中已占用空间的长度
    int len;

    // buf 中剩余可用空间的长度
    int free;

    // 数据空间
    char buf[];
};

```

为什么使用SDS

- 快速获取长度
- C 语言中避免缓冲区溢出
- 方便修改，避免重新分配
- free 字段可以帮助实现"惰性回收内存"
- 二进制安全， 不依赖`\0`

## Linked List

[链表]()

是数据类型 List 的实现方法之一， 

节点重排能力？


