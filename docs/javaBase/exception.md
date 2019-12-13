### 摘要

- <span style="color:red;">世界上最遥远的距离，是我在if 里你在else里，似乎一直相伴又永远分离；</span>
- <span style="color:red;">世界上最痴心的等待，是我当case你是switch，或许永远都选不上自己；</span>
- <span style="color:red;">世界上最真情的相依，是你在try我在catch，无论你发神马脾气，我都默默承受，静静处理。到那时，再来期待我们的finally。</span>

哈哈，开心一笑也笑了，异常体系更完java基础板块就正式结束啦，下面是正文。

### 一.异常概述与异常体系结构

异常：在Java语言中，将程序执行中发生的不正常情况称为“异常”。(开发过程中的语法错误和逻辑错误不是异常)。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191213184826621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

Java把异常当作对象来处理，并定义一个基类`java.lang.Throwable`作为所有异常的超类，有`两个子类Error和Exception`，分别表示错误和异常。

Java程序在执行过程中所发生的异常事件可分为两类：

- **Error**：**Java虚拟机无法解决的严重问题**。如：JVM系统内部错误、资源耗尽等严重情况。比如：**StackOverflowError**(堆栈溢出)和OOM(**OutOfMemoryError**内存用完)。一般不编写针对性的代码进行处理。
- **Exception**：其它因**编程错误或偶然的外在因素**导致的一般性问题，可以使用针对性的代码进行处理。例如：空指针异常、数组角标越界。

java中的Exception类的子类不仅仅只是像上图所示只包含IOException和RuntimeException这两大类，Exception的子类很多很多，主要可概括为`运行时异常(RuntimeException)`和`非运行时异常`，也称为`不检查异常(Unchecked Exception)`和`检查异常(Checked Exception)`。

- **运行时异常**：都是RuntimeException类及其子类异常，如`NullPointerException、IndexOutOfBoundsException`等， 这些异常是**不检查异常**，程序中**可以选择捕获处理，也可以不处理**。这些异常一般是由**程序逻辑错误**引起的，程序应该从逻辑角度尽可能避免这类异常的发生。
- **非运行时异常**：是RuntimeException以外的异常，**编译期间可以检查到的异常**，类型上都属于Exception类及其子类。从程序语法角度讲是**必须进行处理的异常，如果不处理，程序就不能编译通过**。 如`IOException、SQLException`等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

### 二.try-catch-finally

Java提供的是异常处理的**抓抛模型**。

Java程序的执行过程中如出现异常，会生成一个**异常类对象**，该异常对象将被提交给Java运行时系统，这个过程称为**抛出(throw)异常**。

#### 1.异常对象的生成

**由虚拟机自动生成**：程序运行过程中，虚拟机检测到程序发生了问题，如果在当前代码中没有找到相应的处理程序，就会在后台自动创建一个对应异常类的实例对象并抛出——自动抛出

**由开发人员手动创建**：Exception exception = new ClassCastException();——创建好的异常对象不抛出对程序没有任何影响，和创建一个普通对象一样

#### 2.异常抛出机制

若执行try块的过程中没有发生异常，则跳过catch子句。若是出现异常，try块中剩余语句不再执行。开始逐步检查catch块，判断catch块的异常类实例是否是捕获的异常类型。匹配后执行相应的catch块中的代码。

如果异常没有在当前的方法中被捕获，就会被**传递给该方法的调用者**。如果异常没有在调用者方法中处理，它继续被抛给这个**调用方法的上层方法**。这个过程一直重复，直到异常被捕获或被传给main方法（交给JVM来捕获），这一过程称为**捕获(catch)异常**。

如果一个异常**回到main()方法，并且main()也不处理，则程序运行终止**。

程序员通常只能处理Exception，而对Error无能为力。

#### 3.异常处理机制

异常处理是通过try-catch-finally语句实现的。

```java
try{
	...... //可能产生异常的代码
}
catch( ExceptionName1 e ){
	...... //当产生ExceptionName1型异常时的处置措施
}
catch( ExceptionName2 e ){
	...... //当产生ExceptionName2型异常时的处置措施
}
[ finally{
	...... //无论是否发生异常， 都无条件执行的语句
} ]
```

- **try**

  捕获异常的第一步是用try{…}语句块选定捕获异常的范围，将可能出现异常的代码放在try语句块中。

- **catch( Exception e)**

  在catch语句块中是对**异常对象**进行处理的代码。每个try语句块可以伴随一个或**多个catch**语句。对于try里面发生的异常，他会根据发生的异常和catch里面的进行匹配(按照catch块从上往下匹配)，**如果有匹配的catch，它就会忽略掉这个catch后面所有的catch。**

  在catch中可以捕获异常的有关信息：

  - e.getMessage()：获取异常信息，返回字符串
  - e.printStackTrace()：获取异常类名和异常信息及异常出现在程序中的位置。返回值void

- **finally**

  不论在try代码块中是否发生了异常事件，catch语句是否执行，catch语句是否有异常，catch语句中是否有return，finally块中的语句都会被执行；

  **finally中的return、throw会覆盖try、catch中的return、throw；**

  finally语句和catch语句是任选的。

  ```java
  public int test() {
      try {
          int a = 1;
          a = a / 0;
          return a;
      } catch (Exception e) {
          return -1;
      } finally{
          return -2;
      }
  }

  // 方法返回值：-2
  ```

  当执行代码`a = a / 0;`时发生异常，try块中它之后的代码便不再执行，而是直接执行catch中代码；在catch块中，当在执行`return -1`前，先会执行finally块；由于finally块中有return语句，因此catch中的return将会被覆盖，直接执行fianlly中的`return -2`后程序结束。因此输出结果是-2。

- **注意**

  - try、catch、finally三个语句块均**不能单独使用**，三者可以组成 try...catch...finally、try...catch、try...finally三种结构，catch语句可以有一个或多个，finally语句最多一个
  - try、catch、finally三个代码块中**变量的作用域为代码块内部**，分别独立而不能相互访问。如果要在三个块中都可以访问，则需要将变量定义到这些块的外面
  - 多个catch块时候，最多**只会匹配其中一个异常类且只会执行该catch块代码**，而不会再执行其它的catch块，且匹配catch语句的顺序为从上到下，也可能所有的catch都没执行
  - **先Catch子类异常再Catch父类异常**

### 三.throws

throws关键字用于方法体外部的**方法声明部分**，用来声明方法可能会抛出**某些异常类型，也可以是它的父类**。仅当抛出了检查异常，该方法的调用者才必须处理或者重新抛出该异常。当方法的调用者无力处理该异常的时候，应该继续抛出。

```java
import java.io.*;
public class ThrowsTest {
    public static void main(String[] args) {
        ThrowsTest t = new ThrowsTest();
        try {
        	t.readFile();
        } catch (IOException e) {
        	e.printStackTrace();
        }
    }
    public void readFile() throws IOException {
        FileInputStream in = new FileInputStream("data.txt");
        int b;
        while ((b=in.read()) != -1) {
            System.out.print((char) b);
        }
        in.close();
    }
}
```

**重写方法不能抛出比被重写方法范围更大的异常类型 (<=)**。在多态的情况下，对methodA()方法的调用-异常的捕获按父类声明的异常处理。

```java
public class A {
    public void methodA() throws IOException {
    ……
	}
}
public class B1 extends A {
    public void methodA() throws FileNotFoundException {
    ……
	} 
}
public class B2 extends A {
    public void methodA() throws Exception { // 报错
    ……
	} 
}
```

### 四.throw

throw关键字是用于**方法体内部**，用来抛出一个Throwable类型或其子类的异常。

如果抛出了检查异常，则还应该**在方法头部声明**方法可能抛出的异常类型。该方法的调用者也必须检查处理抛出的异常。如果所有方法都层层上抛获取的异常，最终JVM会进行处理，处理也很简单，就是打印异常消息和堆栈信息。

```java
public static void test() throws Exception  
{  
   throw new Exception("方法test中的Exception");  
}  
```

#### throw和throws关键字的区别

- 写法上 : throw 在**方法体**内使用，throws **函数名后或者参数列表后**方法体前 
- 意义 ： throw 强调**动作**，而throws 表示**一种倾向、可能但不一定实际发生** 
- throws 后面跟的是**异常类**，可以一个，可以多个，多个用逗号隔开。throw 后跟的是**异常对象**，或者异常对象的引用。 
- **throws** 用户抛出异常，在当前方法中抛出异常后，**当前方法执行结束**（throws 后，如果有finally语句的话，会执行到finally语句后再结束。）。可以理解成return一样

### 五.自定义异常类

 一般地，用户自定义异常类都是RuntimeException的子类。

- 自定义异常类通常需要编写几个重载的构造器。
- 自定义异常需要提供serialVersionUID
- 自定义的异常通过throw抛出。
- 自定义异常最重要的是异常类的名字，当异常出现时，可以根据名字判断异常类型。

语法：

```java
// 1.定义异常类
class  自定义异常类 extends 异常类型(Exception) {
    // 2.重载构造方法
    
    // 3.将异常信息传递给父类：super(...)

}
```

示例：

```java
// 备注： 这些方法怎么来的？ 重写父类Exception的方法
public class CustomException extends Exception {
    static final long serialVersionUID = 13465653435L;
    
    //无参构造方法
    public CustomException(){
        super();
    }

    //有参的构造方法
    public CustomException(String message){
        super(message);
    }

    // 用指定的详细信息和原因构造一个新的异常
    public CustomException(String message, Throwable cause){
        super(message,cause);
    }

    //用指定原因构造一个新的异常
     public CustomException(Throwable cause) {
         super(cause);
     }
}
```

### 总结: 异常处理的5个关键字

**捕获异常：**

- try：执行可能产生异常的代码
- catch：捕获异常
- finally：无论是否发生异常，代码总被执行

**抛出异常：**

- throw：异常的生成阶段：手动抛出异常对象

**声明异常：**

- throws：异常的处理方式：声明方法可能要抛出的各种异常类


