---
title: Celery启动Worker方式
date: 2023-10-15
tags: [Python, Celery]
---


参考[https://www.aiuai.cn/aifarm2045.html](https://www.aiuai.cn/aifarm2045.html)

启动Celery的Worker时，会生成子进程（或线程）执行池用来处理任务。使用 `--pool` 命令行参数选择进程或线程。

### 1、Solo

Solo 池是一个內联池(inline pool)，意味着，任务不会同时处理，其只是创建一个线程(thread) 并使用该线程执行任务。
Solo适用需要逐一执行(one by one)的任务。不过，实际中不使用并发而仅使用 solo pool 的场景不多。

- 使用方式如：
```
celery -A tasks worker --pool=solo --loglevel=info
```

### 2、Prefork 

prefork池是 Celery 对 Python 标准库 multiprocess 的改造，其能够同时处理多个任务。也是默认执行池。

多进程的方式去实现并发，默认的并发数为当前计算机的 CPU 数，可以通过设置 `-c` 进行自定义设置并发数。没有推荐的并发数，因为最佳的并发数取决于很多因素，如果任务主要是 I/O 限制，可以进行增加并发数，经过测试，设置超过两倍的 CPU 数量效果不是很好，很有可能会降低性能;

- 使用方式如：
```
celery -A tasks worker --pool=prefork -c 4 --loglevel=info
```
- 适用场景如：
  + CPU密集型(CPU-bound)，即，任务的大部分时间主要是 CPU 计算；只有 CPU 越快时才会速度更快.
  + CPU密集型任务如：文件转换、压缩、搜索算法等.

### 3、Eventlet & Gevent

<!--more-->

Eventlet & Gevent 池使用协程来执行任务，不是产生传统线程. 能够同时处理多个任务。
gevent是对eventlet的高级封装,一般使用时 用 gevent 因为此包有 monkey.patch_all() 方法将 所有能转为协程的地方都转为协程,从而增加处理能力
注意的是，一些第三方的库，通常指带有C扩展的，由于无法使用猴子补丁，因此不能从使用 Eventet 中获得好处

- Eventlet Pool 使用方式如：
```
celery -A tasks worker --pool=eventlet --concurrency=500 --loglevel=info
```
- Gevent Pool 使用方式如：
```
celery -A tasks worker --pool=gevent --concurrency=500 --loglevel=info
```
- 适用场景如：
  + I/O 密集型任务，即，任务的主要瓶颈是 I/O 操作的等待时间. 与 Prefork 不同的是，其可以设置并发高的数量，而不受限于 CPUs 的数量.
  + I/O密集型任务如，邮件发送、API请求等.

由于 eventlet 和 gevent 不是 Python 标准库，因此需要单独安装：

```
 pip install celery[eventlet]
 pip install celery[gevent]
```

### 4、Threads

使用 -P threads

问题：

- 脚本执行时，通过signal.signal(signal.SIGALRM, handle)设置信号报错：ValueError: signal only works in main thread of the main interpreter
    ```
	Traceback (most recent call last):
  ...
  ...
  File "/usr/python/lib/python3.11/signal.py", line 56, in signal
    handler = _signal.signal(_enum_to_int(signalnum), _enum_to_int(handler))
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  ValueError: signal only works in main thread of the main interpreter
  ```
- 只在主线程设置信号后，子线程执行脚本超时时，整个进程重启
- 任务并发时，报错：AttributeError: 'TaskPool' object has no attribute 'grow'
	```
	Traceback (most recent call last):
  File "/usr/python/lib/python3.11/site-packages/celery/worker/worker.py", line 203, in start
    self.blueprint.start(self)
  File "/usr/python/lib/python3.11/site-packages/celery/bootsteps.py", line 116, in start
    step.start(parent)
  File "/usr/python/lib/python3.11/site-packages/celery/bootsteps.py", line 365, in start
    return self.obj.start()
           ^^^^^^^^^^^^^^^^
  File "/usr/python/lib/python3.11/site-packages/celery/worker/consumer/consumer.py", line 332, in start
    blueprint.start(self)
  File "/usr/python/lib/python3.11/site-packages/celery/bootsteps.py", line 116, in start
    step.start(parent)
  File "/usr/python/lib/python3.11/site-packages/celery/worker/consumer/consumer.py", line 628, in start
    c.loop(*c.loop_args())
  File "/usr/python/lib/python3.11/site-packages/celery/worker/loops.py", line 97, in asynloop
    next(loop)
  File "/usr/python/lib/python3.11/site-packages/kombu/asynchronous/hub.py", line 362, in create_loop
    cb(*cbargs)
  File "/usr/python/lib/python3.11/site-packages/kombu/transport/redis.py", line 1326, in on_readable
    self.cycle.on_readable(fileno)
  File "/usr/python/lib/python3.11/site-packages/kombu/transport/redis.py", line 562, in on_readable
    chan.handlers[type]()
  File "/usr/python/lib/python3.11/site-packages/kombu/transport/redis.py", line 967, in _brpop_read
    self.connection._deliver(loads(bytes_to_str(item)), dest)
  File "/usr/python/lib/python3.11/site-packages/kombu/transport/virtual/base.py", line 991, in _deliver
    callback(message)
  File "/usr/python/lib/python3.11/site-packages/kombu/transport/virtual/base.py", line 624, in _callback
    return callback(message)
           ^^^^^^^^^^^^^^^^^
  File "/usr/python/lib/python3.11/site-packages/kombu/messaging.py", line 626, in _receive_callback
    return on_m(message) if on_m else self.receive(decoded, message)
           ^^^^^^^^^^^^^
  File "/usr/python/lib/python3.11/site-packages/celery/worker/consumer/consumer.py", line 596, in on_task_received
    strategy(
  File "/usr/python/lib/python3.11/site-packages/celery/worker/strategy.py", line 206, in task_message_handler
    [callback(req) for callback in callbacks]
  File "/usr/python/lib/python3.11/site-packages/celery/worker/strategy.py", line 206, in <listcomp>
    [callback(req) for callback in callbacks]
     ^^^^^^^^^^^^^
  File "/usr/python/lib/python3.11/site-packages/celery/worker/autoscale.py", line 95, in maybe_scale
    if self._maybe_scale(req):
       ^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/python/lib/python3.11/site-packages/celery/worker/autoscale.py", line 87, in _maybe_scale
    self.scale_up(cur - procs)
  File "/usr/python/lib/python3.11/site-packages/celery/worker/autoscale.py", line 113, in scale_up
    return self._grow(n)
           ^^^^^^^^^^^^^
  File "/usr/python/lib/python3.11/site-packages/celery/worker/autoscale.py", line 122, in _grow
    self.pool.grow(n)
    ^^^^^^^^^^^^^^
	AttributeError: 'TaskPool' object has no attribute 'grow'
	```
- Celery设置的Task最大执行时间无效

