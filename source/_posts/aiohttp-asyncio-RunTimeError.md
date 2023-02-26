---
title: aiohttp,asyncio,RunTimeError
date: 2017-07-13 11:04:10
tags:
    - bug
---
首先，你可能遇到了AssertionError: There is no current event loop in thread ‘Thread-1’。这是因为asyncio程序中的每个线程都有自己的事件循环，但只有在主线程中才会自动创建一个事件循环。因此，如果你在子线程中调用asyncio.get_event_loop()，你将会遇到此错误。

为了解决这个问题，你需要在启动线程时显式地创建和设置事件循环，示例代码如下：

```
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)

```

一旦这样做，你应该就能够使用get_event_loop()在特定的线程中正常工作了。
