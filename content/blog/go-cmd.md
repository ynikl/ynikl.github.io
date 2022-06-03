---
title: "Go 常用的命令汇总"
date: 2022-04-01T19:56:59+08:00
draft: false
tags:
- golang
---

# 工具分类


## go build 

编译源代码文件

`-race` 
编译出的目标程序，会启用数据竞争检测


## go doc 

查看包的文档(定义于`doc.go`的注释中), 于包中公开的函数签名

example
```
go doc

go doc encoding/json
```


## go env 

查看 go 相关的环境变量

``` shell
# -w 设置环境变量
go env -w GOPAHT='/some/path'

# -u 恢复成默认设置
go env -u GOPATH
```


## go generate

扫描文件中的指令并执行, 相关指令目的应该是“生成或者修改源文件”


**注释的指令格式**

`//go:generate command argument...`

ps: wire 也是利用命令, 生成依赖注入文件


## go get

管理当前module依赖

```
# 添加依赖包
go get example.com/pkg

# 指定包版本
go get example.com/pkg@1.2.3

# 移除依赖
go get example.com/pkg@none
```

## go install

获取包文件，并编译和安装。可执行文件编译到`$GOBIN`路径下, 包文件编译到`$GOPATH/pkg`


## go list

列出包的数据信息


## go mod

管理 modules

```
edit	修改go.mod
init	初始化
tidy	自动补全依赖包
vendor	生成一个所有依赖的vendor文件夹
```


## go test

跑单元测试

```
go test -v .

# 指定函数
go test -run 函数名

# 性能测试
go test -v -bench . -benchtime 50s

# 单元测试覆盖率
go test -cover

```

## go tool

```
# 不带参数，显示工具列表
go tool
```

### compile

使用`go tool compile -N -l -S main.go`生成汇编代码
