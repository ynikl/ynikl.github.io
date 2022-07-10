---
title: "Make 的基本使用"
date: 2022-07-10T17:47:42+08:00
publishDate: 2022-07-10T17:47:42+08:00
draft: false
tags:
- tool
---

想自己整理一篇基本的 `make` 指令用法, 突然发现 阮一峰大佬已经整理了一篇很完整的博客, 遂放弃.

[阮一峰文章地址](https://www.ruanyifeng.com/blog/2015/02/make.html)

如果不写 c, 主要理解就几个概念就可以使用了

- target 可以用来当作想要执行的命令集的名称
- .PHONY:  可以用来声明命令集名称
- recipes 实际执行的命令集合

## 介绍一下我自己的应用场景

我目前主力编程语言是 go, 我用的编辑器是 vim, 所以我基本就在 shell 里面完成编码任务. 

case 1: 简化本地编译和测试, 自动做 `setup` 和 `teardown`

当我想要尝试一下整个项目是否编译

``` go 
.PHONY: build
build: 
	go build .
	rm -rf [PROJECT NAME]
```

使用上面的 `makefile`, 我就只需要 `make build`, 就不用再删除编译出来文件. QAQ, 可以再加一些单元测试命令, 检测测试是否通过. 因为公司的项目, 在单测这方面做的不是很好, 我自己就是简单 `build` 一下

case 2: git 提交代码自动化操作

当我想要把我代码推送到, 测试分支, 进行集成测试

```
.PHONY: dev
ProjectName="Your Project Name"
TargetBranch="Your want to merge branch"
dev:
	go build 
	rm -f $(ProjectName)
	git add .
	git commit -m $(msg)
	git push
	CurBranch=$(git branch --show-current)
	git checkout ${TargetBranch}
	git pull 
	git merge ${CurBranch} -m "Merge branch '${CurBranch}' into ${TargetBranch}"
	go build
	rm -f $(ProjectName) 
	git push
```

简化 git 的操作流程, 现在只需要`make dev`就可以完成, 还可以在合并之前和之后增加测试, 我自己目前知识简单的 `build` 下而已 QAQ. 
