参考来源：https://javabetter.cn/sidebar/sanfene/nixi.html

[TOC]



## 集合

### HashMap

参考地址：https://tech.meituan.com/2016/06/24/java-hashmap.html

<img src="https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/collection-8.png" width="600px" />

桶数组：存放节点对象（节点有下个节点的属性，所以可以组成链表、树）

链表：存放 hash 冲突的节点

红黑树：解决冲突节点的搜索效率问题

#### 主要的方法

##### tableSizeFor

1. 使用默认值，大小：16 负载因子：0.75f

2. 使用自定义值，

   大小：以自定义值最近的 2 的 n 次方作为大小（如 5 -> 实际 2^3 = 8），原因：方便取模和扩容

   负载因子：不建议修改（内存大时间要求高：降低，内存小时间要求低：升高）

##### hash

计算出 key 的 hashCode ，然后高低 16 位进行异或运算（当 length 较小时减少碰撞）

index = hash & (length -1) 确定所在数组的下标

因 length = 2 * n，故 hash & (length -1) = hash % length，性能更高

##### put

1. 判定数组为空 || length == 0，是：resize()
2. 判定数组的 index 节点为空，是：buckets[index] = new Node(k, v, buckets)
3. 判定已有节点的 key 是否相同，是：覆盖
4. 判定节点是否为树节点，是：putTreeVal
5. 检查链表是否超过 8 ，是：将链表转换为红黑树，插入数据
6. 不超过8节点，key 相同：覆盖，不相同：尾插到链表
7. 数量自增，判定是否超过负载，是：进行扩容

##### resize

1. 检查是否 >= 最大容量（2^30）则不在扩容，并设置 threshold = Integer.MAX_VALUE（实际永远达不到，之后都不会再扩容了）

2. 创建一个新的数组，容量为 length * 2

3. 循环重新计算节点在新的数组中的位置（因扩大2倍，则新位置一定为：原位置 or 原位置 + oldLength）

   1. node.next == null 表示只有一个节点，则 buckets[node.hash & (newCap - 1)]  = node，放到新的数组中
   2. node instanceof TreeNode，则将 node.split(this, newTab, j, oldCap) 将树的元素进行分割为两个树，在原位置 || 原位置+oldLength 存放
   3. node 是链表，将分化为两个链表：node.hash & oldCap == 0 : 在原位置 ? 原位置 + oldLength 

   以 node.hash & oldCap == 0 条件划分的原因：

   1. index =  hash & (length - 1)  如：length =  00100, length - 1 = 00011，那么实际上只取了  hash 后两位的值
   2. 现在进行了扩容，则 newLength = 01000，newLength-1 = 00111，则多了第三位加入
   3. 新加入的第三位又是刚刚的 length 的位数，所以 ==0 则不动，!=0 则到 原位置+ oldLength 去

##### get

1. 计算出 key 的 hash 和应该存放的下标
2. 有节点则判断是否 hash 和 key 相同
3. 没有则判定节点.next 是否有数据
4. 有数据则判定节点的类型进行树的查找 or 链表遍历

### ConcurrentHashMap

JDK7：将存放数据的数组分为多个 Segment ，并继承 ReentrantLock 实现可重入锁

JDK8：

1. 计算 hash 时会 & HASH_BITS，因为要保证 hash 为正数（负数有特殊标记作用，如红黑树 root 节点为 -2）
2. 使用 CAS 失败后 synchroized 实现每个桶的独立加锁
3. 在 Node 的 value 和 next 都使用 volatile 关键字，保证可见性

### CopyOnWriteArrayList

在每次写入的时候使用ReentrantLock 获取锁后，创建一个新的数组，将旧数组复制后，添加新节点进去。适合读多写少的场景

### BlockingQueue 

线程安全的阻塞式队列，插入时用 ReentrantLock 加锁再判定是否已满，满则阻塞



## 多线程

### 线程状态

1. start \ run \ sleep（计时等待） \ wait （等待）\ yield（让行） \ join（等待其他执行）
2. sleep 和 wait 的区别
   1. wait 属于 object 类中的方法，需要获得对象的锁
   2. wait 需要在同步代码中
   3. wait 会释放锁，sleep 不会
   4. sleep 延迟状态自动结束，wait 需要被唤醒

### 线程池

1. Fixed 、Single、Scheduled、Cache
2. 任务的顺序：coreSize > queueSize > maxSize 
3. 队列类型：ArrayBlockingQueue、LinkedBlockingQueue、PriorityBlockingQueue 优先级、DelayQueue 延迟
4. 拒绝策略：直接拒绝 AbortPolicy、调用者运行 CallerRunsPolicy、丢弃最老 DiscardOldestPolicy、直接丢弃 DiscardPolicy
5. Fixed、Single 默认队列大小为 Integer.MAX ，可能 OOM
6. Scheduled、Cache 默认线程池大小为 Integer.MAX，可能 OOM
7. 线程池推荐大小：CPU密集 n+1 、IO密集 2n

### 线程安全

参考地址：https://tech.meituan.com/2018/11/15/java-lock.html

线程安全主要就是保护内存数据（资源）的一致性

​	原子性 - sync

​	可见性 - volatile \ final

​	有序性 - synchroized、锁

#### volatile

volatile 使用 lock 指令实现内存可见性 + 有序性，建立内存屏障并禁止指令重排

1. 强制写入到主内存
2. 写入会导致其他 core 缓存失效
3. 禁止之后的指令重排到之后 lock 之前

注意：volatile 只能保证**基础类型**的数据最新，不能保证对象内的数据为最新（只能保证引用地址最新）

#### ThreadLocal

ThreadLocal\<String> currentName = new ThreadLocal<>();
currentName.set(new String("jiayong.huo"));

1. 线程会创建一个 ThreadLocalMap 来存放，key = currentName、 value = new String("jiayong.huo")
2. key 为弱引用，value 为强引用，key可能会被回收导致内存泄漏
3. **在 get() set() remove() 时会清理 key 被回收的数据**
4. ThreadLocalMap 是简单的线性探索哈希表，出现 hash 冲突用开放定址法（被占用就下一个）
5. 扩容：1.先清理一次 key 被回收的数据 2.容量达到阈值的 3/4 就扩容（阈值=数组.length * 2/3）

#### 锁的分类

要保证多线程访问安全，常见的就是使用锁技术，按照各种锁的特性分类：

<img src="img/lock-type.png" style="zoom: 60%;" />

##### 要不要加锁？

​	悲观锁：直接加锁，如 synchroized 和 Lock

​	乐观锁：不加锁使用 CAS 来实现变量同步

​		**读多写少**的情况下，推荐使用乐观锁（尽量减少加锁的行为）：Version \ Compara And Swap

​		如 AtomicInteger 使用 unsafe 直接读取和操作内存中的数据，用 volatile 保证线程间可见。		

​		1. 先读取指定对象在偏移量对应的值 调用 JNI 方法：判断此位置上的值是否和

​		2. 查到的值相同，相同则更新为新值，不同则返回 false

##### 要不要阻塞？

阻塞 / 自旋锁 / 自适应自旋锁

​	阻塞：阻塞或唤醒线程需要操作系统切换 CPU 状态，耗费CPU时间。当代码简单的时阻塞是不划算的

​	自旋锁：占用 CPU 时间进行循环重试，但应该注意次数。

​	自适应自旋锁：由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如 TicketLock、CLHLock

##### 有没有优先级 ？

​	见下面的 synchroized

##### 要不要尝试插队？

​	公平锁：直接进入队列排队

​	非公平锁：先尝试获取锁，未得到再排队

​	如：在 ReentrantLock 中的 Sync 内部类实现了 AbstractQueuedSynchronizer （AQS）来排队，又有公平锁 FairSync 和非公平锁 NonfairSync 两个子类

​	他们获取锁时的区别只在于：公平锁多一个判断 hasQueuedPredecessors() ，判定是否为等待队列中的第一个才能去获取锁

##### 能不能获取同一把锁？

​	可重入锁：可一定程度避免死锁，使用 status 记录重入的次数，在 status == 0 时释放锁

​	非可重入锁：可用在严格要求被执行的情况（较少）

##### 多个线程同一把锁？

​	共享锁：可多个线程持有，有线程加了共享锁，可以再加共享锁，不能加排他锁

​	排他锁：只有一个线程持有，可以读取和修改数据

​	共享 / 排他锁的主体都是 Sync 实现的，用 state 来表示锁的不同状态，高16位为读锁个数，低16位写锁状态

​	获取锁时判断：1.当读锁存在，则不能获取写锁；只有所有读锁释放了才能获取写锁。2.有写锁则其他线程都进入阻塞

#### synchronized

对象头中 Mark Word 以当前对象的锁策略，记录的数据会发生变化：

​	无锁：HashCode（32bit）、分代年龄（4bit）、锁类型（2bit）

​	偏向锁：线程ID（23bit）、Epoch、分代年龄（4bit）、是否偏向锁（1bit）、锁类型（2bit）

​	轻量级锁：指向栈帧中的锁记录（lock record）指针、锁类型（2bit）

​	重量级锁：指向 monitor 的指针、锁类型（2bit）

**锁升级：**

​	偏向锁：被一个线程多次获得，在对象中记录偏向的线程ID，此自动获得锁，没有线程竞争。

​		1. 不会主动释放锁，等待有竞争时才会释放锁

​		2. 撤销要到全局安全点后，暂停拥有偏向锁的线程，判定对象是否在被锁定的状态。

​	轻量锁：偏向锁被另一线程访问时升级，其他线程通过自旋尝试，不会阻塞

​		1. 进入到同步代码块，虚拟机在**当前线程的栈帧中建立锁记录**（Lock Record）的空间

​		2. 用以存储**需要锁对象**目前的 Mark Word 拷贝

​		3. **拷贝成功后，虚拟机使用 CAS 将对象中的 Mark Word 更新为指向 Lock Record 的指针，并将其 owner 指向对象的 Mark Wrod**

​		4. 更新成功则线程拥有了此对象的锁

​	重量锁：当自旋超过一定次数 || 第三个线程来访问，创建 monitor 对象后将对象的 mark word 设置为 monitor 对象的指针

​		monitor 是线程私有的数据结构，每个线程都有一个可用的 monitor record 列表

​		**每锁住一个对象都会和一个 monitor 关联**，在Owner字段存放线程唯一标识

​		在 Hotspot 虚拟机中，Monitor 由 ObjectMonitor 实现

**使用方法：**

synchronized 修饰代码块（monitorenter monitorexit）和方法 （ACC_SYNCHROIZED）

**修饰的位置不同，存储锁信息的位置有所变化**

​	sychronized(User.class) = User 类的 class 对象

​	static sychronized addUser() { } = User 类的 class 对象

​	sychronized addUser() { } = this 对象

​	sychronized(this) = this 对象

​	sychronized(obj) = obj 对象



#### Lock

1. ReentrantLock 官方提供的互斥锁，锁实现采用自旋，不断循环调用 CAS 操作来避免线程进入内核阻塞状态
2. 公平锁的实现方式：所有的线程请求锁时，都将放入一个队列中。按照 FIFO 的方式获取锁
3. 公平锁因为要放入队列然后在获取，当只有一个线程时，也会执行此操作。涉及到切换，所以效率低

#### AQS 思想

抽象式的队列同步器，是一个用来构建锁和同步器的框架。

支持两种同步方式：1.独占，只能一个线程获取锁  2.共享，多个线程同时获取锁（如：CountDonwLatch）

基本概念：

1. 使用 volatile 修饰 state 变量，保证多线程之间的数据可见性
2. 同步队列由内部的 Node 类实现，包含等待状态、前后节点、线程引用等，是 FIFO 双向链表
3. 使用的 CLH 阻塞队列来维护等待线程，基于链表的自旋锁。**释放锁后只会通知第一个线程去竞争锁**

常见实现：

​	ReentrantLock：可实现公平、非公平锁，由内部类 Sync 实现 AQS 

​	CountDownLatch：主线程阻塞，等到指定数量的子线程完成执行后执行。如所有人准备才开始游戏

​	CyclicBarrier：阻塞指定数量的线程，到达数量后执行。如子任务都完成才开始下一步

​	Semaphore：限制访问的线程数。如限流之类

#### synchronized & ReentrantLock 

1. sync 是 JAVA 关键字，R 只是提供的一个 API
2. 原理：锁状态的升级，R 使用CAS自旋机制实现操作原子性 和 volatile 实现可见性
3. 使用：sync 自动释放，R 需要手动释放锁
4. sync 不可中断，R 可以使用超时方法、调用 interrupt 方法
5. sync 不可设置公平锁，R 可以设置
6. **sync 是单路通知，lock.newCondition() 可以实现多路通知**

#### 锁优化

1. 减少锁应用的范围
2. 减小锁力度（拆分多个锁）
3. 锁粗化（避免多次操作）例：在循环外加锁
4. 根据场景选择不同的锁
5. 使用 CAS 乐观锁 + volatile



## JVM

### 内存区域

<img src="https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-8.png" width=40%> 

#### 方法区 - 线程共享

1.6：称为永久代，在JVM内存中，包含：类常量池、运行时常量池、字符串常量池

1.7：称为永久代，在JVM内存中，包含：类常量池、运行时常量池。将字符串常量池、静态变量 -> 堆

JDK8：称为元空间（Metaspace），在直接内存中，包含：类常量池、运行时常量池

#### 堆 - 所有线程共享

几乎所有 new 对象的空间都分配在这个区域，为啥不是所有？ 

​	从 1.7 开始开启了逃逸分析：1.方法逃逸，对象被方法返回  2. 线程逃逸，对象被其他线程引用

​	若未发生逃逸的时，则可以将对象创建在栈帧中（方便回收）

​	若为发生线程逃逸，则可以减少加锁的操作，减少同步开销

**new 对象的生命周期不由某个方法决定，**需要JVM进行内存回收和管理，基本都是按照分代理论设计：

<img src="https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-5.png" width=50%>

##### 创建对象分配空间

1. 先进行类加载检查，没有则进行类加载
2. 分配内存空间，有两种模式
   1. 指针碰撞：将空间分为已使用和未使用，指针记录下个可用地址；适合管理简单、碎片少，如年轻代
   2. 空闲列表：用一个列表记录未占用的位置；适合对象较少变动的，如老年代
3. 初始化对象，后设置对象头的信息 hash 分代等
4. 调用构造方法，完成赋值

##### 分配空间抢占

当多线程同时创建对象时可能出现抢占同一块空间的情况，JVM 为每个线程保留一块内存空间（TLAB）线程本地分配缓冲区

创建对象先在当前线程的 TLAB 中创建，只有用完或不够大时才直接在堆上分配空间

可以使用 -XX:+PrintTLAB 来打印使用的情况：desired_size 是TLAB的大小

#### 虚拟机栈 - 线程私有

当线程每调用一个方法则会创建一个栈帧，并入栈，后进先出。**方法调用结束后自动释放**

栈帧：局部变量、操作数栈、动态链接、方法出口等

空方法会不会有局部变量数据 ？

​	非静态方法：有 this 引用变量

​	静态方法：没有

#### 本地方法栈 - 线程私有

同上，但此栈中为调用的 native 修饰的方法，此类方法会调用系统加载的动态库，并执行关联的方法

常用于调用操作系统提供的方法或硬件交互

#### 线程计数器 - 线程私有

用以记录当前线程执行到字节码的行数

### 对象的内存布局

<img src="https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/jvm-12.png" width=50% />

mark word：64位系统 8 字节，32位系统 4 字节

类型指针：压缩指针 UseCompressedOops ? 4 byte : 8 byte 

对齐填充：JVM 要求对象的总大小必须是 8 byte 的整数倍（一次寻址的指针大小是 8 byte = cpu L1 缓存行的大小），不够则使用填充对齐

如 new Object() 的大小是多少：64位 = 对象头（8 + 4）+ 填充（4）= 16byte

Object o 这个句柄的大小是多少：UseCompressedOops ? 4 byte : 8 byte 

### 垃圾收集

#### Stop The World

进行垃圾回收时，要找到回收的对象，所以需要暂停所有用户线程（避免关系变动），被称为 STW

当收到垃圾回收的信号时，用户线程会跑到安全点（safe point）后暂停，等到 GC 线程进行垃圾搜集

#### 垃圾标记

目标要找到已经不使用的对象内存空间，常用两种算法

引用计数法：在对象上记录使用的次数。问题：相互引用则无法回收

可达性分析法：找到 GC Roots 后递归整个引用链。

GC Roots 的类型有：

​	虚拟机栈内引用：方法参数、局部变量等

​	本地方法栈的引用：this 对象

​	类静态常量：static 、fainal

​	运行时常量：String 或 Class 类型

#### 清理算法

标记 - 清除：将标记的清理对象直接清除掉，问题：导致内存碎片较多

标记 - 复制：将不清理的对象移动到一块新的内存空间，问题：额外空一块内存空间

标记 - 整理：将不清理的对象移动到一起，问题：移动效率问题

#### 分代清理

新生代和老年代对应对象的存活的周期不一样，所以不同代采用不同的算法，提升清理效率

Minor GC：也称为 Young GC，发生在新生代

Major GC：也称 Old GC，发生在老年代，**是 CMS 收集器的特有行为**

Full GC：清理整个 JAVA 堆和方法区，最耗时，在JVM压力过大时发生。

​	触发：1. 老年代的可用空间 < 每次升入老年代的平均值  2. System.gc()

Mixed GC：同时清理新生代和部分老年代，**是 G1 收集器的特有行为**

#### 垃圾收集器

Serial + Serial Old ：最基础、历史最长的收集器，单线程去完成垃圾收集工作

ParNew：是 Serial 的多线程并发版本

Parallel Scavenge：多线程并行收集，关注吞吐量

Parallel old：上面的老年代版本

**CMS：JDK9 标记弃用**，低延迟：1.初始标记 2.并发标记 3.重新标记 4.并发清理

G1：划分成多个小的 Region，面向内存大、高吞吐的场景，但是调优比较复杂

ZGC：**JDK 11 引入**，低延迟的收集器，面对 TB 级别的内存也保持低停顿时间 <10ms

​	为什么这么短：

​	CMS、G1 类似，采用标记-复制算法；会出现 STW 的阶段有：初始标记、再标记、清理、转移

​	转移阶段耗时较长，**G1 未能解决转移过程中的准确定位对象地址的问题**

​	ZGC通过**着色指针和读屏障技术**，解决了转移过程中准确访问对象的问题，实现了并发转移

### 问题排查 & 调优

1. 通过 -XX:+PrintGCDetails 打印 GC 详情的日志 || jstat -gc pid xxx，查看次数和时间是否正常
2. 查看各代的 GC 频率是否正常，可考虑调整比例
3. 使用 heap dump 查看对象的使用占比情况
4. 根据业务选择合适收集器

### 类加载

#### 双亲委派模型

Bootstrap Class Loader -> Extension Class Loader -> Application Class Loader  -> Customer Class Loader（user）

类在被加载时，先委托父级加载器加载，父加载器加载不了继续往上；都加载不了采用子加载器

优点：1.不会被重复加载 2.父加载器权限更大，如 java.lang.* 只能由 Bootstrap 加载

如何破坏机制：重写 ClassLoader 的 loadClass() 方法

常见的破坏双亲机制的场景：热部署框架、SPI 加载 JDBC 驱动（SPI：扩展机制，用以加载和注册三方库）

#### Tomcat 的类加载器

tomcat 可以进行多 web 应用的同时运行，所以多应用的类加载器应该隔离，所以自定义加载器

Catalina ClassLoader：加载 Tomcat 的核心类库

Shared ClassLoader：加载共享库，允许多个 web 应用共享某些库

WebApp ClassLoader：加载 Web 应用程序的类库，支持多应用隔离和**优先加载自定义的类库**（破坏双亲模型）



## Spring

### 主要功能

1. IoC（Inverse of Controller）将对象的管理权限交给框架管理，IoC 容器实际上是 Map
2. DI（Dependancy Injection）使用容器将对象引入
3. AOP（Aspect-Oriented Programming）面向切面编程，降低系统模块耦合度

### Bean

#### 生命周期

singleton：和 IoC 容器一个周期

prototype：使用时创建，使用过程中一直存活，不用被 GC 回收

大致四个阶段：

1. 实例化（instantiation）
2. 属性赋值（populate）
3. 初始化（initialization）
4. 销毁（destruction）

#### 加载流程

1. 实例化（new Object）
2. 设置属性（设置各个属性值，注入依赖的 bean 等）
3. Aware（BeanNameAware、BeanFactoryAware、ApplicationContextAware）
4. BeanPostProcess 的 postProcessBeforeInitialization() 
5. 调用指定 init-method 方法
6. postProcessAfterInitialization()
7. 创建完成，可以使用

#### 清理 Bean

1. 实现的 DisposableBean 接口
2. 调用指定 destory-method 方法

#### 循环依赖

Spring 采用三级缓存的方式来解决循环依赖的问题

一级缓存（singletonObjects）：创建好的所有 singleton Bean

二级缓存（earlySingletonObjects）：在调用 getSingleton() 从三级缓存中移出的对象

三级缓存（singletonFactories）：刚实例化的 Bean **包装到 ObjectFactory 中**

![循环依赖的返回](img/spring三级缓存.png)

#### 为什么是三级缓存

一级：已设置完整属性的对象

三级：所有刚实例化的对象

为什么一三级要分开：区分不同属性状态的对象

二级：区分代理对象和 ObjectFactory 对象

1. 调用三级缓存的 ObjectFactory.getObject()  返回的为一个 BeanProxy 对象
2. 如果当前对象被 AOP 进行切面代理，每次都将返回一个新的对象 （不符合单例）
3. 所以将产生的 BeanProxy 对象放入到二级缓存中，下次直接获取即可



### IoC 容器的启动流程

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses){
    // 1.加载 RootBeanDefinition 2.解析配置类 3.解析@Import和@Bean
    this();
    
    // 注册配置类
    register(componentClasses);
    
    // 下面的加载 bean 流程
    refresh();
}
```

### IoC 容器加载 bean 流程

1. prepareRefresh() **刷新预处理**
2. obitionBeanFactory() 销毁 old BeanFactory **创建新 BeanFacotry** 并注册到 BeanDefitionRegistry
3. prepareBeanFactory() **预处理**，加载 Context 的 ClassLoader
4. /
5. postProcessBeanFactory() **前置处理**
6. invokeBeanFactoryPostProcessors()  **实例化 BeanFactroyPostProcess** 的 bean
7. registerBeanPostProcessors() **注册 BeanPostProcess 的后置处理**
8. /
9. initMessageResource() **初始化国际功能**
10. initApplicationEventMulticaster() **注册事件派发器**
11. onRefresh() **容器刷新**
12. /
13. registerListeners() **注册监听**
14. finishBeanFactoryInitialization() **初始化所有非懒加载的单例 Bean**
15. finishRefresh() **Context 刷新**



### AOP

AOP 是一种编程范式，用以提高程序模块化的程度。将非核心业务或通用业务（日志记录，事务）分离出来

@Transaction 是典型的 AOP 应用，通过 AOP 来实现事务的管理

核心概念：Aspect 切面、Join point 连接点、Advice 通知、Pointcut 切点、Weaving 织入 等

常用注解：@Aspect \ @Pointcut \ @Before @After @AfterReturning @AfterThrowing @Around 

#### 两种实现方式

JDK 动态代理：由 java 提供的 Proxy 实现，只能代理接口不能代理类本身。如：Proxy.newProxyInstance()

CGLIB 代理：依赖 CGLIB 库，可以代理没有实现接口的类，创建代理对象的开销较大。如：new Enhancer().setSuperClass().setCallback()

#### Spring AOP 和 AspectJ

Spring AOP：基于动态代理，依赖 IoC 容器来管理， Spring自带（支持 AspectJ 的注解风格）

AspectJ：基于编译时增强，可以单独使用，自行引入



### 事务

#### 事务的传播机制

传播机制定义了方法**在被另一个事务方法调用时**的事务行为，这些行为定义了事务的边界和事务上下文如何在方法调用链中传播。

传播机制是使用 ThreadLocal 实现的，所以使用其他线程调用的方法不会被传播

1. REQUIRED	有事务则加入，没有则创建
2. SUPPORTS	有事务则加入，没有则非事务运行
3. MANDATORY	有事务则加入，没有抛异常
4. REQUIRES_NEW	有事务则挂起，用重新创建
5. NOT_SUPPORTED	有事务则挂起，用非事务运行
6. NEVER	有事务则抛异常，用非事务运行
7. NESTED	局部回滚

#### 事务失效的场景

1. **同类中调用：**如 A 调用 @Transcantion B()，但 A 没有事务，则 B 的事务不会生效
2. 使用非代理的方式运行：new A().create()
3. 方法非 public：在 TranscationInterceptor 的 intercept 方法中会调用 **computeTransactionAttribute** 检查 isPublic
4. rollback 设置错误：只有 RuntimeException 和 Error 的子类才能触发回滚
5. 数据库不支持

### Spring Boot

提供一套默认配置，通过约定大于配置的理念帮助我们快速搭建项目

原理：

1. 主要实现自动加载默认配置的注解为 @EnableAutoConfiguration，其通过 AutoConfigurationImportSelector 加载需要自动装配的类，

2. 配合 @Import() 将相应的类导入到容器
3. 获取注入类的方法是：selectImports() ，它实际调用的是：**getAutoConfigEntry()** 方法，是获取自动装配的核心

#### 启动原理

- 第一步，创建 SpringApplication 实例，负责应用的启动和初始化；
- 第二步，从 application.yml 中加载配置文件和环境变量；
- 第三步，创建上下文环境 ApplicationContext，并加载 Bean，完成依赖注入；
- 第四步，启动内嵌的 Web 容器。
- 第五步，发布启动完成事件 ApplicationReadyEvent，并调用 ApplicationRunner 的 run 方法完成启动后的逻辑。



## MySQL

文章资料：

​	崩溃恢复过程：https://mp.weixin.qq.com/s/9Fe99BY7u0mbBs_Jtuea-Q

基础架构大致分为三层：

​	连接层：处理客户端连接，包括验证身份、权限校验、连接管理等

​	服务层：MySQL 的核心，负责查询解析、优化、执行等操作

​	存储引擎层：负责数据的实际存储和提取，支持多种引擎 InnoDB、MyISAM、Memory

### 数据存储

#### 存储形式

以表的形式存储数据，分为

​	表空间 Tablespece：常见由数据段（叶子节点数据）、索引段（非叶子节点的数据）、回滚段（事务执行过程中用于回滚的旧数据）组成

​	段 Segment：多个区组成

​	区 Extent：通常由 64 个连续页组成，大小为 1MB

​	页 Page：是 InnoDB存储的基本单元，大小为 16KB，索引树的一个节点就是一个页

​	行 Row：数据按照行进行组织和管理，格式有：COMPACT \ DYNAMIC \ REDUNDANT 等

#### 存储引擎

InnoDB：5.5+ 的默认存储引擎，支持事务、行级锁、外键、B+树索引等，

​	**Buffer Pool：以页为单位在内存中的缓冲区**

​		查询：优先从此中获取数据，避免直接访问磁盘

​		修改：也先在缓存页面中修改；当数据页被修改后，会在 Buffer Pool 中变为脏页

​					**脏页不会立刻写回磁盘**，通常采用改良的 LRU 算法来将脏页**定期刷新**到磁盘

​		双写机制：将其先写入 2mb 连续空间的**双写缓冲区**（double wirte buffer），然后再将其写入磁盘

​		调优：可设置合理的缓存大小（通常物理内存的 70%），并配置多个 buffer pool 提升并发能力

MyISAM：5.5- 的默认引擎，适合读比较多的场景

#### 日志文件

错误日志（Error Log） ：记录 MySQL 启动、停止、运行中出现的错误信息

一般查询日志（General Query Log）：所有连接信息和查询语句

慢日志记录（Slow Query Log）：记录超过指定时间的 sql 语句（默认：关闭）

**二级制日志（Binary Log）**：**服务层，用以数据恢复和主从同步等**，

​	记录所有修改（INSERT \ UPDATE \ DELETE）的 sql 语句和执行时间（默认：关闭）	

在 mysql.cnf 配置如下：

​	log_bin=mysql-bin：开启 binlog，由 mysql-bin.00000X （数据文件）和 mysql-bin.index （索引文件）组成

​	max_binlog_size=104857600：单个日志文件大小，100mb

​	expire_logs_days=7：最长记录天数

​	binlog-do-db=db_name：记录指定库

​	binlog-ignore-db=db_name：排除指定库

​	sync_binlog=0 ：表示数据先写入到操作系统的缓存，当缓存区满后由操作系统将数据写入磁盘（fsync）**性能高但可能丢失数据**

**重做日志（Redo Log）**：**引擎层，用以崩溃恢复、保证事务持久性**，记录物理级别记录表的写操作

​	Write-Ahead Logging（WAL）：先写日志，后写数据；确保发生故障时可以通过日志恢复数据

​	由两部分组成： ib_logfile0 、ib_logfile1 总大小固定（每个默认为 48MB），以循环的方式写入，写满则从头覆盖

​	先写到 redo log buffer 连续的内存空间的缓冲区中，什么时候写入磁盘？

 1. buffer 空间不足

 2. **事务提交时**

 3. 后台线程控制

 4. 正常关闭服务

 5. 触发 checkpoint 规则后，将 buffer pool 中的脏页写入进磁盘（数据文件），并更新 redo log 中的 checkpoint 地址

    如上 redo log 以块（redo log block，512byte）循环进行存储写入的，则需要两个位置标记：可覆盖块位置和写入块位置

    ​	checkpoint 位置：已经落盘的地址（之前的位置可覆盖，已清空）

    ​	wirte pos 位置：已经写入数据的地址

    当写入的地址追上可写的地址时，表示空间不足，**需要执行刷盘写入到磁盘表空间文件**，则可以释放部分 redo log 空间

**回滚日志（Undo Log）**：**用于回滚和 MVCC**，记录被修改前的值

#### 两阶段提交

常出现在 **分布式事务** 或者 **主从复制、InnoDB 和 binlog 协同写入** 的场景中

​		------------- Prepare 阶段 -------------

1. InnoDB 引擎执行 SQL，生成 Undo log（用于回滚）和 Redo log（用于持久化恢复）；

2. **将 Redo log 记录为 “prepare” 状态**（还没真正提交）；

3. **记录 binlog 日志**，写入 binlog 缓冲区（没有 fsync 到磁盘）；

4. 这时，InnoDB 引擎和 binlog 都处于 “待提交” 状态。

   ------------- Commit 阶段 -------------

5. **将 binlog 持久化到磁盘（fsync）**  注：当 sync_binlog=0时可能并不会落盘

6. **Redo log 改为 “commit” 状态**；

7. 事务正式提交，数据对外可见。



### 索引

#### 按存储方式

1. 聚簇（集）索引：保存索引 和 数据（不用再次查询）

   主键索引 = 聚簇索引 = 覆盖索引 

   **一定会有主键索引，一定是聚簇索引：没有则使用其他唯一索引，没有则使用隐藏 row_id**

   **为什么 innodb 一定会有主键索引？ 因聚簇索引叶子节点会保存整行数据，叶子节点就是表的物理数据页，主键索引就是数据表本身！！！**

2. 非聚簇（集）索引：保存索引 和 主键值（通过主键值再次查询，即回表）

   1. 唯一索引 （UNIQUE）

   2. 联合索引：需要注意 **最左前缀匹配** 问题

      联合索引的索引键是以最左的字段来确定的，所以查询时未指定最左字段则无法使用索引

   3. 全文索引（FULLTEXT）

#### 按数据结构

1. B+ 树索引：非叶子节点存储索引，放入缓存提升效率；叶子节点存储数据，做双向链表进行范围查询

2. HASH 索引：使用数据的 hash 值作为索引项，数据和索引项一对一（不能进行范围搜索，排序等）

   注：InnoDB 引擎不能指定为 HASH 索引，如 UNQUIE HASH(user_name)；

   内部使用 Adaptive Hash Index,AHI 技术（也不是用户显式创建的），引擎发现某个索引被经常访问则自动创建

### 查询

#### 查询执行

1. client 发送 select 语句到 MySQL 后，建立连接、获取权限、管理连接

2. ~~在 query cache 中匹配是否有相同的语句（大小写），有则直接返回~~（8.0后移除）

3. 解析器：将 select 语句交给解析器进行语法解析，生成解析树

4. 优化器：查询优化器生成执行计划，进行索引优化

5. 存储引擎执行 SQL 语句，得到查询结果。执行顺序：

   form > join > on > where > group by > avg,sum... > having > select > distinct >order by > limit

6. 开启 Query Cache 则放入，后返回结果

#### 查询优化

通过 mysql slow log、ORM 框架、连接池工具 druid、 专用工具 p6spy 等记录慢查询的语句

用 explain 初步分析关键值： type（是否全表扫描）、rows（预估的行数）、 extra（详细的描述）

2. 语句优化：

   1. **查询条件使用索引**
   2. count 会全表扫描，预估可以使用 explain 方式
   3. 分页 start 值过大会缓慢，使用子查询 + 表连接解决
   4. 删除表会产生 undo 和 redo 日志，确定删除使用 trancate

3. 索引优化：

   1. 无法使用索引：><、!= 、IS NULL、OR、IN、NOT IN
   2. **使用短索引**
   3. 索引的区分度要高，唯一索引的重复率 < 0.1
   4. 定义外键的列一定要有索引
   5. 更新频繁的列不适合做索引
   6. 查询的数据较少可以通过联合索引来进行覆盖（要符合最左匹配原则）
   
3. 多表优化：

   1. 使用 JOIN 代替子查询
   2. 小表作为驱动表（如 left join 则小表放左表）
   3. 不要关联太多表

4. 设计优化：

   1. 表 < 200
   2. 列 < 40
   3. 行 < 500w
   4. 单表索引 < 5

5. 配置优化：

   **配置连接数**、禁用 Swap、增加内存、升级 SSD

#### JOIN 查询

<img src="https://www.runoob.com/wp-content/uploads/2019/01/sql-join.png" width=50%>

#### UNION

用以合并两个或多个 SELECT 语句的结果集

### 更新执行

<img src="https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-812fb038-39de-4204-ac9f-93d8b7448a18.jpg" width="60%" />

1. 执行器：调用引擎层接口，从内存或磁盘中读取指定修改的数据到内存中
2. 执行器：修改对应的内存数据，后调用引擎层的接口更新数据
3. 引擎层：更新数据到内存，并将操作**记录到 redo log 中**，为 prepare 状态；后告知执行器
4. 执行器：生成这个**操作的 binlog 并记录到磁盘**，后调用引擎的提交事务接口
5. 引擎层：将 redo log 中的记录改为 commit 状态，更新完成



### 数据库事务

#### 数据库事务的四特性 ACID

1. 原子性（Actomicity）：事务中的所有操作要么全部完成，要么全部不完成，它是不可分割的单位

   用 undo log 进行回滚保证事务中所有操作都成功或者失败

2. 一致性（Consistent）：确保事务从一个一致的状态转换到另一个一致的状态

   由其他三大特性一起保证

3. 隔离性（Isolation）：多个事务之间不应该相互干扰

   MVCC 和 锁

4. 持久性（Durable）：事务的结果应该被永久的保存，即使崩溃也能恢复

   以 redo log、双写机制、两阶段提交、checkpoint 机制共同保证

   1. redo log：当事务被提交，会将事务的修改操作写入到 redo log 中，并强制刷盘。然后再将内存中（buffer pool）的数据页写入磁盘

   2. 双写机制：解决16K数据页在操作系统一次写入4K，可能出现不完整的问题。

      在将内存的数据也刷入磁盘时，先将数据页刷到一个 2m 连续空间的双写缓冲（double write buffer）中

      当崩溃后先从双写缓冲中恢复，然后再从 redo log 中恢复

   3. 两阶段提交：redo log 和 binlog 配合进行预提交，当 binlog 落盘后修改为 commit 状态
   
   4. checkpoint 机制：负责将 buffer pool 中的脏页刷到 .idb 文件磁盘中，并更新 redo log 中的 checkpoint_lsn 地址

#### 数据错误术语

1. 脏读：读取未提交的数据，后未提交的数据被回滚
2. 不可重复读：多次读取的结果不一致
3. 幻读：同一事务中多次执行**相同范围查询**，得到不同的结果（数据被其他事务修改）

#### 事务的隔离级别

读未提交（Read Uncommitted）：加行共享锁实现，允许读取未提交的事务数据  ——脏读、不可重复读、幻读

读已提交（Read Commit）：加行排他锁实现，允许读取已提交的事务数据  ——不可重复读、幻读

可重复读（Repeatable Read）【默认】：事务生成ReadView实现，多个事务读取同一数据是一致的  ——幻读

顺序读（Serializable）：加表级锁，所有事务依次读取

解决幻读的方案：

1. 采用 Serializable 级别，所有事务按照顺序执行（效率低）
3. 采用 Gap Lock + Next-Key Lock 方案，对一个范围进行加锁
3. 采用 MVCC （Mulite-Version Concurrency Control）方案，确定事务可以读取到的版本

#### MVCC

是 InnoDB **实现事务隔离**、提升并发性能的核心机制。

它的设计初衷是：让数据库在保持一致性读的同时，尽可能地避免加锁，从而提升并发性能

实现了 **“非阻塞读”** ——保存数据的历史版本，让读操作不需要加锁就能直接读取快照，也不会阻塞写操作，提高读的并发性能。

每次修改数据前，先将记录拷贝到 Undo Log 生成新的版本，并且每条记录会包含三个隐藏列：

1. DB_TRX_ID：事务 id，是一个全局递增的数字
2. DB_ROLL_PTR：每次修改时都会把老版本写入到 undo 日志中，这个回滚指针就指向这条聚簇索引的上一版本的位置，组成版本链
3. DB_ROW_ID：无主键时生成

**Read View：**是一个快照视图，用以确定可以读取到那些版本的数据

生成情况：读已提交 -> 事务每次读取数据都生成；可重复读 -> 第一次执行快照读生成

其中记录了当前活跃事务的 ID 集合、最小事务 ID、最大事务 ID 等信息，

通过与 DB_TRX_ID 进行对比，判断当前事务是否可以看到该数据版本

#### 锁级别

##### 全局锁：

对整个数据库实例进行加锁，会处于只读状态，所有写操作都会被阻塞，直到锁被释放

加锁：FLUSH TABLES WITH READ LOCK （FTWRL）刷新所有表为读锁

解锁：UNLOCK TALBES

##### 表级锁：（Serializable ）

1. 意向锁（Intention Lock）：MyISAM 和 InnoDB 都支持
2. MDL 锁：对表结构做修改时的锁
3. AUTO-INC 锁：自增时的锁，插入后自动释放

##### 行级锁：（RC 和 RR）

底层通过给索引加锁实现，意味着**只有通过索引条件才能使用行级锁。否则对扫描到的所有行加锁，接近表锁（本质仍为行锁）**

1. 记录锁（Record Lock）：对某一行的 **索引项** 进行加锁，有锁才能修改和删除

2. 间隙锁（Gap Lock）：对条件范围内的 **各个索引区间** 进行加锁（不是锁住某些**具体值**，而是值之间的**区间**），防止插入

   必要条件（开启事务 + RR以上级别 + 加锁 ）满足时，会产生的情况：

   1. 范围查询（无论什么索引） ⇒ 一定加间隙锁
   2. 未命中记录（无论什么索引） ⇒ 会锁定插入点的间隙（防幻读）
   3. 普通索引，即使是精确匹配 ⇒ 也可能加间隙锁（因存在重复可能）

3. 临键锁（Next-Key Lock）：对 **某一行的索引项 + 索引区间** 都加锁（记录锁+间隙锁）锁定一个左开右闭的区间，可解决幻读问题

   默认的行锁类型就是临键锁，可能出现情况：

   1. 唯一索引 + 等值 + 命中 = 退化为记录锁
   2. 唯一索引 + 等值 + 未命中 = 退化为插入点前后索引的间隙锁
   3. 唯一索引 + 范围 = 所有命中记录和所有间隙
   4. 非唯一索引 = 二级索引加锁 + 回表主键索引加锁 + 等值（记录/间隙锁）OR 范围（临键锁）

#### 锁类型

##### 读写锁

1. 共享锁（Shared Lock，S）：也叫读锁，允许多个事务同时读取数据，但阻塞写操作
2. 排他锁（Exclusive Lock，X）：也叫写锁，独占数据，阻塞其他事务的读写

**注意：【在事务中】普通的 select （即未指定使用锁）是不会加行锁的，而是使用 MVCC 实现一致性，是无锁的**

在表级和行级都有读写锁。表级：MySQL 定义 、行级：InnoDB 定义

表级读写锁使用：（MyISAM 默认会自动加）

1. LOCK TABLES xx READ （可选 LOCAL：可以在表尾插入）
2. LOCK TABLES xx WRITE
3. （解锁）UNLOCK TALBES

行级读写锁使用：

​	使用 select ... LOCK IN SHARD MODE（行级读锁）

​	使用 select ... FOR UPDATE（行级写锁）

##### 意向锁

在 InnoDB 中支持多粒度锁，允许 表级锁 & 行级锁 共存。**意向锁是一个表级锁**

因为行和表都有锁，那么需要考虑如何让这两种锁共存 ？就有了

参考：https://www.51cto.com/article/743293.html

1. 意向共享锁（Intention Shared Lock，IS）：当有意向对 **某些行** 加共享锁（读），InnoDB 会自动先获取**表的意向读锁**
2. 意向排他锁（Intention Exclusive Lock，IX）：当有意向对 **某项行** 加排他锁（写），InnoDB 会自动获取**表的意向写锁**

作用：

1. 当在使用行级锁时，引擎会自动添加表级意向锁（意向锁不互斥：多行锁可以同时）
2. 当其他事务在需要获取表级锁（互斥的情况）的时候，则只需要判定是否存在意向锁即可
3. **避免了互斥的情况下，需要去判定每行是否存在排他锁的问题**

互斥：

1. 意向锁之间都不互斥
2. 只有共享锁之间兼容，其他都互斥

##### 乐观/悲观锁

MySQL 中的表锁和行锁都是悲观锁

乐观锁常见的实现方式版本号机制或时间戳机制：version 或 timestamp 实现

#### 锁状态

通过：SHOW ENGINE INNODB STATUS 命令可以查看事务和锁的信息



### 集群

#### 数据一致性问题

1. 异步复制：主库写入成功后直接返回，可能主从数据不一致
2. 半同步复制：等从库同步到 relay-log 后主库返回结果
3. 全同步复制：所有从库都执行成功后主库返回结果

#### 集群方案

1. **MySQL Replication 主从架构**

   基本原理：

   1. 从节点：开启 IO 线程，连接主节点的 dump 线程

   2. 主节点：事务提交后以事件的方式记录到 binlog， dump 线程将变更的数据发送给连接的从节点

   3. 从节点：接受到 binlog 文件并保存到 Relay log 文件中

   4. 从节点：SQL 线程监控文件实时变化，按顺序执行 Relay log 中的事件

      注：binlog 记录格式：STATEMENT | ROW | MIXED

   实现方案：

    1. 主节点：开启指定库 binlog 功能 

       log-bin=mysql-bin

       binlog-do-db=数据库名

   2. 从节点：关闭写功能、指定同步库

      read_only=1

      replicate-do-db=需同步数据库

   3. 主节点：查看当前数据状态 show master status; 创建同步用户 “replication”

   4. 从节点：配置主节点信息，执行命令：

      **CHANGE MASTER TO** 

      ​	MASTER_HOST='192.168.10.111', 

      ​	MASTER_USER='replication', 

      ​	MASTER_PASSWORD='123456', 

      ​	MASTER_LOG_FILE='mysql- bin.000006',

      ​	MASTER_LOG_POS=2303;

   5. 从节点：建立连接：START SLAVE;

2. MySQL Group Replication（MGR）：多个节点组成复制组，必须 n/2 + 1 的节点通过后才提交

3. **InnoDB Cluster**：高可用的解决方案

4. InnoDB CluseterSet：多个副本（不同区域）的 innodb cluster，提供容灾能力

5. InnoDB ReplicaSet：使用 MySQL Router 进行引导配置，非常适合扩展读取

6. Master-Master for MySQL（MMM）：支持双主的切换和管理的脚本，使用 prel 编写

7. **Master-High Avaliablity for MySQL（MHA）：**一款优秀的高可用环境下故障切换和主从提高的软件，可自动提升节点、切换主节点

8. Galera Cluster：多主集群，是一种高冗余架构

9. **MySQL Cluster：**支持 ACID 事务的实时数据库，基于分布式架构

#### MySQL Replication 主从延迟

1. 重要数据直接读取主库

2. 业务允许短暂不一致，发出提示

3. 半同步复制：至少一个从节点接收到 binlog 数据并写入 relay log 成功后，才提交主库的事务

   **当指定时间内未收到任何从库的确认，那主库会自动切换为：异步复制模式，确保不阻塞应用**

   主库插件：INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
   
   从库插件：INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

#### MHA | QMHA 升级主节点步骤

1. 尝试修改 old-master 为 read_only = on，避免集群多点写入
2. 将 binlog server 保存 old-master 的 binlog 文件同步到  new-master 上
3. 在 new-master 使用 binlog 进行数据补齐，避免丢失数据



### 分库分表

将一个表拆分成多个（降低数据量压力），再将多个表放入到多个库（降低并发压力）分散压力

#### 分库和集群

​	集群：将一个数据库复制到多个机器（多个机器被视为一个库）

​	分库：将数据分布到多个机器（多个库）

#### ShardingSphere

参考地址：https://shardingsphere.apache.org/document/5.2.0/cn/user-manual/shardingsphere-jdbc/spring-boot-starter/rules/sharding/

1. 引入 `sharding-jdbc-spring-boot-starter` 包

2. 指定配置：

   ```yaml
   spring:
   	shardingsphere:
   		# 1.配置多个库信息
   		datasource:
   			names: d1, d2
   			d1:
   				type: com.zaxxr.hikari.HikariDataSource
   				driverClassName: com.mysql.cj.jdbc.Driver
   				jdbcUrl: jdbc:mysql://localhost:3306/sharding_db
   				username:
   				password:
   		# 2. 配置分片信息
   		rules:
               sharding:
               	# 1.默认分片设置
               	default-database-strategy:
               		standard:
               			sharding-column: xxx
               			sharding-algorithm-name: xxx
               	default-tables-strategy: 同上
               	default-key-generate-strategy: 同上
               	default-sharding-column: 同上
               	# 2.指定表分片配置
                   tables:
                       order:
                           # 2.2.1 分在那些库、表内
                           actual-data-nodes: d$->{1..2}.t_order_$->{1..2}
                           # 2.2.2 主键生成策略
                           key-generator:
                               cloumn: id
                               type: SNOWFLAKE | UUID
                           # 2.2.3 分库策略
                           database-strategy:
                               standard:
                                   sharding-cloumn: id
                                   sharding-algorithm-name: id_hash_mod
                           # 2.2.4 分表策略
                           table-strategy:
                               hint:
                                   sharding-algorithm-name: region-hint-algorithms
                   # 3.定义分片算法定义 type: INLINE\HASH_MOD\CLASS_BASED
                   sharding-algorithms:
                       # 内置算法
                       id_hash_mod:
                           type: HASH_MOD
                           props:
                               sharding-count: 2
                       # 直接写分片算法表达式
                       order-inline:
                           type: INLINE
                           props:
                             algorithm-expression: t_order_${id % 2 + 1}
                       # 自定义算法类
                       region-hint-algorithms:
                           type: CLASS_BASED
                           props: 
                               strategy: HINT
                               algorithmClassName: com.chagee.sharding.RegionShardingAlgorithm
   ```

##### 分片策略

2. strandard：

   1. precise-algorithm-class-name：新增时的自定义分片策略
   2. range-algorithm-class-name：查询范围时确定应该去哪些分片执行的策略

3. complex：多个分片键确定分片

4. hint：和 SQL 不相关的条件去分片（直接自行确定应该在那些库和表）

   例：下单人是成都的，那我就直接指定他去成都库查就行了
   
4. none：不分片

##### 读写分离方案

1. **在  MySQL 中设置读写库同步**

2. 配置如下：

   ```yaml
   spring:
     shardingsphere:
       datasource:
         names: # 数据源名称列表，例如：master,slave0,slave1
           # master:
           #   url: jdbc:mysql://localhost:3306/db
           #   username: root
           #   password: root
           #   driver-class-name: com.mysql.cj.jdbc.Driver
           # slave0: ...
           # slave1: ...
       rules:
         readwrite-splitting:
           data-sources:
             <readwrite-splitting-data-source-name>:
               static-strategy:
                 write-data-source-name: <主库数据源名>
                 read-data-source-names:
                   - <从库1>
                   - <从库2>
               load-balancer-name: <负载均衡算法名称>
           load-balancers:
             <负载均衡算法名称>:
             	# 示例：ROUND_ROBIN / RANDOM / WEIGHT
               type: ROUND_ROBIN
               props:
               	# 如 weight 配置时可用：slave0: 3, slave1: 7
               	salve0: 3
               	salve1: 7
   ```



### 常见问题

#### 双写不中断迁移

1. 让新库加入到所有的读写操作中
2. 通过新库的起始 id 就可判断之前的是旧数据
3. 通过程序将旧库中的数据写入到新库中，写的时候判断 updateTime
4. 循环执行，直至新老一致，则可关闭旧库

#### 分布式自增唯一主键

1. 可用 redis 的方式 （10W级）
2. 雪花算法
3. 自定义主键生成策略

#### 富查询

多维度实时查询最常见的方式是：将查询的字段放到 ElasticSearch 中，那后续扩展加字段呢 ？

#### 千万大表如何添加字段

大表添加字段使用 ALTER TABLE 命令会长时间锁表，甚至数据库崩溃。

在 8.0 之后的版本可以直接使用 ALTER 命令，因为采用了 INSTANT 算法。

#### 深分页

1. 按游标进行查询 offset ?
2. 查询带上上一次查询排序后的最大 id（不能跳页）

#### 数据倾斜

分表 & 取余



## MyBatis

### 生命周期

#### 构建会话工厂

1. 读取配置文件，包括基础配置和 .xml 映射文件
2. 初始化基础配置，环境变量等。如数据源
3. 构建 DefaultSqlSessionFactory 实例

#### 执行阶段

1. 通过 SqlSessionFactory 创建 SqlSession
2. 可以通过 SqlSession 实例来直接执行已映射的 SQL 语句 | 更常用的方式是先获取 Mapper(映射)，然后再执行 SQL 语句
   1. Executor：真正的执行者，提供相应的查询、更新、事务方法
      1. SimpleExecutor：update 和 select，使用后就关闭
      2. ReuseExecutor：update 和 select，以 sql=key  statement=value 缓存到 Map 来重复使用
      3. BatchExecutor：用以进行批处理，缓存多个 Statement 对象依次执行
   2. StatementHandler：处理数据库会话，将语句发送的数据库实例中
   3. ParameterHandler ：参数处理器，设置预编译 SQL 语句的参数
   4. ResultSetHandler：用来包装结果集，默认用 DefaultResultSetHandler
3. 调用 session.commit()提交事务
4. 调用 session.close()关闭会话

### 缓存

一级缓存：默认打开，使用本地缓存，作用域为 SqlSession 

二级缓存：默认关闭，使用本地缓存或自定义存储源，作用域为 Mapper 文件（namespace）。**存储的对象需要实现 Serializable 接口**

写入方式：查询一级缓存没有才查询二级，再查询数据库

更新方式：

1. 当使用 INSERT \ UPDATE \ DELETE 会清空二级缓存
2. 当 SqlSession 关闭时，会将一级缓存写入到二级缓存

#### 适用场景

1. 读多写少的表
2. 需要多次**重复**查询的表
3. 查询频繁的非强实时性要求的场景（二级缓存非强一致性）

### Mapper 接口不需要实现类

使用动态代理的方式吗，来生成对应的执行方法，最后交给 SqlSession 对象执行

1. 先获取 MapperProxyFactory 代理工厂，生成 MapperProxy 代理对象
2. 在 MapperProxy 中通常会生成一个 MapperMethod 对象，使用 cachedMapperMethod 初始化
3. 然后执行 MapperMethod 对象中的 execute 方法，里面使用 SqlSession 真正的执行 SQL 语句

### 分页查询

Mybatis 提供：使用 RowBounds 对象进行分页，是针对 ResultSet 的结果集在内存中分页，并不是物理分页

分页插件：使用 mybatis 提供的拦截器，拦截 Executor 的 query 方法，根据 dialect 方言**添加物理分页参数**



## Redis

### 线程模型

#### BIO - NIO - IO multiplexing

1. BIO（Blocking IO）：server 读取 client 发送的数据会阻塞，导致其他 client 无法连接（只能处理一个 client）
2. BIO + 线程池：读取 client 消息由其他线程执行，主线程就不会阻塞（耗费线程资源）
3. NIO（Nonblocking IO）：将 client 放入到一个数组中，隔一段时间遍历一次（无效遍历，线程态和内核态（read）切换）
4. IO multiplexing：将一批文件描述符通过系统调用给内核，让内核进行遍历，就不用切换了（应运而生 select poll epoll）

#### IO 多路复用

多路复用是实现事件驱动（Reator 模式）一种底层技术：收发消息时不会阻塞，整个进程就被充分利用起来了。

文件描述符（file descrptor，fd）：是**操作系统为每个已打开的文件（包括网络socket、管道pipe、设备/dev/null、常规文件.txt）分配的一个非负整数**，用来唯一标识该文件。

简单理解就是：一个服务端进程可以同时处理多个 Socket 描述符，适合高并发连接、大量连接但读写频率低

- **多路**：多个客户端连接（连接就是 Socket 描述符）
- **复用**：使用单进程就能够实现同时处理多个客户端的连接

其发展可以分 **select -> poll -> epoll** 三个阶段来描述

​	select：使用位图来管理最多 1024 个 fd；使用时将 fd 从用户态复制到内核态；线性遍历 O(n)

​	poll：使用动态数组来管理 fd（不限数量）；线性遍历 O(n)

​	epoll：使用红黑树和链表管理 fd；只需要首次注册到到内核一次；回调事件驱动 O(1)

##### Redis 应用方式：

1. 通过包装常用的 select、poll、evport、kqueue 这些 IO 多路复用函数库来实现

2. 为每个 IO 多路复用函数库（ 各系统提供）都实现相同 API，所以底层实现是可以互换的

   **注：Linux：select poll epoll；Windows：select IOCP；macOS：kqueue**

服务器处理的事件分为：文件事件（主功能） 和 时间事件（处理AOF持久化等）

<img src= "https://cdn.tobebetterjavaer.com/stutymore/redis-20240918114125.png" width=60%>

#### Redis 6.0 多线程

参考：https://segmentfault.com/a/1190000041488709

将耗时的 Socket 读取、请求解析、写入等 IO 任务拆分给一组独立线程执行

**让主线程只需要命令执行**，高效处理多个连接请求（核心线程仍然是线程安全的）



### 数据类型 

参考链接：https://www.cnblogs.com/booksea/p/17729973.html | https://www.cnblogs.com/hunternet/p/12742390.html

| 数据类型      | 编码方式                               | 使用条件                |
| :------------ | :------------------------------------- | ----------------------- |
| String 字符串 | int、embstr、raw                       | 长度 < 44 = embstr      |
| Hash 哈希表   | ziplist、hashtable                     | <64byte <512 = ziplist  |
| List 列表     | ziplist、linkedlist、quicklist（3.2+） | <64byte <512 = ziplist  |
| Set 集合      | intset、hashtable                      | 整数 = intset           |
| ZSet 有序集合 | ziplist、skiplist                      | <64byte <128 =  ziplist |

#### 编码结构

**SDS 数组：**通过 len 和 free 记录字符数组的长度，优化 C 字符串；embstr、raw 都是封装的 sdshdr 的结构

**Dict：**又叫散列表（hash）键值对的结构

​	**整个数据库都是用此结构存储**  hash + 数组 + 链表（解决哈希碰撞）

​	当数据量大的时候，扩容（阈值 0.75）后重新计算 hash 值和移动数据到新数组中，会非常耗时

​	所以解决一次性扩容耗时过多的问题，引入渐进式 rehash ：

1. 创建新的 ht[1] 空间，让字典同时持有 ht[0] 和 h[1] 两个哈希表
2. 设置标记 rehashidx = 0 表示开始 rehash 操作
3. 新增在 ht[1] 上；修改、删除、查询在老 ht[0] 上
4. 在任何操作时，都会附带将 ht[0] 在 rehashidx 索引上的所有键值对移动到 ht[1] 上，然后 rehashidx 自增 1

**ZipList：**类似数组在一片连续的内存存储数据，**允许存储数据的大小不同**（长度灵活，节约空间）

​	zlbytes 占用字节数，zltail 尾到头多少字节，zllen 节点数量(<65535)，zlend 末端标记

**QuickList：**= ZipList + LinkedList 将链表分段，每段使用 ZipList 来紧凑存储，多个用双向指针串联

​	因双向链表前后指针占用浪费，节点单独分配内存，加剧碎片化

**intSet：**可以存储 int16_t、int32_t、int64_t 整数，通过升级的方式节约内存（不可降级）

**SkipList：**在双向链表的基础上加多级索引，提高检索效率（数据量大）



### 持久化

两种持久化策略不冲突，可同时使用

#### RDB（redis database）

默认开启的持久化策略；

按照**一定的时间将内存中的数据**以快照形式保存到硬盘中的 dump.rdb（可能丢失最后一次持久化后的数据）

**主动触发命令：**

​	save：将当前所有数据保存到 .rdb 文件中，会阻塞所有用户线程直到写磁盘结束

​	bgsave：创建子线程来执行备份任务，不会阻塞

**自动触发场景：**

​	1. 使用配置中的 save \<seconds> \<changes> 定义时间段；可指定多个配置

​		save 300 10 = 至少有 10 个被修改，则 300 秒触发快照

​		save 60 10000 = 至少 10000 个被修改，则 60 秒触发快照

 2. 通过 SHUTDOWN 命令正常关闭时，自动触发一次，用于重启恢复
 3. 从节点和主节点建立连接时，自动触发 RDB 持久化后将文件发送给从节点

#### AOF（append only file）

**目前更为主流**；将**每次写命令**记录到单独的日志文件中（文件大，恢复慢，启动效率低）

在 redis.conf 使用 appendfsync 指定刷新策略：

​	=always 表示每执行一个命令就写入文件，从而保证数据安全

​	=everysec（默认） 每秒写入文件，崩溃会丢失一秒的数据

​	=no 由操作系统的内核缓存冲洗决定

文件随着操作无限增大 ？

​	重写压缩 - 创建子线程，将当前最新的数据状态转换为一条命令，写入到新的 AOF 文件中

​	主动重写命令：bgrewriteaof

#### Hybrid AOF（v4.0+）

**当执行 AOF 重写时**，不再从空文件开始，而是将当前内存快照（RDB）写入 AOF 文件头部，然后追加写命令形成完整的数据恢复日志。

#### 数据恢复

1. AOF 开启且存在 AOF 文件时，优先加载 AOF 文件。
2. AOF 关闭或者 AOF 文件不存在时，加载 RDB 文件。



### 架构模式

#### 主从模式

通过读写分离，优化读和写不能同时进行的问题。但是主节点一旦挂掉，不能自动切换

所有的数据处理都在 master 上进行

执行命令：slaveof \<master-ip> \<master-port>

##### 主从复制原理

<img src="https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/redis-aa8d2960-b341-49cc-b04c-201241fd15de.png" width=50%>

1. salve 发送 psync <run_id> \<offset>到 master，如果是首次连接则全量复制（psync ? -1）
2. master 收到psync -1 全量复制，则回复 +FULLRESYNC 响应（任务ID-runId 和偏移量-offset）；slave 收到后保存
3. master 启动后台线程，生成 RDB 快照
4. master 发送 RDB，slave 先保存到硬盘
5. master 会将发送 RDB 过程中的写命令保存到客户端缓冲区内；当 RDB 接受完后同步
6. salve 接收到主节点的所有数据后，清空自身旧数据
7. slave 开始加载 RDB 文件，完成后若开启 AOF 则立即进行 bgrewriteaof 操作
8. 如果断开连接，自动重连后会将增量数据发送到 slave

##### 心跳检测

slave 默认会以每秒一次的频率向 master 发送 ACK 命令

1. 检测连接状态，lag 值 = 0-1，超过则说明主从之间连接有故障

2. 通过配置防止 master 在不安全的情况下执行写命令

   min-slaves-to-write 3 ：表示在 slave < 3 时 master 不执行写入命令

   min-slaves-max-lag 10：表示在 lag > 10 时 master 不执行写入命令

3. 检测命令丢失，进行重试

#### 哨兵模式

给主从模式配置哨兵，以实现自动的主从切换功能。提高系统可用性

适合读 > 写的情况，如果写多会对 master 形成同步压力

哨兵节点 = 只是一个特殊配置的 redis 节点，不提供数据功能而已

实现方式：在 sentinel.conf 配置文件中指定：SENTINEL monitor m-name m-ip m-port 即可监控

**主节点掉线后迁移机制**：

1. -------------- 探测阶段 --------------

2. 主观掉线：Sentinel 会每秒一次向建立连接的主节点发送 **PING** 命令，返回 PONG，如果 **down-after-milliseconds** 内未回复则认为其掉线

3. 客观掉线：Sentinel 向其他监控此节点的哨兵查询主机状态，超过 **quorum** 数量的哨兵都认为其下线，则认为客观掉线

4. -------------- 选举阶段 --------------

5. **选举 Leader Sentinel**：（临时性，leader 角色并不会持续存在）

   使用类似 Raft 算法选举出 Leader；备注：类似 raft 算法并没有严格的Raft 状态： Follower | Candidate | Leader）

   3. 每个节点都广播 is-master-down-by-addr 给其他 sentinel

      1. 创建一个 epoch（投票轮）
      2. **给自己投票**，并向其他节点发送 RequestVote（投票） 的消息

   4. 其他节点收到消息后，**若未投票则投票支持**

   5. 谁先收到了一半节点的同意，则转换为 Leader

   6. 向其他节点发送 AppendEnties（通知）

      注：Raft 协议采用**每个节点随机超时时间**，先转为 Candidate 的节点会先发起投票，从而获得多数票

6. -------------- 转移阶段 --------------

7. Leader Sentinel 确定升级节点：在 slave 中选择一个升级

   1. 去掉客观、主观掉线的节点
   2. 选择 slave-priority 最高的节点
   3. 选择数据偏移量大的节点（数据多）
   4. 选择 runid（pid）最小的节点（越小说明重启的少）

8. 故障转移：

   1. 让其他 slave 复制 new-master
   2. 客户端连接时返回 new-master 地址
   3. 新增 new-master 的 replicaof 配置； sentinel.conf 修改监控对象为 new-master

#### 集群模式

为避免单一节点负载过高导致不稳定，集群模式用**一致性哈希|哈希槽**将 key 分到各个节点上去

高可用：集群各个节点会探测相互是否存活，多个节点判定一个节点挂了，则将其踢出集群并选择一个从节点作为新主节点（哨兵类似）

避免了单点问题，但节点间同步会消耗部分性能

在写比较多的情况下使用此模式

##### 数据分区

1. hash % node_number（取余）：当出现节点加入或者退出，所有节点都将受到影响
2. hash + 顺时针（一致性hash）：计算 hash 后顺时针找到先遇见的节点存放，有节点变动只影响下一个节点
3. hash % 16383（redis cluster）：将所有的 key 分为 0-16383 个槽位，计算 hash 后取余

##### 创建集群

1. 设置节点：所有节点需要开启 cluster-enable yes 启动集群模式
2. 启动集群：redis-cli --cluster create IP1:port1...IP6:port6 --cluster-replicas 1
3. 节点握手：节点通过 Gossip 协议彼此通信，发送 cluster meet{ip}{port} 进行握手
4. 分配槽：集群通过 cluster addslots 命令，将 16384 个槽分配到各个节点

##### 故障发现

1. 集群内节点定期使用 ping/pong 消息实现节点通信
2. 若在 cluster-node-timeout 时间内通信一直失败，则发送节点认为接收节点主观下线（pfail）
3. 发现节点通过 Gossip 消息传播到其他节点，其他节点收集故障节点线下报告
4. 若超过半数以上**主节点**都标记为主观下线，则升级为客观下线（fail）

##### 故障转移

1. 资格检查：1.每个从节点检查与主节点断线时间是否 >cluster-node-timeout 2.谁的 offset 更新
2. 发起选举：哪个从节点最早完成检查，就可以最先触发 FAILOVER_START，向所有节点发送 FAILOVER_AUTH_REQUEST（请求投票）
3. 投票：每个主节点会收到处理故障选举的消息，一个主节点一票，回复 FAILOVER_AUTH_GRANTED
4. 替换：收到超过 n/2 + 1 的从节点成为新主节点，设置为 master、停止复制、设置 configEpoch（提醒其他节点epoch更新）、广播 PONG

##### 集群伸缩

```bash
# 创建集群
redis-cli --cluster create IP1:port1...IP6:port6 --cluster-replicas 1

# 添加节点
redis-cli --cluster add-node 127.0.0.1:7004 127.0.0.1:7000

# 重新分配槽位（迁移槽）
redis-cli --cluster reshard 127.0.0.1:7000

# 删除节点
redis-cli --cluster del-node 127.0.0.1:7000 <node-id>
```





### 事务

一次性、顺序性、排他性的执行一个队列中的一系列命令

实现方式：

1. MULTI：开启一个事务，总是返回OK。之后发送的指令会被放到队列中，直到  EXEC 被执行
2. EXEC：执行事务队列中的所有命令。按照先后顺序返回结果
3. WATCH：乐观锁，为事务提供 Check-And-Set（CAS）行为。监控一个或多个键，一旦被修改之后，其他指令不会执行；直到 EXEC
4. RECARD：客户端清空队列，并放弃执行事务
5. UNWATCH：取消 WATCH 对所有键的监控

### 分布式锁

#### WATCH 实现 Redis 乐观锁

WATCH 是 redis 自带的命令，可以监控一个或多个变量是否被更改，更改则其他指令不会执行

1. WATCH xxx
2. GET xxx 然后在 value += 1
3. EXEC，如果 value 被修改过则回滚

#### set NX 防止超卖

set NX（redis 2.6+ 取代 setnx 命令）只有在当前 key 不存在时才会成功

加锁：jedis.set(key, value, "NX", "EX", expireTime)

释放锁：使用 redis + lua 

```java
String lua = "if redis.call('get', KEYS[1]) == ARGV[1] then"
    			+ " return redis.call('del', KEYS[1]) "
    		+ "else "
    			+"return 0 end";
Object result = jedis.eval(lua, Collections.singletonList(lockKey);
```

#### Redisson 分布式锁

在业务需要强一致性，不能重复获得锁的情况下可以使用。性能较低比较重量级

##### 实现方式

在 RedissonLock 类中，通过 Lua 脚本封装 Redis 命令来实现                                                                                                      

##### 自动续期 Watch Dog

在 Redis 中获取到锁，但是程序实际的执行时间超过了设置的过期时间，则需要自动续期

在 Redis 的看门狗机制中，检查锁的过程并不是单独的一个步骤，而是与锁的续期操作绑定在一起，通过 Lua 脚本完成的

检查与续期是一个整体的原子操作，以确保只有持有锁的客户端才能成功续期



### 内存淘汰策略

当 redis 内存不够时，全局的移除策略

1. noeviction：不移除，返回报错信息（默认）
2. allkeys-lru：移除最近最少使用的
3. allkeys-random：随机移除

设置了过期时间的键移除策略

1. volatile-lru：移除最近最少使用的
2. volatile-random：随机移除
3. volatile-ttl：移除快过期的

### Redis 优化

1. 使用短 key，KV 值太大可以拆分成几个小的
2. 尽量少用 keys *（这个命令是阻塞的）
3. 尽量设置过期时间，以保证会被清理
4. 如果不需要持久化的可以关闭以提升性能
5. 读写峰值单机 10W 左右，超过可使用 local-cache 配合，甚至多层 redis 缓存

### 缓存失效策略

1. 定时清理：给设置了过期时间的 key 创建定时器
2. 惰性清理：在获取时判断（对内存不友好 --没有正常释放）
3. 定时扫描清理：在 100ms 内随机检查 20 个，若存在 25% 以上则循环删除

### 使用时读写策略

1. Cache Aside（旁路策略）：写数据后删除缓存，无缓存读数据库后写入到缓存

   缓存一致性：删除 -> 修改DB -> 等待再删除（双删）

2. Read / Write Through（读写穿透）：将 cache 作为主要存储，所有操作由组件写入数据库

3. Write Behind caching（异步缓存写入）：只更新 cache，批量异步更新 DB；例：多次写入影响效率，先放到缓存再一次写入

请求数据不一定在 cache 中：提前将热点数据写入到 cache

### 常见问题

**缓存雪崩：**同一时间多个缓存失效，导致请求都去查询数据库，短时间承受大量请求而崩掉

- Redis 高可用，主从 + 哨兵，Redis Cluster 
- 使用 hystrix 等限流 & 降级措施
- 设置不同的过期时间，避免同时失效
- 设置缓存标记，标记失效则更新数据缓存（那是不是还得配合锁 ？）

**缓存穿透：**缓存和数据库中都没此数据，数据库接受了大量无效请求而崩掉

- 加强参数校验，剔除无效参数
- 在缓存中新增无数据 null 标记，并设置短一点的过期时间。防止同一 ID 暴力攻击
- 采用足够大的布隆过滤器

**缓存击穿：**数据库有数据，缓存中没有。并发访问数据库中的同一记录

- 热点数据永不过期，异步线程处理
- 查询后回写加互斥锁
- 缓存预热

**数据不一致：**缓存机器被打满，写入缓存失败；在 rehash 时网络波动多次 rehash，导致脏数据

- 进行重试，再不行加入到 mq 中，等待缓存机器恢复后删除，再次使用时从数据库加入缓存
- 缓存时间调短，早点更新新数据 ？
- 不使用 rehash 进行漂移，使用多层缓存 ？

**数据并发竞争：**大量用户同时查询同一缓存

- 回写加互斥锁，查询失败快速返回
- 保持缓存多备份，减少并发

**HotKey：**短时间内被频繁访问，导致系统崩溃（明星爆瓜）

- 找出来：可以预见的提前评估；使用工具（如 spark）找出最近历史的热点数据
- 将同一个 key 加个后缀，分成多份，分散压力
- 使用应用内的缓存，注意设置上限

**HotKey 重建：**并发量特别大 或 重建缓存不能短时间完成

- 互斥锁，只有一个线程重建
- 缓存永不过期，由程序来检查是否过期，然后异步重建

**BigKey：** String > 5mb | size > 5000

- 拆分成多个 key-value 的小 key
- 序列化后压缩存储的数据
- 定期清理失效数据

### Redis 热升级

1. 创建一个 redis 壳程序，将 server 的所有属性保存为全局变量
2. 将 redis 的逻辑代码全部封装到动态链接库 so 中
3. 在后续升级时通过指令壳程序，重新加 redis-4.so 到 redis-5.so 文件 ，即可完成毫秒级升级



## Elastic Search

基础概略：https://www.cnblogs.com/jajian/p/11223992.html

### 基础概念

**倒排索引：**将数据拆分成词语，以词语为主题，记录其在那些 id 数据中存在，实现较快的搜索

**正排索引：**记录 id 数据中的所有词语，用以进行聚合、排序等操作。

1. doc_value 默认是开启的，不支持 text，存在磁盘中
2. field_data 需要在 _mapping 中自定义，使用堆外内存

**主分片：**number_of_shards 默认值：5 < v7.0 < 1 创建index后不可修改（分片数确定数据存储在哪个分片）是一个 Lucense 实例

**分片副本：**number_of_replicas 主分片的备份，实现高可用，减缓查询压力。创建后可以进行手动调整

**滚动索引：**调用接口设置其满足的条件则滚动，再用一个别名来访问多个索引



### 写过程和原理

#### 过程

1. 客户端发送请求数据到集群中的一个节点
2. 节点通过 hash(_routing) % shards_num 计算出在应该存储在哪个主分片
3. 将请求数据转发到主分片所在的节点上
4. 主分片写入成功后，将数据发送到副本上
5. 等待所有副本（可修改 index 配置：wait_for_active_shards=1）返回结果后，主分片节点将结果返回给终端请求的节点
6. 如果没达到 wait_for_active_shards 的数量 OR 超过 30s ，则返回错误

#### 原理

参考：https://cloud.tencent.com/developer/article/1531723

![es-write](img\es-write.png)

1. 请求数据发送到主节点，解析数据后写入节点的 memory  buffer 中

2. 然后将 Id 和 doc 写入到 Translog 中进行备份

   设置 translog 写到磁盘的方式：

   1. index.translog.sync_intervel（默认 5s）将新增的 translog 写入到磁盘中
   2. index.translog.duriablity（request 每个请求| async 异步| fsync 文件刷新） 设置刷新到磁盘的方式

   **备注：**这个时候 geyById 的方式查询则可以直接使用  translog 中的数据（实时）

3. 节点在 buffer 超过阈值>1s（index.refresh_interval） >jvm 10% 则触发 **refresh** 操作，将缓存中的数据生成一个 Segment

4. Segment 是在 Filesystem Cache 中

   **备注：**因为 Segment 是不可变的，所以被放入内核中的文件系统缓存则会一直存在，除非文件系统缓存空间不足

5. 当 >30min || Translog > 512mb（index.translog.flush_threshold_size）则会触发 **flush** 操作

   1. 将当前 memory buffer 中的数据写入到 segment
   2. 调用 lucene 的 commit 方法将**所有的 segment fsync** 到硬盘，生成提交点
   3. 清空 translog 文件


##### Segment Merge

由上可知内存中的数据 1s 就会被写入到 Filesystem Cache 一个新的 Segment 中，那么就会产生很多小的 Segment 文件

遍历这些小的文件会影响效率，所以**占用不高时**会在后台执行一个定时进程合并小段到大段中去

1. 合并进程选择一部分大小相近的段，在后台将其合并到更大的段中
2. 合并结束后，old-segment 被删除，new-segment **被 flush 到磁盘**
3. 写入一个包含新段的提交点，new-segment 被打开可以搜索



### 更新 / 删除过程

#### 删除

1. 每个 Segement 中都维护了一个 **.del 文件**，删除时将文档 Id 状态更新为 deleted
2. 在查询时剔除掉 .del 中的文档，而不会物理删除
3. 当 **Segment merge** 的时候不会将 .del 中的文档进行合并到新的 Segment 中（变相删除）

#### 更新

Lucene 不支持部分更新，则 ElasticSearch 实现此功能：

1. 从 Segment 或 translog 中读取到同 ID 的文档，进行对比合并后更新 versionMap = v1
2. 加锁
3. 检查 versionMap 中的 version 是否和当前一致，不一致则重新准备更新
4. 将 version +=1 ，再将其加入到 Lucene 中，**先删除后新增**
5. 写入成功后将 versionMap = v2 
6. 释放锁



### 查询过程

#### GET 

直接通过 id 查询数据（实时的）

1. 先查询内存中的 translog 
2. 然后查询磁盘的 translog 
3. 然后再查询磁盘上的 Segment

#### Search 

查询分为两阶段：Query Then Fetch

**Query：**

1. 客户发送一个请求到节点（协调节点），节点创建一个 from + size  的队列
2. 这个节点会将请求转发给 **每个分片**（主分片 or 副本 随机轮询）
3. 每个分片执行查询并将其添加到本地的 from + size  的有序优先队列中（例：每个节点的 top10）
4. 每个分片返回各自队列中的 **ID 和 排序值** 给协调节点

**Fetch：**

1. 协调节点根据 ID 判定那些加入到**总结果的队列**中
2. 向相关分片提交多个 GET 请求
3. 每个分片加载出 ID 对应的数据返回给协调节点
4. 协调节点收集到相关数据后返回给客户端



### 集群配置

Zen Discovery 是内建的、默认的发现模块，提供**单播 和 基于文件**的发现

只需要配置文件中的 cluster.name（集群名称）和 discovery.zen.ping.unicast.hosts（单播的主机列表）即可自动组建集群

**主节点：**node.master = true

**数据节点：**node.data = true

一个节点既可以是主节点 也可以是数据节点

#### 选举主节点

主节点负责创建、删除索引、跟踪节点加入集群、追踪节点状态、分片分配等，**不负责文档级别的管理**

只有 node.master = true 的节点才有**选举权和被选举权**，其他节点不参与选举操作

参考：https://www.cnblogs.com/shanml/p/16684887.html|https://developer.aliyun.com/article/1082593

1. 节点启动后执行 PING 操作，当节点数超过 mininum_master_nodes 才开始选举，不够则重复 PING
2. 节点给自己所知道的备选节点中 ID 为最小的投票
3. 如果得票超过 discovery.zen.mininum_master_nodes && 自己也投票给自己，那么这个节点就是新的 master
4. 如果不满足则重复此过程

#### 脑裂问题

产生原因：

1. 网络抖动导致部分节点认为 master 掉线
2. master 节点负载过大，导致 es 进程失去响应
3. GC 较大内存导致失去响应

解决方案：

1. 延迟：提高 discovery.zen.ping_timeout 的值
2. 负载：设置 node.data = false 避免进行数据操作
3. 选举：设置 mininum_master_nodes = (主节点 / 2) + 1 超过半数才能选举 master



### 常见优化

#### 配置优化

1. 内存按照 50% 的比例分配给 jvm，并固定大小（ xmx = xms）
2. 剩余的 50% 留给文件系统缓存
3. 推荐 xmx 不超过 31gb （数值不固定）以确保虚拟机采用 zero-based 压缩指针算法来节约空间

#### 写优化

1. 初始化数据时设置 number_of_replicas = 0 ，避免副本建立索引
2. 使用自动生成 ID 
3. 合理的选择类型和分词器
4. 禁用搜索评分，延长索引刷新间隔

#### 读优化

1. 使用 filter 代替 query，减少评分环节
2. 使用滚动索引将数据集中

#### 深分页问题

1. 不允许跳页，采用 scroll 方式查询
2. 页面禁用深度跳页



## RabbitMQ

### 产品比较

|          | RabbitMQ            | Kafka             | RocketMQ            |
| -------- | ------------------- | ----------------- | ------------------- |
| 性能     | 1W+                 | **10W+**          | **10W+**            |
| 延迟     | **us**              | ms                | ms                  |
| 可用性   | 主从高可用          | 多副本分布式架构  | 分布式架构          |
| 可靠性   | **基本不丢**        | 配置优化后 0 丢失 | 配置优化后 0 丢失   |
| 支持语言 | **几乎所有**        | JAVA，PHP，Python | JAVA，C++（不成熟） |
|          |                     |                   |                     |
| 劣势     | erlang 不好二次开发 | 批量处理延迟较高  | 兼容不好            |

**为什么选择 RabbitMQ：**

1. 消息延迟低，适合业务场景
2. 支持语言多，方便进行集成开发

**为什么选择 Kafka：**

1. 强大的性能和吞吐量
2. 支持流式处理，是大数据处理、日志搜集等方面的标杆

**为什么选择 RocketMQ：**

1. 性能好吞吐量大、功能全面
2. 有活跃的中文社区



### 基础概念

![rabbitmq](img\rabbitmq.png)

broker：一个消息队列服务器

v-host：虚拟主机（可以理解为 RabbitMQ 的多个实例）

exchange：交换机，用以接受消息

queue：队列，用以存储消息

routing-key：用以绑定 exchange 和 queue

connection：终端和服务器建立的连接

channel：在连接上创建的信道（一个连接多个信道）

#### Exchange 类型

参考：https://www.rabbitmq.com/getstarted.html 有 7 中类型，常用 5 种

1. 直接：投递给 **默认** 的交换机，一个消费
2. 工作：投递给 **默认** 的交换机，多个消费（轮询）
3. direct：绑定 routing-key 找到对应的队列
4. topic：支持 routeing-key 中携带匹配字符，找到相匹配的队列（*：一个字符 #：多个字符）
5. fanout：把消息复制同步到所有绑定的队列中
6. headers：匹配消息头（模式：all | any）

#### 生产流程

1. producer 连接服务器（broker）建立连接（connection）并在连接上创建一个信道（channel）
2. producer 配置 exchange 和 queue 然后将其使用 routing-key 关联起来
3. producer 发送消息到服务器
   1. 消息头中包含：routing-key \ priority（优先级）\ delivery-mode（存储模式）
4. 服务器根据 routing-key 找到对应的队列进行投递，没有则以 mandatory 属性进行退回（true） or 丢弃（false）

#### 消费流程

1. consumer 连接到服务器（broker）建立连接（connection）并在连接上创建一个信道（channel）
2. 向 broker 请求消费对应队列中的消息，设置响应的回调函数
3. 等待回应，接受返回消息
4. 返回 ack 确认消息
5. 服务器从队列中删除已经确认的消息

#### 死信队列

当消息成为 Dead Message 时，可以被重新发送到另一个交换机（DLX：dead letter exchange）

成为死信的条件：

1. 消息超时（TTL：Time to live，参数：x-message-ttl）
2. 消息被拒收 channel.basicRejected & requeue = false
3. 队列超过长度限制

#### 延迟消息



###  集群搭建

1. 在多个虚拟机上安装 rabbitmq

2. 指定每个虚拟机的名称 /etc/hostname 

   命令：rabbitmqctl status 会显示： Status of node 'rabbit@node1'  ....

3. 复制 node1 的 cookie 到 node2,3（/var/lib/rabbitmq/.erlang.cookie）

   **注意：** 1. chmod 400 /var/lib/rabbitmq/.erlang.cookie    2. 用户组为：rabbit:rabbit

4. 修改每个节点的 /etc/hosts 文件

   192.168.100.10 node1 

5. 使用（rabbit-server -detached）启动三个独立的集群

6. 将节点加入到某个集群中去

   1. rabbitmqctl stop_app
   2. rabbitmqctl reset_app
   3. rabbitmqctl join_cluster rabbit@node1(status看到的) --ram（内存节点 default=disk）

#### 镜像集群

可以实现消息多节点存储，在 admin 中进行配置 policy 规则

1. pattern：匹配的正则表达式

2. ha-mode：镜像队列的模式

   all（所有节点） | exactly（指定数量） | nodes （指定节点）

3. ha-params：模式的值

4. ha-sync-mode：同步的方式

   automatic （自动）| manually（手动）

#### 集群代理

使用 HAProxy 进行代理配置即可



### 常见问题

**如何保证休息不丢失** ？

生产端：

1. 开启事务 txSelect() | txRollback() | txCommit()  
2. 设置为 channel.confirm 模式（失败发送 Nack 消息）

队列：**关键消息**开启持久化（deliveryMode = 2）

消费端：关闭自动确认（autoAck = false），进行手动确认（注意：1.忘记 ack 导致堆积 2.确认太慢导致堆积）

**如何让消息有序 ?** 

消息到队列中肯定是有序的，一个队列对应一个消费者即可

**如何让消息优先处理 ？**

设置消费优先级 x-max-priority（1-10）属性，数字大会被优先消费

**如何保证消息不被重复消费 ？**

幂等性：消息被执行多次 和 一次的影响相同

1. 添加版本号（不同则不执行）
2. 添加主键约束（后续执行会冲突并失败）
3. 记录关键 key 是否已经被处理

**如何处理消息堆积的情况 ？**

排查：堆积一定是生产和消费能力不匹配的问题导致的

1. 增加更多的消费者，提高消费速度
2. 使用新的队列接受消息，保证新消息的服务正常
3. 排查消费者能力不足的问题
4. 根据数据状态找回丢失的消息，进行重新投递

 

## RocketMQ

### 部署

#### 单机部署

下载安装包后运行 bin 目录下的 nameserver 和 broker 程序

#### 主从高集群

1. 各节点安装服务，开放防火墙等

2. 根据节点数量规划 nameserver 和 broker 及副本数量

3. 创建各个 broker 的配置文件

   ```properties
   brokerClusterName=cluster-name
   brokerName=order-broker
   # 0.master >0.slave
   brokerId=0
   nameserverAddr=10.11.1.10:9876;10.11.1.11:9876;
   ```

4. 使用 -c 指定配置文件启动 broker 和 nameserver

#### 监控

下载打包好的 rabbitmq-dashboard.jar 运行即可



### 常见模式

#### 基本模式

创建新消息：new Message("topic", "tags", "".getBytes())

同步发送：DefaultMQProducer.send(message)

异步发送：DefaultMQProducer.send(message, new SendCallback() { onSuccess(); onException(); })

单向发送（不管消息是否成功）：DefaultMQProducer.sendOneway(message)

消息拉取：DefaultPullConsumer.subscribe("topic")  consumer.poll()

在指定队列中拉取：

```java
// 获取 topic 的队列
Collection<MessageQueue> queues = consumer.fetchMessageQueues("topic");
// 分配队列
List<MessageQueue> listQueues = new ArrayList(queues);
consumer.assgin(listQueues);
// 从指定队列中拉取
consumer.seek(listQueues.get(0), offset);
```

消息推来：

```java
// 并发处理
DefaultMQPushConsumer.setMessageLister(new MessageListenerConcurrently() { 
	@Override
	public ConusmeConcurrentlyStatus consumeMessage(List<MessageExt> list, MessageListenerConcurrently context){ 
		// TODO 处理消息
        
        return ConsumerConcurrentlyStatus.
	} 
});

```

#### 顺序消息

是让指定的某一个队列中的消息是有序的，而不是让所有队列中的消息是有序的

生产：DefaultMQProducer.send(message, new MessageSelector() { // TODO 根据 ID 参数指定投递的队列 }, id)

消费：DefaultMQPushConsumer.setMessageLister(new MessageListenerOrderly(){ xxx })

#### 广播消息

**consumer**.setMessageModel(MessageModel.BROADCASTING（每个消费者都发） | CLUSETERING（选择一个消费者）)

#### 延迟消息

message.setDelayTimeLevel(1)   级别：1s 5s 10s 30s 1m 2m 3m 4m 5m 10m 20m 30m 1h 2h

message.setDelayTimeMs(1000L)

#### 过滤消息

tag：consumer.subscribe("topic", "tag_x || tag_y")

SQL：.subscribe("topic", MessageSelector.bySql("TAGS is not null and PROPS.key = value")) 

#### 事务消息

1. 实现事务回调实现方法：

   ```java
   public class TransactionListenerImpl implements TransactionListener{
       // 回调的事务执行方法
       @Override
       public LocalTransactionState executeLocalTransaction(Message message, Object o){
           
           return LocalTransactionState.COMMIT_MESSAGE | ROLLBACK_MESSAGE | UNKNOW;
       }
       
       // 回调的事务定时检测方法
       @Override
       public LocalTransactionState checkLocalTransaction(Message message, Object o){
           
           return LocalTransactionState.COMMIT_MESSAGE | ROLLBACK_MESSAGE | UNKNOW;
       }
   }
   ```

2. 绑定事务回调：producer.setTransactionListener(new TransactionListenerImpl())

3. 提交事务消息：producer.sendMessageInTransaction(message, args)



### 部分原理

#### 持久化原理

![rocketmq-file](img\rocketmq-file.png)

CommitLog ：存放消息的文件，默认为 1G

IndexFile： 消息的索引文件，记录整个 broker 中的消息 key hash 和在 commitLog 中的偏移量。

1. IndexHead ：beginTimestamp \ endTimestamp \ beginPhyoffset \ endPhyoffset \ hashSlot count \ index count
2. hashslot（500w个） ：hash code
3. index（2000w个）：hashcode \ phyoffset \ timedif \ 

ConsumeQueue：定位消息的索引文件。记录 topic 和 queue 的消息具体存放的位置：commitLogOffset \ msgSize \ tagsCode

##### 存储优化

page cache：操作系统对文件的缓存，访问读取文件时会顺序的对其他相邻的数据文件进行预读取

顺序读写：ConsumeQueue 的文件小并且顺序读取的，接近内存性能

零拷贝：减少**用户态与内核态**的上下文切换和内存拷贝的次数、提升I/O的性能。常见实现是 Mmap **在 Java 中 MappedByteBuffer**

#### 事务原理

![rocket-transcation](img\rocket-transcation.png)

1. 服务器接收到消息，被标记为 ”暂不可投递“ 状态（半消息）
2. 确认半消息接受给生产者，生产执行本地的事务处理方法，并返回事务状态给服务器
3. 若当前状态为 UNKNOW ，则服务器定时回查消息（15次），生产者会返回当前状态
4. 根据回查状态处理消息

#### 延迟消息原理

临时存储 + 定时任务

1. 延时消息会被投递到 **SCHEDULE_TOPIC_XXX 临时主题**中，主题下有多个级别的延时队列
2. 然后启动一个定时任务检测投递时间
3. 将到达投递时间的消息进行投递

#### 负载均衡原理

生产者：设置 sendLatencyEnable 开关（根据上次请求的延时，延缓下次分配任务的时间）默认情况：轮询

消费者：

​	**前提：**push 消费模式也是封装 pull + 轮询 来实现的，所以两个模式都需要知道去哪个队列获取消息

​	**心跳发送：**

 1. Consumer 会将心跳发送到**所有 broker** 中
 2. 收到后会将它维护在 ConsumerManager 的本地缓存的 **consumerTable** 中
 3. 将封装后的通道消息保存在本地缓存的 **channelInfoTable** **中**

​	**负载均衡：**

​	Consumer 在启动时会启动 **RebalanceService** 的启动（20s），这个线程调用 RebalanceImpl.rebalanceByTopic() 实现负载均衡

1. 查询当前 topic 中的所有队列的集合（mqSet）

2. 查询当前 topic 的消费者列表

3. 用**队列平均分配算法**计算出消费者对应的队列是哪些

4. 更新当前的 processQueueTable

      过期的队列：标记为 Dropped = true，并获取（1s）队列锁后移除

      还在的队列：**判断是否过期 并 移除 push 模式的队列**

      新的队列：直接放入

5. 为每个 queue 创建一个 ProcessQueue 加入到 ProcessQueueTable 

6. 创建 pullRequest 放入到 **PullMessageService** 线程中的 pullRequestQueue 等待取出后发送请求



### 常见问题

#### 如何保证消息可用性 ？

生产者：

1. 投递消息的确认
2. 事务

消息中间件：持久化（同步刷盘 or 异步刷盘）

消费者：手动确认消息接受

#### 如何实现消息过滤 ？

1.TAG 2.bySql 3.Filter（自定义函数）

#### 如何处理消息积压 ？

queue > consumer：添加更多的消费者

queue <= consumer：消息迁移到其他 queue 



## Kafka

### 基础概念

**controller broker：**管理**整个集群**中的分区和副本状态（先到先得）

message：消息

**batch：**消息并不会直接投递，而是多个数据批量处理

topic：消息的主题

partition：消息的分区（可以理解为队列）

replicas：消息的副本（仅为复制备份的作用，在 2.4+ 之后提供**有限的**读取服务）

producer：生产者

consumer：消费者

consumerGroup：组内的所有消费者协调在一起来消费订阅主题（subscribed topics）的所有分区

​	（同一个消费者组不会出现对同一个 partition 的消费）

**consumer offset：**消费者记录各个 partition 的 offset，互不影响

### 处理流程

#### 发送消息的流程

1. producer 创建 Sender 线程并设置为守护线程，调用其发送消息

2. broker 接受到消息后，以过滤器 -> 序列化器 -> 分区器的流程进行处理

3. 将处理后的消息放入**消息缓存区**内

4. 当消息达到 batch.size（数量） || linger.ms （时间）就将消息写入到 partition 中

   确认消息的节点：

   acks = 0 ：到达缓冲区后就确认

   acks = 1：被写入到 Leader partition 就确认

   acks = all：被写入到 **所有 partition** 就确认

5. 投递失败 & retries > 0 则会按照指定的重试次数进行尝试

#### 副本同步

1. Follower 发送 FETCH 更新请求到 Leader 上
2. Leader 读取出底层日志文件的消息数据 
3. **并且更新它记录的**当前 Follower 的 LEO（log end offset，下一条消息的位置）和 HW（high watermark，备份的位置）
4. 再尝试更新**当前分区**的高水位值
5. Follower 收到数据后写入底层日志，并更新自己记录的 LEO 和 HW 值

#### 重新选举 Leader

前提：

1. 在 Zookeeper 中会针对**每个 partition** 维护一个 in-sync replica （ISR，在同步的副本）集合

2. 数据量和 Leader 差距太大会被踢出，只有在其中的 Follower 才有资格被选举
3. 如果 ISR 是空的，则以配置（unclean.leader.election.enable）确定是否在被踢出的列表（非同步副本）中选举**（丢数据）**
4. Kafka 只有一个服务器在线也可以提供服务（冗余度低）

选举：调用配置的分区选择算法，进行选举（常见：Zab(自由竞争) \ Raft）

#### 分区分配 Rebalance

条件：

1. 分组成员发生变化
2. 订阅的主题数量发生变化
3. 订阅的主题分区数量发生变化

过程：

1. 分区的 Leader 将使用分区分配方案，确定各个 topic 下的 partition 应该被哪些 consumer 消费
2. 将结果封装成 SyncGroup 发送到 coordinator（broker 的用户组管理）
3. 将结果放入 SyncGroup 的 response 中发送到 consumer



## 分布式系统

### 分布式理论

#### CAP 原则

P（partition tolerance）：分区容错；这是分布式的基础需求，都会满足

A（availability）：可用性；保证系统的可用性优先，即使出现部分数据不一致 or 功能降级等，都可以接受

C（consistency）：一致性；任何时候所有节点的状态应该都是一致的

**模型：**

CP：系统在网络分区的情况下保持一致性，如：zookeeper

AP：保证服务的可用性，可能出现数据错误的情况，如：eureka

##### 为什么 CAP 不能兼得 ？

1. **在网络正常等基础服务正常的情况下，CAP 是都可以保障的**
2. 但在出现异常的情况下，C 和 A 就会出现冲突，进而只能优先选择其中的一种方案
3. 所以我认为其实并不是不能兼得，而是选择其优先的功能

#### BASE 理论

核心：即便不能达到强一致性（CP）也采用合适的方式来实现最终一致性的效果

系统状态恢复：Basically Available -> Soft state -> Eventually consistent



### 分布式事务

#### 实现方案

**2PC（两阶段）：**1.事务协调者发出事务准备，等待响应 2.以响应的结果做提交 or 回滚

​	问题：事务协调者挂了、回应后阻塞、数据不一致

**3PC（三阶段）：**1. 发起事务准备，等待回应 2.进行预提交 3.以响应的结果做提交 or 回滚

​	问题：只是解决了同步阻塞 和 单点故障的问题（预提交、提交超时检查）

**TCC（Try Confirm Cancel）：**将事务的提交给**应用层**控制

​	问题：需要提供 Try Confirm Cancel 调用，提高业务耦合度

**MQ 消息事务：**用消息的事务来代替本地事务 ？



## Netty

参考连接：https://dongzl.github.io/netty-handbook/#/_content/chapter05

### 基础概念

ServerBootstrap：引导程序

**Channel：**建立的网络连接 C/S 的通道

<<<<<<< Updated upstream
1. NioScoketChannel：异步 TCP Socket 连接（客户端）
2. NioServerSocketChannel：异步 TCP Socket 连接（服务器）
3. NioDatagramChannel：异步的 UDP 连接
4. NioSctpChannel：异步的 SCTP 连接
=======
	1. NioScoketChannel：异步 TCP Socket 连接（客户端）
	2. NioServerSocketChannel：异步 TCP Socket 连接（服务器）
	3. NioDatagramChannel：异步的 UDP 连接
	4. NioSctpChannel：异步的 SCTP 连接
>>>>>>> Stashed changes

**NioEventLoop：**一个不断循环**执行任务**的线程（线程 + 任务队列）

​	IO 任务：selectKey 中就绪事件，connect \ accept \ read \ write 等（由 processSelectedKeys 触发）

​	非 IO 任务：添加到队列的任务，register() \ bind() 等

​	**Channel  -- bind() --> NioEventLoop（绑定）** 

NioEventLoopGroup：管理 eventLoop 的生命周期，多个 NioEventLoop 的集合



ChannelPipeline：管理由 ChannelHanlderContext 组成的双向链表，用于拦截入站事件和出站事件

ChannelHandlerContext：保存 channel 上下文信息，**关联一个 ChannelHandler 对象**

**ChannelHandler：处理 IO 事件**，并将事件转发给 Channel Pipeline 的下一个程序

注意：一个事件可能会经过多个 handler 的处理（如编码、解码、校验等）所以在 pipeline 中形成双向链表的方式

### 线程模型

**Reactor：**是一种为处理服务请求并发,提交到一个或者多个服务处理程序的事件设计模式。

当请求抵达后，服务处理程序使用解多路分配策略，然后同步地派发这些请求至相关的请求处理程序

1. 单 Reactor - 单线程：一个线程负责多路复用和业务处理的功能 [参考连接](https://dongzl.github.io/netty-handbook/#/_content/chapter05?id=_54-%e5%8d%95-reactor-%e5%8d%95%e7%ba%bf%e7%a8%8b)
2. 单 Reactor - 多线程：一个线程负责多路复用，多个线程负责业务处理
3. 主从 Reactor - 多线程：MainReactor 负责连接（accept），然后 channel 分发给 SubReactor 处理，其他线程处理业务

### 代码实现

启动服务器：

```java
public class NettyServer {
    
    public static void main(String[] args) throws Exception {
        
        // 1. bossGroup 只是处理连接请求 , 真正的和客户端业务处理，会交给 workerGroup完成
        // 2. 两个都是无限循环
        // 3. bossGroup 和 workerGroup 可以指定线程个数（默认：cpu core * 2）
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(); //8
        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class) 
	                // 设置线程队列得到连接个数
                    .option(ChannelOption.SO_BACKLOG, 128) 
                	//设置保持活动连接状态
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
	                // 该 handler对应 bossGroup , childHandler 对应 workerGroup
		            .handler(null) 
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel channel) throws Exception {
                            // 可以使用一个集合管理 SocketChannel，用来推送消息
                            
                            // 设置 channel 的 handler
                            channel.pipeline()
                                .addLst("encode", new ProtubufEecode())
                                .addLst("decode", new ProtubufDecode())
                                .addLst(new MyMessageHandler());
                        }
                    });
            
            // 绑定端口 和 关闭监听
            ChannelFuture cf = bootstrap.bind(6668).sync();
            cf.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

自定义事件处理器：

```java
public class MyMessageHandler extends ChannelInboundHandlerAdapter {
    
    /**
    * 接受消息的处理。还可以覆盖：channelActive()建立连接 \ channelInactive 断开连接 等等方法
    */
	@Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        
    }
}
```

### 工作流程

![](https://ask.qcloudimg.com/http-save/yehe-6841114/ocmrm2pw9j.png)

NioEventLoop ：一个不断循环的执行处理任务的线程，每个 NioEventLoop 都有一个 Selector，用于监听绑定在其上的 socket 的网络通讯

1. BossGroup 专门负责接收客户端的连接，WorkerGroup 专门负责网络的读写（**包含 subReactor**）
2. 每个 BossNioEventLoop 循环执行的步骤有 3 步
   1. 轮询 accept 事件
   2. 处理 accept 事件，与 client 建立连接，生成 NioScocketChannel，并将其注册到某个 worker NioEventLoop 上的 Selector
   3. 处理任务队列的任务，即 runAllTasks
3. 每个 Worker NIOEventLoop 循环执行的步骤
   1. 轮询 read，write 事件
   2. 处理 I/O 事件，即 read，write 事件，在对应 NioScocketChannel 处理
   3. 处理任务队列的任务，即 runAllTasks
   4. 每个 Worker NIOEventLoop 处理业务时，会使用 pipeline（管道），pipeline 中包含了 channel，即通过 pipeline 可以获取到对应通道，管道中维护了很多的处理器

#### runAllTasks

执行给当前 eventLoop 指定的其他任务，主要包含下面三种：

1. 用户自定义的普通任务：ChannelHandlerContext.eventloop().execute(new Runnable() {  xxx })
2. 非当前线程调用的 Channel 方法：如保存了 context 然后在其他地方调用了 context.xxx 方法（给某个终端通知）
3. 用户自定义的定时方法：ChannelHandlerContext.eventloop().execute(new Runnable() {  xxx }, 60, TimeUnit.SECONDS)

### 粘包和拆包

发送端为了将多个发给接收端的包，更有效的发给对方，使用了优化方法（`Nagle` 算法）

将多次间隔较小且数据量小的数据，合并成一个大的数据块，然后进行封包

解决方案：

1. 自定义协议 + 编解码器
2. 指定每次读取的长度



## Docker



## Kubernates



## 设计模式



## 算法

