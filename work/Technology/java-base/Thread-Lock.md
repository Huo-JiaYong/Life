## 多线程

多线程能够充分利用 CPU 的运算能力，提高程序的运行速度



## 线程安全

#### 为什么会有线程安全的问题

当多个线程访问同一资源时，可能出现多个线程交替访问 / 修改。从而导致资源不一致

#### 线程安全的定义

> 当多个线程访问一个对象的时候，如果不用考虑这些线程在运行时环境下的调度和交替执行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都可以获取正确的结果，那这个对象就是线程安全的. 

#### 如何实现线程安全

1. 互斥同步

   1. synchronized 关键字

      可以用以修饰方法或代码块

   2. Lock 接口

2. 非阻塞同步

   需要硬件指令完成 如：CAS compare-and-swap

3. 无同步方案

   可重入代码 / 线程本地存储 ThreadLocal

#### 何为同步

在 JAVA 的模式工作模式中，每个线程都有对应的内存缓存，用以减少线程与主内存的频繁通信造成的浪费

而内存缓存就会导致主内存中的值可能不是最新的

那么使用 synchronized 或者 lock 包裹的代码块，就不使用线程的内存缓存。在代码执行完后直接更新主内存就不会出现同步问题

#### Lock

Lock 是在 jdk 1.5 加入的 java.util.concurrent.locks 中加入的新接口，用以实现更加灵活的锁

##### 与 synchronized 的比较

1. synchroinzed 的锁只能由一个线程持有，其他线程进入等待状态。ReentrantReadWriteLock 读写锁能只在写的时候线程独占，读的时候多线程读
2. Lock 可以中断等待
3. synchronized 自动释放锁，而 Lock 必须调用 Lock.unlock() 释放锁
4. synchronized 释放锁后线程自己竞争，有可能导致某个线程长时间都未获得锁。Lock 可以设置为公平锁
5. **synchronized 和 Lock 都是可重入锁**

##### Lock 常见方法

1. lock()  获取锁
2. tryLock() tryLock(Long time) 尝试获取锁返回 true/false 
3. lockInterruptibly() 获取可中断的锁
4. interruptibly() 中断当前线程的等待

常见 Lock 实现

1. ReentrantLock 可重入锁
2. ReentrantReadWriteLock 可重入读写锁
3. new Lock(true) 公平锁

#### ThreadLocal

通过隔离线程变量的方式来实现线程的同步，其实数据是属于线程的。所以可以用以解决线程同步的问题

#### volatile

被 volatile 修饰的变量会直接写入到主内存中去，更新都会直接更新到主内存。所以获取到的值总是最新的



## 阻塞的代价

> JAVA 的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统介入，需要在用户态与核心态之间切换，这种切换会消耗大量的系统资源
>
> 因为用户态与内核态都有各自专用的内存空间，专用的寄存器等，用户态切换至内核态需要传递给许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器值、变量等，以便内核态调用结束后切换回用户态继续工作
>
> 1. 如果线程状态切换是一个高频操作时，这将会消耗很多CPU处理时间；
> 2. 如果对于那些需要同步的简单的代码块，获取锁挂起操作消耗的时间比用户代码执行的时间还要长，这种同步策略显然非常糟糕的。
>
> **synchronized会导致争用不到锁的线程进入阻塞状态，所以说它是java语言中一个重量级的同步操纵，被称为重量级锁，为了缓解上述性能问题，JVM从1.5开始，引入了轻量锁与偏向锁，默认启用了自旋锁，他们都属于乐观锁。**

#### 锁的等级

1. 无锁	**所有对象初始化的状态**
2. 无锁 -> 偏向锁  **当只有一个线程访问对象的时候**
3. 偏向锁 -> 轻量级锁  **当多个线程竞争的时候**
4. 轻量级锁 -> 重量级锁   **当线程多次竞争都未获得锁的时候，并阻塞其他线程**

#### 常见的锁

1. 自旋锁

   当持有锁的线程能够很快的释放锁的话，那么当前等待锁的线程就不需要进入等待状态。只需要欺骗 CPU 当前线程正在执行，当锁被释放的时候就可以直接获取锁

   **因此适用于 执行时间较短 的多线程中**

2. 排队自旋锁

   在自旋锁的基础上加入排队机制，让释放锁时的争抢锁更加公平。如等的太久的线程优先获取

3. 偏向锁

   对象会偏向**第一个**访问的线程，当不存在多个线程争抢锁的时候，第一个线程会减少加锁和解锁的操作。用以提升效率

   当多个线程访问对象时，JVM 会将偏向锁线程挂起并恢复至普通轻量锁

#### CAS 导致的 Cache 一致性流量

![CPU](../images/cpu.jpg)

> Core1和Core2可能会同时把主存中某个位置的值Load到自己的L1 Cache中，当Core1在自己的L1 Cache中修改这个位置的值时，会通过总线，使Core2中L1 Cache对应的值“失效”，而Core2一旦发现自己L1 Cache中的值失效（称为Cache命中缺失）
>
> **则会通过总线从内存中加载该地址最新的值，大家通过总线的来回通信称为“Cache一致性流量”**，
>
> 因为总线被设计为固定的“通信能力”，如果Cache一致性流量过大，总线将成为瓶颈。
>
> 而当Core1和Core2中的值再次一致时，称为“Cache一致性”，
>
> **从这个层面来说，锁设计的终极目标便是减少Cache一致性流量。**

而 CAS 恰好会导致 Cache 一致性流量，

如果有很多线程都共享同一个对象，当某个 Core CAS 成功时必然会引起总线风暴，这就是所谓的本地延迟，

本质上偏向锁就是为了消除 CAS，降低Cache一致性流量



## 常见 API

Executor	Executor ExecutorServices ThreadPoolExecutor

Collections	CopyOnWriteArrayList LinkedBlockingQueue ConcurrentHashMap

Atomic		AtomicBoolean AtomicInteger AtomicIntegerArray

Lock		Lock ReentrantLock ReentrantReadWriteLock

Tools		Semaphore Executors

## 参考链接

1. [Synchronized 的原理及自旋锁，偏向锁，轻量级锁，重量级锁的区别](http://blog.csdn.net/kirito_j/article/details/79201213) ★★★
2. [锁的等级](https://www.cnblogs.com/wewill/p/8058292.html) ★★★
3. [volatile 揭秘](https://www.cnblogs.com/tangyanbo/p/6538488.html) ★★★
4. [Lock 和 synchronized 比较详解](https://www.cnblogs.com/handsomeye/p/5999362.html) ★★
5. [JAVA 线程安全的实现方式](http://blog.csdn.net/fcc7619666/article/details/52022025)