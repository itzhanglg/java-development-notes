## JVM体系结构和描述

### 虚拟机与Java虚拟机介绍

#### 虚拟机

虚拟机（Virtual Machine）顾名思义就是虚拟的计算机。是一款软件，用来执行一系列虚拟计算机指令。虚拟机一般分为 系统虚拟机 和 程序虚拟机。

- Visual Box，VMware就属于**系统虚拟机**，是对物理计算机的仿真，提供了一个可运行完整操作系统的软件平台
- Java虚拟机就是典型的**程序虚拟机**，专门为执行单个计算机程序设计，在Java虚拟机中执行的指令称为Java字节码指令

#### Java虚拟机

Java虚拟机是一台执行Java字节码的虚拟计算机，JVM平台的各种语言可以共享Java虚拟机带来的跨平台性、优秀的垃圾回收器和可靠的即时编译器。

**Java虚拟机作用**：Java虚拟机就是二进制字节码的运行环境，负责装载字节码到其内部，解释/编译为对应平台上的机器指令执行。每一条Java指令，Java虚拟机规范中都有详细定义，如怎么取操作数，怎么处理操作数，处理结果放在哪里。

**特点**：

- 一次编译，到处运行
- 自动内存管理
- 自动垃圾回收功能

### 跨语言的平台JVM

Java7以后，Java虚拟机平台就可以运行非Java语言编写的程序。**Java虚拟机**根本不关心运行在其内部的程序是使用何种编程语言编写，**它只关心"字节码"文件**。

![image-20200703000823097](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200703000823097.png)

- 编译器的前端：各个语言自己的编译器
- 编译起的后端：Java虚拟机的解释器和 JIT即时编译器

信息产业领域三大难题：cpu、操作系统、编译器。

Java平台上的多语言混合编程正成为主流，各种语言之间的交互不存在任何困难，就像使用自己语言的原生API一样方便，因为它们最终都运行在一个虚拟机上。Java虚拟机在从“Java语言虚拟机”向“多语言虚拟机”方向发张。

### Java及JVM历史事件

- 1990年：Sun公司中James Gosling 等领导的小组Green Team开发出新的程序语言**Java**
- 2000年：JDK1.3发布，**Java HotSpot Virtual Machine**正式发布，成为Java默认虚拟机
- 2003年底：Java平台的**Scala**正式发布，同年**Groovy**也加入了Java阵营
- 2006年：JDK6发布，Java开源并建立了**OpenJDK**。HotSpot虚拟机也成为了OpenJDK中默认虚拟机
- 2008年：Oracle收购了BEA，得到了**JRockit**虚拟机
- 2009年：Oracle收购了Sun，获得了Java商标和最具有价值的**HotSpot**虚拟机
- 2011年：JDK7发布，正式启用了新的垃圾回收器**G1**
- 2018年：JDK11发布，LTS（长期支持版本）版本JDK，发布革命性的**ZGC**
- 2019年：JDK12发布，加入RedHat领导开发的**Shenandoah GC**

OpenJDK与OracleJDK：JDK11之前，OracleJDK（商用）比OpenJDK（开源）多一些闭源的功能。但在JDK11中，两者基本完全一致。

### JVM的发展历程

#### Sun Classic VM

- 1996年Java1.0版本，Sun公司发布了Sun Classic VM的 Java 虚拟机，JDK1.4时完全被淘汰。
- 这款虚拟机**只提供解释器**。
- 如果使用JIT编译器，就需要进行外挂。一旦使用JIT编译器，解释器就不再进行工作。**解释器和编译器不能配合工作**。

#### Exact VM

- jdk1.2时，sun提供了该虚拟机
- Exact Memory Management：准确式内存管理
  - Non-Conservative/Accurate Memory Management
  - **虚拟机可以知道内存中某个位置的数据类型**
- 具备高性能虚拟机的雏形，还没完全实现
  - 热点探测
  - 编译器与解释器混合工作模
- 使用短暂，该虚拟机还没怎么开始就被HotSpot替换了

#### SUN公司的 HotSpot VM

2009年，Sun公司被Oracle公司收购。

- JDK1.3时，HotSpot VM称为默认虚拟机
- HotSpot从服务器、客服端到移动端，嵌入式都有应用
- HotSpot名称指的就是他的热点代码探测技术
  - **通过计数器找到最具编译价值代码**，触发即时编译或栈上替换
  - **通过编译器与解释器协同工作，在最优化的程序响应时间与最佳执行性能中取得平衡**

#### BEA 的 JRockit

2008年，BEA被Oracle公司收购。Java之父高斯林目前就职于谷歌，研究人工智能和水下机器人

- **专注于服务器端应用**：**可以不关注启动速度，JRockit内部不包含解析器实现，全部代码靠即时编译器编译后执行**
- JRockit JVM是世界上最快的JVM
- Oracle在JDK 8中大致整合两大优秀虚拟机，在HotSpot的基础上移植JRockit的优秀特性

#### IBM 的 J9

- IBM Technology for Java Virtual Machine，简称 IT4J，内部代号 J9
- 与HotSpot定位一致，在服务端、桌面端、嵌入式等都有用途
- 与IBM自己的硬件相耦合，是最快的虚拟机，与外面其他配合使用还是比不过JRockit

#### Graal VM

- 2018年4月，Oracle Labs公开了Graal VM，称“**Run Programs Faster Anywhere**”
- Graal VM在HotSpot  VM基础上增强成**跨语言全栈虚拟机，可以作为“任何语言”的运行平台使用**。之前已经集成的 Java、Scala、Groovy、Kotlin；C、C++、JavaScript、Ruby、Python、R等
- **Graal VM未来有可能会替代掉HotSpot虚拟机**

**总结**:

- 三款高性能Java虚拟机：HotSpot、JRockit、C9，使用在通用硬件平台上。
- 两款高性能Java虚拟机中的战斗机：Azul VM 和 BEA Liquid VM，他们与特定硬件平台绑定、软硬件配合的专有虚拟机。
- 具体JVM的内存结构取决于不同厂商的JVM实现或者同一厂商发布的不同版本，都有可能存在一定差异。

### JVM位置

硬件  --》 操作系统  --》 JVM  --》 字节码文件  --》 用户User。

![image-20200704095539202](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704095539202.png)

图片来源于：https://docs.oracle.com/javase/8/docs/

### JVM的整体结构

![image-20200704103334167](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704103334167.png)

以HotSpot为代表：**采用解释器与即时编译器并存的架构**

- 字节码文件（Class files）
- 类装载器子系统（Class loader）
- 运行时数据区（Runtime Data Area）
  - 线程共享：方法区（Method Area）、堆（heap）
  - 线程私有：Java虚拟机栈（Java stack）、本地方法栈（Native Method Stack）、程序计数器（Program Counter Register）
- 执行引擎（Execution Engine）
- 本地方法接口（Native Method Interface）、本地方法库（Native Method Library）

### Java代码的执行流程

![image-20200704095622133](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704095622133.png)

为什么不一次性都编译所有代码，这样不更好的节省效率么？

- **编译过程耗时，程序运行速度慢，启动慢**
- **code catch的内存有可能会溢出，也及时方法区内存不够**

### JVM的架构模型

Java编译器输入的指令流一种是基于**栈的指令集架构**，另一种基于**寄存器的指令集架构**。

**基于栈式架构的特点**：

- 使用**零地址指令**方式分配
- 指令流中的指令大部分是零地址指令，其执行过程**依赖于操作栈**。**指令集更小**，编译器容易实现
- 不依赖硬件支持，可移植性更好，更好的**实现跨平台**

**基于寄存器架构的特点**：

- 指令集架构则完全**依赖硬件，可移植性差**
- **性能优秀**和执行更高效
- 用**更少的指令**完成一项操作

大部分情况下，基于寄存器架构的指令集往往都以一地址指令、二地址指令和三地址指令为主，而基于栈式架构的指令集是以零地址指令为主。

总结：

**由于跨平台性的设计，Java的指令都是根据栈来设计的**。不同平台CPU架构不同，所以不能设计为基于寄存器的。**优点是跨平台性，指令集小，编译器容易实现；缺点是执行性能比寄存器差，实现同样的功能需要更多的指令**。

字节码文件可以在命令行操作： `javap -v 文件名.class`  进行反编译,查看字节码指令，也可以使用 `jps` 指令查看当前应用的进程。

### JVM的生命周期

#### 虚拟机的启动

Java虚拟机的启动是通过**引导类加载器**（bootstrap class loader）创建一个**初始类**（initial class）来完成的，这个类是由虚拟机的具体实现指定的。

#### 虚拟机的执行

程序开始执行时，虚拟机才运行，程序结束时才停止。**执行一个所谓的Java程序时，真真正正执行的是一个叫做Java虚拟机的进程**。

终端窗口命令：jps 命令可查看当前程序的进程情况

#### 虚拟机的退出

- 程序正常执行结束
- 程序执行过程中遇到异常或错误而异常终止
- 操作系统出现错误导致Java虚拟机进程终止
- 某线程调用Runtime类或System类的exit方法，或Runtime类的halt方法，并且Java安全管理器也允许这次exit或halt操作







