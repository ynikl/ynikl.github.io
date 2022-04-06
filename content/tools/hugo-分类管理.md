---
title: "Hugo 文章分类管理"
date: 2022-03-09T11:29:17+08:00
draft: false
tags:
- hugo
---

# Hugo 文章分类管理
## 文件夹
内容存放在`content/`目录下方, `content/`下方的子目录会形成资源URI, 

例如
`content/blog/doc.md` 文章访问目录即为 `https//XXX.github.io/blog.doc.md`

## 分类管理 <taxonomies> 

## 文章内容表述 (Front Matter)

使用<taxonomies>进行分类, 默认只有tags

``` yaml
tags:
- Go
- fast
```

可以在`config`添加自定的分类选项

``` yaml
taxonomies:
	series: series
	category: categories
```

## 文章路径 
| 类别     | 描述         | 地址                  |
| ---      | ---          | ---                   |
| home     | 网站homepage | /index.html           |
| page     | 指定页面     | /post/页面/index.html |
| section  | 分区         | /section/index.html   |
| taxonomy | 分类         | /tags/index.html      |
| term     | 分类系列     | /tags/go/index.html   |

## `_index.md` 和 `index.md`
在文件夹下方添加`_index.md`会识别成section.
在文件夹下方添加`index.md`会被识别成文章.

**千万不要在默认的content目录下方添加`index.md`**

