---
title: helm 报错：another operation (install/upgrade/rollback) is in progress处理方法
date: 2023-02-18 18:01:54
top: true
tags:
    - devops
    - k8s
---
### 背景

本文介绍使用Flux CD管理Helm Releases的方法。

### 问题描述

在使用Helm Release时，有时会出现报错信息："another operation (install/upgrade/rollback) is in progress"。

### 解决方案

1. 运行`helm history <name> --namespace <ns>`命令，查看Release的历史版本。

   ![/medias/1675238086770.jpeg](/medias/1675238086770.jpeg)

2. 运行`lux suspend hr <name> -n <ns>`命令，暂停Release的自动升级。
3. 选择`status=pending-upgrade`的版本，运行`helm uninstall <name> --namespace <ns> <REVISION>`命令，卸载该版本。
4. 运行`flux resume helmrelease <name> -n <ns>`命令，重新启动Release。

以上步骤可解决该错误。
