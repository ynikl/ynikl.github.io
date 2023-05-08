---
title: "Git 查看文件指定范围的修改记录"
date: 2022-12-12T15:29:36+08:00
publishDate: 2022-12-12T15:29:36+08:00
draft: false
tags:
- git
---

查看一个文件指定范围内的所有修改记录

```
git log -p -2 -L1081,+5:'hello/world.go'
```

`-p -2` 或者 `--patch -2`

往前展示两个 commit 的 diff . 在 git 中 commit 和 patch 是同一个意思参考下文.

[git commands patching](https://git-scm.com/book/en/v2/Appendix-C:-Git-Commands-Patching)
> A few commands in Git are centered around the concept of thinking of commits in terms of the changes they introduce, as though the commit series is a series of patches. 

`-L` 语法
`-L<start>,<end>:<file>, -L:<funcname>:<file>`

限制指定查看范围.

