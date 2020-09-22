## 堆空间

### 1.概述

一个JVM实例只存在一个堆内存,堆也是Java内存管理的核心区域。Java堆区在JVM启动的时候即被创建,其空间大小也就确定了。是JVM管理的最大一块内存空间。堆内存的大小是可以调节的。

《Java虚拟机规范》规定,堆可以处于**物理上不连续**的内存空间中,但在**逻辑上它应该被视为连续**的。**所有的线程共享Java堆**,在这里还可以**划分线程私有的缓冲区**(Thread Local Allocation Buffer, TLAB)。

《Java虚拟机规范》中对Java堆的描述是:**所有的对象实例以及数组都应当在运行时分配在堆上**。(The heap is the run-time data area fromwhich memory for all class instances and arrays is allocated)。**几乎所有对象实例都在堆上分配(有可能出现对象逃逸现象，在栈上分配)**。**在方法结束后,堆中的对象不会马上被移除,仅仅在垃圾收集的时候才会被移除**（频繁的GC会影响用户线程的执行）。堆是GC ( Garbage Collection,垃圾收集器)执行垃圾回收的重点区域。

现代垃圾收集器大部分都基于分代收集理论设计，堆空间细分为：

Java7及之前堆内存逻辑上分为三部分：

- **新生代**（Young Generation Space）：Young/New区
  - 伊甸园区：Eden区
  - 幸存者区：Survivor1区（from区）、Survivor2区（to区）
- **老年代**（Tenure Generation Space）：Old/Tenure区
- 永久区（Permanent Space）：Perm区

Java8及之后将永久代改为**元空间**（Meta Space），即Meta区。

### 2.堆内存大小与OOM

idea测试时可以在Run/Debug Configurations中VM options里填入参数：`-Xms20m -Xmx10m -XX:+PrintGCDetails` 进行测试，运行程序后可以到 jdk8/bin 中执行 jvisualvm.exe 程序进行查看，可以在工具栏中安装 JConsole Plugins、Visual GC、Tracer插件进行观看。修改java jdk版本：在项目结构中修改模块的JDK版本，Configurations中修改JRE为对应的java运行环境。

Java堆区用于存储Java对象实例,那么堆的大小在JVM启动时就已经设定好了,大家可以通过选项"-Xmx"和"-Xms"来进行设置。

- "-Xms"用于表示**堆区（年轻代和老年代）的起始内存**,等价于-xx: InitialHeapsize，-X 是jvm的运行参数；ms 是memory start
- "-Xmx"则用于表示**堆区（年轻代和老年代）的最大内存**,等价于-xx:MaxHeapsize

一旦堆区中的内存大小超过"-Xmx"所指定的最大内存时,将会抛出OutofMemoryError异常。**通常会将-Xms和-Xmx两个参数配置相同的值**,其目的是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小,从而提高性能。

默认情况下,**初始内存大小为物理电脑内存大小的64分之一**，**最大内存大小为物理电脑内存大小的4分之一**。

下面看下示例：在Run/Debug Configurations中设置参数`-Xms600m -Xmx600m` ，后运行下列代码

```java
//返回Java虚拟机中的堆内存总量
long initialMemory = Runtime.getRuntime().totalMemory() / 1024 / 1024;
//返回Java虚拟机试图使用的最大堆内存量
long maxMemory = Runtime.getRuntime().maxMemory() / 1024 / 1024;

System.out.println("-Xms : " + initialMemory + "M");
System.out.println("-Xmx : " + maxMemory + "M");

//  System.out.println("系统内存大小为：" + initialMemory * 64.0 / 1024 + "G");
//  System.out.println("系统内存大小为：" + maxMemory * 4.0 / 1024 + "G");
```

查看设置的参数，可以在命令行使用 `jps` 和 `jstat -gc` 进程ID 查看,或者启动之前设置控制台打印GC信息 ``-XX:+PrintGCDetails`，效果如下：

![image-20200716075110877](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200716075110877.png)

​	













### 3.年轻代与老年代



### 4.对象分配过程



### 5.Minor GC、Major GC、Full GC



### 6.堆空间分代思想



### 7.内存分配策略



### 8.对象分配内存：TLAB



### 9.堆空间的参数设置



### 10.堆是分配对象的唯一选择吗





