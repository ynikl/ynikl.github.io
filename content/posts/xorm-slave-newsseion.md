---
title: "xorm 的 session 和 salve 的区别"
date: 2022-07-22T19:08:05+08:00
publishDate: 2022-07-22T19:08:05+08:00
draft: false
tags:
- golang
---

简单分析下xorm 里面 session 和 slave 里面 close 的代码

```
xorm.EngineGroup.Slave()
```

- Slave () 会直接返回 xorm.Engine对象（指代我们项目到数据库的逻辑连接（里面有tcp复用））
- 如果这时候再调用 `close` 的会，会直接把 xorm.Engine 关闭（即数据库连接关闭）。 
- 如果用这个 engine， 执行查询动作如 find， get时. 内部会有自动执行 `NewSsession` 和 `Close`, 所以不在需要手动调用`close`


```
xorm.EngineGroup.NewSsession()
```
- 会返回一个`xorm.Session`对象, 对应我们数据库操作中的"会话事务", 是否自动提交之类选项.(未手动开始事务时,都是自动提交)
- 调用 `close`, 对清除 session 中未提交的事务和一些缓存的 `sql` 前置处理语句
- 对于普通 `select` 语句调不调用是没什么太大影响 (但是还是建议`new`之后调用`close`)
