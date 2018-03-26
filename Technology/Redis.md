## 个人认识

redis 是一种内存数据库，即所有的数据都在内存中。可以通过持久化将数据保存到硬盘上

以快速存取为最大特点，速度最高可达 10W/s 

主要用来缓存数据

## 问题解决

1. 组建集群失败，卡在 check cluster join ... 

   A：每个节点对应一个 redis-server.exe，表示多个服务端。通过同一个 exe 启动的不同配置文件被认为是修改配置，即只启动了最新配置的 server 

## 安装软件

1. windows （官方不支持 windows, 由微软的开源团队维护 win 版本）
   1. 下载 redis.msi 安装（可以直接在百度软件中下载）
   2. 安装自动添加了 redis 服务，采用默认 redis.conf 配置
   3. redis-server --service-install redis.win.conf --loglevel verbose 安装服务
   4. redis-server --service-stop/start 关停/启动
2. linux 
   1. wget http://download.redis.io/releases/redis-4.0.5.tar.gz
   2. tar -zxvf redis-4.0.5.tar.gz
   3. cd redis-4.0.5
   4. make
3. 客户端连接
   1. redis-cli -c[cluster模式] -h xx -p xx

## 基本配置

只要有基础参数即可运行单例的 redis，若没有配置则会自动加载默认参数

1. bind [ 域名 ]
2. post  [ 端口 ]
3. daemonize [ Linux 后台 ]

## 搭建集群

集群主要用来应对超过单例的数据量的时候，扩展容量并能启动备份数据的作用

通过分段的方式来将 16384 个插槽平分给所有的节点，对 key 进行 CRC16 方式计算出对应所在的插槽值

默认集群需要超过 6 个节点，并且子节点必须是单数？

1. 新增配置
   1. cluster-enable yes
   2. cluster-config-file nodes.config [ 集群配置文件 ]
   3. cluster-node-timeout 5000 [ 掉线时间 ]
   4. 复制集群配置并修改不同端口
   5. **复制 redis-server.exe 对应每个配置文件**，即每个配置文件对应一个 exe
   6. 启动集群的所有节点
2. 开启集群
   1. 安装 ruby [ 百度下载 ]，**提示安装 msys2 可不用安装**
   2. 使用官方提供的 redis-trib.rb 来开启集群
   3. redis-trib.rb --replicas 1[每个节点的子节点数量] host:port host:port
3. 通过 redis-cli -c 使用集群模式连接

## 搭建高可用集群

通过 redis-sentinel 方式，为主节点建立多个监听节点

当主节点断线时会通过 slave-priority 的值，将最大值的节点提升为主服务器

1. 新增配置 [ 子节点 ]
   1. slaveof host port
2. sentinel 配置
   1. **slave monitor [master-name]**
   2. slave down-after-milliseconds xxx
   3. slave failover-timeout xxx
3. 启动节点和 sentinel ,通过 info replication 和 info sentinel 来查看集群情况
4. 当主节点掉线时，会通过 sentinel 来选举出某个子节点作为主节点
5. **选举出的子节点会修改物理配置文件内容**

## 在 Spring-boot 中使用 

1. 单例 redis 

   1. 加入 spring-data-redis 依赖
   2. 直接使用 RedisTemplate 即可 [ 可以不用管配置文件，有默认配置 ]

2. **集群 / 高可用 在 spring-boot 1.4 中加入，直接配置即可**

   **之前版本手动加载配置文件到程序中并生成 RedisTemplate**

3. 集群 redis 

   ```yaml
   spring:
     redis:
       cluster:
         nodes: 
         	# - 表示是个数组
           - 127.0.0.1:6379
   ```

4. 高可用 redis

   ```yaml
   spring:
     redis:
     	cluster:
     	  nodes:
     	  	- 127.0.0.1:6379 ...
       sentinel:
         master: master-name
         nodes: 
           - 127.0.0.1:16379
           - 127.0.0.1:26379
   ```