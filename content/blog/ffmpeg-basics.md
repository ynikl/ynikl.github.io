---
title: "FFmpeg 基本使用"
date: 2022-08-16T20:58:41+08:00
publishDate: 2022-08-16T20:58:41+08:00
draft: true
tags:
- golang
- thoughts
---

## 概念

### 文件格式

不同的视频文件类型, 用户存储特定的数据流( 在 ffmpeg 中称为 containers)
可以存储声音或者视频数据.

### 相关缩写

- encoding (E)
- decoding (D)
- video (V)
- audio (A) 
- subtitles (S)

## 使用

### 通用参数

`-i` 输入源, 可以是文件也可以是 url

`-vf` option for video filters 
`-af` option for audio filters.

`-y` 输出文件会强制覆盖已经存在的文件

### 帮助相互

``` 
" 查看支持格式
ffmpeg -formats

" 编解码器
ffmpeg -codecs 
``` 

### 分辨率

- s : w x h 参数 宽乘以高

```
" 缩小分辨率
ffmpeg -i input.avi -s 640x480 output.avi

" 扩大分辨率
ffmpeg -i input.mpg -vf scale=iw/PHI:ih/PHI output.mp4
```

### 裁剪

### 格式转化

格式转化流程:

Demuxer (分解复用) : 将合成信号恢复成原本独立的信号数据
Decoder (解码器) : 解码
Encoder (编码器) : 编码
Muxer ( _ ): 将多个信号数据合并

``` 
ffmpeg -y -i input.avi output.mp4

" 改变格式, 但不更改编解码方式
ffmpeg -i input.avi -q 1 -c copy output.mov


```

## 参考
