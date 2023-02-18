---
title: helm 报错：another operation (install/upgrade/rollback) is in progress处理方法
date: 2023-02-18 18:01:54
top: true
tags:
    - devops
    - k8s
---
### 背景
使用flux cd来管理helmreleases
### 错误信息
单个helmrelease报错：another operation (install/upgrade/rollback) is in progress。
### 处理方式
1. 运行 `helm history <name> --namespace <ns>`
   ![](/medias/1675238086770.jpeg)
2. 运行： `lux suspend hr <name> -n <ns>`
2. 选择status=pending-upgrade的版本，运行： `helm uninstall <name> --namespace <ns> <REVISION>`
3. 运行： `flux resume helmrelease <name> -n <ns>`

搞定😊
