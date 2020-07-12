## 本地方法与本地方法栈

### 一.本地方法

![image-20200712110242214](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200712110242214.png)

#### 1.什么是Native Method

**一个Native Method就是一个Java调用非Java代码的接口**。一个Native Method是这样一个Java方法，该方法的实现由非Java语言实现。

"A native method is a Java method whose implementation is provided by non-java code."**在定义一个native method时**,并不提供实现体(有些像定义一个Java Interface) ,因为**其实现体是由非java语言在外面实现的**。本地接口的作用是融合不同的编程语言为Java所用,它的初衷是融合C/C++程序。

标识符 native 可以与其它Java标识符连用，但是 abstract 除外。如：

```java
public class IHaveNatives {
    public native void Native1(int x);
    public native static long Native2();
    private native synchronized float Native3(Object o);
    native void Native4(int[] ary) throws Exception;
}    
```

**Navtive 方法是 Java 通过 JNI 直接调用本地 C/C++ 库**，可以认为是 Native 方法相当于 C/C++ 暴露给 Java 的一个接口，Java 通过调用这个接口从而调用到 C/C++ 方法。当线程调用 Java 方法时，虚拟机会创建一个栈帧并压入 Java 虚拟机栈。当它调用的是 native 方法时，虚拟机会保持 Java 虚拟机栈不变，虚拟机只是简单地动态链接并直接调用指定的 native 方法。

#### 2.为什么要使用Native Method

![image-20200712111159289](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200712111159289.png)

- **与Java环境外交互**：有时Java应用需要与Java外面的环境交互,这是本地方法存在的主要原因。Java需要与一些底层系统,如操作系统或某些硬件交换信息时的情况。本地方法为我们提供了一个非常简洁的按口,无需去了解Java应用之外的繁琐的细节。
- **与操作系统交互**：JVM支持着Java语言本身和运行时库,它是Java程序赖以生存的平台,它由一个解释器(解释字节码)和一些连接到本地代码的库组成。它经常依赖于一些底层操作系统的支持。**通过使用本地方法,我们得以用Java实现了jre的与底层系统的交互,甚至JVM的一些部分就是用C写的**。
- **Sun's Java**：**Sun的解释器是用C实现的,这使得它能像一些普通的C一样与外部交互**。jre大部分是用Java实现的,它也通过一些本地方法与外界交互。例如:类java.lang.Thread的 setPriority() 方法是用Java实现的,但是它实现调用的是该类里的本地方法 setPriority0()。这个本地方法是用C实现的,并被植入JVM内部。

#### 3.JVM怎样使Native Method跑起来

当一个类第一次被使用到时，这个类的字节码会被加载到内存，并且只会加载一次。在这个被加载的字节码的入口维持着一个该类所有方法描述符的list，这些方法描述符包含这样一些信息：方法代码存于何处，它有哪些参数，方法的描述符（public之类）等等。

如果一个方法描述符内有native，这个描述符块将有一个指向该方法的实现的指针。这些实现在一些DLL文件内，但是它们会被操作系统加载到java程序的地址空间。当一个带有本地方法的类被加载时，其相关的DLL并未被加载，因此指向方法实现的指针并不会被设置。当本地方法被调用之前，这些DLL才会被加载，这是通过调用`java.system.loadLibrary()`实现的。

### 二.本地方法栈

#### 1.概念

![image-20200712112127701](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200712112127701.png)

**Java虚拟机栈用于管理Java方法的调用,而本地方法栈用于管理本地方法的调用**。

- 本地方法栈是一个后入先出（Last In First Out）栈,也是线程私有的，生命周期随着线程启动而产生，线程结束而消亡。本地方法是使用C语言实现的。
- 允许被实现成固定或者是可动态扩展的内存大小。(在内存溢出方面是相同的)
    - 如果线程请求分配的栈容量超过本地方法栈允许的最大容量, Java虚拟机将会抛出一个**StackOverflowError**异常。
    - 如果本地方法栈可以动态扩展,并且在尝试扩展的时候无法申请到足够的内存,或者在创建新的线程时没有足够的内存去创建对应的本地方法栈,那么Java虚拟机将会抛出一个**OutOfMemoryError**异常。
- 它的具体做法是**Native Method stack中登记native方法,在Execution Engine执行时加载本地方法库**。

**当某个线程调用一个本地方法时,它就进入了一个全新的并且不再受虚拟机限制的世界。它和虚拟机拥有同样的权限**。

- 本地方法可以通过本地方法接口来访问虚拟机内部的运行时数据区。
- 它甚至可以直接使用本地处理器中的寄存器。
- 直接从本地内存的堆中分配任意数量的内存。

#### 2.本地方法是如何工作的

当一个线程调用一个本地方法时，本地方法又回调虚拟机中的另一个Java方法。下面这幅图展示了java虚拟机内部线程运行的全景图。一个线程可能在整个生命周期中都执行Java方法，操作他的Java栈；或者他可能毫无障碍地在Java栈和本地方法栈之间跳转。

![image-20200712115456828](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200712115456828.png)

该线程首先调用了两个Java方法，而第二个Java方法又调用了一个本地方法，这样导致虚拟机使用了一个本地方法栈。图中的本地方法栈显示为 一个连续的内存空间。假设这是一个C语言栈，期间有两个C函数，他们都以包围在虚线中的灰色块表示。第一个C函数被第二个Java方法当做本地方法调用， 而这个C函数又调用了第二个C函数。之后第二个C函数又通过本地方法接口回调了一个Java方法（第三个Java方法）。最终这个Java方法又调用了一个Java方法（它成为图中的当前方法）。

 就像其他运行时内存区一样，**本地方法栈占用的内存区也不必是固定大小的，他可以根据需要动态扩展或者收缩**。某些是实现也允许用户或者程序员指定该内存区的初始大小以及最大，最小值。

**并不是所有的JVM都支持本地方法。因为Java虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等**。如果JVM产品不打算支持native方法,也可以无需实现本地方法栈。**在Hotspot JVM中,直接将本地方法栈和虚拟机栈合二为一**。



### 参考链接

- [JVM中的本地方法栈（Native Method Stacks）和Java虚拟机栈（Java Virtual Machine Stacks）](https://www.javatt.com/p/48067)
- [JVM学习笔记(一)——本地方法栈及native方法](https://blog.csdn.net/qq_28885149/article/details/52672475)

