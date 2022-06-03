---
title: "Mysql 不支持事务嵌套"
date: 2022-06-02T16:16:58+08:00
publishDate: 2022-06-02T16:16:58+08:00
draft: false
tags:
- mysql
---

mysql 在事务中再开启事务，前一个事务会被自动提交

[stackoverflow](https://stackoverflow.com/questions/1306869/are-nested-transactions-allowed-in-mysql)

