---
title: "Golang Gin"
date: 2023-01-01T22:08:25+08:00
publishDate: 2023-01-01T22:08:25+08:00
draft: false
tags:
- golang
---

# Golang Gin 框架的使用和理解

[框架代码地址](https://github.com/gin-gonic/gin)

## 中间的几个关键对象

- Engine
- RouterGroup
- HandlerFunc
- Context

### Engine 

gin 的框架实例, 包含路由地址, 中间件, 框架配置. 通过`New()`或者`Default()`创建

- `New` 不带任何中间件
- `Default` 会带 `logger` 和 `recover`

通过 `Engine.Use()` 添加中间件到Engine的`RouterGroup`

`Engine.Run` 启动, 并绑定到参数的地址

### RouterGroup
```
type RouterGroup struct {
	Handlers HandlersChain
	basePath string
	engine   *Engine
	root     bool
}
```

用于存储中间件的处理方法 -- 存储再 `HandlersChain` 里面, 即`[]HandlerFunc`

真正的路由地址存储在`engine.trees` -- 路由树, 检索请求地址和对应处理方法.

Engine 包含 RouterGroup, RouterGroup 也会存储 Engine 的地址. 
目的是为了, 加点api地址时, 根据当前具体配置的中间件, 再添加engine的路由树.
也就是说, 过程中修改了中间件, 不会影响到已经配置的路由的中间件.

```
func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
	absolutePath := group.calculateAbsolutePath(relativePath)
	handlers = group.combineHandlers(handlers)
	group.engine.addRoute(httpMethod, absolutePath, handlers)
	return group.returnObj()
}
```

注册路由之前, 先将当前的 http 处理方法, 与当前中间件配置合并, 再添加到engine. 

#### RouterGroup

`RouterGroup.Group`	创建一个新的 `RouterGroup` , 意味着已经添加的过的 middleware
都需要重新进行添加, 因为是在注册路由的地址的时候, 中间件才会生效.

相似的路由捆绑在一块, 路由组写法, 可以做绑定相同中间件处理. 
```
// Simple group: v1
v1 := router.Group("/v1")
{
  v1.POST("/login", loginEndpoint)
  v1.POST("/submit", submitEndpoint)
  v1.POST("/read", readEndpoint)
}
```
中间的花括号只起美化代码作用, 非必需

### HandlerFunc

中间件和请求处理方法的函数签名

``` go
type HandlerFunc func(*Context)
```

处理请求的时候, HandlerFunc 执行完毕 -- 代表该请求处理结束


### Context

gin 自己实现的 Context 结构

- 在中间件中传递参数
- 控制调用流程
- 获取参数
- 返回结果
 
Context 结构的重要组成部分介绍
``` go
type Context struct {
	// 存储URL 参数
	Params   Params
	
	// 写入请求的结果
	Writer    ResponseWriter
	
	// 请求的所有处理方法集合
	handlers HandlersChain
	
	// 当前执行到第几个方法
	index    int8
	
	// 存储流转于中间件的参数式
	Keys map[string]any
}
``` 
 
#### 在中间件中传递参数
 
- `context.Get`
- `context.Set` 两个方法在中间件中传递参数 


#### 流程控制

- `context.Next()`

``` go
func (c *Context) Next() {
	c.index++
	for c.index < int8(len(c.handlers)) {
		c.handlers[c.index](c)
		c.index++
	}
}
```

将当前代码执行点移交到下一个 `HandlerFunc` 具体流程看, [Middleware](#Middleware)


#### 获取参数

- `context.Param()` 获取 Query Param 的字符串参数
- `context.Query()` 
- `context.QueryMap()` 或者 `context.PostFormMap` 获取 map 的参数
- `context.Bind` 将参数包含query参数和body参数解析之后, 反序列化到传入的结构体. 
  tag `binding:"required"` 如果没有对应参数会报错

  
#### 调用结束

- `conetxt.JSON` 以json的形式, 返回结果
  举个例子: 如果是json的的化就是 `jsonBinding` 进行反序列化
- 报错 500 `context.AbortWithStatus()`


#### Cookie 操作

- `context.Cookie()`
- `context.SetCookie()`


### Middleware

符合 [HandlerFunc](#HandlerFunc) 签名的函数. 就可以`Engine.Use`注册层路由. 路由设置
的处理函数, 在本质上可以称为中间件.

1. gin 会按照 routegroup `Use` 中间件的先后顺序存储到路由表中
2. 接收请求, 依次执行注册时设置的所有中间件函数.

#### Next

调用流程图:

```
middleware 1
| - doing m1 work
| - call Context.Next()
|     - call middleware 2
|       - doing m2 work
|       - call Context.Next()
|         - call middleware 3
|         - - done m3
|       - continue m2 work
|     - - done m2 work
| - coninue m1 work
| - done 
finish
```

当调研 `Context.Next()` 时会调用下一个中间函数, 当下一个函数执行完成之后, 执行
代码会返回到原先调用 `Context.Next()`的地方 -- 递归

#### Abort

中间处理时可以调用 `Context.Abort()` 

``` go
func (c *Context) Abort() {
	c.index = abortIndex
}
```

会将函数调用链的index指向一个超大整数 -- 放弃后面所有的处理函数, 但是 **调用Abort后, 同一个函数接下的代码还会继续执行**

#### 小demo

[demo](https://gist.github.com/ynikl/20b603bfd743d2540d482939ac87d133)

<script src="https://gist.github.com/ynikl/20b603bfd743d2540d482939ac87d133.js"></script>
``` go
func main() {
	eg := gin.New()
	eg.Use(m1, m2)
	eg.GET("/greet", m3, greet)
	eg.GET("/abort", abort, m3, greet)
	eg.Run(":8080")

}

func m1(c *gin.Context) {
	fmt.Println("before m1 next")
	c.Next()
	fmt.Println("after m1 next")
}

func m2(c *gin.Context) {
	fmt.Println("\tbefore m2 next")
	c.Next()
	fmt.Println("\tafter m2 next")
}

func m3(c *gin.Context) {
	fmt.Println("\t\tbefore m3 next")
	c.Next()
	fmt.Println("\t\tafter m3 next")
}

func abort(c *gin.Context) {
	fmt.Println("\t\t\tbefore abort")
	c.Abort()
	fmt.Println("\t\t\tafter abort")
}

func greet(c *gin.Context) {
	fmt.Println("\t\t\t\thow are you doing?")
	c.JSON(200, "great")
}
```

接口 `/greet` 的输出
``` sh
before m1 next
        before m2 next
                before m3 next
                                how are you doing?
                after m3 next
        after m2 next
after m1 next
```

接口 `/abort` 的数据
``` sh
before m1 next
        before m2 next
                        before abort
                        after abort
        after m2 next
after m1 next
```

### 其他

gin.H
``` go
type gin.H map[string]interface{} 
// 常用于 engine.JSON() 时返回 json 数据
```

## 几个问题

### Gin 的 context 有什么作用和怎么用?

context 是 Gin 代码请求流转的核心, 存储处理请求的所有必须参数


基本的使用方法:

1. 需要知道如何获取参数 -- `Bind` 或者 `Param` 方法
2. 控制处理函数流程 -- `Next` 或者 `Abort` 方法
3. 返回结果 -- `JSON`

### Gin 的整体框架流程是怎么样的? 从接受到一个请求再到返回请求中间的流程?

请求的注册入口, `Engine.ServeHTTP`

整体流程

1. 存储路由路径和注册的处理函数到"路由树" -- `nodetree`
2. 接收请求
3. 根据接收到的请求地址, 从路由树中取出注册的函数, 组成`HandlerChain` 函数处理链
4. 把函数处理链赋值到`gin.Context`中
5. 由`gin.Context`开始依次调用注册处理函数, 所以函数遍历完成, 处理结束
6. 通过`gin.Context.Writer`写入http请求结果 
7. 请求结束

