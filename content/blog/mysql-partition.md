---
title: "Mysql 分区"
date: 2023-04-27T16:16:48+08:00
publishDate: 2023-04-27T16:16:48+08:00
draft: false
tags:
- mysql
---

## 可以解决什么问题

- 表太大, 无法全部放入内存中
- 表有热点数据, 其他均是历史数据
- 分区更容易维护, 批量删除,修复
- 可以跨多个硬件设备
- 减少单个索引互斥访问
- 独立备份和恢复分区

主要目的就是对表, 进行一个粗粒度的过滤;

## 原理

将一个表在物理上分层多个更小的部分, 但是在逻辑上, 仍是只有一个表.  

执行 SQL 时, 可以通过合适的过滤今天, 过滤掉那边不需要查询的分区, 以此提高性能.

支持类型为: 水平分区, 不支持垂直分区;

### 分区的数据操作流程

先打开并锁住所有的分区底层表, 过滤掉多余的分区. 再进行操作. 

## 分区类型

- range
- list
- hash
- key

创建分区时, 分区的列, 必须是主键或者唯一索引的一部分

```
create table t1 (
col1 int null,
col2 int null, 
col3 int null,
unique key(col1, col2, col3)
)

partitionby hash(col3)
partitions 4;
```

### Range

创建分区 

```
create table t (
id int) 
partition by range (id)(
partition p0 values less than (10),
partition p1 values less than (20));
```

增加 maxvalue 分区
```
alter table t add partition ( partition p2 value less than maxvalue);
```

创建分区后, 表就会变成多个 ibd 文件组成, 查看详情:

```
select * from information_schema.PARTITIONS where table_schema=databese() and 
table_name = 't';
```

### LIST

与 range 类似, 离散版

```
create table t (
a int, 
b int) engine=innodb
partition by list(b)(
partion p0 values in (1,3,5,7,9),
partion p1 values in (0,2,4,5,8)
);
```

### HASH

HASH 分区的目的是将数据均分地分布到各个分区中, 保证分区的数据量都是一样的.

```
create table t_hash(
a int,
b datetime) engine=innodb
partition by hash (YEAR(b))
partitions 4
```

还支持 LINEAR HASH


### KEY

与HASH类似, HASH 使用用户定义的函数分区, KEY 使用 MySQL 的函数进行区分

### COLUMNS

以上四种分区方法都是需要对整型进行操作. 

RANGE 和 LIST 的进化, columns 可以直接对非整型数据进行分区.

### 子分区

处理超大的表时, 可以在分区下方, 在继续划分子分区.

### Null 值

Mysql 把 null 视为小于任何一个非 null 值.

- range 下所有的 null 都会被划分在最左分区.
- list 需要显示指明那个分区存放 null 值
- hash 和 key 都是等于0值. 最左分区.

### 性能

数据场景分两种, OLTP, OLAP

#### OLAP 

在 OLAP 下, 需要频繁扫面大表, 可以通过对应搜索字段进行分区, 直接过滤掉无需扫描的分区.

#### OLTP

获取的数据量较小, 一般是通过索引获取几条记录. B+树的索引, 一般需要2-3次IO

### 分区列与索引列不匹配

如果检索的字段或者索引, 没有在分区字段上, 每个分区都会有独立的索引, 就会导致每个分区都需要进行索引检索.
导致IO的次数增长, 导致查询变慢. (1000万条数据, 10个分区话, 就会需要 10 * (2或3次)IO)


### 查看 Partition 是否开启

```
mysql> SELECT
    ->     PLUGIN_NAME as Name,
    ->     PLUGIN_VERSION as Version,
    ->     PLUGIN_STATUS as Status
    -> FROM INFORMATION_SCHEMA.PLUGINS
    -> WHERE PLUGIN_TYPE='STORAGE ENGINE';

+--------------------+---------+--------+
| [Name](Name)               | Version | Status |
+--------------------+---------+--------+
| binlog             | 1.0     | ACTIVE |
| CSV                | 1.0     | ACTIVE |
| MEMORY             | 1.0     | ACTIVE |
| MRG_MYISAM         | 1.0     | ACTIVE |
| MyISAM             | 1.0     | ACTIVE |
| PERFORMANCE_SCHEMA | 0.1     | ACTIVE |
| BLACKHOLE          | 1.0     | ACTIVE |
| ARCHIVE            | 3.0     | ACTIVE |
| InnoDB             | 5.7     | ACTIVE |
| partition          | 1.0     | ACTIVE |
+--------------------+---------+--------+
10 rows in set (0.00 sec)
```

``` sql
select * from information_schema.PARTITIONS;
```

## 使用优化

访问分区表的时候, 需要在 where 条件后面增加一个分区列, 让优化器过滤分区.

Mysql 无法根据表达式进行过滤, 与索引一致, 只能通过值进行过滤. 


## 参考

- 高性能的MySQL (第三版)
- MySQL 技术内幕: InnoDB存储引擎
