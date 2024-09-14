---
title: "Restful Delete"
date: 2024-04-18T16:13:36+08:00
publishDate: 2024-04-18T16:13:36+08:00
draft: true
tags:
- golang
- thoughts
---

DELETE 在 Restful 中的应用

## 传参规则

- 没有严格规定是否只能使用 PATH 参数作为传参
- Query 参数也是可以使用的
- 参考了飞书/语雀的删除操作， 使用的是 POST 方式

[stackover flow](https://stackoverflow.com/questions/2539394/rest-http-delete-and-parameters?rq=1)
