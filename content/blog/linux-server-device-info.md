---
title: "查看 Linux 的负载情况"
date: 2022-12-16T14:31:33+08:00
publishDate: 2022-12-16T14:31:33+08:00
draft: false
tags:
- linux
---

## 查看负载

系统平均负载

``` shell
uptime
```

> 那么什么是系统平均负载呢？ 系统平均负载是指在特定时间间隔内运行队列中的平均进程数。
> 如果每个CPU内核的当前活动进程数不大于3的话，那么系统的性能是良好的。如果每个CPU内核的任务数大于5，那么这台机器的性能有严重问题。

## 查看内存信息

``` shell
free -h
```

## 查看 cpu

型号

``` 
cat /proc/cpuinfo | grep 'physical id' | sort | uniq | wc -l
```

核数

``` 
cat /proc/cpuinfo |grep "cores"|uniq|awk '{print $4}'
```



## 参考文章
- https://www.eet-china.com/mp/a87720.html
- https://colobu.com/2019/02/22/how-to-find-cpu-cores-in-linux/
- https://wangchujiang.com/linux-command/c/uptime.html
