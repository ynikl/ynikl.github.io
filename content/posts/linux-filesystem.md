---
title: "Linux 文件系统简单操作流程"
date: 2022-11-05T13:52:16+08:00
publishDate: 2022-11-05T13:52:16+08:00
draft: false
tags:
- linux
---

## 文件系统原理

- BIOS：启动主动运行的韧体，会认识第一个可启动的装置
- MBR：第一个可启动装置的第一个磁区内的主要启动记录区块，内含启动管理程序；
- 启动管理程序(boot loader)：一支可读取核心文件来运行的软件；

## 相关命令

### 查看磁盘信息

```
fdisk -l
```

macOS
``` 
diskutil list
```

查看磁盘用量

``` 
df -h
```

查看文件大小
``` 
du -h
```

### 新磁盘的安装流程

1. 对磁盘进行分割，以创建可用的 partition ；
2. 对该 partition 进行格式化( format )，以创建系统可用的 filesystem；
3. 在 Linux 系统上，需要创建挂载点 ( 亦即是目录 )，并将他挂载上来；

操作磁盘分区, fdisk 后面跟具体的物理磁盘
```
fdisk /dev/hdc
```

创建一个ext4文件系统

```
mkfs -t ext4 /dev/vdb1
```

挂载磁盘分区
``` 
mkdir /mnt/hdc6
mount /dev/hdc6 /mnt/hdc6
```

## 参考
[Linux 磁盘与文件系统管理](http://cn.linux.vbird.org/linux_basic/0230filesystem.php#filesys_1)
