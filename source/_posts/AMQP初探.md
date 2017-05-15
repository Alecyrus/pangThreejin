---
title: AMQP初探
date: 2016-06-16 04:57:21
tags:
 - AMQP
---

## AMQP是什么
 AMQP 全称为( Advanced Message Queuing Protocol), 它是一个跨平台的提供统一消息服务的应用层标准协议，它允许来自不同供应商的生产以及消费者之前的 相互通信，因此，基于AMQP协议开发的客户端与消息中间件之前的通信并不受供应商，开发语言等条件的限制。 

 基于AMQP协议的常见消息中间件有QPID、RabbitMQ、ActiveMQ等等。
 
---------
## 基本概念

### 生产者 Producer
产生消息，并将消息递交给Exchange
### 交换机 Exchange
负责消息将正确地转发到相应的队列
### 队列 Queue
存储消息的容器(队列)，先进先出
### 消费者 Consumer
从队列中取出消息并进行处理
### 路由键 `Routing_key`
在生产者将消息发送到Exchange时需要用到的参数。
> 该参数说明了**消息的属性**，换言之，说明了该消息是什么类型的消息。
### 绑定键 `Binding_key`
在交换机将生产者递送来的消息转发到与此交换机绑定的队列时，需要用到的参数。
> 该参数说明了**队列的属性**，换言之，说明了该队列愿意接纳什么样类型的消息

---------
## 消息转发方式
取决于交换机的类型，如下：
### Direct型
`单播型`： 在路由键和绑定键**完全匹配**的消息和队列之间建立转发通道
### Fanout型
`广播型`： 将消息和当前**所有队列**(和当前exchange建立绑定)之前建立转发通道
### Topic 型
`组播型` ： 路由键和绑定键是**模糊匹配**的，匹配规则如下：
>格式： `<elem>.<color>.<specific>`
>
> `* ` : 匹配一个单词，表示该处必须有一个单词
> 
>  `#` : 匹配任意个单词，表示该处有没有单词都行
>  
>  示例： 消息路由键为`*.color.#`将与队列绑定键为`parameter.color`或者`parameter.color.red`匹配 

### Header 型
>  Message是AMQP模型中所操纵的基本单位，它由Producer产生，经过Broker被Consumer所消费。它的基本结构有两部分: Header和Body。Header是由Producer添加上的各种属性的集合，这些属性有控制Message是否可被缓存，接收的queue是哪个，优先级是多少等。

所以，对于RabbitMQ，由于routing_key以及其他一些参数，充当了消息头的角色，故，RabbitMq中并未实现header    
```python
# send message
message =  "Hello World!"
channel.basic_publish(exchange='',
                  routing_key='color',
                  body=message,
                  properties=pika.BasicProperties(
                     delivery_mode = 2, 
                      ))
```
