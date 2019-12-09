### 一.概述

#### 1.java语言概述

1.  是**SUN**(Stanford University Network，斯坦福大学网络公司 ) 1995年推出的一门高级编程语言; java之父---**James Gosling(詹姆斯.高斯林)**.

2.  应用领域:

    -   **Java SE(Java Standard Edition)标准版**: 支持面向桌面级应用（如Windows下的应用程序）的Java平台，提供了完整的Java核心API，此版本以前称为J2SE;
    -   **Java EE(Java Enterprise Edition)企业版**: 是为开发企业环境下的应用程序提供的一套解决方案。该技术体系中包含的技术如:Servlet 、Jsp等，主要针对于Web应用程序开发。版本以前称为J2EE;
    -   Java ME(Java Micro Edition)小型版: 支持Java程序运行在移动终端（手机、PDA）上的平台，对Java API有所精简，并加入了针对移动终端的支持，此版本以前称为J2ME;
    -   Java Card: 支持一些Java小程序（Applets）运行在小内存设备（如智能卡）上的平台.

3.  java发展:

    -   **1996年，发布JDK 1.0，约8.3万个网页应用Java技术来制作**
    -   **2004年，发布里程碑式版本：JDK 1.5，为突出此版本的重要性，更名为JDK 5.0**
    -   2009年，Oracle公司收购SUN，交易价格74亿美元
    -   2011年，发布JDK 7.0
    -   **2014年，发布JDK 8.0，是继JDK 5.0以来变化最大的版本**
    -   随后发布了 JDK 9.0, JDK 10.0, JDK 11.0, JDK 12.0, JDK 13.0

4.  java语言特点:

    -   **面向对象**: 两大基本概念(类, 对象)与 三大特性(封装, 继承, 多态)

    -   **健壮性**: 吸收了C/C++语言的优点,去掉了影响程序健壮性的部分(指针、内存的申请与释放等)

    -   **跨平台性**: 

        -   跨平台性：通过Java语言编写的应用程序在不同的系统平台上都可以运行。 “Write once , Run Anywhere”

        -   原理：只要在需要运行 java 应用程序的操作系统上，先安装一个Java虚拟机 (JVM Java Virtual Machine) 即可。由JVM来负责Java程序在该系统中的运行
          ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209174206796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 2.java运行机制与运行过程

1.  **Java虚拟机 (Java Virtal Machine)**

    -   JVM 是一个虚拟的计算机，具有指令集并使用不同的存储区域。负责执行指令，管理数据、内存、寄存器
    -   对于不同的平台，有不同的虚拟机, 只有某平台提供了对应的java虚拟机，java程序才可在此平台运行
    -   Java虚拟机机制屏蔽了底层运行平台的差别，实现了“一次编译，到处运行”

2.  **垃圾收集机制 (Garbage Collection)**

    Java技术提供了垃圾收集器，用于自动检测每一块分配出去的内存空间，然后将无价值的内存块自动回收.

    在java中判断内存空间是否符合垃圾收集器的收集标准有两个：

    -   **为对象赋予了空值NULL后再没有调用过**
    -   **为对象赋予了新值，即重新分配了内存空间**

3.  **运行原理**：

    -   编写java源文件，以.java作为后缀名
    -   编译为字节码文件，使用java编译器将.java源文件编译成JVM能接受的指令集合，且以字节码.class的形式保存于文件中
    -   解释执行字节码.class文件，JVM读取字节码，取出指令，并且将其解释为能够将计算机执行的语言

    **java源程序 --(java编译器编译)--> .class文件 --(JVM解释)--> 计算机语言**
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209174309384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 3.java环境搭建

1. 什么是JDK ，JRE

    -   JDK(Java Development Kit): **Java开发工具包**

        JDK是提供给Java开发人员使用的，其中包含了java的开发工具，也包括了JRE。所以安装了JDK，就不用在单独安装JRE了。其中的开发工具：编译工具(javac.exe) 打包工具(jar.exe)等

    -   JRE(Java Runtime Environment): **Java运行环境**

        包括Java虚拟机(JVM Java Virtual Machine)和Java程序所需的核心类库等，如果想要 运行一个开发好的Java程序，计算机中只需要安装JRE即可

2. JDK 、JRE 、JVM 关系
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209174350784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)
    JDK = JRE + 开发工具集（例如Javac编译工具等）
    JRE = JVM + Java SE标准类库

    总结: 使用JDK 的开发工具完成的java 程序，交给JRE 去运行

3. 环境变量

    -   **Java_HOME**:JDK的安装路径  (JAVA_HOME: JDK安装路径)
    -   **PATH**：使系统可以在任何路径下识别java命令  (PATH: %JAVA_HOME%\bin)
    -   **CLASSPATH**：Java加载类的路径  (CLASSPATH: .;%JAVA_HOME%\lib)'.'表示当前路径

4. 运行步骤：在cmd窗口中执行java程序时先用javac命令将java源文件编译成字节码（.class）文件`javac 源文件名.java`，再用java命令将字节码文件解释为能被计算机执行的语言`java 字节码文件  (不能加后缀名)`。文档注释（/** */）可以通过javadoc命令生成API文档`javadoc -d 文档存放目录 源文件名.java `。

#### 4.helloword程序

-   Java应用程序的执行入口是main()方法.`public static void main(String[] args) {...}`

-   Java语言严格区分大小写

-   Java方法由一条条语句构成，每个语句以“;”结束

-   **一个源文件中最多只能有一个public类。其它类的个数不限，如果源文件包含一个public类，则文件名必须按该类名命名**

-   注释:

    -   单行注释:  `//注释文字`

    -   多行注释: `/* 注释文字 */`

    -   文档注释（Java 特有）: 注释内容可以被JDK提供的工具 javadoc 所解析，生成一套以网页文件形式体现的该程序的说明文档。

        /**
        @author 指定java 程序的作者
        @version 指定源文件的版本
        */

### 二.变量与运算符

#### 1.关键字和保留字
- 关键字(keyword)的定义和特点
    -   定义：被Java 语言赋予了特殊含义，用做专门用途的字符串
    -   特点：关键字中所有字母都为 小写

| 定义类型         | 关键字                                      |
| ------------ | ---------------------------------------- |
| 定义数据类型       | class interface enum byte short int long float double char boolean void |
| 定义流程控制       | if else switch case default while do for break continue return |
| 定义访问权限修饰符    | private protected public                 |
| 定义类,函数,变量修饰符 | abstract final static synchronized       |
| 定义类与类之间关系    | extends implements                       |
| 定义建立实例及引用实例  | new this super instanceof                |
| 异常处理         | try catch finally throw throws           |
| 包            | package import                           |
| 其他修饰符        | native strictfp transient volatile assert |
| 定义数据类型值      | true false null                          |

- 保留字(reserved word)

    现有Java版本尚未使用，但以后版本可能会作为关键字使用

    goto 、const

#### 2.标识符

1.  标识符
    -   Java 对各种 变量、 方法和 类等要素命名时使用的字符序列称为标识符
2.  标识符命名规范
    -   **由字母，数字，下划线，和‘$’中的任意字符组合而成**
    -   **数字不可以开头**
    -   **需要具有一定意义，且不能是系统关键字**
    -   **严格区分大小写**
3.  名称命名规范
    -   **包名**：多单词组成时所有字母都小写：xxxyyyzzz
    -   **类名**、接口名：多单词组成时，所有单词的首字母大写：XxxYyyZzz
    -   **变量名、方法名**：多单词组成时，第一个单词首字母小写，第二个单词开始每个单词首字母大写：xxxYyyZzz
    -   **常量名**：所有字母都大写。多单词时每个单词用下划线连接：XXX_YYY_ZZZ

#### 3.变量

1. **变量**

   - 变量的概念

     - **内存中的一个内存区域**
     - 该区域的数据可以在同一类型范围内不断变化
     - **变量是程序中最基本的存储单元**。包含变量类型、变量名和存储的值

   - 注意

     - java中每个变量·必须先声明，后使用
     - 变量的作用域：定义所在的一对{}内容，变量只有在其作用域内才有效
     - 同一作用域内，不能定义重名的变量

   - 声明变量

     - 语法：<数据类型><变量名称>
     - 示例：int i;

   - 变量赋值

     - 语法：<变量名称> = <值>
     - 示例：i = 6;

   - 声明和赋值变量

     - 语法：<数据类型> <变量名> = <初始化值>
     - 示例：int i = 6;

   - 分类-按声明的位置不同

       ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209174838191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

       ​

2. **数据类型**

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209174909540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

   - **整数类型**：默认为int型，声明long需加'l'或'L'

     ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209174940245.png)

     ​

   - **浮点类型**：默认为double型，声明float型需要加'f'或'F';float可以精确到7位有效数字，double精度是float的两倍。

     ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209174957210.png)

     ​

   - **字符类型**：占2个字节。

   - **布尔类型**：只允许true和false，无null。不可以使用0或非0的整数替代false和true。

3. **基本数据类型变量间转换**

   - **自动类型转换：容量小的类型自动转换为容量大的数据类型**。![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209181137722.png)

     ​

     当和字符串进行连接运算时(+)时，

     `3+4+"hello"	//7hello`，

      `"hello"+3+4	//hello34`，

     `'a'+1+"hello"	//98hello`，

     `"hello"+'a'+1	//helloa1`

   - **强制类型转换：将容量大的数据类型转换为容量小的数据类型**。使用时要加上强制转换符：`()`，但可能造成精度降低或溢出。

4. **基本数据类型与String间转换**

   字符串不能直接转换为基本类型，但通过基本类型对应的包装类则可以实现把字符串转换成基本类型。如：`Integer.parseInt()`

#### 4.运算符

运算符是一种特殊的符号，用以表示数据的运算、赋值和比较等。

1. **算术运算符**

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209181337849.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

  ​

2. **赋值运算符**

   符号：=

   **当“=”两侧数据类型不一致时，可以使用自动类型转换或使用强制类型转换原则进行处理**。

   扩展赋值运算符： +=,  -=,  *=,  /=,  %=

3. **比较运算符**

   比较运算符的结果都是boolean型，也就是要么是true，要么是false；

   比较运算符“==” 不能误写成“=”。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209181407858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

   ​

4. **逻辑运算符**

   `& —逻辑与` `| —逻辑或` `！ —逻辑非`
   `&& —短路与` `|| —短路或` `^ —逻辑异或`
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209181450478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

   “&”和“&&”的区别：

   **单&时，左边无论真假，右边都进行运算；**

   **双&时，如果左边为真，右边参与运算，如果左边为假，那么右边不参与运算。**

   “|”和“||”的区别同理，||表示：当左边为真，右边不参与运算。

   异或( ^ )与或( | )的不同之处是：**当左右值都相等时，结果为false**。

5. **位运算符**

   位运算是直接对整数的二进制进行的运算。

   **二进制或运算符**（or）：符号为`|`，表示若两个二进制位都为`0`，则结果为`0`，否则为`1`。

   **二进制与运算符**（and）：符号为`&`，表示若两个二进制位都为1，则结果为1，否则为0。

   **二进制否运算符**（not）：符号为`~`，表示对一个二进制位取反。

   **异或运算符**（xor）：符号为`^`，表示若两个二进制位不相同，则结果为1，否则为0。

   **左移运算符**（left shift）：符号为`<<`。

   **右移运算符**（right shift）：符号为`>>`。

   **头部补零的右移运算符**（zero filled right shift）：符号为`>>>`。

   位运算只对整数有效，遇到小数时，会将小数部分舍去，只保留整数部分。所以，将一个小数与`0`进行二进制或运算，等同于对该数去除小数部分，即取整数位。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209181616283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

   ​

   无 <<< 运算符
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209181633493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

   ​

6. **三元运算符**

   格式：`(条件表达式)？表达式1：表达式2`

   条件表达式为true，运算结果是表达式1；为false，运算结果是表达式2；

7. **运算符的优先级**

   只有单目运算符、三元运算符、赋值运算符是从右向左运算的。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209181700982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

### 三.流程控制

流程控制语句是用来控制程序中各语句执行顺序的语句，可以把语句组合成能完成一定功能的小逻辑模块。

其流程控制方式采用结构化程序设计中规定的三种基本流程结构，即：**顺序结构、分支结构、循环结构**。

#### 1.顺序结构

程序从上到下逐行地执行，中间没有任何判断和跳转。

Java中定义**成员变量**时采用合法的**前向引用**。

#### 2.分支结构

1. **if-else结构**

   - if(条件表达式){ 执行代码块; }
   - if(条件表达式){ 执行代码块1; }else{ 执行代码块2; }
   - if(条件表达式){ 执行代码块1; }else if(条件表达式){ 执行代码块2; }else{ 执行代码块3; }

2. **switch-case结构**

   ```java
   switch(表达式){
       case 常量1:
           语句1;
           // break;
       case 常量2:
           语句2;
           // break;
       default:
           语句;
           // break;
   }
   ```

   -  switch(表达式)中表达式的值 必须是下述几种类型之一：**byte ，short ，char ，int ，枚举 (jdk 5.0) ，String (jdk 7.0)**
   -  case子句中的值必须是 常量，不能是变量名或不确定的表达式值
   -  同一个switch语句，所有case子句中的常量值互不相同
   -  **break语句用来在执行完一个case分支后使程序跳出switch语句块；如果没有break，程序会顺序执行到switch结尾**
   -  default子句是 可任选 的。同时，位置也是灵活的。当没有匹配的case时，执行default

3. 如果判断的具体数值不多，又符合byte、short、char、int、String、枚举等类型，建议使用switch-case，效率稍高；对区间判断，对结果为boolean类型判断，建议使用if语句。

#### 3.循环结构

在某些条件满足的情况下，反复执行特定代码的功能。循环语句的四个组成部分：**初始化部分(init_statement)、循环条件部分(test_exp)、循环体部分(body_statement)、迭代部分(alter_statement)**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209181750324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

1. **for循环**

   ```java
   for(1初始化部分; 2循环条件部分; 4迭代部分){
       3循环体部分;
   }
   ```

   - 循环条件部分为boolean类型表达式，当值为false时，退出循环
   - **初始化部分可以声明多个变量，但必须是同一个类型，用逗号分隔**
   - **可以有多个变量更新，用逗号分隔**

2. **while循环**

   ```java
   1初始化部分;
   while(2循环条件部分){
   	3循环体部分;
   	4迭代部分;
   }
   ```

   - **注意不要忘记声明迭代部分**。否则，循环将不能结束，变成死循环
   - for循环和while循环可以相互转换

3. **do-while循环**

   ```java
   1初始化部分;
   do{
       3循环体部分;
       4迭代部分;
   }while(2循环条件部分);
   ```

   - **do-while循环至少执行一次循环**

4. **嵌套循环(多重循环)**

   将一个循环放在另一个循环体内，就形成了嵌套循环。

   **嵌套循环就是把内层循环当成外层循环的循环体**。当只有内层循环的循环条件为false时，才会完全跳出内层循环，才可结束外层的当次循环，开始下一次的循环。

   设外层循环次数为m次，内层为n次，则内层循环体实际上需要执行m*n次。

5. **break、continue使用**

   - break
     - break语句用于**终止**某个语句块的执行
     - break语句出现在多层嵌套的语句块中时，可以通过标签指明要终止的是哪一层语句块
   - continue
     - continue语句用于跳过其所在循环语句块的**一次执行**，继续下一次循环
     - continue语句出现在多层嵌套的循环语句体中时，可以通过标签指明要跳过的是哪一层循环
   - return
     - return：并非专门用于结束循环的，它的功能是结束**一个方法**。当一个方法执行到一个return语句时，这个方法将被结束
     - 与break和continue不同的是，return直接结束整个方法，不管这个return处于多少层循环之内


   - 注意
     - **break只能用于switch 语句和 循环语句中**
     - **continue 只能用于 循环语句中**
     - **continue是终止 本次循环，break是终止 本层循环**
     - **break、continue之后不能有其他的语句，因为程序永远不会执行其后的语句**
     - **标号语句必须紧接在循环的头部。标号语句不能用在非循环语句的前面**







