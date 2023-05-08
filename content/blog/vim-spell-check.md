---
title: "Vim Spell Check"
date: 2022-06-16T23:00:35+08:00
publishDate: 2022-06-16T23:00:35+08:00
draft: true
tags:
- vim
---

最近发现自己写代码时候经常出现代码拼写错误, 所想设置 vim 自带英文拼写检测

开启 vim 自带拼写检查
```
# 增加 cjk 表示不对中文, 日文, 韩文插件. btw: cjk 就是中日韩缩写
set spell spelllang=en_us,cjk

# 可以通过 :h spell 查看文档
```

然后, 通过 `zg` 添加特殊单词. 但是我发现, 这个对于识别一些特定的计算机相关名词不是很有好, 比如 `cmd`

下一个目标, 要找一个可以兼容大小写驼峰, 和能自动识别编程程序的插件. 

幸好我找到了: [spelunker](https://github.com/kamykn/spelunker.vim)


