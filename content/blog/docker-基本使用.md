---
title: "Docker 基本使用"
date: 2022-04-07T13:34:55+08:00
draft: false
tags:
- docker
---

## 启动一个容器

打个样
```
docker run -itd --rm --name hello image_name 
```

-- it 
将当前的终端和容器内的终端连接在一起, 正式所谓的交互模式

--rm
当容器停止之后, 会自动删除改容器

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

	

### 查看日志
`docker logs CONTAINER`
可以查看容器日志

-f
可以持续输出容器内部的最新日志

## 管理容器
启动
```
docker start CONTAINER
```

暂停
```
docker stop CONTAINER
```

提交
```
docker commit --author="ian" CONTAINER NEW-IMAGENAME:TAG
```

## 管理镜像

查看所有的镜像列表
```
docker image ls
```

删除镜像
``` 
docker rmi IMAGE
```

删除 `<none>` 名称的镜像
```
docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
```

## FQA

### CMD和ENTRYPOINT的区别

RUN、CMD 和 ENTRYPOINT 这三个 Dockerfile 指令看上去很类似，很容易混淆。本节将通过实践详细讨论它们的区别。

简单的说：

RUN 执行命令并创建新的镜像层，RUN 经常用于安装软件包。

CMD 设置容器启动后默认执行的命令及其参数，但 CMD 能够被 docker run 后面跟的命令行参数替换。

ENTRYPOINT 配置容器启动时运行的命令。

[CMD和ENTRYPOINT的区别](https://www.cnblogs.com/LucasSong/p/12701357.html)



