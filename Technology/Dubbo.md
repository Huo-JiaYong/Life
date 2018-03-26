## 个人认识

Dubbo 是一种服务化的框架，将程序提供的功能作为服务开放

能有效降低各个程序之间的耦合

RPC （remote procedure call） 远程服务调用

## 软件安装

Dubbo 是一种框架，只需要引入 jar 包

1. Zookeeper 

   zookeeper.tar.gz 中包含 .exe 程序

   复制 zookeeper-sample.conf 重命名 zoo.conf （在 zkServer.cmd 中可以修改目标配置的文件名）

## 在 Spring-boot 中使用

1. Server

   1. 引入 io.dubbo.springboot -> spring-boot-starter-dubbo

   2. 新增 application.yml 配置

      ```yaml
      spring:
      	dubbo:
      		application:
      			name: xxx
      		registry:
      			address: zookeeper://127.0.0.1:2181 [注册中心]
      		protocol: [指定协议]
      			name:dubbo
      			port:20880
      		scan: xxx.xxx [扫描包]
      ```

   3. 在需要暴露的服务中使用 @Service 注解 

      注：要使用 dubbo 的注解

2. Client

   1. 同上

   2. 新增 application.yml 配置

      ```yaml
      spring:
      	dubbo:
      		application:
      			name:
      		registry:
      			adddress: zookeeper://127.0.0.1:2181
      		scan: xx.xx
      ```

   3. 在需要使用服务的地方中直接通过 @Service 注解来获取接口，并直接调用方法即可