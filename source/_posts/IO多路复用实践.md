---
title: python IO多路复用实践
date: 2023-02-18 17:51:02
top: true
tags:
    - python
---
最近几天一直在看tornado源码，发现tornado虽然标榜使用异步模型实现， 但是实际上是使用IO多路复用实现的事件循环。为了加深对IO多路复用的印象，我决定自己实现一个简易的HTTP客户端以比较同步客户端和IO多路复用客户端的性能差别。

废话少说，现在先来看看同步客户端与IO多路复用客户端最直观的区别。首先使用tornado实现一个简单的服务器：

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

我们让服务端在处理请求时暂停1秒，以便更方便地观察两种方式实现的客户端区别。接下来先实现同步客户端：

```
# -*- coding:utf-8 -*-
import socket
import time

REQUEST_STR = '{method} {path} HTTP/1.0\\r\\n\\r\\n'

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
    print("{} 请求{}次, 运行时间: {:.1f}秒".format(block_client.__name__, count, time.time() - start_time))

```

启动服务端，开始测试同步客户端请求服务端需要多少时间：

```
>>> C:\\Users\\Administrator>python client.py
block_client 请求3次, 运行时间: 3.0秒

```

不出所料，请求三次由于服务器暂停了1秒，总计使用时间为3秒。接下来实现IO多路复用客户端，来看看IO多路复用的表现：

```
from selectors import DefaultSelector, EVENT_WRITE, EVENT_READ
import socket
import time

REQUEST_STR = '{method} {path} HTTP/1.0\\r\\n\\r\\n'
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
print("{} 请求{}次, 运行时间: {:.1f}秒".format(NoBlockClient.__name__, count, time.time() - start_time))

```

运行客户端：

```
>>> C:\\Users\\Administrator>python no_block_client.py
NoBlockClient 请求3次, 运行时间: 1.0秒

```

非常明显的看见时间上的区别，虽然代码看起来特别的复杂，但是说的直白一点就是将每一个函数对应一个事件注册进sleelct中，由sleect进行监控，如果有事件触发就运行对应的函数。

我们分析下IO多路复用客户端代码：

1. `select = DefaultSelector()` 该行代码返回了当前平台IO多路复用最佳实现方式，分别为：select、poll、epoll、dev/poll、kqueue。由于我是使用win来测试，所以DefaultSelector()返回了select模型。
2. `NoBlockClient.request` 函数的目的是创建一个非阻塞套接字连接至目标服务器，并将当前套接字注册进select中。当其状态为可写时，运行NoBlockClient._send，相当于一个回调函数。
3. `NoBlockClient._send` 负责将消息发送至已连接的服务器，最后如同NoBlockClient.request一样注册事件选择回调函数。
4. `NoBlockClient._recv` 当注册事件为可读时，将运行该函数，读取服务器返回数据，至此一次完整的请求就结束了。
5. `NoBlockClient.run` 函数主要为了让select开始循环监听这些注册事件的状态，并运行回调函数。

selectors模块是对select模块的封装，使得我们不用在意当前平台需要使用什么模型，而是直接返回当前平台最佳的模型，并将各个模型的API进行整合，让使用者能够更方便地写出跨平台代码。最经常使用的几种模型包括：select，poll，epoll。接下来将讲述这几种模型的区别、优缺点以及大概的实现方式。

1. select：select将被注册的事件放入一个列表中并拷贝到内核空间进行监听。如果这些事件其中一个有了变化，那么select将再次把包含事件的列表拷贝进用户空间，这就造成了资源上的极大浪费。如果select只监听一个或者两个事件还好，但是当select需要监听的事件越来越多时，select的性能将会直线下降。而且select将时间拷贝到用户空间时并不会告诉用户哪一个事件被触发了，而是要用户自己去遍历。因为select监听的事件越多性能越差，所以通常系统内核都会对select模型监听数量进行限制。在Python源码中（[github: selectmodule.c](https://github.com/python/cpython/blob/master/Modules/selectmodule.c)），使用宏定义将win中select监听事件限制在了512。当然，select的最大优点则是几乎所有的平台都支持select。
2. poll：其实现方式几乎与select相同，所以select有的缺点poll也拥有，这里就不多说了。详情见[poll事件机制](https://blog.csdn.net/luojian5900339/article/details/54581852)。
3. epoll：epoll相比较select、poll有了质的改变，epoll将注册的事件都插入到了红黑树中，红黑树中的每一个节点都是一个注册的事件。由于红黑树查询、插入、删除时间复杂度都是O(logn)，所以epoll能够更加方便地对事件进行管理，并且其在事件被触发时仅仅返回被触发的事件，而不是像select全部返回，这大大增加了效率。更加详细的epoll模型介绍见[EPOLL的理解和深入分析](https://blog.csdn.net/apacat/article/details/51375950)。
