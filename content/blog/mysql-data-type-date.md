---
title: "Mysql- 数据类型 - 数字类型"
date: 2022-07-31T17:49:46+08:00
publishDate: 2022-07-31T17:49:46+08:00
draft: true
tags:
- mysql
---

## 概览

Mysql 支持以下数据类型

- 数字类型
- 串类型(字符和字节)
- 日期类型
- 空间
- JSON

## 日期类型

Mysql 支持的数据类型

- DATE
- TIME
- DATETIME
- TIMESTAMP
- YEAR

### Date

只存储日期数据, 不包含时间. `YYYY-MM-DD`, 范围是从 '1000-01-01' to '9999-12-31'


### DateTime

存储日期, 也存储时间 `'YYYY-MM-DD hh:mm:ss'`

范围是从'1000-01-01 00:00:00' to '9999-12-31 23:59:59'

### TIMESTAMP	

- 存储Unix时间戳数据
- 会受到服务器时区影响-- 存储的时候转化成标准的Unix时间戳(0时区), 取数据时反之
- [时区环境变量设置](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_time_zone)

范围 '1970-01-01 00:00:01' UTC to '2038-01-19 03:14:07' UTC.

相关函数: 

- FROM_UNIXTIME 把 Unix 时间戳转化成日期
- UNIX_TIMESTAMP 把日期转化成 Unix 时间戳

### YEAR

显示形式 `YYYY`, 可选显示位数`YYYY(M)`

- 默认4位显示 '1991'

### TIME

只有时间部分,没有日期部分 `hh:mm:ss`, 范围从 '-838:59:59' 到 '838:59:59'

### 自动更新

DateTime 和 Timestamp 在 **Mysql 8.0**, 支持自动初始化和当数据更新时自动更新.

``` sql
CREATE TABLE t1 (
  ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  dt DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 存储毫秒级别的时间

支持存储毫秒级别的时间类型有:

- TIME
- DATETIME
- TIMESTAMP

声明模式为 `type_name(fsp)`, fsp 为0-6, 表示小数点后个数

``` sql
CREATE TABLE fractest( c1 TIME(2), c2 DATETIME(2), c3 TIMESTAMP(2) );

INSERT INTO t1 VALUES
('17:51:04.777', '2018-09-08 17:51:04.777', '2018-09-08 17:51:04.777');
```

### 存储空间

| 类型      | 大小    | 其他           |
| Year      | 1-bytes |                |
| DateTime  | 5 bytes | 3-bytes 小数点 |
| timestamp | 4 bytes | 3-bytes 小数点 |

### 使用推荐

*高性能 Mysql* 里面总结 DateTime 和 Timestamp 的使用选择:

- 非特殊情况, 尽量使用 timestamp, 因为空间效率更高.
- 没有必要用 INT 存储, 保存时间戳. 因为没有任何收益.


## 参考
[Mysql 8.0 官方文档](https://dev.mysql.com/doc/refman/8.0/en/)
[高性能 Mysql]
