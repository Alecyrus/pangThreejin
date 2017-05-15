---
layout: post
title: RabbitMQ RPC中的一些细节
date: 2016-07-22 05:01:07
tags:
 - RPC
---

### 摘要
RabbitMQ官网的RPC例子只是详细的说明了回调队列的使用，但对于稍微复杂的使用场景并未过多介绍，这里记录一下实际开发时的细节问题.

-------
### RPC服务端与客户端的连接问题
如果RPC服务器与客户端运行在不同的机器上，那么应该直接采用指定具体URL的方式来连接，如下
```python
sparameters = pika.URLParameters(amqp)
conn = pika.BlockingConnection(parameters)
channel = conn.channel()
```


值得注意的是URL的格式: `amqp://username:password@<ip address>:5672/<virtual host>`, 这里的`username`和`password`值的是RabbitMQ的账户与密码，`virtual host`则是虚拟主机名称，虚拟主机可以理解为namespace。而且用户的权限控制也是以虚拟主机作为粒度的。

----------
### 从回调队列看消息队列的特点
1. 指定`exclusive=True`,则当连接断开后会RabbitMQ会自动删除这个队列

2. 由于队列的`exclusive`参数为`True`,故只能在一边声明，一般我们会在客户端声明，即哪里对回调队列中的消息处理，就在哪里声明

3. 由于RabbitMQ中没用超时的概念，而是通过一个`no_ack`参数来作消息确认的，默认值为`False`，即需要对消息进行确认。

4. 下面通过两种场景来说明`no_ack`参数
> 若参数值为`False`, 那么消费者如果因为某些原因宕机后，接收到了但还没还未来得及处理的消息会一直存在在队列中，并由RabbitMQ重新发送给符合转发条件的消费者，也就是消息的持久性。换句话说，消费者只有在收到消息后，并且向RabbitMQ反馈自己已经处理完消息后，RabbitMQ才会在队列中删除该消息。\
> 若参数值为`True`，则不需要对消息进行确认，RabbitMQ确认有消费者接收到消息后就会将消息删除。

5. 队列的持久性，`no_ack`参数可以保证消息的持久性，但那时在队列持久的前提下，如果队列挂了，消息自然也就挂了，比如，RabbitMQ挂掉的时候。因此如果要保证在RabbitMQ挂掉重启后，队列中的消息不丢失，需要指定队列的`durable`参数为`True`，这就是队列的持久性

-----------------
### RPC中Exchange的使用
RabbitMQ官网对于Exchange的使用已有很详细的教程，这里不再赘述，值得注意的是，我们去理解RabbitMQ关于RPC的模型时，抛开那些很专业性的描述不谈，其实其RPC的实现，就是双重的生产消费者，只不过有一重被约定为“Hello world”中给的模型。Client和Server之间就是生产者与消费者的关系，它们之间，你可以实现很复杂的通信模式，而Server处理完消息，返回值给Client时，就是一个"Hello World"的通信模式，而Client通过`correlation_id`参数来辨认回调队列中消息的归属。 

考虑到高并发的效率，消息队列中的消息，不II应该是数量越多越好，也不应该是消息体越长越好，为了保证稳定性，消息应该限定在一定字节内，其余的操作可以交由数据库去处理。

----------------
### 参考：

1.[pika docs][1]
2.[RabbitMQ Tutorials][2]

  [1]: https://pika.readthedocs.io/en/0.10.0/examples/using_urlparameters.html
  [2]: http://www.rabbitmq.com/getstarted.html

