---
title: "FFmpeg 基本使用"
date: 2022-08-19T00:14:54+08:00
publishDate: 2022-08-16T20:58:41+08:00
draft: false
tags:
- ffmpeg
---

接手公司一个视频相关项目, 也是使用`ffmpeg`工具.  需要快速了解下.

## 概念

## 码率, 帧率, 文件大小

帧率(frame rate) : 视频一秒中有多少帧画面(frames per second, fps), 决定视频流畅度. 

- interlaced: 古代黑白相机的交错扫描呈现, 描述单位i : 50i
- progressive: 现代整页整页呈现, 描述单位p: 60p


码率(bit rate) :视频一秒中有多少位, 决定视频的质量

- ABR: 平均码率
- CBR: 常量码率
- VBR: 动态码率


文件大小 = 视频文件 + 音频文件

视频文件 =  码率 * 时间(s) / 8
音频文件 =  码率 * 时间(s) / 8

### 文件格式

不同的视频文件类型, 用户存储特定的数据流( 在 ffmpeg 中称为 containers)
可以存储声音或者视频数据.

### 相关缩写

- encoding (E)
- decoding (D)
- video (V)
- audio (A) 
- subtitles (S)

### 文件元数据 metadata

描述媒体文件自身的信息, 比如:
```
Metadata:
	publisher : Ninja Tune
	track : 1
	album : Ninja Tuna
	artist : Mr. Scruff
	album_artist : Mr. Scruff
	title : Kalimba
	genre : Electronic
	composer : A. Carthy and A. Kingslow
	date : 2008
```

### 声音

数字音频是通过对声音的模拟信息, 抽样且用数据信号表示.

音频使用 bit depths 来表示声音的解析度:

8bit, 12bit, 14bit ...

声音的样本频率用 Hz 表示

8000Hz, 11025Hz, 16000Hz ...

### FFmpeg 其他套件

- ffplay 播放器
- ffprobe 查看媒体文件的信息
- ffserver 流服务器

## 使用

### 通用参数

`-i` 输入源, 可以是文件也可以是 url

`-vf` option for video filters 
`-af` option for audio filters.

`-filter_complex` 当多个输入源的时候使用

`-y` 输出文件会强制覆盖已经存在的文件

### 帮助相互

``` 
" 查看支持格式
ffmpeg -formats

" 编解码器
ffmpeg -codecs 
``` 

### 调整帧率

```
ffmpeg -i input.avi -r 30 output.mp4
```

### 调整码率

```
ffmpeg -i input.avi -b:v 1500k output.mp4
```

### 调试视频大小

- s : w x h 参数 宽乘以高

```
" 缩小分辨率
ffmpeg -i input.avi -s 640x480 output.avi

" 扩大分辨率
ffmpeg -i input.mpg -vf scale=iw/PHI:ih/PHI output.mp4
```

### 视频旋转, 翻转

旋转
```
ffmpeg -i CMYK.avi -vf transpose=2 CMYK_transposed.avi
```

竖直翻转
```
ffmpeg -i meta.mp4 -vf vflip output_flip.mp4
```

### 裁切 

裁切视频的画中画, 裁切视频的中心 1/2 的视频

```
ffmpeg -i input.avi -vf crop=iw/2:ih/2 output.avi
```

### 模糊化

```
ffmpeg -i input.mpg -vf boxblur=1.5:1 output.mp4
```

锐化
```
ffmpeg -i input -vf unsharp output.mp4
```

### 叠加

```
ffmpeg -i input1 -i input2 -filter_complex overlay=x:y output
```

### 裁剪

获取一个时间段内的音频
`-t` 参数为秒

```
ffmpeg -i input.mp4 -t 180 output_3_min.mp4
```

`--ss` 设置开始时间点 ( seek from start, 从视频开始过多少秒开始操作)

```
" 直接截断开头3分钟
ffmpeg -i input.mp4 -ss 180 output_without_start_3.mp4
```

**截取某一段时间视频**
```
" 截取第4分钟, 一分钟视频
ffmpeg -i input.mp4 -ss 180 -t 60 clip_4th_min.mp4
```

### 图片操作

从视频中截图
```
ffmpeg -i videoclip.avi -ss 01:23:45 image.jpg
```

翻转图片
```
ffmpeg -i orange.jpg -vf hflip orange_hflip.jpg
ffmpeg -i orange.jpg -vf vflip orange_vflip.jpg

" transpose [0, 1, 2, 3]
ffmpeg -i image.png -vf transpose=1 image_rotated.png

```

转换图片格式
```
ffmpeg -i illustration.png illustration.jpg
```



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

### 混音

将两个声合成一个文件
```
ffmpeg -i demo.mp3 -i louder_sound.aac -filter_complex amix=inputs=2 sounds.wav
```

加强耳机的立体声效果
```
ffmpeg -i music.mp3 -af earwax -q 1 music_headphones.mp3
```

## 参考

- *FFmpeg Basics 2012 by Frantisek Korbel*
