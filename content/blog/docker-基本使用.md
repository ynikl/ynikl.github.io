---
title: "Docker 基本使用"
date: 2022-04-07T13:34:55+08:00
draft: true
---

## 启动一个容器

打个样
```
docker run -itd --rm --name hello image_name 
```

-- it 

外挂文件夹
参数 `-v `

`docker run -it -v /home/dock/Downloads:/usr/Downloads ubuntu64 /bin/bash`

## 与容器交互

### 进入容器

`docker attach`


exec

`docker -it exec CONTAIN_NAME bash`

exex 会再目标容器内部执行一个命令, 命令名为 bash, 就是起一个 shell 咯. 
	加上 `-it`, 进入交互的终端模式




## 管理容器


## 管理镜像



