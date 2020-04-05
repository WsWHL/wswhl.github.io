---
title: RabbitMQ Delayed Message
date: 2020-04-05 14:45:20
categories: 
  - 每日一记
tags:
  - MQ
  - 消息队列
---
### 利用自身死信队列特性

   设置队列（ Queue TTL）或消息（Per-Message TTL）时，当消息到达队列后在TTL规则指定时间内未被消费时会成为死信，利用DLX特性当消息***到达死亡时间且处于队头***时，可以自动将其重新转发到另一个Exchange或Routing Key，将其路由到指定处理队列以达到延迟的目的。
<!--more -->
   实现步骤：

   - 创建交换机（Exchange）

     设置交换机名称为`delay`，此名称可自定义，后面参数值与此对应。

   - 创建死信队列

     设置队列参数

     ``` json
     {
             "x-queue-type": "classic",
             "x-dead-letter-exchange": "delay",	// 死信交换机
          "x-dead-letter-routing-key": "delay_key",	// 死信路由Key
     }
  ```
  绑定此队列到`delay`交换机，设置路由`routing key`为`delay`。
  
  - 创建处理队列
  
     创建死信处理队列，用于死信消费。
   
     绑定次队列到`delay`交换机，设置路由`routing key`为`delay-key`。
     
   - 生产死信延迟消息
   
     延迟消息参数有两种，一种是设置队的有效期参数`x-expires`，另一种是针对消息设置生存时间`x-message-ttl`。
   
     ***延迟时间单位 millisecond***
   
     第一种：
   
     ``` python
         channel.basic_publish(
             exchange="delay",
             routing_key="delay",
             body="[%s]Send message: Hello World!"
             % time.strftime("%Y-%m-%d %H:%M:%S", time.localtime()),
             properties=pika.BasicProperties(delivery_mode=2, expiration="3000"),
         )
     ```
   
     第二种：
   
     ``` python
     channel.basic_publish(
             exchange="delayed_exchange",
             routing_key="delayed-key",
             body="[%s]Send message: Hello World!"
             % time.strftime("%Y-%m-%d %H:%M:%S", time.localtime()),
             properties=pika.BasicProperties(delivery_mode=2, headers={"x-message-ttl": "3000"}),
         )		
     ```
   
     缺陷问题：
   
     ​		由于消息队列的先进先出原则，该方法在消息不同生存时间刻度的情况下无法保证准确的及时转发到死信处理队列，当生存时间长的消息到达该队列后，后面生存时间短的消息到达该队列时会被生存时间长的消息阻塞，使其在到达失效时间时无法及时出队。
   
     ***到达死亡时间且处于队头。***
   
### 使用延迟消息插件

   使用延迟消息扩展插件：[rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)，该扩展插件能够针对不同生存时间刻度的消息进行排序，以保证生存时间短的消息能够及时达到队头进行出队。

   使用步骤：

   - 安装扩展插件

     下载（[Community Plugins](https://www.rabbitmq.com/community-plugins.html)）该扩展文件将其放置RabbitMQ扩展目录。

     启用扩展插件：

     ``` bash
     rabbitmq-plugins enable rabbitmq_delayed_message_exchange
     ```

   - 配置延迟消息类型交换机

     创建交换机时选择类型为：`x-delayed-message`，设置参数`x-delayed-type`为`direct`，绑定消息处理队列即可。

   - 生产延迟消息

     设置消息生存时间参数`x-delay`，单位毫秒。

     ``` python
     channel.basic_publish(
             exchange="delayed_exchange",
             routing_key="delayed-key",
             body="[%s]Send message: Hello World!"
             % time.strftime("%Y-%m-%d %H:%M:%S", time.localtime()),
             properties=pika.BasicProperties(delivery_mode=2, headers={"x-delay": "3000"}),
         )
     ```

     ***不同时间刻度的消息能够在其失效时及时出队。***