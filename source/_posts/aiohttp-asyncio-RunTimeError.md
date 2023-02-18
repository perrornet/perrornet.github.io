---
title: aiohttp,asyncio,RunTimeError
date: 2017-07-13 11:04:10
tags:
    - bug
---
首先，你得到的AssertionError: There is no current event loop in thread ‘Thread-1’.是因为asyncio程序中的每个线程都有自己的事件循环，但它只会在主线程中为你自动创建一个事件循环。所以如果你asyncio.get_event_loop在主线程中调用一次，它将自动创建一个循环对象并将其设置为默认值，但是如果你在一个子线程中再次调用它，你会得到这个错误。相反，您需要在线程启动时显式创建/设置事件循环：
```
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)
```
一旦你这样做，你应该能够使用get_event_loop()在特定的线程。