---
title: "Mysql 快速备份数据"
date: 2022-12-20T10:55:10+08:00
publishDate: 2022-12-20T10:55:10+08:00
draft: false
tags:
- mysql
---

```
CREATE TABLE dbto.table_name like dbfrom.table_name;
insert into  dbto.table_name select * from dbfrom.table_name;
```

[原文](https://stackoverflow.com/a/63457341/9992963)
