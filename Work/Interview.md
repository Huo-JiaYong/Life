## 2025.05.09 致上联品（外包到川航）

1. **自我介绍还是有点磕巴，要整一套说法来衬托自己比较牛。**经历应该还是可以的

2. **写在上面就是用了 ！别说自己在项目中没用（所以工具都要想好在项目中的应用场景）**

3. redis 的集群部署方式和同步数据

   主从复制：一个主节点写，从节点读

   Sentinel：实现高可用，当主节点掉线的时候进行主节点选举

   1. 定时监控：定期通过 PING 命令向节点发送心跳包
   2. 主观下线：若节点未回复则认为其客观下线
   3. 客观下线（多节点确认）：当多个 Sentinel 节点都认为其下线
   4. 选出新主节点：当主节点被发现下线后，则通过类似 Raft 算法选举出新主节点，通知其他节点

   Redis Cluster：

   1. 设置节点：所有节点需要开启 cluster-enable yes 启动集群模式
   2. 节点握手：节点通过 Gossip 协议彼此通信，发送 cluster meet{ip}{port} 进行握手
   3. 分配槽：根据节点将 16384 个槽分配到各个节点

4. RDB 和 AOF 的区别

   基本没错，可以多说一些细节

5. mysql 的分库分表

   使用 shardingsphere 进行分库分表的，需要模拟出一个场景

6. Sentinel 的使用方式

   主要是用来进行限流和降级的，也可以模拟出一个场景

7. seata 的集群部署方式

   启动多个 seata 实例，并将其注册到同一 nacos 上即可，通过 seata.register.nacos.cluster: master/slave 来确定主从节点

8. ES 分词器和倒排索引

   应该说几个常见的：Standard analyer、simple analyer（转小写去数字）、stop analyer（去 the a 等）、中文IK 、aliNLP 等

9. mybatis 的动态 SQL ，深入下原理

   就是\<if> \<foreach> \<where> \<set> 标签，

**总结：**问的比较偏实践，但也应该掌握。



## 2024.01.28 霸王茶姬

1. **回答语言的串联，言简意赅让别人感觉你是个高手**

2. 什么是快速失败 和 安全失败 ？ 

   快速失败：用迭代器进行迭代的时候其他的操作都会失败 检查modCount

   安全失败：复制一个数组进行迭代

3. 常见的设计模式背一下

4. AQS 的实现机制 ？

5. synchroized 的原子性、可见性、有序性、可重入 ？

   原子性：代码块单线程访问

   **可见性：加锁前清空工作内存的共享变量值，从内存中重新读。其他线程无法获取共享变量。必须把最新值刷会主变量**

   **有序性：单线程访问执行结果的有序性**

   **可重入：锁对象有计数器，每次+1，退出-1，直到清零释放锁**

6. 锁升级的过程 ？（不可逆）

   偏向锁 - 轻量锁 - 重量锁

7. redisson 分布式锁的机制

   通过 lua 脚本执行语句，**释放锁：直接删除，让其他线程可以插入**

8. RabbitMQ 的延迟消息实现

   死信队列：设置过期时间，消息将投递到 x-dead-letter-exchange 中

   延迟插件：rabbitmq-delayed-message-exchange

9. JAVA 的内存模型

   每个线程都有自己的控制器、运算器、共享变量副本（L1 cache ） 多个线程之间可能还有共享的 L2 cache ，再进一步就是主内存



## 2017.12.25 叮当师傅 [ 二手车交易平台 ]

1. MQ的作用

   消息队列中间件是 **分布式系统** 中重要的组件

   主要解决应用耦合，异步消息，流量削锋等问题

   1. **从分布式系统的角度来说这个消息队列有什么用**
   2. 常见的场景：利用其异步的特性实现异步处理，分布式中各个系统的业务通信，利用其长度可控和异步性质实现流量削峰，利用点对点实现消息通信

2. Dubbo 的作用

   是一种高性能的 RPC （remote procedure call） 远程服务调用框架

   用以实现各个系统之间的松耦合

   **实现服务之间的高性能的输入和输出**

3. Tomcat 的集群式部署

   打开 tomcat 中的 server.xml 配置中的 cluster 配置

4. 前端的加密方式

   数据加密 RSA 公钥加密 私钥解密=---0ppppppppppppppppp

#### 面试总结

1. 不要使用任何怀疑的语气

   说出自己清楚的点，不清楚的不说

   **不要使用怀疑自己的语气，即使错了**

   自己都不相信，凭什么别人会相信

2. 不要主动提及前公司是外包性质

   **提问到相关的技术，别通过外包的性质来逃避。没用过就是没用过**

   问到网站访问量怎么办？A：如果在秒杀的时候大概会有 100 200 QPS 的样子，平时的话就20 30 QPS吧

3. 别把东西想的太复杂

   **做过就相当于在项目中用过**

   他也不可能知道你遇见了那些问题，而不一定能问到相同的问题

## 2018.1.2 企鹅医生 [ 医疗行业 ]

1. 微信支付和退款流程

   1. 扫码商品码 h5
      1. 用户扫码提交，微信调用回调 URL
      2. 商户调用统一下单 API，生成预付交易
      3. 点击付款并输入密码
      4. 异步通知商户或商户主动
   2. 扫支付码
      1. 用户下单，商户调用统一下单 API 
      2. 获取 CODE_URL 并生成二维码
      3. 用户扫描二维码付款并支付

2. 秒杀场景

   1. 去除重复点击  ——前端限定点击次数
   2. 去除多个浏览器多次请求 ——根据 UID 来限定访问频度
   3. 去除数据库的多次请求 ——通过消息队列来限定人数

3. Spring FactoryBean 和 BeanFactory 的区别

   FactoryBean 是一个Bean，他实现了 FactoryBean 接口 获取的是 FactoryBean.getObject() 对象而不是工厂对象

   BeanFactory 是工厂类，用以创建和获取 Bean 对象

4. AOP 用到了那些设计模式

   适配器模式：两个类有不同的功能，通过适配类来将一个类的方法添加到另一个类中

5. redis 集群动态新增节点

   1. redis-trib.rb add-node 新增节点
   2. redis-tirb.rb reshard 重新分配哈希槽（key 使用 crc16 确定分配槽）

6. ArrayList 和 LinkedList 特点

   ArrayList 新增 查找快 删除慢（需要移动元素）

   LinkedList 新增 查找慢 删除快 （只需要修改指针）

7. 什么情况下必须使用 mybatis 的 $

   1. 在非用户设定值的情况下可以使用
   2. 在固定值的情况下（如 order by）可以使用
   3. **必须使用的场景未知...**

8. ConcurrentHashMap 为什么要分为 16 个区

   1. 16 个段是默认设置，可以修改
   2. 每个段被称为 segment

#### 二面 [ 技术组长 ]

1. 如何实现 hash 和 有序的 map

   LinkedHashMap 在 HashMap 的基础上，通过维护一个链表来实现有序的 HashMap

2. Futrue 是怎么拿到的线程处理结果

   实现 Callable<T> 类的 Call 方法需要返回指定的泛型对象，

   线程的自旋锁可以保证其获取到结果

3. Concurrency 包中常用的 API

   **BlockingQueue** 阻塞队列

   1. 当队列为空的时候，get 请求被阻塞
   2. 当队列满的时候会提示队列已满

   **Lock** 锁对象

   1. 让锁更加公平的分配
   2. 可以指定唤醒的线程

4. Spring 加载对象的生命周期

   根据不同的加载类型来确定加载的顺序（ 其实也是创建对象，只是创建对象的工作由spring 来完成）

   1. singleton 【默认】在每次使用的时候都返回同一个对象
   2. prototype 会在每次使用对象的时候都产生一个新对象
   3. Lazy 懒加载 
      1. lazy-init = false 【默认】 在 Spring 启动的时候就创建单例对象
      2. lazy-init = true 在使用的时候才创建对象

5. Spring MVC 是如何加载到 tomcat 中的  [参考链接](https://www.cnblogs.com/luoluoshidafu/p/6442055.html)

   1. 使用 ContextLoaderListener 监听 Web 容器启动，调用实现 ServletContextListener 接口中的 contextInitialized() 方法加载 WebApplicationContext （根上下文）。并将其存储到 ServletContext 下
   2. 然后调用实现的父类方法：ContextLoader.initWebApplicationContext() 加载 ApplicationContext
   3. 根据设置的 url-parent 将所有的请求都通过 Spring MVC

6. Spring MVC 是如何工作的 [参考链接 底部](https://www.cnblogs.com/hadoop-dev/p/6912652.html)

   1. DispatcherServlet 前端控制器接收发过来的请求，交给 HandlerMapping 处理器映射器
   2. HandlerMapping 处理器映射器，根据请求路径找到相应的 HandlerAdapter 处理器适配器（处理器适配器就是那些拦截器或 Controller）
   3. HandlerAdapter 处理器适配器，处理一些功能请求，返回一个 ModelAndView 对象（包括模型数据、逻辑视图名）
   4. ViewResolver 视图解析器，先根据 ModelAndView 中设置的 View 解析具体视图
   5. 然后再将 Model 模型中的数据渲染到 View 上

7. PageHelper 是如何实现分页的 [参考链接](http://www.ccblog.cn/92.htm)

   Mybatis 留有 plugin 的接口，而 PageHelper 使用拦截器获取了 MyBatis 的处理。并在流程中的 doProcessPage() 方法中通过查询总数和根据设置的分页数来将参数加入到 sql 查询中

8. 如何更新 redis 的数据

   使用 Spring 的 @Cacheable 注解，即可指定 key。


#### 面试总结：

1. 不懂的东西还很多，每个东西都知道一点。但是稍微问深一点就不知道了
2. **注重细节！！！**

## 2018.3.5 上市吧信息 [外包]

1. 为什么重写 equals() 必须同时重写 hashCode()

   **如果 equals() 判定两个对象相等，那么两个对象的 hashcode 就必须一样**

2. 如何让 A 线程执行完后 B 线程再执行

   1. b.wait() a.notify()
   2. b.join()

   **这是对线程基础方法的不了解**

3. 如何指定线程执行时间，超过时间中止

   1. while 循环询问时间
   2. interrupt() 中断线程

4. 当在查询时如何利用索引

   索引是用来加快查询速度的，一般用在 where 子句中

   组合索引：一个索引中有多列

#### 面试总结：

1. 问题基本能回答上来，但是整个人还是懵逼状态
2. 信心还是欠缺，说普通话声音太小....
3. **如果对方觉得你工资叫的太高，就可以直接问公司的价格**

## 2018.3.7 四川奇迹 [ 外包 ]

1. 如何找出需要回收的对象

   **不要说的太多！！！** 只需要回答上问题就行了，**说的太多别人感觉你在背书**

2. 你做的项目在运行么

   **在**，看不看是他的事

3. 如何设计一个回滚机制

   将传入的变量和新值保存到一个 map 中，只有当提交的时候才更新变量

   **回滚机制的根本在于：将修改缓存，只有提交的时候才修改真实数据**

#### 面试总结：

1. 不要像背书一样回答问题！！！
2. **处理实际问题的时候：想清楚他的核心思想，可以随便说，但是不要不说**
3. 工资根据公司的来
   1. 填表的工资：可以稍微低点  11K
   2. 聊得很好的工资：12K
   3. 聊的一般的工资：11K
4. 如果问到工作经验，就说去深圳的时候有 15 K

## 2018.3.7 网思科平 [ 互联网安全 ]

1. Redis 为什么需要六个节点做集群

   在加入**高可用**的时候，当主节点掉线了，需要和其他主节点确认。奇数能保证判定成功

2. **Redis 的常用配置 ** [详解](https://www.cnblogs.com/AlanLee/p/7053577.html)

   ```shell
   # 引入公共配置
   include xx/xx/comm.conf

   # 后台运行
   daemonize yes

   port 6379
   bind 127.0.0.1

   # 密码 (因为速度很快，所以破解更简单)
   requirepass xxx

   # 日志
   loglevel DEBUG VERBOSE NOTICE WARNING
   logfile xxx/xx.log

   # 持久化
   dbfilename xxx.rdb

   # Redis 的数据备份机制：按时间段和操作数同时备份。超过最大次数就备份一次，或者某个时间段备份一次
   save 900 1
   save 300 10
   save 60 10000
   # 当 60 秒内有 10000 个请求就保存
   # 没有 -> 300 秒内有 10 个请求就保存
   # 没有 -> 900 秒内有 1 请求就保存

   # 从服务器
   salveof 127.0.0.1 6379

   # 最大内存
   # Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key
   maxmemory <bytes>
   # 移除机制
   # volatile-lru -> 利用 LRU 算法移除设置过过期时间的 key (LRU: 最近使用 Least Recently Used)
   # allkeys-lru -> 利用 LRU 算法移除任何 key
   # volatile-random -> 移除设置过过期时间的随机 key
   # allkeys->random -> remove a random key, any key 
   # volatile-ttl -> 移除即将过期的 key(minor TTL)
   # noeviction -> 不移除任何可以，只是返回一个写错误
   maxmemory-policy volatile-lru

   # 同步机制，将所有的写请求都记录下来
   appendonly yes/no
   appendfilename xxx.aop
   appendfsync everysec
   ```

3. linux 777 权限

   > 读、写、运行三项权限可以用数字表示，就是r=4,w=2,x=1，777就是rwxrwxrwx，意思是该登录用户（可以用命令id查看）、他所在的组和其他人都有最高权限

4.  如果用户加入购物车的商品没货了不能下单怎么办

   用户在进入购物车的时候，获取购物车中的信息包含：此商品的库存数量 或 商品的库存状态（有、没有）即可看到是否能够下单

5. NIO 和 BIO

   > BIO：同步阻塞式 IO，服务器实现模式为一个连接一个线程，即**客户端有连接请求时服务器端就需要启动一个线程进行处理**，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。 
   > NIO：同步非阻塞式 IO，服务器实现模式为一个请求一个线程，即**客户端发送的连接请求都会注册到多路复用器上**，多路复用器轮询到连接有 I/O 请求时才启动一个线程进行处理。 

#### 面试总结

1. 总的来说面试还行。很多细节性的东西没有回答上，导致认为技术不行、砍价
2. **心里要有个B数，是否回答的好。别人的砍价是否能够接受？**
3. **如果出现砍价，那么当时一定要做出决定。不要说回去再想想（这相当于拒绝）**

## 2018.3.8 省心宝 [ 汽车交易 ]

1. Interceptor 和 filter

   拦截器是用以拦截指定 URL 并执行指定的拦截器方法

   preHandler() 、postHandler() 、 afterHandler() 

   Filter 是过滤器，interceptor 是拦截器

   前者基于回调函数实现，必须依靠容器支持。因为需要容器装配好整条 FilterChain 并逐个调用

   后者基于代理实现，属于 AOP 的范畴。

2. Listener、Filter、Servlet 执行顺序

   L - F - S 

3. Response 的类型有哪些

   text/html text/xml text/plain

   image/jpeg image/png

   application/json application/xml application/pdf

4. URL 重写 [参考链接](http://tuckey.org/urlrewrite/)

   定义：URL 直接暴露参数从交互上和搜索方面不好

   如：xxx.com/fruit?id=1 (apple)，**不能明显的表达出请求的资源是什么**

   改：xxx.com/fruit/apple，就能明显的知道请求的资源是什么，并且能被**搜索引擎抓取**

   实现：通过引入 rewrite 工具包，配置 filter ，修改配置文件

   ```xml
   <rewrite>
     <rule>
     	<from></from>
       <to></to>
     </rule>
   </rewrite>
   ```

   **具体的映射规则，可能需要一个全文检索来实现**

   **在前后端分离的基础上，重写已经没有意义了**

5. JDK 动态代理 和 CGLib 有啥区别

   CGLib 生成的代理类是**委托类**的**子类**，不能被 final 修饰

   动态代理用的是**反射方法**，CGLib 用**类似索引的方法**直接调用委托类的方法

6. 反射获取类的私有方法

   Field field = Class.getDeclaredField( NAME );

   field.setAccessible(ture)  允许获取私有字段

7. 注解设置 AOP 的执行顺序

   AOP：@Order(num) 

   **num 越小最先执行**

8. UNION 和 UNION ALL

   **必须拥有相同数量的列，并且数据类型要相似**

   union 会选取不同的值

   union all 会列出所有值

9. MyBatis 一对一和一堆多

   <collection property="fieldName" ofType="xx.xx.ClassName"> OneToMany

   <association property="fieldName" javaType="xx.xx.ClassName" > ManyToOne

10. DUBBO 协议的最大字节数

  Dubbo 采用**单一长连接**的方式，保证服务提供者不会被单一消费者压死

  长连接能有效减少**连接握手验证**时间

  适合于小数据量大并发的服务调用

  **一般建议不超过 100K 数据**

#### 面试总结：

1. 从一开始上来就被第一个问题问住了，影响了后面的发挥
2. **要注重技术的细节性、实际使用，往往是在这些地方作为考点**

# 2018.3.12 探时科技 [ 集团平台 ]

1. JVM 调优实战

   调优的关键不在于参数的设置，而在于**如何定位到问题**

   -Xms -Xmx -Xss -XX:NewSize -XX:NewRatio -XX:SurvivorRatio

   -XX:UseSerialGC -XX:UseParallelOldGC -XX:UseConcMarkSweepGC

   -XX:PrintGCDetails

2. ClassLoader 的实际使用 [参考链接](http://ifeve.com/classloader/)

   1. Bootstrap ClassLoader 是最基础的，没有 parent 

   2. Bootstrap（rt.jar） -> Extension（lib/ext） -> System（catalina.home/bin/bootstrap.jar） -> Common（catalina） -> Webapp（WEB-INF/lib）

   3. ClassLoader 的委派机制，是先到 parent 中查找类或资源（**依次递归**），找不到再本地进行查找。

      **避免相同的类被多次加载**

   4. 反转委派原则（JavaEE），即现在本地查找，找不到再去 parent 中查找（**依次递归**）

      **原因是应用服务器中所携带的类库并不是应用所期待的**，

   5. **JVM 中确定一个类型的坐标是通过类加载器和全类名做到的**

   实际应用：动态加载类，加密（对类文件进行加密算法之后在加载的时候进行解密）

3. WebSocket 和 Socket 的区别

   WebSocket 是一种 HTML5 的协议，实现了**双网工通信**

   Socket 是网络通信中的一端，对于服务器来说相当于 TCP/UDP 的一层外壳

4. 长连接和短连接

   长连接：客户端和服务器建立连接后，客户端不主动断开连接的话，连接一直存在

   短连接：客户端每次与服务器通信都建立连接，请求完了直接关闭

   >  区别：整个客户和服务端的通讯过程是利用一个 Socket 还是多个 Socket 进行的

5. synchronized 和 Lock 的区别

   sync **只能由一个线程持有锁**，其他线程进入阻塞状态

   Lock 更加灵活、公平，可以由多线程进行读的访问（ReadWriteLock），公平锁

   **Lock 可以中断等待状态**

6. SQL 调优 [参考链接](https://www.cnblogs.com/moonlightL/p/7634294.html)

   1. 建立索引，不要在索引上计算
   2. Where 将条件小的放在前面，使用 where 代替 having 
   3. 使用 IN 关键字注意结果集有可能出现 null 的情况，IN 不会使用索引（使用 exists / not exists）
   4. "a%" 会使用索引，其他 "a%c" "%a" 会导致搜索范围过大
   5. 使用 Left join 连接，将小表放在 From 后面
   6. 使用 UNION ALL 代替 UNION（确定记录不重复时），UNION 会将记录进行比较，ALL 不会

   在查询中最主要的优化在于：**将查询限制在小的范围中，并且避免全表扫描**

7. 觉得自己在哪个技术方面特长

   认为自己现在还处在**发展期**，还有很多新的技术要学习。并没有深入的去专研某一方面

#### 面试总结：

1. 往往在问到实际经验的时候就很慌，就说在项目中没用。这样是不行的，**实际经验就跟着你熟悉的参数来，这些参数在什么情况下会使用，说出来就行了**
2. **想好再说！！！**，**想好再说！！！**，**想好再说！！！**
3. 出现对某个东西不是特别清楚的话，**首先说清楚自己懂的那部分**，不清的可以不用回答
4. 为什么总是有一种你对这个东西不熟悉、你没做过的感觉？？？
   1. **自身信心不足，老是回答这个在项目中没有使用。自己写了代码了就算使用了**
   2. **技术应用场景不熟悉**
   3. **信心比黄金更珍贵！！！**