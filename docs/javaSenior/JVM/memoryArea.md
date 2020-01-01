转载链接：[Java 内存区域详解](https://snailclimb.gitee.io/javaguide/#/docs/java/jvm/Java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F)

### 一.摘要

因为 Java 程序员把内存控制权利交给 Java 虚拟机，一旦出现内存泄漏和溢出方面的问题，如果不了解虚拟机是怎样使用内存的，那么排查错误将会是一个非常艰巨的任务。

### 二.运行时数据区域

Java 虚拟机在执行 Java 程序的过程中会把它管理的**内存划分成若干个不同的数据区域**。JDK. 1.8 和之前的版本略有不同。

**JDK1.8之前：**

<div align="center">

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191231192113566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

</div>

**JDK1.8：**

<div align="center">  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191231192124935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

</div>

<div style="color:red;">

- 线程私有的：程序计数器、虚拟机栈、本地方法栈
- 线程共享的：堆、方法区、直接内存(非运行时数据区的一部分)
- JDK8 将方法区从运行时数据区移到直接内存，并改名为元空间

</div>

#### 1.程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。**字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。**

另外，**为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。**

**从上面的介绍中我们知道程序计数器主要有两个作用：**

1. 字节码解释器通过改变程序计数器来**依次读取指令**，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器**用于记录当前线程执行的位置**，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

**注意：程序计数器是唯一一个不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**

**小结：**<div style="color:red;">

- 字节码解释器通过改变程序计数器来依次读取指令，实现代码流程控制
- 多线程情况下，程序计数器用于记录当前线程执行的位置，知道线程切换回来运行到哪了

</div>

#### 2.Java虚拟机栈

**与程序计数器一样，Java 虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。**

**Java 内存可以粗糙的区分为堆内存（Heap）和栈内存 (Stack),其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。** （实际上，Java 虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息。）

**局部变量表主要存放了编译器可知的各种数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

**Java 虚拟机栈会出现两种错误：StackOverFlowError 和 OutOfMemoryError。**

- **StackOverFlowError：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **OutOfMemoryError：** 若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出 OutOfMemoryError 错误。

Java 虚拟机栈也是线程私有的，每个线程都有各自的 Java 虚拟机栈，而且随着线程的创建而创建，随着线程的死亡而死亡。

**扩展：那么方法/函数如何调用？**

Java 栈可用类比数据结构中栈，Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出。Java 方法有两种返回方式：return语句 和 抛出异常。不管哪种返回方式都会导致栈帧被弹出。

**小结：**<div style="color:red;">

- Java 虚拟机栈描述Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的
- Java 虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息。平常说的栈内存也就是局部变量表，局部变量表主要存放基本数据类型，对象引用
- 每次函数调用都会有对应的栈帧被压入，每次函数调用结束都会有对应的栈帧被弹出
- 会出现 StackOverFlowError 和 OutOfMemoryError 两种错误

</div>

#### 3.本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 StackOverFlowError 和 OutOfMemoryError 两种错误。

**小结：**<div style="color:red;">

- 虚拟机栈为虚拟机执行 Java 方法服务，而本地方法栈则为虚拟机使用到的 Native 方法服务
- 其他和虚拟机栈发挥的作用相似

</div>

#### 4.堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是垃圾收集器管理的主要区域，因此也被称作**GC 堆（Garbage Collected Heap）**.从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代：再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

在 JDK 7 版本及JDK 7 版本之前，堆内存被通常被分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永生代(Permanent Generation)

![JVM堆内存结构-JDK7](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS0xMS9KVk0lRTUlQTAlODYlRTUlODYlODUlRTUlQUQlOTglRTclQkIlOTMlRTYlOUUlODQtSkRLNy5qcGc?x-oss-process=image/format,png)

JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

![JVM堆内存结构-JDK8](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS0xMS9KVk0lRTUlQTAlODYlRTUlODYlODUlRTUlQUQlOTglRTclQkIlOTMlRTYlOUUlODQtamRrOC5qcGc?x-oss-process=image/format,png)

**上图所示的 Eden 区、两个 Survivor 区都属于新生代（为了区分，这两个 Survivor 区域按照顺序被命名为 from 和 to），中间一层属于老年代。**

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加  1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

> 修正（[issue552](https://github.com/Snailclimb/JavaGuide/issues/552)）：“Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了survivor区的一半时，取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值”。
>
> **动态年龄计算的代码如下**
>
> ```c++
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> 	//survivor_capacity是survivor空间的大小
>   size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
>   size_t total = 0;
>   uint age = 1;
>   while (age < table_size) {
>     total += sizes[age];//sizes数组是每个年龄段对象大小
>     if (total > desired_survivor_size) break;
>     age++;
>   }
>   uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> 	...
> }
>
> ```
>
> 

堆这里最容易出现的就是  OutOfMemoryError 错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **`OutOfMemoryError: GC Overhead Limit Exceeded`** ： 当JVM花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **`java.lang.OutOfMemoryError: Java heap space`** :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发`java.lang.OutOfMemoryError: Java heap space` 错误。(和本机物理内存无关，和你配置的对内存大小有关！)
3. ......

推荐阅读：[Java8内存模型—永久代(PermGen)和元空间(Metaspace)](http://www.cnblogs.com/paddix/p/5309550.html)

**小结：**<div style="color:red;">

- 几乎所有对象实例及数组都在堆里分配内存
- JDK7及以前：Java 堆可分为 新生代、老年代和永久代，新生代 又可分为 Eden 空间、From Survivor、To Survivor 空间等；JDK8及之后永久代被元空间取代，元空间使用的是直接内存(永久代使用的是JVM的堆内存空间，而元空间使用的是物理内存，直接受到本机的物理内存限制)
- 堆容易出现 OutOfMemoryError 错误

</div>

#### 5.方法区

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然 **Java 虚拟机规范把方法区描述为堆的一个逻辑部分**，但是它却有一个别名叫做 **Non-Heap（非堆）**，目的应该是与 Java 堆区分开来。

方法区也被称为永久代。很多人都会分不清方法区和永久代的关系，为此我也查阅了文献。

##### 2.5.1 方法区和永久代的关系

> 《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。  **方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。** 也就是说，永久代是 HotSpot 的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。

##### 2.5.2 常用参数

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```java
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

##### 2.5.3 为什么要将永久代 (PermGen) 替换为元空间 (MetaSpace) 呢?

1. 整个永久代有一个 JVM 本身设置固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

> 当你元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。  

1. 元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。  
2. 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

**小结：**<div style="color:red;">

- 方法区存放已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据
- 方法区为堆的一个逻辑部分，别名为Non-Heap(非堆)，也称为永久代。方法区是 java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，其他虚拟机实现并没有永久代说法
- JDK8 方法区(HotSpot 的永久代)被元空间取代，使用直接内存，这样能加载的类就更多了，存放的是类的元数据。但若不指定元空间大小，随着类的创建，虚拟机会耗尽所有可用的系统内存

</div>

#### 6.运行时常量池

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用）

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 错误。

**JDK1.7 及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。** 

![](https://imgconvert.csdnimg.cn/aHR0cDovL215LWJsb2ctdG8tdXNlLm9zcy1jbi1iZWlqaW5nLmFsaXl1bmNzLmNvbS8xOC05LTE0LzI2MDM4NDMzLmpwZw?x-oss-process=image/format,png)
——图片来源：https://blog.csdn.net/wangbiao007/article/details/78545189

推荐阅读：[Java 中几种常量池的区分](https://blog.csdn.net/qq_26222859/article/details/73135660)

**小结：**<div style="color:red;">

- 运行时常量池是方法区的一部分，存放编译期生成的各种字面量(字符串、常量、基本数据类型值、其他)和符号引用(类和结构的完全限定名、字段名和描述符、方法名和描述符)
- JDK7及之后 运行时常量池从方法区移出到堆中

</div>

#### 7.直接内存

**直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 错误出现。**

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）** 与**缓存区（Buffer）** 的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

**小结：**<div style="color:red;">

- 直接内存不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，内存分配不会受到java堆的限制，但也会受到本机内存大小以及处理器寻址空间的限制

</div>

### 三.HotSpot虚拟机对象探秘

通过上面的介绍我们大概知道了虚拟机的内存情况，下面我们来详细的了解一下 HotSpot 虚拟机在 Java 堆中对象分配、布局和访问的全过程。

#### 3.1 对象的创建

下图便是 Java 对象的创建过程，我建议最好是能默写出来，并且要掌握每一步在做什么。
![Java创建对象的过程](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS02L0phdmElRTUlODglOUIlRTUlQkIlQkElRTUlQUYlQjklRTglQjElQTElRTclOUElODQlRTglQkYlODclRTclQTglOEIucG5n?x-oss-process=image/format,png)

##### Step1:类加载检查

 虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

##### Step2:分配内存

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，**选择那种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定**。

**内存分配的两种方式：（补充内容，需要掌握）**

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的

![内存分配的两种方式](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS02LyVFNSU4NiU4NSVFNSVBRCU5OCVFNSU4OCU4NiVFOSU4NSU4RCVFNyU5QSU4NCVFNCVCOCVBNCVFNyVBNyU4RCVFNiU5NiVCOSVFNSVCQyU4Ri5wbmc?x-oss-process=image/format,png)

**内存分配并发问题（补充内容，需要掌握）**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

##### Step3:初始化零值

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

##### Step4:设置对象头

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

##### Step5:执行 init 方法

 在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

**小结：**<div style="color:red;">

- 类加载检查：虚拟机遇到new指令，检查常量池中是否能定位到这个类的符号引用及代表的类是否已被加载  --  **比如孕妇是否在医院，若不在医院立马联系孕妇到医院**
- 分配内存：虚拟机把一块确定大小的内存从java堆中划分出来  --  **比如小孩出生**
- 初始化零值：虚拟机将分配的内存空间都初始化为零值(不包括对象头)  --  **比如小孩出生后的属性**
- 设置对象头：虚拟机要对对象头进行必要的设置(对象是哪个类的实例、如何找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息)  --  **比如小孩出生后注册身份：出生日期，出生地址，身份证号等**
- 执行init方法：执行new指令后会接着执行`<init>`方法，将对象进行初始化  --  **显示的赋值如小孩姓名、年龄等**

</div>

#### 3.2 对象的内存布局

在 Hotspot 虚拟机中，对象在内存中的布局可以分为 3 块区域：**对象头**、**实例数据**和**对齐填充**。

**Hotspot 虚拟机的对象头包括两部分信息**，**第一部分用于存储对象自身的自身运行时数据**（哈希码、GC 分代年龄、锁状态标志等等），**另一部分是类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。

**实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

**对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。** 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

#### 3.3 对象的访问定位

建立对象就是为了使用对象，我们的 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问方式有**①使用句柄**和**②直接指针**两种：

1. **句柄：** 如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息；

   [外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-nTQRdphl-1577791232639)(https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/对象的访问定位-使用句柄.png)]

2. **直接指针：**  如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而 reference 中存储的直接就是对象的地址。

![对象的访问定位-直接指针](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9teS1ibG9nLXRvLXVzZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAxOS02LyVFNSVBRiVCOSVFOCVCMSVBMSVFNyU5QSU4NCVFOCVBRSVCRiVFOSU5NyVBRSVFNSVBRSU5QSVFNCVCRCU4RC0lRTclOUIlQjQlRTYlOEUlQTUlRTYlOEMlODclRTklOTIlODgucG5n?x-oss-process=image/format,png)

**这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。**



### 参考链接

- 《深入理解 Java 虚拟机：JVM 高级特性与最佳实践（第二版》
- 《实战 java 虚拟机》
- <https://docs.oracle.com/javase/specs/index.html>
- <http://www.pointsoftware.ch/en/under-the-hood-runtime-data-areas-javas-memory-model/>
- <https://dzone.com/articles/jvm-permgen-%E2%80%93-where-art-thou>
- <https://stackoverflow.com/questions/9095748/method-area-and-permgen>
- [深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)




