---
title: "Mysql- 数据类型 - 数字"
date: 2022-07-31T17:49:46+08:00
publishDate: 2022-07-31T17:49:46+08:00
draft: false
tags:
- mysql
---

## 概览

Mysql 支持以下数据类型

- 数字类型
- 串类型(字符和字节)
- 日期和时间
- 空间
- JSON

## 数字类型

### 整数类型 Interger

| 类型      | 存储大小 bytes | 其他别名                   |
| ---       | ---            | ---                        |
| TinyInt   | 1              | bool, boolean = tinyint(1) |
| SmallInt  | 2              |                            |
| MediumInt | 3              |                            |
| Int       | 4              |                            |
| BigInt    | 8              |                            |

**int(M) 表示显示宽度, 最大显示宽度为(255), M 与存储空间的大小无关. 空间大小由具体类型决定.**

如果具体数值达不到宽度, 左边就会用0值补齐至 M 位.

### 浮点 ( Floating-Point )

| 类型   | 存储    | 补充   | 范围 |
| ---    | ---     | ---    | ---  |
| Float  | 4 bytes | 单精度 |      |
| Double | 8 bytes | 双精度 |      |

`Float(p)` p 表示小数点后的精度位数

`Float(M, D)` Mysql 语法: M表示总显示位数, D表示小数点后个数 -- 由Mysql自己做约分处理. Mysql 8.0 后废弃该语法.

### 定点 ( Fixed-Point )

用于需要准备保存字段数据, 如金钱相关字段.

`Decimal(M, D)` 其中, M 表示字段中有效数据个数, D 表示小数点后个数

Decimal(5,2) 的精度为 `-999.99 - 999.99`

### 位 ( Bit-Value )

`BIT(M)` 用于存储位值, M范围(1-64), 当存储的数值小于 M, 会进行左边补0

example
```
mysql> CREATE TABLE t (b BIT(8));
mysql> INSERT INTO t SET b = b'11111111';
mysql> INSERT INTO t SET b = b'1010';
mysql> INSERT INTO t SET b = b'0101';
```

```
mysql> SELECT b+0, BIN(b), OCT(b), HEX(b) FROM t;
+------+----------+--------+--------+
| b+0  | BIN(b)   | OCT(b) | HEX(b) |
+------+----------+--------+--------+
|  255 | 11111111 | 377    | FF     |
|   10 | 1010     | 12     | A      |
|    5 | 101      | 5      | 5      |
+------+----------+--------+--------+
```

## 参考
[Mysql 8.0 官方文档](https://dev.mysql.com/doc/refman/8.0/en/)
