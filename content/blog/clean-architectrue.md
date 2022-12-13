---
title: "代码整洁架构"
date: 2022-12-05T09:21:02+08:00
publishDate: 2022-12-05T09:21:02+08:00
draft: false
tags:
- golang
- architecture	
---

代码整洁架构

## 核心思想

最重要的是依赖顺序需要内收 -- 业务逻辑不能依赖框架

## 分层

简单分层四层

- Entities
- Use Cases
- Interface Adapters
- Framework and Drivers

### Entity 实体抽象层

我的理解应该是在公司业务, 或者项目领域上对于业务模型的抽象. 不容易改变 (除非公司
业务, 或者项目方向改变). 应该是与 *领域驱动设计* 不谋而合

### Use Cases 使用场景层

业务使用场景, 应该是存放相关不同业务场景的具体实现流程

### Interface Adapters 接口转化器层

负责 Use Cases 数据 与外部使用数据转换器实现. 

比较特别的例子, 将 Use Cases 产生的数据与外部的 UI 表现层所需要的数据格式做转化.

### Framework and Divers

数据库和框架层, 外部工具包接口依赖之类的.


## 依赖倒置

当遇到跨层依赖的时候, 内层需要引用到外层逻辑时: 比如, Use Case 要呈现 UI 数据结构
(Interface Adapters 层) 时, 不能直接引用 UI 层的数据模型.

而是, 通过依赖倒置. 将 UI 层的处理逻辑, 注入 Use Case 层进行处理, 实现目标.

## golang 整洁模板

引用自 [golang clean template](https://github.com/evrone/go-clean-template)

```
├─cmd 应用入口
│  └─app
├─config
├─docs // 存放文档
├─internal
│  ├─app
│  ├─controller // 控制器
│  │  ├─amqp_rpc
│  │  └─http
│  │      └─v1
│  ├─entity // 实体层
│  ├─middleware // 中间件
│  └─usecase
│      ├─repo // 数据库操作
│      └─webapi // RESTful API
├─migrations
├─pkg //以被外部程序安全导入的包
│  ├─crypto
│  ├─httpresponse
│  ├─httpserver
│  ├─logger
│  ├─mysql
│  ├─postgres
│  ├─rabbitmq
│  └─redis
```


## 参考文章

- [结构架构介绍](https://www.artacode.com/post/golang/template/)
- [the clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [goang clean template](https://github.com/evrone/go-clean-template)
