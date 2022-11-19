---
title: "【翻译】使用 Godoc 给 go 代码添加文档"
date: 2022-05-25T16:11:10+08:00
draft: false
tags:
- 翻译
---

在看官方文档的时候，突然心血来潮，想翻译一下一篇博客玩玩。

原文章地址
[Godoc: documenting Go code](https://go.dev/blog/godoc)


Go 语言官方对于代码的文档非常重视，文档对于软件的可阅读性和可维护性有着至关重要的地位。当然，文档必须要是准确可理解的，也需要易于编写和维护。理性情况下，文档应该与代码紧密联系。这样子才能方便于程序员修改和编写。

所以，我们开发了 godoc 文档工具。本文描述了的 godoc 关于的“文档”的方法论，和解释如何按照使用规范给你自己的项目写出一个好的文档。

godoc 会解析源文件代码和注释，生成基于HTML的页面或者纯文本格式的文档。这样子文档就与代码紧密联系。举个例子，通过在 godoc 的页面点击，你就可以在一个函数的文档说明和源代码中快速跳转。

godoc 在概念上与 Python 的“Docstring”和 Java 的“Javadoc"相似，但是 godoc 设计的更加简单化。godoc 读取的代码注释的不需要特定的结构化（Docstring 使用），也不需要特定的语法（JavaDoc 使用），godoc 从代码读取的注释就是简单的“代码注释”，就算你不使用 godoc 也可以直接阅读的注释。

使用的规范很简单：在类型，变量，常量，函数，包声明的上方写下注释即可， 中间不要有空行。godoc 会把这些注释以文字的形式呈现在被注释的对象旁边。举个例子，下方就是 fmt 包的`Fprint`函数的注释。

``` go
// Fprint formats using the default formats for its operands and writes to w.
// Spaces are added between operands when neither is a string.
// It returns the number of bytes written and any write error encountered.
func Fprint(w io.Writer, a ...interface{}) (n int, err error) {
```

值得注意的是该注释是以被注释的对象命名开头的一个完整的句子。 这个使用规范可以方便我们生成各种各样的格式文档，从简单的纯文本到 UNIX 是 man 的帮助页，还可以使用其他工具更见简单地获取到信息， 比如提取出第一行或者句子。

在包的声明处的注释，需要写整个包的概括说明。这些注释可以很简洁，就像 sort 包中的简短描述：

```
// Package sort provides primitives for sorting slices and user-defined
// collections.
package sort
```

也可以很详细，比如 gob 包。有另一个使用惯例：像这种有这大量注释文档的包，单独一个`doc.go`文件，放置包的注释。

不论包的注释文档大小，第一句注释会被展示在 godoc 的呈现包列表中。

那些没有与最外层声明（可以简单理解为包内全局对象）连接在一块的注释会被 godoc 忽略。但是，有一个例外。那些写在最外层且以“BUG(who)”开头的注释，会被识别为已知的 bug，且会被包含在包文档的“Bugs”分区中。这个“who”部分应该填写可以提供更加详细信息的人名。举个已经在 bytes 包中注释的问题:

```
// BUG(r): The rule Title uses for word boundaries does not handle Unicode punctuation properly.
```

某些时候，当一个结构体字段，或者函数，类型，甚至一个整个包变成了冗余或者没有使用必要，但是还是需要与旧代码保持兼容。这时，可以增加一个一段落以“Deprecated:”开头后面跟废弃信息的注释。标识该对象不应该再被使用。

下面展示一些 godoc 把注释转化成网页的规则：

- 不同的段落需要以空行分割。否则将会被识别成同一段。
- 带有格式的文本，需要要缩进。
- URL 会被转化成网页连接，无需特殊处理。

上方的这些规则不需要你做任何的特殊处理。


事实上，godoc 的极简处理方式使得它非常容易使用。所以，很多 go 项目，包括标准库，都已经开始开始遵循 godoc 的注释文档规范。

你自己的项目也可以通过编写符合文中规范的注释生成漂亮的文档。任何下载在`$GOROOT/src/pkg` 或者任何在 `GOPATH` 空间下的 go 代码包，都可以被 godoc 的命令行或者 HTTP 的接口访问， 你也可以通过在命令后添加`-path`参数或者直接使用`godoc .`来指定源码的路径。在[godoc 文档](https://pkg.go.dev/golang.org/x/tools/cmd/godoc) 你可以查看到更加详细的内容。
