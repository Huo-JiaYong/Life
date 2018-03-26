## 个人认识

Rabbit 是 AMQP(Advanved Message Queue Protocol) 的一种使用 erlang 实现的消息队列框架

[详细概念](http://blog.csdn.net/whoamiyang/article/details/54954780)

主要应用场景：

1. 异步处理
2. 应用解耦
3. 流量削峰

## 软件安装

1. erlang

   Rabbit 基于 erlang 开发，所以需要安装 erlang 

   1. 下载 [erlang][http://www.erlang.org/downloads]，安装即可

2. Rabbit

   1. 下载 [Rabbit][http://www.rabbitmq.com/download.html], 安装
   2. 使用 sbin 目录下的 rabbitmq-server.bat 启动服务 port: 5672
   3. 使用 rabbitmq-plugins enable rabbitmq_management 开启 manage 服务，可通过 localhost:15672 访问

## 基础概念

1. 分发机制

   1. Round-robin dispatching 

      默认机制，将第 n 个 Message 分发给第 n 个 Consumer。n 是取余后的

   2. Message acknowledgment

      所有的消息发送都需要 Consumer 确认后，才会再给这个客户端发送消息

   3. fair dispatch [ 公平分发 ]

      默认机制可能会存在分配不均的情况，可以通过 prefetch_count 指定权重

2. 路由方式

   1. fanout 广播匹配，每个消息都给每个队列发送一次

   2. direct 直接匹配，通过 Routering key 匹配相应的队列一次

   3. topic 主题匹配，通过 . 来分隔routering key 层级.

      如：crm.office.xx 可以通过 * # 来匹配

3. 持久化

   持久化消息队列能够有效的防止消息丢失，通过 durable=true 开启

## 在 Spring-boot 中使用

引入 spring-boot-starter-amqp 

1. Service 

   1. 在需要使用的地方自动注入 AmqpTemplate.convertAndSend("queueName", object)
   2. 创建一个队列的配置： @Configuration @Bean return Queue("queueName");
   3. 设置 application.yml 的 rabbitmq 属性

2. Client

   1. 在处理的方法或类上加上 @RabbitListener(queues = "queueName")

      若在类是使用注解，需要在处理方法上 @RabbitHandler 注解

**这只是基础使用，参数配置等使用明天再看...**