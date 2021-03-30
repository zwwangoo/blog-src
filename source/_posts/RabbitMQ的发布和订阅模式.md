---
title: RabbitMQ的发布和订阅模式
date: 2021-03-02
tags: [Python, 中间件]
---

RabbitMQ可以通过交换机(Exchange)把消息发送给所有绑定该交换机的队列，实现发布和订阅的模式。

> 发布者会把消息发送给RabbitMQ的交换机（Exchange），Exchange的一侧是发布者，另一侧则是一个或多个由订阅者创建的队列，由Exchange决定一条消息的生命周期。

### 使用Python的pika库

- 发布者声明交换机(Exchange)，交换机名称为 `exchange_1`,交互机类型为`fanout`

  ```
  channel.exchange_declare('exchange_1', durable=True, exchange_type='fanout')
  ```

  `durable=True` 设置交换机持久化，如果交换机不存在就创建。

- 订阅者1声明队列`Q1`，并持久化(`durable=True`)，并与交换机`exchange_1`进行绑定

  ```
  channel.queue_declare('Q1', durable=True)
  channel.queue_bind(exchange='exchange_1', queue='Q1')
  ```

- 订阅者2声明队列`Q2`，并持久化(`durable=True`)，并与交换机`exchange_1`进行绑定

  ```
  channel.queue_declare('Q2', durable=True)
  channel.queue_bind(exchange='exchange_1', queue='Q2')
  ```

- 启动订阅者1和订阅者2，启动生产者发布消息，1和2都可以接收到相同的消息。**如果发布者发布的时候，无队列与交换机绑定，那么交换机会丢掉消息**;因为上面订阅者1和2都做了持久化的设置，当服务都出现问题时，交换机仍然会把消息发送给`Q1`和`Q2`

### 代码演示

发布者

<!-- more -->

```
import pika

amqp_config = {
    'user': 'guest',
    'password': 'guest',
    'host': '127.0.0.1',
    'port': '5672'
}

connection = pika.BlockingConnection(
    pika.URLParameters(
        'amqp://{user}:{password}@{host}:{port}/%2F'.format(**amqp_config)))
channel = connection.channel()

exchange = 'exchange_1'
# exchange_type = 'fanout' 交换机类型为广播模式
channel.exchange_declare(exchange, durable=True, exchange_type='fanout')
data = 'message'
# 发布消息，因为用的是fanout, 所以routing_key值为空
channel.basic_publish(exchange=exchange, routing_key='', body=data)
connection.close()

```

订阅者

```
import time
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

queue_name = 'Q1'
exchange = 'exchange_1'

channel.exchange_declare(exchange, durable=True, exchange_type='fanout')
channel.queue_declare(queue_name, durable=True)
# 绑定队列和交换机
channel.queue_bind(exchange=exchange, queue=queue_name)


print('Waiting for Message.')


def callback(ch, method, properties, body):

    print('Received %r' % (body,))
    time.sleep(10)
    ch.basic_ack(delivery_tag=method.delivery_tag)  # 告诉生产者，消息处理完成


channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue=queue_name, on_message_callback=callback)

try:
    channel.start_consuming()
except KeyboardInterrupt:
    pass

connection.close()
```

