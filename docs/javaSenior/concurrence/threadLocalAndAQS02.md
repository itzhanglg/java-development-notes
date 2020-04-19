AQS是`AbstractQueuedSynchronizer`的简称,中文涵义是 **抽象队列同步器**,可以理解为:

- 抽象: 即抽象类,只有方法的声明,没有方法的实现
- 队列: 基于队列(FIFO)的方式实现
- 同步: 实现同步的功能

**AQS是一个用来构建锁和同步器的框架**,使用AQS能简单高效的构造出广泛的同步器,如 ReentrantLock, Semaphore,  ReentrantReadWriteLock, FutureTask 等都是基于AQS的,我们只要**子类实现它的几个 protected 方法**就可以轻松的构造出自己需要的同步器.









