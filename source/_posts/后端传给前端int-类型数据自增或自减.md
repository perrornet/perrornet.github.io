---
title: 后端传给前端int 类型数据自增或自减
date: 2018-03-16 19:53:35
tags:
    - bug
---
由于我使用的python3 ,在python中int 类型不像其他语言的int类型， python 将long类型也加入到了int中， 所以再python中能够正确显示的int类型在其他的语言中不一定能够正确的显示， 当python 传的int 超过了浏览器所能解析的最大值时就会出现这种情况， 建议将较大的int 类型转换成string 在传给前端。
