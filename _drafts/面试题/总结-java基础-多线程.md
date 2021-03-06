面试-java基础多线程
===
# 多线程

## 你们业务中的多线程的运用场景，你们是怎么运用的
- 异步发消息，我们系统中大量运用了消息队列，消息服务器用的rabbitmq，发送消息我们是在一个新的线程中发的消息，目前是新启了个线程去处理，应该是改造为从线程池取线程，发送消息，这样更好，可以减少一些线程创建的开销，线程的创建也需要些内存开销。
- 定时任务，数据分片之后，交由线程池去处理，在我们的分布式定时任务开发出来以前，我们实现定时任务是一次性取出大量数据，然后自己按照分片的思想，把数据切割成多个集合，然后分给线程池中的多个线程，并行去处理，加快处理速度。现在我们开发了分布式定时任务，对数据分片后，我们把任务交给了ignite 集群去并行处理。
- 上传excel 异步处理耗时线程池，excel 提交后，新启一个线程异步的去处理业务，在原有线程给出委婉提示，让用户稍后在某处查看处理进度，对于业务处理的结果，进度等信息记录在单独的表中，给用户可反馈的信息。
- 批量生成兑换码，一次性生成几百万的兑换码，任务提交后，异步的去生成
- 业务的异步处理，用@sync 注解
- tomcat 的多线程，tomcat 的业务线程池
- dubbo 多线程，dubbo 中的netty 的业务线程池
- 异步日志
- 把请求入队列，然后异步的去处理，不同的业务用不同的线程

##   在Java中wait和seelp方法的不同；

答：最大的不同是在等待时wait 会释放锁，而sleep 一直持有锁。wait 通常被用于线程间交互，sleep 通常被用于暂停执行。

##  讲一下线程状态转移图
## 多线程 **wait（）sleep（）

答：聊到多线程，大致先问了多线程的实现方式，这个问题不用多说，很简单，java提供了两种方式，**一个是继承Thread类，另一个是实现Runnable接口**，由于java不支持多继承，所以在多继承的时候，我们得优先选用 实现 Runnable接口，因为我们可以通过实现接口的办法，间接的实现多继承！【问了实现方式之后面试官又问了这两种方法的使用】，其次问到了wait（）和sleep（）的区别，这两个方法区别在于：**sleep（）是Thread类的，而wait（）是Object类的，**sleep是睡眠，指定时间后线程会继续执行，**不放弃对cpu资源的占用（即不放弃对象锁）**，相当于暂停指定t，**wait（）是等待，需要唤醒，它会释放对cpu资源的占用（即会放弃对象锁）**，调用**notify（）和notifyAll（）**唤醒。

## 线程模型了解吗？哪些是线程私有的？


##   线程和进程的概念、并行和并发的概念


##   进程间通信的方式

##   说说 CountDownLatch、CyclicBarrier 原理和区别

##   说说 Semaphore 原理

##   说说 Exchanger 原理

##   ThreadLocal 原理分析，ThreadLocal为什么会出现OOM，出现的深层次原理


##   线程的生命周期，状态是如何转移的

##   AtomicInteger底层实现原理； 

##   synchronized与ReentraLock哪个是公平锁；

##   CAS机制会出现什么问题； 
 
##   线程状态以及API怎么操作会发生这种转换；

##   常用的避免死锁方法；
##   如何调试多线程的程序；

##   一个线程连着调用start两次会出现什么情况？（由于状态只有就绪、阻塞、执行，状态是无法由执行转化为执行的，所以会报不合法的状态！）
##   wait方法能不能被重写？（wait是final类型的，不可以被重写，不仅如此，notify和notifyall都是final类型的），wait能不能被中断；

##   多线程是解决什么问题的？线程池解决什么问题？


## 进程和线程的区别** 
  从调度、并发性、拥有的资源和系统开销四个方面回答的。

## 创建线程有哪几种方式，你最喜欢哪种，为什么

##   启动一个线程是调用 run() 还是 start() 方法？start() 和 run() 方法有什么区别

##   调用start()方法时会执行run()方法，为什么不能直接调用run()方法

##   sleep() 方法和对象的 wait() 方法都可以让线程暂停执行，它们有什么区别

##  yield方法有什么作用？sleep() 方法和 yield() 方法有什么区别

## Java 中如何停止一个线程

##   stop() 和 suspend() 方法为何不推荐使用

##  如何在两个线程间共享数据

##   如何强制启动一个线程

##   如何让正在运行的线程暂停一段时间

##   什么是线程组，为什么在Java中不推荐使用

##  你是如何调用 wait（方法的）？使用 if 块还是循环？为什么


##   有哪些不同的线程生命周期

##  线程状态，BLOCKED 和 WAITING 有什么区别

##  画一个线程的生命周期状态图

## ThreadLocal 用途是什么，原理是什么，用的时候要注意什么

##   Java中用到的线程调度算法是什么

##  什么是多线程中的上下文切换

##  你对线程优先级的理解是什么

##  什么是线程调度器 (Thread Scheduler) 和时间分片 (Time Slicing)
## 如何停止运行一个线程 
interrupt



# 线程池

##   线程池的实现？四种线程池？重要参数及原理？任务拒接策略有哪几种？


##  Java线程池的几个参数的意义和实现机制；

##   Java线程池使用无界任务队列和有界任务队列的优劣对比；
##   线程池，如何设计的，里面的参数有多少种，里面的工作队列和线程队列是怎样的结构，如果给你，怎样设计线程池？

## 线程池有哪些？如何使用的？

## 如何初始化线程池？有哪些线程池？饱和策略怎么设置的？


##   线程池，如何根据CPU的核数来设计线程大小，如果是计算机密集型的呢，如果是IO密集型的呢？

##   线程池内的线程如果全部忙，提交⼀个新的任务，会发⽣什么？队列全部塞满了之后，还是忙，再提交会发⽣什么？

##   ExecutorService你一般是怎么⽤的？是每个Service放一个还是个项目放一个？有什么好处？

  可参考：《[Java多线程编程核心技术](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247484881&idx=2&sn=b0ecf85cd7c9e543c84e7a9859c20a26&chksm=e9c5fc60deb27576a6a9c453dabc585f43d9f29fd8a8f37ed0e7cc2f012c86b23fbd21763a39&scene=21#wechat_redirect)》
## 创建线程的方式中用线程池的话你用哪个实例？

## ThreadPoolExecutor 的构造器中的常用参数有哪些？
## 线程池的构造类的方法的5个参数的具体意义？

## 线程池的参数，每个参数解释一遍，然后面试官设置了每个参数，给了是个线程，让描述出完整的线程池执行的流程

## 让我设计一个线程池
  因为我简历中有写到我对多线程、并发这一块理解比较好。所以他老问这方面的题。这个问题因为我之前看过ThreadPoolExecutor的源代码，所以我就仿照那个类的设计思路来想的，详细讲了一下核心池、创建线程可以用工厂方法模式来进行设计、线程池状态、阻塞队列、拒绝策略这几个方面。设计的还算比较周全。 

## 核心线程是什么概念？

## ****线程池的构造类的方法的5个参数的具体意义？** 

## ****单机上一个线程池正在处理服务如果忽然断电该怎么办？（正在处理和阻塞队列里的请求怎么处理）？

## java线程池中基于缓存和基于定长的两种线程池，当请求太多时分别是如何处理的？定长的事用的队列，如果队列也满了呢？交换进磁盘？基于缓存的线程池解决方法呢？

## 线程池原理，单核操作系统是否应该使用多线程，为什么

##   Java线程池的核心属性以及处理流程；

## Java中的线程池共有几种？

Java四种线程池

第一种：newCachedThreadPool

　　创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。

第二种：newFixedThreadPool

　　创建一个指定工作线程数量的线程池

第三种：newScheduledThreadPool

创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。

第四种：newSingleThreadExecutor

　　创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。

分析线程池的实现原理和线程的调度过程

## 线程池的工作原理，几个重要参数，然后给了具体几个参数分析线程池会怎么做，最后问阻塞队列的作用是什么？

## 为什么会出现死锁？你来写一个死锁，如何改一改这个代码避免死锁？（我把synchronized改成重入锁的tryLock）
ThreadPool



##   提交任务时，线程池队列已满时会发会生什么

##   newCache 和 newFixed 有什么区别？简述原理。构造函数的各个参数的含义是什么，比如 coreSize, maxsize 等


##  线程池的关闭方式有几种，各自的区别是什么

##  线程池中submit() 和 execute()方法有什么区别？
。
## 线程饱和时拒绝后的线程用什么东西处理。
## 线程池的使用时的注意事项

##  什么是线程池，如何使用?

 答：线程池就是事先将多个线程对象放到一个容器中，当使用的时候就不用new 线程而是直接去池中拿线程即可，节
    省了开辟子线程的时间，提高的代码执行效率

# 线程安全

## 讲一讲AtomicInteger，为什么要用CAS而不是synchronized？

## ****如何保证共享变量修改时的原子性？**

##  synchronized 和volatile 关键字的作用；
答：1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。2）禁止进行指令重排序。

   volatile 本质是在告诉jvm 当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized 则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
    1.volatile 仅能使用在变量级别；synchronized 则可以使用在变量、方法、和类级别的
    2.volatile 仅能实现变量的修改可见性，并不能保证原子性；synchronized 则可以保证变量的修改可见性和原子性
    3.volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞。
    4.volatile 标记的变量不会被编译器优化；synchronized 标记的变量可以被编译器优化
# 线程同步有什么作用？都有哪些方式？他们的区别是什么？

## Java内存模型，volatile和i++的线程安全

## java同步方式，有什么同步器 ， 使用过什么同步器 

## 怎样让一个线程放弃锁，放弃锁后什么时候能再次获取锁 


## volitile 怎么保证可见性。 怎么保证有序性

## synchronized 怎么使用，你写的代码里哪里用了。不算你刚才写的双检查锁的单例模。这个单例双检查锁为什么要这样写##   synchronized关键字锁住的是什么东西？在字节码中是怎么表示的？在内存中的对象上表现为什么？

##   Java中有哪些同步方案（重量级锁、显式锁、并发容器、并发同步器、CAS、volatile、AQS等）

##  乐观悲观锁的设计，如何保证原子性，解决的问题；

##   sync原理详细，sync内抛异常会怎样，死锁吗？还是释放掉？怎么排查死锁？死锁会怎样？有没有什么更好的替代方案？


##   AQS原理，ReentranLock源码，设计原理，整体过程。

##  说说线程安全问题，什么是线程安全，如何保证线程安全

##   重入锁的概念，重入锁为什么可以防止死锁


##   volatile 实现原理（禁止指令重排、刷新内存）

##   synchronized 实现原理（对象监视器）

##   synchronized 与 lock 的区别

##   AQS同步队列

##   CAS无锁的概念、乐观锁和悲观锁

##   乐观锁的业务场景及实现方式

##   偏向锁、轻量级锁、重量级锁、自旋锁的概念

*   可参考：《[Java多线程编程核心技术](http://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247484881&idx=2&sn=b0ecf85cd7c9e543c84e7a9859c20a26&chksm=e9c5fc60deb27576a6a9c453dabc585f43d9f29fd8a8f37ed0e7cc2f012c86b23fbd21763a39&scene=21#wechat_redirect)》

## 数据一致性如何保证；Synchronized关键字，类锁，方法锁，重入锁；

## Volatile有什么用，能不能实现原子性，为什么？

## 乐观锁和悲观锁是什么，分别应用于什么情况？

## 什么状况下不适合用CAS 

## Synchronized与Lock与volatile有什么区别 


##  Atomic包的实现原理是什么； 

##   CAS又是怎么保证原子性的； 
## Synchronize关键字为什么jdk1.5后效率提高了

##  数据一致性如何保证；Synchronized关键字，类锁，方法锁，重入锁；

volatile和synchronized简介：

在Java中,为了保证多线程读写数据时保证数据的一致性,可以采用两种方式：

　 1）使用synchronized关键字

　 2）使用volatile关键字：用一句话概括volatile,它能够使变量在值发生改变时能尽快地让其他线程知道。

两者的区别：

1）volatile本质是在告诉jvm当前变量在寄存器中的值是不确定的,需要从主存中读取,synchronized则是锁定当前变量,只有当前线程可以访问该变量,其他线程被阻塞住.

2）volatile仅能使用在变量级别,synchronized则可以使用在变量,方法.

3）volatile仅能实现变量的修改可见性,而synchronized则可以保证变量的修改可见性和原子性.
4）volatile不会造成线程的阻塞,而synchronized可能会造成线程的阻塞.
9、synchronized和lock的区别？

##   请说出你所知的线程同步的方法

##  synchronized 的原理是什么

##  synchronized 和 ReentrantLock 有什么不同

##  什么场景下可以使用 volatile 替换 synchronized
##   同步块内的线程抛出异常会发生什么

##   当一个线程进入一个对象的 synchronized 方法A 之后，其它线程是否可进入此对象的 synchronized 方法B

##  使用 synchronized 修饰静态方法和非静态方法有什么区别

##  如何从给定集合那里创建一个 synchronized 的集合

锁

##  Java Concurrency API 中 的 Lock 接口是什么？对比同步它有什么优势

##  Lock 与 Synchronized 的区别？Lock 接口比 synchronized 块的优势是什么

##  ReadWriteLock是什么？

##  锁机制有什么用

## 什么是乐观锁（Optimistic Locking）？如何实现乐观锁？如何避免ABA问题
##  解释以下名词：重排序，自旋锁，偏向锁，轻量级锁，可重入锁，公平锁，非公平锁，乐观锁，悲观锁   

## 嗯，刚刚谈到了锁，一下面试官就扯出了锁，看你简历上ssh mybatis都熟悉，那你知道悲观锁和乐观锁吧？答知道，然后讲了下两个的区别以及应用场景

##  Atomic原子性怎么保证的， CAS锁
## 你提到Lock，知道哪些相应的锁？
   读写锁，可重入锁
## 知道AQS吗，他的实现是怎样的？AQS可重入吗？
知道，读写锁，可重入锁都是通过AQS实现的，AQS维护一个链表，并主要提供tryacquire和tryrelease方法。默认为非公平锁，此时当一个线程需要请求锁时...

## AQS如何实现可重入

维护一个int类型的status作为计数器，同一个线程acquire就加1，release就减1.到0就释放锁。读写说则是将status分为两部分使用。内部维护一个shift变量做位运算的变化。。。（AQS可以看占小狼的blog或者并发编程的艺术）


## 指令重排序指什么？指令重排序的好处是什么？如何防止指令重排序。
编译器重排序，cpu重排序，内存重排序。好处是流水线技术，提高并发性能等。通过禁止编译器优化，以及汇编使用Lock信号，java中的cpp加入volatile等防止。
## 内存可见性具体指什么？volatile通过什么机制防止

讲了下JMM，以及计组原理中的三级cache，buffer，缓存行等。

顺便扯了下c语言的volatile只保证防止编译器优化以及内存可见性的语义，而不能保证顺序性。然后是C11的acquire，release语义 接着回归java，扯了下内存屏障的实现与作用。（并发编程的艺术）然后扯了下#LOCK信号，包括总线锁，mesi的缓存一致性等。最后是先行发生的语义（语无伦次，不过基本点都讲到了）

## synchronized内部分为几种锁，他们的使用场景是什么
偏向锁，轻量级锁，重量级锁（又有自旋锁等）,然后详细讲了实现和使用场景（周志明的书和并发编程的艺术都有讲，此处省略）。

##  `synchronized`和`lock`有什么区别？
##   `volatile`的作用，如果`volatile`修饰的对象经过了大量的写，会出现什么问题？
## 重入锁、对象锁、类锁的关系

对象锁(方法锁)，是针对一个对象的，它只在该对象的某个内存位置声明一个标识该对象是否拥有锁，所有它只会锁住当前的对象，一般一个对象锁是对一 个非静态成员变量进行synchronized修饰，或者对一个非静态成员方法进行synchronized进行修饰，对于对象锁，不同对象访问同一个被 synchronized修饰的方法的时候不会阻塞

类锁是锁住整个类，当有多个线程来声明这个类的对象时候将会被阻塞，直到拥有这个类锁的对象呗销毁或者主动释放了类锁，这个时候在被阻塞的线程被挑选出一个占有该类锁，声明该类的对象。其他线程继续被阻塞住。

## 什么状况下不适合用CAS

## 怎样让一个线程放弃锁，放弃锁后什么时候能再次获取锁
## 偏向锁是什么？自旋锁和偏向所讲一个


# 其他
##   ThreadPool用法与优势


##   怎么认为一个类是线程安全？线程安全的定义是什么？Java有多少个关键字进行同步？为什么这样设计？（聊了一大堆，一堆为什么）；

##   两个线程设计题。记得一个是：t1,t2,t3，让t1，t2执行完才执行t3，原生实现。

## 说下多线程，我们什么时候需要分析线程数，怎么分析，分析什么因素;

##   你有遇到过临界区问题吗？有遇到过吗？你在项目遇到这个问题是怎样解决的？

##   多个线程同时读写，读线程的数量远远⼤于写线程，你认为应该如何解决并发的问题？你会选择加什么样的锁？


##   wait/notify/notifyAll⽅法需不需要被包含在synchronized块中？这是为什么？

##   产生死锁的四个条件（互斥、请求与保持、不剥夺、循环等待）

##   如何检查死锁（通过jConsole检查死锁）


##   常见的原子操作类

##   什么是ABA问题，出现ABA问题JDK是如何解决的


##   Java 8并法包下常见的并发类


## 进程间共享内存的方式有哪些？（8种）

## 线程安全的集合有哪些？

## 何时用单线程，何时用多线程？


## 如果两个线程都使用一个ByteBuf 怎么保证它的安全，具体说一下代码实现



## java线程之间的通信 


## AtomicInteger原理 

## ThreadLocal有什么了解

## 自己实现线程安全类
## 同步的方法；多进程开发以及多进程应用场景；

## 使用无界阻塞队列会出现什么问题？
## 如何保证共享变量修改时的原子性？

## 如何控制某个方法允许并发访问线程的个数；
## i++是否线程安全
 
## 多线程：怎么实现线程安全，各个实现方法有什么区别，volatile关键字的使用，可重入锁的理解，Synchronized是不是可重入锁** 
  这里我就主要讲了Synchronized关键字，还有并发包下面的一些锁，以及各自的优缺点和区别。volatile关键字我主要从可见性、原子性和禁止JVM指令重排序三个方面讲的，再讲了一下我在多线程的单例模式double-check中用到volatile关键字禁止JVM指令重排优化。 

##  Java中的threadlocal是怎么用的? threadlocal中的内部实现是怎么样的? 哪种引用?
## java中的"final"关键字在多线程的语义中，有什么含义


## ThreadLocal 了解吗，什么时候用，讲讲他的内部实现。


##  多线程：如何实现一个定时调度和循环调度的工具类。但提交任务处理不过来的时候，拒绝机制应该如何处理；线程池默认有哪几种拒绝机制； 

##  多线程：如何实现一个ThreadLocal； 

##  说说你了解的一个线程安全队列； 


## 线程池的使用时的注意事项

##   服务器只提供数据接收接口，在多线程或多进程条件下，如何保证数据的有序到达；

##   ThreadLocal原理，实现及如何保证Local属性；

##   如何控制某个方法允许并发访问线程的个数；

##   同步的方法；多进程开发以及多进程应用场景；


##   Java的并发、多线程、线程模型；


## Java并发包当中的类，它们都有哪些作用，以及它们的实现原理，这些类就是java.concurrent包下面的。与上面一样，咱们也简单的模拟一个并发包的连环炮。

比如面试官可能会先问你，如果想实现所有的线程一起等待某个事件的发生，当某个事件发生时，所有线程一起开始往下执行的话，有什么好的办法吗？

这个时候你可能会说可以用栅栏（Java的并发包中的CyclicBarrier），那么面试官就会继续问你，你知道它的实现原理吗？

如果你继续回答的话，面试官可能会继续问你，你还知道其它的实现方式吗？

如果你还能说出很多种实现方式的话，那么继续问你，你觉得这些方式里哪个方式更好？

如果你说出来某一个方式比较好的话，面试官依然可以继续问你，那如果让你来写的话，你觉得还有比它更好的实现方式吗？

如果你这个时候依然可以说出来你自己更好的实现方式，那么面试官肯定还会揪着这个继续问你。

##  有T1，T2，T3三个线程，怎么确保它们按顺序执行？怎样保证T2在T1执行完后执行，T3在T2执行完后执行


## 写代码：解决生产者消费者问题

用了Semaphore手写的代码 大概花了一些时间 跟面试官讲了下实现。。。

## 多线程用过吧？写过一些demo，那你说说多线程之间的通信，回答用wait sleep notify notifyAll配合使用 然后就问 wait和sleep一样吗？回答不一样，然后巴拉巴拉谈了下，关于对象锁的释放，是否需要唤醒。。。

## threadlocal使用时注意的问题（ThreadLocal和Synchonized都用于解决多线程并发访问。但是ThreadLocal与synchronized有本质的区别。synchronized是利用锁的机制，使变量或代码块在某一时该只能被一个线程访问。而ThreadLocal为每一个线程都提供了变量的副本，使得每个线程在某一时间访问到的并不是同一个对象，这样就隔离了多个线程对数据的数据共享。而Synchronized却正好相反，它用于在多个线程间通信时能够获得数据共享）


## 想让所有线程都等到一个时刻同时执行有哪些方法（介绍了下CountDownLatch和CyclicBarrier）

## 假如有两个线程，一个线程A，一个线程B都会访问一个加锁方法，可能存在并发情况，但是线程B访问频繁，线程A访问次数很少，问如何优化。(然后面试官说有了解过重度锁和轻度锁吗)


