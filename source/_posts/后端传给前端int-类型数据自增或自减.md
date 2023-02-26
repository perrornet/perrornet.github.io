---
title: 后端传给前端int 类型数据自增或自减
date: 2018-03-16 19:53:35
tags:
    - bug
---
由于我使用的是Python 3，Python 中的 int 类型与其他语言中的 int 类型不同，Python 将 long 类型也加入到了 int 中。因此，在其他语言中能够正确显示的 int 类型，在 Python 中不一定能够正确显示。当 Python 传递的 int 超过了浏览器所能解析的最大值时，就会出现这种情况。建议将较大的 int 类型转换成字符串后再传给前端。
