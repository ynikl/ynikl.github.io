---
title: "计算机中的：大端和小端"
date: 2022-06-04T00:19:23+08:00
publishDate: 2022-06-04T00:19:23+08:00
draft: false
tags:
- computer science
---

大端和小端的命名出处是来自于《格列佛游记》中吃鸡蛋分成从大端开始吃的“大端派”和从小端开始吃的“小端派”

- 大端的优势是高位计算，和可读性
- 小端的优势的低位运算

各自的优劣分析本质上还是因为内存的连续性，需要修改和读取的位数越少越有优势。

以上是阅读[阮一峰的博文-字节序探析：大端与小端的比较](https://www.ruanyinfeng.com/blog/2022/06/endianness-analysis.html)的简单总结。本来自己也是想要整理一篇关于大小端分析的文章。刚好阮老师发文了，收获甚多，就不在自己整理了。


## 参考连接
- [阮一峰的博文-字节序探析：大端与小端的比较](https://www.ruanyinfeng.com/blog/2022/06/endianness-analysis.html)
