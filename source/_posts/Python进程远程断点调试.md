---
title: Python进程远程断点调试
date: 2026-05-21
tags:
  - 工具使用
---

pdb 是 Python 标准库自带的模块，适合在本地开发环境（如终端、命令行）中直接调试代码。**无法直接访问终端的后台多进程任务**。在内网环境的服务器上调试时，无法下载其他包，这里实现的是只使用 Python 自带的 sys, socket, pdb 模块。

在项目中新增函数，可以放在要调试的py脚本最上方：

```python
import sys
import socket
from pdb import Pdb

def set_remote_trace(host='0.0.0.0', port=4444):
    """纯标准库实现的远程 Pdb 断点"""
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind((host, port))
    server.listen(1)

    print(f"=== 正在等待远程连接 {host}:{port} ===", flush=True)
    client, address = server.accept()
    print(f"=== 已连接来自 {address} 的调试请求 ===", flush=True)

    # 将标准输入输出重定向到 Socket 连接
    # Python 3 必须处理好文本流与字节流的转换
    stream = client.makefile('rw', buffering=1, encoding='utf-8')

    # 启动 Pdb 并将输入输出绑定至 socket 终端
    debugger = Pdb(stdin=stream, stdout=stream)
    debugger.set_trace(sys._getframe().f_back)
```

在需要调试的地方加入：`set_remote_trace()` 例如：

```python
def my_process():
    x = 100
    y = 200
    
    set_remote_trace()  # 触发远程断点
    
    result = x + y
    print(result)
```

重启服务后，观察日志出现 `=== 正在等待远程连接 0.0.0.0:4444 ===` 即可进入调试阶段

使用：

```bash
nc 127.0.0.1 4444
# 或者
telnet 127.0.0.1 4444
```

后续使用和pdb一致。