## 类加载子系统学习

### 一.概述

![image-20200704104831860](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704104831860.png)

类加载器子系统负责从文件系统或网络中加载Class文件，class文件在文件开头有特定的文件标识。**字节码文件会经过类加载阶段、链接阶段、初始化阶段后被执行引擎所执行**。ClassLoader只负责字节码文件的加载，至于是否可以运行由执行引擎决定。

**加载的类信息存放在方法区（JRockit和J9虚拟机没有方法区的概念）的内存空间。运行时常量池信息也会存放到方法区中，可能还包括字符串字面量和数字常量**。

字节码文件存在于本地硬盘上，被加载到JVM中，被称为**DNA元数据模板**，放在方法区。该过程中类加载器扮演着一个运输工具。

### 二.类的加载过程

![image-20200704110912077](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704110912077.png)

类的加载过程分为加载阶段，链接阶段（验证、准备、解析）和初始化阶段三个阶段过程。

#### 1.加载阶段

![image-20200704112049569](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704112049569.png)

加载阶段：通过一个类的全限定类名获取该类的二进制字节流，将这个字节流所代表的静态存储结构转为方法区的运行时数据结构，**在内存中生成一个代表该类的Class对象**（字节码对象）作为该类的各种数据访问入口。

常见加载 .class 文件的方式：

- 本地系统直接加载
- 网络获取，如web应用
- 压缩包中读取，如 jar、war包
- 运行时计算生成，如动态代理技术
- 由其它文件生成，如 jsp 页面

#### 2.链接阶段

**验证**（Verify）：确保**Class文件的字节流中包含的信息符合当前虚拟机要求，保证加载类的正确性**。主要包括四种验证：文件格式验证、元数据验证、字节码验证、符号引用验证。如java编译器编译后的class文件开头为`CA FE BA BE` 。

**准备**（Prepare）：

- **为类变量（static修饰的变量）分配内存并且根据类型设置对应的默认初始值**，如零，null，false等
- final修饰的static变量在java编译时就分配了，该过程**不包含静态常量的分配**
- 实例变量随对象一起分配到Java堆中，类变量分配到方法区中，该过程**不包含实例变量的分配**

**解析**（Resolve）：**将常量池内的符号引用转换为直接引用的过程**，解析操作往往会随着JVM在执行完初始化之后再执行。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。

- 符号引用是一组符号来描述所引用的目标
- 直接引用是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄

#### 3.初始化阶段

**初始化阶段就是执行类构造器方法`<clinit>()`的过程**，该方法是javac编译器自动收集类中所有类变量的赋值动作和静态代码块中的语句合并而来。

方法中的指令按语句在源文件中出现的顺序执行，若该类由父类，JVM会先执行父类的`<clinit>()`，再执行子类的`<clinit>()`方法。多线程情况下，一个类的`<clinit>()`方法会被同步加锁。

`<clinit>()`是给类变量和静态代码块中的语句显示赋值，而`<init>()`方法是给成员变量和构造器方法显示的赋值。

可以在 idea 中安装 jclasslib Bytecode Viewer、ASM Bytecode Viewer插件进行查看类的相关字节码反编译后的指令。推荐： [idea查看字节码 bytecode 插件](https://blog.csdn.net/dataiyangu/article/details/89226410?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

### 三.类加载器的分类

![image-20200704135254558](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704135254558.png)

JVM支持两种类型的类加载器，分别为**引导类加载器**（Bootstrap ClassLoader）和**自定义类加载器**（User-Defined ClassLoader）。在Java虚拟机规范中将**自定义加载器定义为所有派生于抽象类ClassLoader的类加载器**。

拿扩展类加载器 ExtClassLoader 和系统类加载器 AppClassLoader来看，按 F4 或 Ctrl+Shift+U 查看相关信息

ExtClassLoader：

![image-20200704140043745](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704140043745.png)

AppClassLoader：

![image-20200704140141163](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704140141163.png)

扩展类加载器和系统类加载器的 Java Class Diagrams图都为：

![image-20200704140318514](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704140318514.png)

扩展类加载器和系统类加载器都派生于ClassLoader类，都属于自定义类加载器。

再举个例子加深这几个类加载器之间的关系：

```java
//获取系统类加载器
ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

//获取其上层：扩展类加载器
ClassLoader extClassLoader = systemClassLoader.getParent();
System.out.println(extClassLoader);//sun.misc.Launcher$ExtClassLoader@1540e19d

//获取其上层：获取不到引导类加载器
ClassLoader bootstrapClassLoader = extClassLoader.getParent();
System.out.println(bootstrapClassLoader);//null

//对于用户自定义类来说：默认使用系统类加载器进行加载
ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
System.out.println(classLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

//String类使用引导类加载器进行加载的。---> Java的核心类库都是使用引导类加载器进行加载的。
ClassLoader classLoader1 = String.class.getClassLoader();
System.out.println(classLoader1);//null
```

上面例子显示 Java核心类库都是用引导类加载器进行加载的，引导类加载器（C或C++编写）比较高贵，一般平民是联系不到的；自定义类默认使用系统类加载器（AppClassLoader）进行加载，再向上层扩展类加载器（ExtClassLoader）去找。

#### 1.虚拟机自带类加载器

**引导类加载器**(Bootstrap ClassLoader)：也称为启动类加载器。由C/C++语言编写，嵌套在JVM内部，用来**加载Java核心库**（JAVA_HOME/jre/lib/rt.jar、resource.jar等内容），提供JVM自身需要的类。引导类加载器没有继承ClassLoader类，**加载扩展类和系统类加载器并指定为父类加载器**。只加载包名为java、javax、sun等开头的类。

**扩展类加载**(Extension ClassLoader)：由Java语言编写，派生于ClassLoader类，父类为启动类加载器。**从JDK的安装目录的 jre/lib/ext 子目录（扩展目录）下加载类库**。若自己创建的JAR放在此目录下，也就自动由扩展类加载器加载。

**应用类加载器**(AppClassLoader)：由Java语言编写，派生于ClassLoader类，父类为扩展类加载器。**加载环境变量classpath或系统属性path指定路径下的类库**。一般Java应用的类都是由它来完成加载的。

![image-20200704202111749](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704202111749.png)

举个例子来说明各个加载器加载的类库：

```java
System.out.println("**********启动类加载器**************");
//获取BootstrapClassLoader能够加载的api的路径
URL[] urLs = sun.misc.Launcher.getBootstrapClassPath().getURLs();
for (URL element : urLs) {
    System.out.println(element.toExternalForm());
}
//从上面的路径中随意选择一个类,来看看他的类加载器是什么:引导类加载器
ClassLoader classLoader = Provider.class.getClassLoader();
System.out.println(classLoader);

System.out.println("***********扩展类加载器*************");
String extDirs = System.getProperty("java.ext.dirs");
for (String path : extDirs.split(";")) {
    System.out.println(path);
}

//从上面的路径中随意选择一个类,来看看他的类加载器是什么:扩展类加载器
ClassLoader classLoader1 = CurveDB.class.getClassLoader();
System.out.println(classLoader1);//sun.misc.Launcher$ExtClassLoader@1540e19d
```

引导类加载器主要加载的是 `JAVA_HOME/jre/lib` 下子目录的类库，扩展类加载器主要加载的是 `jre/lib/ext` 下子目录的类库，应用类加载器主要加载的是环境变量classpath或系统属性path指定路径下的类库。

![image-20200704145504406](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704145504406.png)

#### 2.用户自定义类加载器

`sun.misc.Launcher`是java虚拟机的入口应用。

![image-20200704163715991](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704163715991.png)

ClassLoader类是一个抽象类，除了启动类加载器外，所有的类加载器都继承ClassLoader类。

![image-20200704161155558](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704161155558.png)

获取ClassLoader的途径：

- 获取当前类的ClassLoader：``class.getClassLoader()``
- 获取当前线程上下文的ClassLoader：`Thread.currentThread().getContextClassLoader()`
- 获取系统的ClassLoader：`ClassLoader.getSystemClassLoader()`
- 获取调用者的ClassLoader：`DriverManager.getCallerClassLoader()`

自定义类加载器应用场景：隔离加载类、修改类加载的方式、扩展加载源、防止源码泄露。

自定义类加载器实现步骤：

1. 继承抽象类ClassLoader类
2. JDK1.2之前，重写loadClass方法；JDK1.2之后建议重写findClass方法
3. 若没有复杂的需求，可以直接继承URLClassLoader类，避免重写findClass方法及获取字节码流的方式

案例：

```java
public class CustomClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            byte[] result = getClassFromCustomPath(name);
            if(result == null){
                throw new FileNotFoundException();
            }else{
                return defineClass(name,result,0,result.length);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        throw new ClassNotFoundException(name);
    }

    private byte[] getClassFromCustomPath(String name){
        //从自定义路径中加载指定类:细节略
        //如果指定路径的字节码文件进行了加密，则需要在此方法中进行解密操作。
        return null;
    }

    public static void main(String[] args) {
        CustomClassLoader customClassLoader = new CustomClassLoader();
        try {
            Class<?> clazz = Class.forName("One",true,customClassLoader);
            Object obj = clazz.newInstance();
            System.out.println(obj.getClass().getClassLoader());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 四.双亲委派机制

Java虚拟机对class文件采用的是**按需加载**方式，当需要时才会将它的class文件加载到内存中生成class对象。加载过程中，采用的是**双亲委派模式**，把请求委托给父类处理，父类处理不了就依次委派给子类再进行处理。

![image-20200704170437351](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704170437351.png)

- 类加载器接收到类加载请求，会把该请求委派给父类加载器去执行
- 父类加载器还存在父类加载器，则进一步向上委托，依次递归，最终请求到达顶层的启动类加载器
- 启动类加载器开始执行类加载请求，若处理不了，就反向委派给子类加载器进行加载，依次递归反向委派，若可以处理，就成功返回

优势：避免类的重复加载；保护程序安全，防止核心API被随意篡改。

双亲委派机制案例：

![image-20200704171708396](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200704171708396.png)

使用到SPI核心类中的接口时，依次递归委派给引导类加载器，引导类加载器加载rt.jar中API接口核心类，当接口调用实现类中方法时，由于是第三方jar包，启动类加载器反向委派给线程上下文类加载器(系统类加载器)进行加载jdbc.jar,也就是加载SPI接口的实现类。

**沙箱安全机制**：自定义string类,但是在加载自定义string类的时候会率先使用引导类加载器加载,而引导类加载器在加载的过程中会先加载jdk自带的文件. (rt.jar包中java\lang\string.class),报错信息说没有main方法,就是因为加载的是rt.jar包中的string类。这样可以保证对java核心源代码的保护,这就是沙箱安全机制。

### 五.其它重要内容

JVM中**表示两个class对象是否为同一个类存在两个必要条件**：

- 类的全限定类名必须相同
- 加载这个类的ClassLoader必须相同

当同一个class文件被不同的ClassLoader实例对象加载，这两个类对象也是不相等的。

**当一个类型是由自定义加载器加载的，JVM会把这个类加载器的引用作为类型信息的一部分保存到方法区**。当解析一个类型到另一个类型的引用时，JVM要保证这两个类型的类加载器是相同的。

Java程序对类的使用方式分为：主动使用和被动使用。

**主动使用**分为下列七种情况:

- 创建类的实例
- 访问某个类或接口的静态变量,或者对该静态变量赋值
- 调用类的静态方法
- 反射(比如: Class.forName ("com.atguigu.Test"))
- 初始化一个类的子类
- Java虚拟机启动时被标明为启动类的类
- JDK 7开始提供的动态语言支持: `java.lang.invoke.MethodHandle`实例的解析结果`REF_getStatic, REF _putStatic, REF_invokeStatic`句柄对应的类没有初始化,则初始化

除了以上七种情况,其他使用Java类的方式都被看作是对**类的被动使用**,都**不会导致类的初始化**。



推荐:

- [JVM成神之路-类加载机制-双亲委派,破坏双亲委派](https://blog.csdn.net/w372426096/article/details/81901482)
- [深度分析Java的ClassLoader机制(源码级别)](https://www.hollischuang.com/archives/199)
- [Java类的加载,链接和初始化](https://www.hollischuang.com/archives/201)
- [JVM类加载机制详解(二)类加载器与双亲委派模型](https://blog.csdn.net/zhangliangzi/article/details/51338291)

