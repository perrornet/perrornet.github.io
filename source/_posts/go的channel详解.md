---
title: Explanation of Go channels
date: 2023-02-18 17:51:42
top: true
tags:
    - golang
---
This article is based on the analysis of the source code of version golang 1.12.1, where the chan source code is located in the file go/src/runtime/chan.go.

## What is a Channel?
In the source code, a channel is actually a structure, with the following structure:

```
type hchan struct {
    qcount uint // the number of data in the queue
    dataqsiz uint // the size of the circular queue
    buf unsafe.Pointer // the element array that stores the data
    elemsize uint16
    closed uint32
    elemtype *_type // the type of the element
    sendx uint // the number of sends
    recvx uint // the number of receives
    recvq waitq // the structure to put the receiver when blocking
    sendq waitq // the structure to put the sender when blocking

    // Used to ensure concurrent safety of data
    lock mutex
}

```

`sendx` and `recvx` need to be incremented each time data is read or written. The roles of `recvq` and `sendq` are to store the channel structures of the receiver or sender when the buffer is full.

## Creating a Channel

To create a channel, we use the `make` function, which actually calls the `makechan` function to create the channel. The function signature is as follows:

```
func makechan(t *chantype, size int) *hchan

```

`T` is the type of data to be sent or received by the channel, and `size` is the second parameter of the `make` function, which specifies the buffer size of the channel. By default, it is 0, indicating an unbuffered channel. The function returns a pointer to the `hchan` structure. When creating a channel, the size of `t` is first checked. If it exceeds 65536 bytes, an error will be triggered. If the size of the data type passed through the channel meets the requirements, a memory space of a certain capacity will be created according to the characteristics of the `t` type, and this memory space will be copied to `buf`. It is worth mentioning that there is a variable called `debugChan` in the `chan` package, which is set to `false` by default. When it is set to `true`, some debug information will be printed when operating on the channel. This will help us quickly discover bugs caused by the channel.

## Sending Data to a Channel

Sending data to a channel is implemented by the `chansend` function at [chan.go:142](https://github.com/golang/go/blob/master/src/runtime/chan.go#71). The function first checks whether the current channel is nil and whether it is a buffered channel. If it is, it returns directly. If it is an unbuffered channel, it calls the `gopark` function at `proc.gp:284` to set the current goroutine to a waiting state. It then checks whether the channel is ready to receive data. If it is, it calls the `send` function at `chan.go:264` to send the data. The main function of `send` is to `copy` the data to be sent to `buf`. If the current channel is unbuffered, it locks the current goroutine by calling `lock`. After the current data is received, it calls `goready` to notify the runtime that the current goroutine is ready to run again. If the channel is buffered, it returns directly.

## Receiving Data from a Channel

Receiving data is actually the same as sending data, except that when the channel is buffered and no data has been sent to the channel, it directly stores the address of the variable that receives the chan data into the `sendq` structure. When there is data to be sent, if it finds data in `recvq`, it directly stores the data into the address in this structure instead of using `buf` to `copy` the data.

## Conclusion

The channel uses the `copy buf` method to communicate and ultimately implements sharing memory through communication. When a goroutine is blocked, the system thread puts it into the `hchan.sendq` or `hchan.recvq` list. The `sudog` type structure in this list is the current goroutine, and this `sudog` structure contains a variable that holds a pointer to the channel. Previously, I only used chan without knowing its principle. After reading the source code, I understood the parts that I did not understand before. When writing this article, I followed the source code, so there may be some contradictions in the order.
