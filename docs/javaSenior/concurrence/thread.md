### 一:基本概念

#### 1.程序、进程、线程概念

**程序(program)**是为完成特定任务、用某种语言编写的一组指令的集合。即指一段静态的代码，静态对象。

**进程(process)**是代码在数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

**线程(thread)**是进程的一个执行路径，一个进程中至少有一个线程，进程中的多个线程共享进程的堆和方法区资源，但每个线程都有自己的程序计数器、虚拟机栈和本地方法栈。线程是CPU分配的基本单位。

在windows中查看任务管理器的方式，可以看到window当前运行的进程(.exe文件的运行)：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191231192832930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

小结：

- **程序(program)是为完成指定任务,用某种语言编写的一组指令集合.指一段静态的代码**
- **进程(process)是正在运行的一个程序.是一个动态的过程**
- **线程(thread)是一个程序内部的一条执行路径**

#### 2.线程与进程的关系与区别

#### 3.单核CPU,多核CPU的理解

1.  单核CPU,实质是一种假的多线程,在一个时间单元内,也只能执行一个线程的任务.
2.  多核CPU是多线程,每个核单独执行一个线程的任务.
3.  一个java应用程序java.exe,至少有三个线程:**main()主线程,gc()垃圾回收线程,异常处理线程.**

#### 4.并行与并发

1.  **并行:多个CPU同时执行多个任务.(多个人同时做不同的事)**
2.  **并发:一个CPU(采用时间片)同时执行多个任务.(多个人做同一件事)**

#### 5.多线程优点

1.  提高应用程序的响应.
2.  提高CPU的利用率
3.  改善程序结构

#### 6.何时需要多线程

1.  程序需要同时执行两个或多个任务.
2.  程序需要实现一些需要等待的任务.
3.  需要一些后台运行的程序时.

### 二:线程的创建和使用

JVM允许程序运行多个线程,通过**java.lang.Thread**类体现.JDK1.5之前有两种方式创建线程: **继承Thread类 和 实现Runnable接口**.

#### 1.继承Thread类

1.  Thread类特性

    1.  **每个线程都是通过某个特定Thread对象的run()方法完成操作的**,常把run()方法的主体称为**线程体**.
    2.  **通过Thread对象的start()方法启动线程**,不是直接调用run()方法

2.  Thread类构造器

    1.  **Thread(): 创建Thread对象**
    2.  Thread(String threadname): 创建线程并指定线程实例名
    3.  **Thread(Runnable target): 创出线程的目标对象,它实现了Runnable接口的run()**
    4.  Thread(Runnable target, String name): 创建新的Thread对象

3.  流程

    1.  **继承Thread**
    2.  **重写Thread中的run()**
    3.  **创建线程对象**
    4.  **调用对象的start(): 启动线程,调用run()**

4.  示例

    ```java
    //1. 创建一个继承于Thread类的子类
    class MyThread extends Thread {
        public MyThread(){}
        public MyThread(String name){ super(name); }

        //2. 重写Thread类的run()
        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                if (i % 2 == 0) {
                    /*public Thread() {
                        init(null, null, "Thread-" + nextThreadNum(), 0);
                    }*/
                    System.out.println(Thread.currentThread().getName() + ":" + i);
                }
            }
        }
    }

    public class ThreadTest {
        public static void main(String[] args) {
            //3. 创建Thread类的子类的对象
    //        MyThread t1 = new MyThread();
            MyThread t1 = new MyThread("分线程");
            //设置线程名称
    //        t1.setName("线程一");

            //4.通过此对象调用start():①启动当前线程 ② 调用当前线程的run()
            t1.start();

            //问题一：我们不能通过直接调用run()的方式启动线程。
    //        t1.run();
            //问题二：再启动一个线程，遍历100以内的偶数。不可以还让已经start()的线程去执行。
            /*if (threadStatus != 0)
                throw new IllegalThreadStateException();*/
    //        t1.start();       // java.lang.IllegalThreadStateException

            //我们需要重新创建一个线程的对象
            MyThread t2 = new MyThread();
            t2.start();
            Thread.currentThread().setName("主线程");

            //如下操作仍然是在main线程中执行的。
            for (int i = 0; i < 100; i++) {
                if (i % 2 == 0) {
                    System.out.println(Thread.currentThread().getName() + ":" + i);
                }
            }
        }
    }
    ```

5.  注意点

    1.  **对象调用run()方法,没有启动多线程模式**,
    2.  run()方法由JVM掉用,什么时候,执行过程都由CPU调度决定.
    3.  **启动多线程,必须调用start()方法**.
    4.  **一个线程对象不能重复调用start()方法,否则发生 "线程状态不符" 异常**

#### 2.实现Runnable接口

1.  流程

    1.  **实现Runnable接口.**
    2.  **重写Runnable接口中的run()方法.**
    3.  **通过Thread(Runnable target)构造器创建线程对象.**
    4.  **调用start()方法: 开启线程,调用Runnable子类接口的run()方法.**

2.  示例

    ```java
    //1. 创建一个实现了Runnable接口的类
    class MyThread2 implements Runnable{
        //2. 实现类去实现Runnable中的抽象方法：run()
        @Override
        public void run() {
            for (int i = 0; i <= 100 ; i++) {
                if (i % 2 == 0){
                    System.out.println(Thread.currentThread().getName()+":"+i);
                }
            }
        }
    }

    public class ThreadTest2 {
        public static void main(String[] args) {
            //3. 创建实现类的对象
            MyThread2 r1 = new MyThread2();
            //4. 将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
            Thread t1 = new Thread(r1);
            t1.setName("线程1");
            //5. 通过Thread类的对象调用start():① 启动线程 ②调用当前线程的run()
            // -->调用了Runnable类型的target的run()
            t1.start();

            //再启动一个线程，遍历100以内的偶数
            Thread t2 = new Thread(r1);
            t2.setName("线程2");
            t2.start();
        }
    }
    ```

#### 3.两种方式的联系与区别

`public class Thread extends Object implements Runnable`

1.  联系: **都需要重写run()方法,将线程要执行的逻辑都声明在run()中**
2.  实现Runnable接口方式的好处
    1.  **避免了单继承的局限性**
    2.  **多个线程可以共享同一个接口实现类的对象,适合处理多个线程有共享数据的情况**

### 三:Thread类的相关方法

1.  **void start(): 启动当前线程,并执行当前线程的run()方法**
2.  **run(): 被CPU调度时执行的操作,将创建的线程要执行的操作声明在此方法中**
3.  **String getName(): 返回线程的名称**
4.  **void setName(String name): 设置线程的名称**
5.  **static Thread currentThread(): 返回执行当前代码的线程. 在Thread子类中指this, 通常用于主线程和Runnable实现类.**
6.  **static void yield(): 释放当前cpu的执行权**
7.  **join(): 在线程a中调用线程b的join(),此时线程a就进入阻塞状态，直到线程b完全执行完以后，线程a才结束阻塞状态**
8.  **static void sleep(long millitime): 让当前线程“睡眠”指定的millitime毫秒。在指定的millitime毫秒时间内，当前线程是阻塞状态**
9.  **boolean isAlive(): 判断当前线程是否存活**

### 四:线程的优先级及分类

1.  调用策略: **时间片**(单元时间内做任务切换) 或 **抢占式**(高优先级的线程抢占CPU)
2.  调度方法: 同优先级线程组成先进先出队列(先到先服务),使用时间片策略; 对高优先级,使用抢占式策略
3.  线程优先级
    1.  **MAX_PRIORITY: 10;  MIN_PRIORITY: 1;  NORM_PRIORITY: 5;(默认优先级)**
    2.  **getPriority(): 返回线程等级值;  setPriority(int newPriority): 设置线程等级值**
    3.  **说明: 线程创建时继承父线程的优先级; 低优先级只是获得调度的概率低,并不一定在高优先级线程之后才被调用**
4.  线程分类
    1.  分为**守护线程 和 用户线程**.
    2.  守护线程是用来服务用户线程的,在**start()方法前调用thread.setDaemon(true)可以把一个用户线程变为一个守护线程.**
    3.  java垃圾回收就是一个守护线程.**若JVM中都是守护线程,当前JVM将退出**.

示例:

```java
class HelloThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if(i % 2 == 0){

//                try {
//                    sleep(10);
//                } catch (InterruptedException e) {
//                    e.printStackTrace();
//                }

                System.out.println(Thread.currentThread().getName() + ":" + Thread.currentThread().getPriority() + ":" + i);
            }

//            if(i % 20 == 0){
//                yield();
//            }
        }
    }

    public HelloThread(String name){
        super(name);
    }
}


public class ThreadMethodTest {
    public static void main(String[] args) {
        HelloThread h1 = new HelloThread("Thread：1");

//        h1.setName("线程一");
        //设置分线程的优先级
        h1.setPriority(Thread.MAX_PRIORITY);
        h1.start();

        //给主线程命名
        Thread.currentThread().setName("主线程");
        Thread.currentThread().setPriority(Thread.MIN_PRIORITY);

        for (int i = 0; i < 100; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + ":" + Thread.currentThread().getPriority() + ":" + i);
            }

//            if(i == 20){
//                try {
//                    h1.join();
//                } catch (InterruptedException e) {
//                    e.printStackTrace();
//                }
//            }
        }

//        System.out.println(h1.isAlive());
    }
}
```

### 五:示例

```java
/**
 * 示例：创建两个分线程，其中一个线程遍历100以内的偶数，另一个线程遍历100以内的奇数
 *
 * @author zlg
 * @create 2019-10-25 1:56
 */
public class ThreadDemo {
    public static void main(String[] args) {
//        MyThread1 m1 = new MyThread1();
//        MyThread3 m2 = new MyThread3();
//        m1.start();
//        m3.start();

      	// 方式二:采用匿名内部类
        //创建Thread类的匿名子类的方式:遍历偶数
        new Thread(){
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    if(i % 2 == 0){
                        System.out.println(Thread.currentThread().getName() + ":" + i);
                    }
                }
            }
        }.start();

		//创建Thread类的匿名子类的方式:遍历奇数
        new Thread(){
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    if(i % 2 != 0){
                        System.out.println(Thread.currentThread().getName() + ":" + i);
                    }
                }
            }
        }.start();

    }
}

//方式一:采用两个类分别执行不同任务
// 遍历100以内偶数
class MyThread1 extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
    }
}

// 遍历100以内奇数
class MyThread3 extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if(i % 2 != 0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
    }
}
```



### 六:线程的生命周期

#### 1.Thread.State类

Thread.State类定义了线程的几种状态,在一个完整的生命周期中通常要经历五种状态:

1.  **新建**: 当一个Thread类或其子类的对象被声明并创建时
2.  **就绪**: start()方法后,将进入线程队列等待CPU时间片
3.  **运行**: 当就绪的线程被调度并获得CPU资源时,便进入运行状态
4.  **阻塞**: 被人为挂起或执行输入输出操作时,让出CPU并临时终止自己的执行,进入阻塞状态
5.  **死亡**: 线程完成全部工作或被提前强制性终止或出现异常导致结束

#### 2.线程状态转换图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124165624626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

### 七:线程的同步

#### 1.多线程的安全问题

1.  问题原有: 当多条语句在操作同一个线程共享数据时,一个线程对多条语句只执行了一部分,另一个线程就参与进来执行,导致共享数据的错误.
2.  解决办法: 对多条操作共享数据的语句,只能让一个线程都执行完,在执行过程中,其他线程不可以参与执行.

#### 2.三种解决方法

##### 2.1.同步代码块

```java
synchronized(对象){
  // 需要被同步的代码;
}
```

说明：
1. 操作共享数据的代码，即为需要被同步的代码。  -->不能包含代码多了，也不能包含代码少了。
2. 共享数据：多个线程共同操作的变量。比如：ticket就是共享数据。
3. 同步监视器，俗称：锁。任何一个类的对象，都可以充当锁。
         	要求：多个线程必须要共用同一把锁。
     ​    	
     补充：
- 在继承Thread类创建多线程的方式中，慎用this充当同步监视器，考虑使用当前类充当同步监视器。
- 在实现Runnable接口创建多线程的方式中，我们可以考虑使用this充当同步监视器。



##### 2.2.同步方法

```java
public synchronized void add (String id){
  ......
}
```

如果操作共享数据的代码完整的声明在一个方法中，我们不妨将此方法声明同步的。
1. 同步方法仍然涉及到同步监视器，只是不需要我们显式的声明。
2. 非静态的同步方法，同步监视器是：this
         静态的同步方法，同步监视器是：当前类本身

同步的方式，解决了线程的安全问题。---好处
操作同步代码时，只能有一个线程参与，其他线程等待。相当于是一个单线程的过程，效率低。 ---局限性

##### 2.3.示例

```java
/**
 * 示例:创建三个窗口卖票，总票数为100张.采用继承方式
 * 1.问题：卖票过程中，出现了重票、错票 -->出现了线程的安全问题
 * 2.问题出现的原因：当某个线程操作车票的过程中，尚未操作完成时，其他线程参与进来，也操作车票。
 * 3.如何解决：当一个线程a在操作ticket的时候，其他线程不能参与进来。直到线程a操作完ticket时，其他
 *            线程才可以开始操作ticket。这种情况即使线程a出现了阻塞，也不能被改变。
 */
 
//方式一:采用继承类的方式
class Window1 extends Thread{
    // 共享资源需要加上static
    private static int ticket = 100;

    @Override
    public void run() {
        while (true){
            //同步方法
//            handleTicket();

            // 同步代码块
            // 不能使用this,this代表w1,w2,w3
            synchronized (Window1.class) {   //Class clazz = Window1.class,Window1.class只会加载一次
                if (ticket > 0){
                    try {
                        sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(getName()+":"+ticket);
                    ticket--;
                }else {
                    break;
                }
            }
        }
    }

    // 方式一
    /*private void handleTicket() {
        // 同步代码块
        synchronized (Window.class) {   //Class clazz = Window.class
            if (ticket > 0){
                System.out.println(getName()+":"+ticket);
                ticket--;
            }
        }
    }*/
    // 方式二
    // 同步方法: 需要加上static,保证是同一个对象锁
    // 方法不加static时,同步监视器：w1,w2,w3
    private static synchronized void handleTicket() {   // 同步监视器：Window.class
        if (ticket > 0){
            try {
                //sleep()方法属于静态方法,与对象无关
                sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // static方法内容不能使用this,super关键字
            System.out.println(Thread.currentThread().getName()+":"+ticket);
            ticket--;
        }
    }
}

public class ExtendsTest {
    public static void main(String[] args) {
        Window1 w1 = new Window1();
        Window1 w2 = new Window1();
        Window1 w3 = new Window1();

        w1.setName("窗口一");
        w2.setName("窗口二");
        w3.setName("窗口三");

        w1.start();
        w2.start();
        w3.start();
    }
}


//方式二:采用实现接口的方式
class Window2 implements Runnable{
    // 不需要加static
    private int ticket = 100;

    @Override
    public void run() {
        while (true){
            //同步方法
//            handleTicket();

            //同步代码块
            synchronized (this) {   //此时的this:唯一的Window2的对象
                if(ticket > 0){
                    try {
                        Thread.sleep(20);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // getName()属于Thread的方法
                    System.out.println(Thread.currentThread().getName()+":"+ticket);
                    ticket--;
                }else {
                    break;
                }
            }
        }
    }

    //方式一
    /*private void handleTicket() {
        // 同步代码块
        synchronized (this) {   //同步监视器：this
            if(ticket > 0){
                System.out.println(Thread.currentThread().getName()+":"+ticket);
                ticket--;
            }
        }
    }*/
    //方式二
    //同步方法: 不需要加static
    private synchronized void handleTicket() {  //同步监视器：this
        if(ticket > 0){
            System.out.println(Thread.currentThread().getName()+":"+ticket);
            ticket--;
        }
    }
}

public class ImplementTest {
    public static void main(String[] args) {
        Window2 w1 = new Window2();

        Thread t1 = new Thread(w1);
        Thread t2 = new Thread(w1);
        Thread t3 = new Thread(w1);

        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");

        t1.start();
        t2.start();
        t3.start();
    }
}
```

##### 2.4.lock

1.  synchronized 与 Lock的异同？

    相同：二者都可以解决线程安全问题
    不同：synchronized机制在执行完相应的同步代码以后，自动的释放同步监视器
    Lock需要手动的启动同步（lock()），同时结束同步也需要手动的实现（unlock()）

2.  优先使用顺序：

     Lock > 同步代码块（已经进入了方法体，分配了相应资源）> 同步方法（在方法体之外）

3.  代码:

```java
class Window implements Runnable{
    private int ticket = 100;
    //1.实例化ReentrantLock
    private ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        while(true){
            try{
                //2.调用锁定方法lock()
                lock.lock();

                if(ticket > 0){

                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + "：售票，票号为：" + ticket);
                    ticket--;
                }else{
                    break;
                }
            }finally {
                //3.调用解锁方法：unlock()
                lock.unlock();
            }
        }
    }
}

public class LockTest {
    public static void main(String[] args) {
        Window w = new Window();

        Thread t1 = new Thread(w);
        Thread t2 = new Thread(w);
        Thread t3 = new Thread(w);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}
```

### 九:线程的通信

#### 1.涉及到的三个方法

1.  wait():一旦执行此方法，当前线程就进入阻塞状态，并释放同步监视器。
2.  notify():一旦执行此方法，就会唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个。
3.  notifyAll():一旦执行此方法，就会唤醒所有被wait的线程。

#### 2.说明

1.  wait()，notify()，notifyAll()三个方法必须使用在同步代码块或同步方法中。
2.  wait()，notify()，notifyAll()三个方法的调用者必须是同步代码块或同步方法中的同步监视器。否则，会出现IllegalMonitorStateException异常.
3.  wait()，notify()，notifyAll()三个方法是定义在java.lang.Object类中。

#### 3.sleep() 和 wait()的异同？

1.  相同点：一旦执行方法，都可以使得当前的线程进入阻塞状态。
2.  不同点：
    1.  两个方法声明的位置不同：Thread类中声明sleep() , Object类中声明wait()
    2.  调用的要求不同：sleep()可以在任何需要的场景下调用。 wait()必须使用在同步代码块或同步方法中
    3.  关于是否释放同步监视器：如果两个方法都使用在同步代码块或同步方法中，sleep()不会释放锁，wait()会释放锁

#### 4.示例

```java
class Number implements Runnable{
    private int number = 1;
    private Object obj = new Object();
    @Override
    public void run() {
        while(true){

            synchronized (obj) {
                obj.notify();

                if(number <= 100){
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":" + number);
                    number++;

                    try {
                        //使得调用如下wait()方法的线程进入阻塞状态
                        obj.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }else{
                    break;
                }
            }
        }
    }
}

public class CommunicationTest {
    public static void main(String[] args) {
        Number number = new Number();
        Thread t1 = new Thread(number);
        Thread t2 = new Thread(number);

        t1.setName("线程1");
        t2.setName("线程2");

        t1.start();
        t2.start();
    }
}
```





