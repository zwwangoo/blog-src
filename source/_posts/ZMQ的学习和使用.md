---
title: ZMQ的学习和使用
date: 2019-08-11
tags: [物联网, Python, ZMQ]
---

参考：[ZMQ 指南](https://wizardforcel.gitbooks.io/zmq-guide/content/index.html)
代码：[ZMQ-DEMO](https://github.com/suAdminWen/studyForPython/tree/master/zmq-demo)

> ZeroMQ 是基于消息队列的多线程网络库，其对套接字类型、连续处理、帧甚至路由的底层细节抽象，提供跨域多种传输协议的套接字。

## 生命周期

1 创建上下文。ZMQ 的context上下文是线程安全，可以在多线程中使用，不需要主动为其加锁或解锁。
2 创建和销毁套接字。ZMQ套接字是在后台I/O操作的，无论是发送还是接受消息，它都会先传送到一个本地的缓存队列。
3 配置和读取套接字选项。ZMQ套接字不是线程安全的，在 bind 之前不能使用。
4 为套接字连接。常用的四种协议：机器间(`tcp://`)、进程间(`ipc://`)、进程内(`inproce://`)、广播(`pgm://`)。`bind()`连接的节点称之为服务器，有固定的地址，`connect()`连接的节点成为客户端，其地址一般不固定。
5 发送和接收信息。

## ZMQ的核心消息模式

- 请求-应答模式，将一组服务端和一组客户端相连，用于远程过程调用或任务分发。（REQ-REP）
- 发布-订阅模式，将一组发布着和一组订阅者相连，用于数据分发。（PUB-SUB）
- 管道模式，使用PUSH或PULL的形式组装多个节点，可以产生步骤或循环。（PUSH-PULL）


## 实例（Python实现）

### 请求应答模式

请求-应答模式，将一组服务端和一组客户端相连，用于远程过程调用或任务分发。（REQ-REP）

#### 基础实现

![REQ-REP](https://github.com/anjuke/zguide-cn/raw/master/images/chapter1_1.png)

<!--more-->

- server端(`server.py`)

```python
'''
基础的请求-应答模式。
REP 用于响应方，不能主动发送请求，必须是在有响应之后才可以应答。
'''
import time
import zmq

# 创建zmq上下文, 该上下文是线程安全的，可以在多线程中使用。
context = zmq.Context()
socket = context.socket(zmq.REP)
# 绑定套接字到端点
socket.bind("tcp://*:5555")

while True:
    # 等待客户端请求
    message = socket.recv()
    print("Received request: %s" % message)

    # 做些处理
    time.sleep(1)

    # 响应客户端
    socket.send(b"World")
```

- client端(`client.py`)

```python
''' 
基础的请求-应答模式。
REQ 用于请求方，如果没有应答将一直等待。
'''
import zmq

context = zmq.Context()
print('connecting to hello world server')
socket = context.socket(zmq.REQ)
# 连接套接字到端点
socket.connect('tcp://localhost:5555')

for request in range(10):
    # 主动发送请求
    print("Sending request %s ..." % request)
    socket.send(b"Hello")

    # 等待响应
    message = socket.recv()
    print("Received reply %s [ %s ]" % (request, message))
```

#### 进阶实现和理解

![请求-应答模式，REQ和ROUTER通信，DEALER再和REP通信](https://github.com/anjuke/zguide-cn/raw/master/images/chapter2_10.png)


拓展请求-应答模式，使用broker进行负载均衡。

- REQ(client)只需要和ROUTER对话;
- REP(server)只需要和DEALER对话;

在请求-应答模式中使用到的四种套接字类型：

- DEALER是一种负载均衡，它会将消息分发给已连接的节点，并使用公平队列的机制处理接受到的消息。DEALER的作用就像是PUSH和PULL的结合。
- REQ发送消息时会在消息顶部插入一个空帧，接受时会将空帧移去。其实REQ是建立在DEALER之上的，但REQ只有当消息发送并接受到回应后才能继续运行。
- **ROUTER在收到消息时会在顶部添加一个信封，标记消息来源**。发送时会通过该信封决定哪个节点可以获取到该条消息。
- REP在收到消息时会将第一个空帧之前的所有信息保存起来，将原始信息传送给应用程序。在发送消息时，REP会用刚才保存的信息包裹应答消息。REP其实是建立在ROUTER之上的，但和REQ一样，必须完成接受和发送这两个动作后才能继续。


borker实现(`broker.py`):

```python
import zmq

context = zmq.Context()

brokerfe = context.socket(zmq.ROUTER)
brokerfe.bind('tcp://*:5560')

brokerbe = context.socket(zmq.DEALER)
brokerbe.bind('tcp://*:5561')

# 一个线程中如果有多个sokect,同时需要收发数据时,
# zmq提供polling sockets实现，
# 不用在send()或者recv()时阻塞socket
poller = zmq.Poller()
# POLLIN在recv()端，负责刷新recv端口，来接受信息
# POLLOUT在send()端口，负责刷新send端，来发送消息。
poller.register(brokerbe, zmq.POLLIN)
poller.register(brokerfe, zmq.POLLIN)

while True:
    # poller.poll()轮询
    socks = dict(poller.poll())
    if socks.get(brokerfe) == zmq.POLLIN:
        msg = brokerfe.recv_multipart()
        print('brokerfe', msg)
        brokerbe.send_multipart(msg)

    if socks.get(brokerbe) == zmq.POLLIN:
        msg = brokerbe.recv_multipart()
        print('brokerbe', msg)
        brokerfe.send_multipart(msg)

```

client实现(`client.py`)

```python
'''
拓展请求-应答模式，使用broker进行负载均衡。

REQ(client)只需要和ROUTER对话;
'''
import time
import zmq


context = zmq.Context()

client = context.socket(zmq.REQ)
client.connect('tcp://localhost:5560')

while True:
    client.send(b'HELLO')
    msg = client.recv()
    print(msg)
    time.sleep(3)

```

server实现(`server.py`)

```python
'''
拓展请求-应答模式，使用broker进行负载均衡。

REP(server)只需要和DEALER对话;
'''
import zmq

context = zmq.Context()
server = context.socket(zmq.REP)
server.connect('tcp://localhost:5561')

while True:
    msg = server.recv()
    print(msg)
    server.send(b'OK')
```

### 发布订阅模式

发布-订阅模式，将一组发布着和一组订阅者相连，用于数据分发。（PUB-SUB）

在该模式中，**订阅者(`SUB`)在订阅时，需要设置订阅的内容**。如果你不设置订阅内容，那将什么消息都收不到，订阅信息可以是任何字符串，可以设置多次。只要消息满足其中一条订阅信息，SUB套接字就会收到。另外还需要注意的是，订阅者只能使用`revc`接收信息，不能发送信息；发布者`PUB`只能使用`send`发送信息。

如果发布者没有订阅者与之相连，那它发送的消息将直接被丢弃。

#### 基础实现

![发布-订阅模式](https://github.com/anjuke/zguide-cn/raw/master/images/chapter1_4.png)



这里演示一个天气信息发布的例子，包括邮编、温度、相对湿度。我们生成这些随机信息服务端将信息发送给一组客户端。

发布者(server.py)

```python
from random import randrange
import zmq

context = zmq.Context()
socket = context.socket(zmq.PUB)
socket.bind('tcp://*:5556')

while True:
    # 随机生成一些数据
    zipcode = randrange(1, 100000)
    temperature = randrange(-80, 135)
    relhumidity = randrange(10, 60)

    socket.send_string('%i %i %i' % (zipcode, temperature, relhumidity))
```

订阅者(client.py)

```python
import sys
import zmq

context = zmq.Context()
socket = context.socket(zmq.SUB)

print('Collecting updates from weather server...')
socket.connect('tcp://localhost:5556')

zip_filter = sys.argv[1] if len(sys.argv) > 1 else '10001'
if isinstance(zip_filter, bytes):
    zip_filter = zip_filter.decode()

# 这是订阅内容
socket.setsockopt_string(zmq.SUBSCRIBE, zip_filter)

total_temp = 0
for update_nbr in range(5):
    string = socket.recv_string()
    zipcode, temperature, relhumidity = string.split()
    total_temp += int(temperature)

print('Average temperature for zipcode "%s" was %dF' %
      (zip_filter, total_temp/(update_nbr + 1)))

socket.close()
context.term()

```

