---
title: "GO 代码结构中为什么老有 cmd 和 internal 文件夹"
date: 2023-08-03T20:34:58+08:00
publishDate: 2023-08-03T20:34:58+08:00
draft: false
tags:
- golang
---

总结:

- cmd 一个代码库多个启动程序, 清晰明了
- internal 将外部依赖放置在 internal 外面, 避免内部逻辑代码直接引用. 比如, 数据持久层 MySQL 


参考一下 ChatGPT 给我的答案: 

>  在 Go 语言中，使用 cmd 目录和 internal 目录的组合是一种常见的方式来组织项目结构。
> cmd 目录通常包含可执行程序的入口点。这些可执行程序可以是应用程序的命令行接口、工具或测试二进制文件。每个子目录都包含一个可执行程序的主要文件，以及与其关联的其他文件。这种风格使得每个程序都可以独立编译和分发，并且可以让其它人轻松了解和使用它们。
> 
> 对于 internal 目录，其命名约束了该目录下的代码仅供内部使用。这意味着代码不应该由任何可以被外部依赖使用的包引用。这种风格允许对代码的控制更精细，因为程序员可以更好地控制哪些代码可以被其他人使用。
> 
> 使用这样的结构来组织 Go 项目有很多好处，包括：
> 
> 代码更易于维护和理解：使用独立的可执行程序和内部目录，可以让代码更清晰且更容易维护，使开发人员能够更好地理解代码的用途和位置。
> 
> 扩展性强：该结构使得每个可执行程序都可以相对独立地进行开发、测试和部署。这也使得代码更有扩展和修改的完整性。
> 
> 松耦合：使用 cmd 和 internal 目录方式可以大大增强代码间的松耦合度。这使得程序员可以更好地对代码变化做出反应，同时减少代码冲突及协调问题。
> 
> 综上所述，使用 cmd 和 internal 目录方式更易于开发和维护，是现代化 Go 应用程序中广受欢迎的项目结构设计方式。


