---
title: "返读 Coolshell 记录"
date: 2023-05-26T18:42:36+08:00
publishDate: 2023-05-26T18:42:36+08:00
draft: false
tags:
- thought
---

返向读一遍陈皓老师在 coolshell 上发布的文档. 并简单做一下读后记录. 


[ETCD的内存问题](https://coolshell.cn/articles/22242.html)

- 一个清晰的排查 Etcd 内存占用过大问题


[一把梭：REST API 全用 POST](https://coolshell.cn/articles/22173.html)

- RESTFul 为通用惯用标准, 切有各大厂的指导文档背书. 
- 身为程序员要对自己的代码负责, 有程序员的操守, 反对"优雅不能当饭吃"
- 反对讨论问题使用: 讨论都是在主观的“我觉得”，“我认为”，“感觉还好”…… 


[TCP 的那些事儿 (上) ](https://coolshell.cn/articles/11564.html)

- tcp_syncookies 可以用于防止 sync 攻击
- seq_num 是根传输字节相关
- ISN 的与一个假时钟相关, 每4微妙加一

[TCP 的那些事儿（下）](https://coolshell.cn/articles/11609.html)


[我做系统架构的一些原则](https://coolshell.cn/articles/21672.html)

- 完备性比性能更重要
- 控制逻辑进行收口
- 服从标准, 规范, 最佳实践

[双向显示文本](https://coolshell.cn/articles/21649.html)


[如何做一个有质量的技术分享](https://coolshell.cn/articles/21589.html)

- 描述好问题
- 怎么做, 为什么
- 最佳实践和方法总结


[GO编程模式：PIPELINE](https://coolshell.cn/articles/21228.html)

> 在今天，流式处理，函数式编程，以及应用网关对微服务进行简单的API编排，
其实都是受pipeline这种技术方式的影响，Pipeline这种技术在可以很容易的把代码按单一
职责的原则拆分成多个高内聚低耦合的小模块，然后可以很方便地拼装起来去完成比较复杂
的功能。

golang 的 Pipeline 代码大家都会写, 但是我一直没有思考过为什么要这样子写.


[GO编程模式：委托和反转控制](https://coolshell.cn/articles/21214.html)

- 把控制逻辑与业务逻辑分层


未完待续...
