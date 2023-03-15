---
title: "Gocv Intro"
date: 2023-03-05T15:11:02+08:00
publishDate: 2023-03-05T15:11:02+08:00
draft: true
tags:
- golang
- thoughts
---

## step 1  环境

日常开始安装环境, 要求 OpenCV 4.7.0

[官方安装教程](https://gocv.io/getting-started/macos/)

中间检验版本的步骤, 需要在 gocv 的代码仓库中执行

```
❯ go run ./cmd/version/main.go

gocv version: 0.32.0
opencv lib version: 4.7.0
```

## step 2 demo

官方的示例代码仓库在仓库的`cmd/`目录下

1. Hello Video 调用你电脑的摄像头
2. Face 
