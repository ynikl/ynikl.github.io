---
title: "Content Disposition"
date: 2022-09-04T21:32:24+08:00
publishDate: 2022-09-04T21:32:24+08:00
draft: false
tags:
- golang
- http
---

`Content-Disposition` 常见是用在 http 请求的 Response 的 Header 头部. 

告诉请求客户端(浏览器) 如何处理内容; 

`Content-Disposition`是在 MIME 标准定义的. http 中的用法只是其中的一小部分.

## 语法参数


### inline 
 
会在浏览器内部显示
```
Content-Disposition: inline
```  

###  attachment 

 会被保存成文件
 
```
Content-Disposition: attachment
```

后面可以跟 filename, 值为预设文件名称, 中间使用`;`分号隔开.

```
Content-Disposition: attachment; filename="filename.jpg"
```

拓展参数, 有两个文件名称参数可选

- filename*
- filename

`filename*` 采用了 RFC 5987 中规定的编码方式,
假如两个参数都使用 `filename*` 的优先级更高

#### RFC 5987

该提议最终还是引用 [RFC 2231](https://www.rfc-editor.org/rfc/rfc2231#section-4)
中的编码方式. 下面简单介绍一下语法

- `*`星号用于标记该同名参数是支持该编码语法的, 就如( `filename*` 之于 `filename`)
- `'`单个逗号用于分割 **字符集名称** , **语言**, **文件名称**
- `%`百分号用于标记编码方式, 参考[RFC 2047](https://www.rfc-editor.org/rfc/rfc2047), 所以文件名中不能有`%`

```
filename*=us-ascii'en-us'This%20is%20%2A%2A%2Afun%2A%2A%2A
```
使用`us-ascii`编码, `en-us`英语

## 问题: 下载文件名称乱码

根据上面的知识, 就可以解决有的浏览器下载文件时中文名称乱码. 

- Http 返回设置`Content-Disposition`值
- 使用 RFC 2231 的文件名称方式, 制定编码`utf-8`
- 将文件名称移除`%`

``` go
disposition := fmt.Sprintf("attachment;filename*=utf-8''%s", url.QueryEscape(name))
```

## 参考

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition)
[RFC 2231](https://www.rfc-editor.org/rfc/rfc2231#section-4)
