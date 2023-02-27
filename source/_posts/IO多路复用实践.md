---
title: Python IO Multiplexing Practice
date: 2023-02-18 17:51:02
top: true
tags:
    - python
---
In the past few days, I have been studying the Tornado source code and found that although Tornado claims to use an asynchronous model, it actually implements an event loop using IO multiplexing. To deepen my understanding of IO multiplexing, I decided to implement a simple HTTP client myself to compare the performance difference between synchronous and IO multiplexing clients.

Without further ado, let's first take a look at the most intuitive difference between synchronous and IO multiplexing clients. First, let's use Tornado to implement a simple server:

```
# -*- coding:utf-8 -*-
from tornado.web import RequestHandler, Application
from tornado.ioloop import IOLoop
import tornado.gen

class TestHandle(RequestHandler):

    async def get(self, *args, **kwargs):
        await tornado.gen.sleep(1)
        self.write("OK")

if __name__ == '__main__':
    app = Application([('/', TestHandle)])
    app.listen(8080)
    IOLoop.current().start()

```

We pause the server for 1 second when processing requests so that it is more convenient to observe the difference between the two client implementations. Next, let's implement a synchronous client:

```
# -*- coding:utf-8 -*-
import socket
import time

REQUEST_STR = '{method} {path} HTTP/1.0\r\n\r\n'

def block_client(hostname, port, method, path):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((hostname, port))
    sock.send(REQUEST_STR.format(method=method, path=path).encode())
    response_body = []
    while True:
        response_stream = sock.recv(1024)
        if not response_stream:
            return b"".join(response_body).decode()
        response_body.append(response_stream)

if __name__ == '__main__':
    count = 3
    start_time = time.time()
    for _ in range(3):
        block_client("127.0.0.1", port=8080, method="GET", path="/")
    print("{} requests {} times, running time: {:.1f} seconds".format(block_client.__name__, count, time.time() - start_time))

```

Start the server and test how long it takes for the synchronous client to request the server:

```
>>> C:\Users\Administrator>python client.py
block_client requests 3 times, running time: 3.0 seconds

```

As expected, it takes a total of 3 seconds to make three requests because the server paused for 1 second. Next, let's implement an IO multiplexing client and see how it performs:

```
from selectors import DefaultSelector, EVENT_WRITE, EVENT_READ
import socket
import time

REQUEST_STR = '{method} {path} HTTP/1.0\r\n\r\n'
JOBS_COUNT = 0
select = DefaultSelector()

class NoBlockClient:
    def __init__(self):
        self.result = []

    def request(self, method, hostname, port, path):
        global JOBS_COUNT
        JOBS_COUNT += 1
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.setblocking(False)
        try:
            sock.connect((hostname, port))
        except BlockingIOError:
            pass
        select.register(sock.fileno(), EVENT_WRITE, partial(self._send, sock, method, path))

    def _send(self, sock, method, path):
        select.unregister(sock.fileno())
        sock.send(REQUEST_STR.format(method=method, path=path).encode())
        select.register(sock.fileno(), EVENT_READ, partial(self._recv, sock))

    def _recv(self, sock):
        global JOBS_COUNT
        response_stream = sock.recv(1024)
        if not response_stream:
            select.unregister(sock.fileno())
            sock.close()
            JOBS_COUNT -= 1
        else:
            self.result.append(response_stream)

    def run(self):
        while JOBS_COUNT:
            events = select.select()
            for key, mask in events:
                callback = key.data
                callback()
        return self.result

start_time = time.time()
count = 3
no_block_client = NoBlockClient()

for _ in range(count):
    no_block_client.request("GET", "127.0.0.1", 8080, "/")
result = no_block_client.run()
print("{} requests {} times, running time: {:.1f} seconds".format(NoBlockClient.__name__, count, time.time() - start_time))

```

Run the client:

```
>>> C:\Users\Administrator>python no_block_client.py
NoBlockClient requests 3 times, running time: 1.0 seconds

```

The difference in time is very obvious. Although the code looks particularly complicated, to put it bluntly, each function corresponds to an event registered in select. select monitors these events and runs the corresponding function if an event is triggered.

Let's analyze the IO multiplexing client code:

1. `select = DefaultSelector()` This line of code returns the best implementation of IO multiplexing on the current platform, which are select, poll, epoll, dev/poll, and kqueue. Since I am testing on Windows, DefaultSelector() returns the select model.
2. The purpose of the `NoBlockClient.request` function is to create a non-blocking socket connection to the target server and register the current socket in select. When its status is writable, the NoBlockClient._send function is called, which is equivalent to a callback function.
3. `NoBlockClient._send` is responsible for sending the message to the connected server, and finally, it registers the event to select like NoBlockClient.request does.
4. `NoBlockClient._recv` is called when the registered event is readable. It reads the returned data from the server, and the complete request is finished.
5. The `NoBlockClient.run` function is mainly used to let select start looping to monitor the status of these registered events and run callback functions.

The selectors module is a encapsulation of the select module, so we don't have to worry about what model is needed on the current platform. It directly returns the best model for the current platform and integrates the APIs of various models to make it easier for users to write cross-platform code. The most commonly used models include select, poll, and epoll. Next, we will explain the differences, advantages, and disadvantages of these models, as well as their implementation methods.

1. select: select puts the registered events into a list and copies them to the kernel space for monitoring. If one of these events changes, select will copy the list containing the events back to the user space, resulting in a huge waste of resources. If select needs to monitor more and more events, its performance will decrease linearly. Moreover, when select copies the time to the user space, it does not tell the user which event was triggered, but requires the user to traverse them by themselves. Since the more events select monitors, the worse the performance, the system kernel usually limits the number of events that the select model can monitor. In the Python source code ([github: selectmodule.c](https://github.com/python/cpython/blob/master/Modules/selectmodule.c)), the number of events that select can monitor in Windows is limited to 512 using macro definitions. Of course, the biggest advantage of select is that it is supported by almost all platforms.
2. poll: The implementation method of poll is almost the same as that of select, so I won't go into details. See [Poll event mechanism](https://blog.csdn.net/luojian5900339/article/details/54581852) for details.
3. epoll: Compared with select and poll, epoll has undergone a qualitative change. Epoll inserts the registered events into a red-black tree, and each node in the red-black tree is a registered event. Because the time complexity of querying, inserting, and deleting a red-black tree is O(logn), epoll can manage events more conveniently, and it only returns the triggered events when an event is triggered, rather than all events, which greatly increases efficiency. For a more detailed introduction to the epoll model, see [Understanding and in-depth analysis of EPOLL](https://blog.csdn.net/apacat/article/details/51375950).
