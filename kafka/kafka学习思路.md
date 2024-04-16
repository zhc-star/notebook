### P3 Kafka介绍

1. 线程间数据交换, 借助内存和Queue

   问题: 生产者每秒产生50个消息, 消费者每秒20个消息, 容易造成内存溢出. 即使存储到磁盘上, 也会有资源消耗的问题.

2. 进程间数据交换, 借助网络数据流(socket)

   问题: 当有1个进程需要同时给两个进程发送消息时, 需要发送一模一样的两份. 如果给不同进程发送的是不同的消息, 则需要增加逻辑判断, 消耗发送方计算资源

​    归根结底: 解耦可以解决各种问题. 增加消息中间件进行解耦.



### P4 JMS介绍

Java Message Service 可类比 JDBC.

是Java为规范消息中间件的开发和使用, 设置的规范.

各消息中间件及其厂商提供的实现, 可以称之为JMS Provider.



message的组成:

- 消息头
- 消息属性
- 消息主体内容



Kafka 是 JMS Provider 之一.   其底层通过Java的队列实现.



JMS Producer --- JMS Provider --- JMS Consumer

Kafka支持的两种模型:

- P2P模型
- PS模型



相较于ActiveMQ, RabbitMQ, RocketMQ , Kafka并没有以MQ结尾, Kafka只是借鉴了JMS规范的思想.



### P5 组件

Kafka设计的初衷是日志的传输, 所以, Kafka在磁盘中存储数据的文件名以.log结尾.

Kafka Producer

Kafka Broker

Kafka Consumer



### 安装与启动

