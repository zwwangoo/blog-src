---
title: UDP广播
date: 2024-03-22
tags: [udp]
---

通过利用UDP广播，可以实现一些网络应用程序中需要向整个网络发送信息的场景，比如发现服务、局域网游戏或设备发现等。



## UDP广播原理

当使用UDP协议进行广播时，数据包将被发送到一个特定的广播地址，以便所有位于该网络的主机都能接收到这个数据包。UDP广播的原理如下：

1 **UDP协议**：UDP（User Datagram Protocol）是一种无连接的、轻量级的传输协议，它提供了一种快速而简单的数据传输方式。UDP不同于TCP，它不需要在发送数据之前建立连接，也不保证数据的可靠传输和顺序交付。

2 **广播地址**：在IPv4网络中，广播地址是一个特殊的IP地址，用于向网络中的所有主机发送数据包。常见的局域网广播地址是以特定网络段结尾的地址，比如以`.255`结尾的地址。例如，对于IP地址`192.168.1.0`，广播地址为`192.168.1.255`。

3 **发送UDP广播**：要发送UDP广播，发送方将数据包发送到指定的广播地址，而不是特定的单个主机地址。发送方创建一个UDP数据包，并指定目标地址为广播地址。

4 **网络设备处理**：当网络设备（如路由器或交换机）接收到UDP广播数据包时，它会将数据包转发到与该网络相连的所有主机。这使得所有连接到网络的主机都能接收到这个广播数据包。

5 **接收UDP广播**：接收方主机上的应用程序可以监听指定的UDP端口，以接收来自广播地址的数据包。一旦接收到UDP广播数据包，应用程序就可以处理其中的信息。

6 **局限性**：需要注意的是，UDP广播在跨越不同网络的情况下通常不可行，因为广播数据包通常被路由器所阻止。此外，UDP广播可能存在安全风险，因为所有连接到网络的主机都可以接收到广播数据包。

   

##  UDP广播示例DEMO

1 创建了一个UDP socket，并设置其为广播模式。然后，我们不断发送消息到广播地址`<broadcast>:12345`，并每隔1秒发送一次

```python
import socket
import time

# 创建UDP socket
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_socket.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)

# 广播地址和端口
broadcast_addr = ('255.255.255.255', 12345)

# 发送广播消息
while True:
    message = b"Hello, world!"
    udp_socket.sendto(message, broadcast_addr)
    time.sleep(1)
```

2 创建了一个UDP socket，并设置其为可重用地址。然后，我们绑定到广播地址和端口，以便接收广播消息。通过使用`IP_ADD_MEMBERSHIP`选项，我们加入了一个广播组，以便接收来自特定广播地址的消息。每次接收到广播消息时，我们打印出消息内容以及发送者的地址。

```python
import socket

# 创建UDP socket
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

# 绑定广播地址和端口
udp_socket.bind(('', 12345))

# 加入广播组
udp_socket.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, socket.inet_aton('224.0.0.1') + socket.inet_aton('0.0.0.0'))

# 接收广播消息
while True:
    data, addr = udp_socket.recvfrom(1024)
    print(f"Received message from {addr}: {data.decode()}")
```



## docker容器，如何接收其他主机发送的udp广播

在Docker Compose 中，您可以通过指定端口映射的格式来指定要使用的协议。如果您想要映射的端口只接收UDP协议，您可以在端口映射规范中指定协议为`udp`

```yaml
version: '3.9'
services:
  your_service:
    image: your_image
    ports:
      - "12345:12345/udp"
```

问题：

我的网络模式为 `bridge`，监听服务打印的发送者地址为子网ip，如果想使用udp广播接收，需要更改网络模式，或其他处理方式。

发送广播时，body中的数据包含发送者的ip地址，接收广播时，可以通过解析body中的数据，获取发送者的ip地址，然后进行处理。


