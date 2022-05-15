---
title: "Hugo 如何使用已经发布文章做为相对URL"
date: 2022-05-15T16:33:13+08:00
draft: false
tags:
- hogo
- qa
---

hugo 的默认内容都是在 `content/` 路径下方

在 markdown 文章中使用 `{{< ref "/blog/my-first-post.md" >}}` 会在编译时发生地址替换, 带`/` 是表示从 `content/` 目录下的下一个绝对路径


```

[我的文章](\{\{< ref "/blog/my-first-post.md" }})

```

**记得是已经发布的文章, 如果使用不存在, 或者是不参加发布的草稿文章就会发生报错**

[hugo官方文档](https://gohugo.io/content-management/cross-references/)
