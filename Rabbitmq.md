# RabbitMQ
###### 本次版本使用3.9
#四大核心概念
###### 生产者
产生数据发送消息的程序是生产者

###### 交换机
交换机是 RabbitMQ 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息 推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推 送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定

###### 队列
队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存 储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可 以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式

###### 消费者
消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费 者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。


# 工作原理    
![img.png](img.png)    

* Broker

接收和分发消息的应用，RabbitMQ Server 就是 Message Broker

* Virtual host

出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似 于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出 多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等

* Connection

publisher／consumer 和 broker 之间的 TCP 连接

* Channel

如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程 序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客 户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。Channel 作为轻量级的 Connection 极大减少了操作系统建立 TCP connection 的开销

* Exchange

message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发 消息到 queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)

* Queue

消息最终被送到这里等待 consumer 取走

* Binding

exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保 存到 exchange 中的查询表中，用于 message 的分发依据



# 安装  
``
docker pull rabbitmq:3
``
![img_2.png](img_2.png)   
```
docker run -d -v /opt/rabbitmq/data:/var/lib/rabbitmq -p 5672:5672 -p 15672:15672 --name rabbitmq --restart=always --hostname myRabbit    rabbitmq:3
```
![img_3.png](img_3.png)   
安装完成后运行插件后可以访问网页的rabbitmq管理页面，输入默认账号密码guest guest      
![img_1.png](img_1.png)     
![img_5.png](img_5.png)    




![img_6.png](img_6.png)   
每一个vhost的仓库和交换机是不一样的   

###### 添加用户  
![img_8.png](img_8.png)  
###### 设置用户权限   
![img_7.png](img_7.png)  

# java使用rabbitmq   
1. 导入依赖     
```
<dependencies>
<dependency>
<groupId>com.rabbitmq</groupId>
<artifactId>amqp-client</artifactId>
<version>5.8.0</version>
</dependency>
<dependency>
<groupId>commons-io</groupId>
<artifactId>commons-io</artifactId>
<version>2.6</version>
</dependency>
</dependencies>
```
2. 写生产者代码  
![img_10.png](img_10.png)    
信道连接队列，这个例子只是个简单的入门程序，不然信道和队列之间还有个交换机       
![img_9.png](img_9.png)     
参数解析:   
``s: 队列名称 b: 是否持久化  b1: 是否排他  b2: 是否自动删除  DeclareOK:  队列参数``   

![img_11.png](img_11.png)   
参数解析:   
``s:发送到哪个交换机 s1:路由的Key值是哪个 本次是队列的名称 basicProperties:其他参数信息 bytes:要发送的消息的二进制形式``

完整代码如下:   
![img_12.png](img_12.png)    

![img_13.png](img_13.png)   
![img_14.png](img_14.png)   
4. 写消费者代码   
前面的代码和生产者的代码一样   
![img_15.png](img_15.png)    
参数解析:
``s: 消费的队列名字 b: 消费成功后是否自动应答 s1:消费者未成功消费的回调 Consumer: 消费者取消消费的回调``   
   

###### 消费者未成功消费的回调
![img_21.png](img_21.png)   
###### 消费者取消消费的回调   
![img_22.png](img_22.png)  

完整代码:    
![img_16.png](img_16.png)   

 

# 工作线程
**多个工作线程线程接受消息采用轮询接收消息，他们之间是竞争关系**      
**一个消息只能被处理一次，不可以处理多次**
![img_18.png](img_18.png)      
#### 测试轮询分发消息     
###### 工具类   
![img_19.png](img_19.png)    
###### 工作线程代码  
![img_20.png](img_20.png)   
###### 生产者代码
![img_27.png](img_27.png)   
###### 测试轮询结果  
![img_24.png](img_24.png)   
![img_25.png](img_25.png)    
![img_26.png](img_26.png)     
**说明了工作队列是轮询分发消息的**   


# 消息应答