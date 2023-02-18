---
title: go的channel详解
date: 2023-02-18 17:51:42
top: true
tags:
    - golang
---
此片文章基于[golang 1.12.1](https://studygolang.com/dl/golang/go1.12.1.src.tar.gz)版本源码解析而成，chan源码位于go/src/runtime/chan.go文件中。
<!-- more -->


### channel是个什么？

chan 实际在源码中是一个结构体，该结构体结构如下：

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
其中sendx以及recvx为每次读取或者写入都需要将其加一，而recvq、sendq作用是有缓冲channel缓冲已满的情况下会将读取或是发送channel结构体放入其中。

### channel的创建
创建一个chan我们会使用make函数来创建， 实际make函数调用了makechan函数创建，该函数签名如下：func makechan(t *chantype, size int) *hchan ，t是chan接收或者发送的数据类型， size则是make函数的第二个参数，指明了chan的缓冲大小，默认为0即无缓冲chan,返回hchan结构体的指针。创建channel时首先会检测t的大小，如果t的大小超过了65536个字节的话，将会触发错误，在这里我们可以实验一下，创建一个类型，使得该类型在被创建时大小就超过65535字节：
```
// 使用拼接字符串的方式去创建一个结构体， 该结构有65536 / intSize +1个int元素组成
var d int
a := "type A struct{\n"
for i := 0; i<= ((1<<16) / int(unsafe.Sizeof(d))) + 1; i++{
	a += fmt.Sprintf("\t\t\t\ta%d int\n", i)
}
a += "}"
ioutil.WriteFile("./name.go", []byte(a), 07777)
```
结构体创建完成后，在main函数中尝试在chan中传递这种数据:_ = make(chan A)，此时该代码将会编译不通过并发出错误：’ channel element type too large (>64kB)'。当chan传递的数据类型大小符合要求后，将会根据t类型的特征去创建一定容量大小的内存空间，并将此片内存空间复制给buf，直到此时一个channel就算创建hao了。值得一提的是chan包中提供了一个名称为debugChan的变量，该变量默认为False， 当他为True时， 对chan操作时将会打印一些debug信息， 这将会帮助我们快速的发现由于chan发生的BUG。

### 向channel中发送数据
向chan中发送数据由[chan.go: 142](https://github.com/golang/go/blob/master/src/runtime/chan.go#71) chansend函数实现。该函数首先会检查当前channel等于nil并且是一个有缓冲通道则直接返回，如果是一个无缓存通道则调用proc.gp:284 gopark将当前的goroutine设置为waiting状态。随后检测channel时候已经准备好发送数据了，如果已经可以发送数据，则调用chan.go:264 send函数发送数据，send的作用主要是将要发送的数据copy到buf中，如果当前channel是无缓冲的，则调用lock加锁，阻塞当前goroutine，当前数据被接受后，调用goready通知runtime当前goroutine已经准备好再次运行了。如果channel是有缓冲的， 则直接返回。

### 从channel中接受数据
接受数据实际跟发送数据流程是一样的，唯一不同的是当前channel是有缓冲的时候并且channel中没有数据被发送时， 则直接将接受chan数据的变量地址存入到sendq结构体中。当有发送数据时发现recvq中有数据时， 直接将数据存入到这个结构体中的地址中，而不再会使用buf去copy数据。

### 总结
channel使用copy buf的方式来通信， 最后实现以通信的方式来共享内存。当一个goroutine阻塞的时候，系统线程会把它放入到hchan.sendq或者hchan.recvq list中， 该list中的sudog类型的结构题就是当前goruntine， 而这个sudog结构体中保存着一个变量，该变量保存着channel相关的指针。之前使用chan仅仅只是会使用，而不知其原理， 看了其源码，之前不理解的地方也恍然大悟。写这篇文章时是对着源码写的，可能有一些顺序上的矛盾。
