---
title: go的channel详解
date: 2023-02-18 17:51:42
top: true
tags:
    - golang
---
此篇文章基于[golang 1.12.1](https://studygolang.com/dl/golang/go1.12.1.src.tar.gz) 版本源码解析而成，chan源码位于go/src/runtime/chan.go文件中。

## Channel是个什么？

在源码中，channel实际是一个结构体，其结构如下：

```
type hchan struct {
   qcount   uint           // 队列数据中的数据数量
   dataqsiz uint           // 循环队列的大小
   buf      unsafe.Pointer // 储存数据的元素数组
   elemsize uint16
   closed   uint32
   elemtype *_type // 元素类型
   sendx    uint   // 发送次数
   recvx    uint   // 接受次数
   recvq    waitq  // 当阻塞时，存放接受者的结构体
   sendq    waitq  // 当阻塞时，存放发送者者的结构体

   // 用于保证数据的并发安全
   lock mutex
}

```

其中，`sendx`以及`recvx`是每次读取或者写入都需要将其加一。而`recvq`、`sendq`的作用是，当缓冲区已满的情况下，会将读取或是发送channel结构体放入其中。

## Channel的创建

创建一个chan我们会使用`make`函数来创建，实际`make`函数调用了`makechan`函数创建，该函数签名如下：

```
func makechan(t *chantype, size int) *hchan

```

其中，`t`是chan接收或者发送的数据类型，`size`则是`make`函数的第二个参数，指明了chan的缓冲大小，默认为0即无缓冲chan，返回`hchan`结构体的指针。创建channel时首先会检测`t`的大小，如果`t`的大小超过了65536个字节，将会触发错误。当chan传递的数据类型大小符合要求后，将会根据`t`类型的特征去创建一定容量大小的内存空间，并将此片内存空间复制给`buf`。值得一提的是，`chan`包中提供了一个名称为`debugChan`的变量，该变量默认为`false`，当它为`true`时，对chan操作时将会打印一些debug信息，这将会帮助我们快速的发现由于chan发生的BUG。

## 向channel中发送数据

向chan中发送数据由[chan.go:142](https://github.com/golang/go/blob/master/src/runtime/chan.go#71) `chansend`函数实现。该函数首先会检查当前channel等于nil并且是一个有缓冲通道则直接返回，如果是一个无缓存通道则调用`proc.gp:284` `gopark`将当前的goroutine设置为waiting状态。随后检测channel时候已经准备好发送数据了，如果已经可以发送数据，则调用chan.go:264 `send`函数发送数据。`send`的作用主要是将要发送的数据`copy`到`buf`中，如果当前channel是无缓冲的，则调用`lock`加锁，阻塞当前goroutine。当前数据被接受后，调用`goready`通知runtime当前goroutine已经准备好再次运行了。如果channel是有缓冲的，则直接返回。

## 从channel中接受数据

接受数据实际跟发送数据流程是一样的，唯一不同的是，当channel是有缓冲的时候，并且channel中没有数据被发送时，则直接将接受chan数据的变量地址存入到`sendq`结构体中。当有发送数据时发现`recvq`中有数据时，直接将数据存入到这个结构体中的地址中，而不再会使用`buf`去`copy`数据。

## 总结

channel使用`copy buf`的方式来通信，最后实现以通信的方式来共享内存。当一个goroutine阻塞的时候，系统线程会把它放入到`hchan.sendq`或者`hchan.recvq` list中，该list中的`sudog`类型的结构题就是当前goroutine，而这个`sudog`结构体中保存着一个变量，该变量保存着channel相关的指针。之前使用chan仅仅只是会使用，而不知其原理，看了其源码，之前不理解的地方也恍然大悟。写这篇文章时是对着源码写的，可能有一些顺序上的矛盾。
