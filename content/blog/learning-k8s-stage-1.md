---
title: "Learning K8s Stage 1"
date: 2023-05-29T12:57:34+08:00
publishDate: 2023-05-29T12:57:34+08:00
draft: true
tags:
- golang
- thoughts
---


[熟悉K8s](https://github.com/caicloud/kube-ladder/blob/master/tutorials/lab2-application-and-service.md)

### YAML

kubernetes yaml 整体分为 5 个部分：apiVersion, kind, metadata, spec, status；其中 apiVersion 表明当前 kubernetes API 的分组；kind 表明当前操作的资源类型； metadata 是资源的元数据，对于每种资源都是固定的，例如资源的名字，所处的 namespace, label 等；spec 是用户对资源的 “说明书”，即用户对资源的各种配置信息；status 是资源当前的状态，
