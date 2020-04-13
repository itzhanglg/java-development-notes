:heavy_check_mark:  : 表示该知识点已学;	:heavy_exclamation_mark:  : 表示学过的知识点中还未涉及到的;    :question:  : 表示学过的知识点中还有疑惑的​

#### 1.并发编程

- Java内存模型(JMM)  :heavy_check_mark:
    - java当中的线程通信和消息传递  
    - 什么是重排序和顺序一致性? Happens-Before? As-If-Serial?  
- 原子操作常用知识讲解  :heavy_check_mark:
    - 基本类型的原子操作比如经典的AtomicBoolean,AtomicInteger,AtomicLong  
    - 数组类型的原子操作比如AtomicIntegerArray,AtomicLongArray,AtomicReferenceArray  :heavy_exclamation_mark:
    - 引用类型的原子操作如AtomicReference,AtomicReferenceFieldUpdater等  :heavy_exclamation_mark:
    - CAS的概念和知识,Compare And Swap 以及它的缺陷
- Volatile和DCL的知识  :heavy_check_mark:
    - DCL的单例模式,什么是DCL?如何来解决DCL的问题  :heavy_exclamation_mark: 
    - Volatile的使用场景和Volatile实现机制,内存语义,内存模型
- Synchronized的概念和分析  :heavy_check_mark:
    - 同步,重量级锁以及Synchronized的原理分析
    - 自旋锁,偏向锁,轻量级锁,重量级锁的概念,使用以及如何来优化他们  :question:
- 并发基础之AQS的深度分析
    - 同步状态的获取和释放,线程阻塞和唤醒
    - AbstractAueuedSynchronize同步器的概念,CLH同步队列是什么?
- 线程池和并发并行
    - Executor,ThreadPoolExecutor,Callable &amp;amp;amp;amp; Future ScheduledExecutorService
    - ThreadLocal, Fork & amp;amp;amp;amp; Join? 什么是并行? 线程池如何保证核心线程不被销毁?
- Lock和并发常用工具类
    - java当中的Lock,ReentrantLock,ReentrantReadWriteLock,Condition
    - java当中的并发工具类CyclicBarrier,CountDownLatch,Semphore
    - java当中的并发集合类ConcurrentHashMap,ConcurrentLinkedQueue

#### 2.源码解析

- 常用集合类源码解析  :heavy_check_mark:
    - JDK7/JDK8中的HashMap源码解析  
    - JDK7/JDK8中的ConcurrentHashMap源码解析  :heavy_exclamation_mark:
    - 红黑树分析与应用详解  :heavy_exclamation_mark:
    - LinkedHashMap,ArrayList等集合源码解析
- 线程池源码
    - CachedThreadPool源码和使用场景解析
    - FixedThreadPool源码和使用场景解析
    - SingleThreadExecutor源码和使用场景解析
    - ScheduleThreadPool源码和使用场景解析
- 并发工具类源码  :heavy_check_mark:
    - Atomic系列源码解析  
    - Locks系列源码解析  :heavy_exclamation_mark:
    - ConcurrentHashMap源码解析  :heavy_exclamation_mark:
    - ForkJoin源码解析  :heavy_exclamation_mark:
    - CountDownLatch,CycliBarrier,Semaphore等并发工具类源码解析  :heavy_exclamation_mark:

#### 3.其它

- JDK新特性  :heavy_check_mark:
    - Lambda表达式详解  
    - Optional,Stream详解  
    - 函数式接口,方法引用详解  
    - 重复注解,扩展注解,并行数组详解  :heavy_exclamation_mark:
    - CompletableFuture详解  :heavy_exclamation_mark:
    - JDK自带HTTP Client工具详解  :heavy_exclamation_mark:
    - JAVA模块化详解  :heavy_exclamation_mark:
    - Switch新语法,JAVA脚本详解  :heavy_exclamation_mark:
- JVM
    - java虚拟机是什么?是如何做到跨平台的?
    - 有哪些字节码指令?字节码指令详解
    - 虚拟机运行时数据区详解
    - 类加载器与类的生命周期详解
    - 有哪些垃圾回收器?各个垃圾回收器工作流程与底层原理详解?
    - JVM执行子系统,类文件结构,类加载机制,字节码执行引擎,字节码编译模式,如何改变字节码编译模式?



