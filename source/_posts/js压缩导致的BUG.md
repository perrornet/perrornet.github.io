---
title: js压缩导致的BUG
date: 2023-05-22 15:26:26
tags:
  - 前端
---
## 问题描述

基于gin-vue-admin项目开发了一个后台管理项目，在上线后一直挺好的，但是最近一段时间总是出现各种问题， 比如登录成功后白屏、切换路由之后白屏以及动态切换路由显示错误。
这些问题非常奇怪，在本地开发环境一切正常，只要上了服务器就有问题。

## Debug过程

1. 首先想到的是线上build的前端node版本与本地node版本不同，开发环境使用的是16.17.0，服务器中使用的是16.所以就先将服务器中的node版本降级到16.17.0，但是问题依旧。
2. 在查看服务器版本的报错信息之后发现是某个js出现了错误：`o.exports=a(1.valueOf)o.exports=a(1.valueOf)` 这行代码导致的问题，于是对比服务器上文件以及本地文件发现了问题，本地文件是`o.exports=a(1 .valueOf)`，服务器文件是`o.exports=a(1.valueOf)`，缺少了一个空格，随即联想到是cloudflare的js压缩导致的问题。
3. 尝试关闭cloudflare auto minify功能后再次尝试，问题解决。
4. 但这也不是办法，再给cloudflare反馈的同时，需要继续查出为什么会这样， 于是在项目中搜索`1 .valueOf`类似的代码，发现了`1..valueOf`这样的代码，应该就是这个问题导致了
5. 该代码在[chunk-vendors.2e7c88f1.js](https://github.com/flipped-aurora/gin-vue-admin/blob/main/server/resource/page/js/chunk-vendors.2e7c88f1.js)中，但是由于作者没有写上任何关于该文件的来源（没有版本号、commit hash），但是可以了解到这是可以正常运行的。
6. 那么为什么`1..valueOf`会被改成`1 .valueOf`这样了？只有一种可能那就是又被压缩了，于是发现了[chunk-vendors.2e7c88f1.js](https://github.com/flipped-aurora/gin-vue-admin/blob/main/server/resource/page/js/chunk-vendors.2e7c88f1.js)根本没有直接输出，而最后`1 .valueOf`代码是由vite生成的于是上面的debug步骤作废。
7. 于是发现了https://cn.vitejs.dev/config/build-options.html#build-minify 文档中显示默认使用了esbuild来压缩文件，于是尝试着使用terser来压缩文件，问题解决。