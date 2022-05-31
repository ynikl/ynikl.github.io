---
title: "并发与并行的区别"
date: 2022-05-31T22:36:34+08:00
publishDate: 2022-05-31T22:36:34+08:00
draft: false
tags:
- golang
- thoughts
---

举个例子，电脑的鼠标，键盘或者其他设备的驱动程序，他们是并发的，但不是并行的。他们也不需要并行去运行。

并发是很多程序（形容运行任务，不是广义上的程序）的独立运行，并发是一种程序设计结构

并行是一次性运行很多程序，并行是一种程序的运行现象，当成程序设计成并发的，他很容易就变成了并行， 但并行不一定是并发的目的。


[Concurrency is not Parallelism by Rob Pike](https://www.youtube.com/watch?v=oV9rvDllKEg&list=WL&index=5&t=9s)
