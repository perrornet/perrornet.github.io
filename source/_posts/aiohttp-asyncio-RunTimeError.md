---
title: aiohttp,asyncio,RunTimeError
date: 2017-07-13 11:04:10
tags:
    - bug
---
Firstly, you may have encountered the AssertionError: There is no current event loop in thread 'Thread-1'. This is because each thread in an asyncio program has its own event loop, but only the main thread will automatically create an event loop. Therefore, if you call asyncio.get_event_loop() in a sub-thread, you will encounter this error.

To solve this problem, you need to explicitly create and set an event loop when starting the thread. Sample code is as follows:

```
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)

```

Once you do this, you should be able to use get_event_loop() to work properly in a specific thread.
