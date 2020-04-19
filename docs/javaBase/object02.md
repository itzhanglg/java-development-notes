### 1.关键字

#### 1.this

##### 1.理解

-   this 可以调用类的属性、方法和构造器
-   它在方法内部使用，即这个方法所属对象的引用(**谁调用就是谁的引用**)
-   它在构造器内部使用，表示该构造器正在初始化的对象

##### 2.this调用属性与方法

```java
class Person { // 定义Person类
	private String name ;
	private int age ;
	public Person(String name,int age){
		this.name = name ;	// this调用属性
		this.age = age ; 
    }
	public void getInfo(){
		System.out.println("姓名：" + name) ;
		this.speak();	// this调用方法
	}
  	public void speak(){
  		System.out.println(“年龄：” + this.age);
  	}
}
```

-   在任意方法或构造器内，如果使用当前类的成员变量或成员方法可以在其前面添加this，增强程序的阅读性。不过，通常我们都习惯省略this
-   **当形参与成员变量同名时，如果在方法内或构造器内需要使用成员变量，必须添加this来表明该变量是类的成员变量**
-   **使用this访问属性和方法时，如果在本类中未找到，会从父类中查找**

##### 3.this调用本类的构造器

```java
class Person{ // 定义Person类
	private String name ;
	private int age ;
	public Person(){ // 无参构造器
		System.out.println("新对象实例化") ;
	}
	public Person(String name){
		this(); // 调用本类中的无参构造器
		this.name = name ;
	}
	public Person(String name,int age){
		this(name) ; // 调用有一个参数的构造器
		this.age = age;
	}
	public String getInfo(){
		return "姓名：" + name + "，年龄：" + age ;
	}
}
```

-   **可以在类的构造器中使用"this(形参列表)"的方式，调用本类中重载的其他的构造器, 构造器中不能通过"this(形参列表)"的方式调用自身构造器**
-   **在类的一个构造器中，最多只能声明一个"this(形参列表)" 且 必须声明在类的构造器的首行**

#### 2.super

##### 1.使用

使用super调用父类中的指定操作：

- super可用于访问父类中定义的**属性**
- super可用于调用父类中定义的**成员方法**
- super可用于在子类构造器中调用父类的**构造器**

##### 2.注意

- 子父类出现同名成员时，可以用super表明调用的是父类的成员

- **super的追溯不仅限于直接父类**

- super代表父类内存空间的标识，this代表本类对象的引用

- 每个构造器的第一行默认都隐含有`super()`语句，当显示声明时，隐式会自动被覆盖。

  ```java
  class Person {
      protected String name = "张三";
      protected int age;
      public String getInfo() {
      	return "Name: " + name + "\nage: " + age;
      }
  }
  class Student extends Person {
      protected String name = "李四";
      private String school = "New Oriental";
      public String getSchool() {
      	return school;
      }
      // super调用父类中成员方法
      public String getInfo() {
          return super.getInfo() + "\nschool: " + school;
      }
  }
  public class StudentTest {
      public static void main(String[] args) {
      	Student st = new Student();
      	System.out.println(st.getInfo());
      }
  }
  ```

##### 3.调用父类的构造器

子类中所有的构造器 **默认**都会访问父类中 **空参数**的构造器，当父类中没有空参数的构造器时，子类的构造器必须通过**this( 参数列表)或者super( 参数列表)**语句指定调用本类或者父类中**相应的构造器**。同时，只能”二选一”，且必须放在**构造器的首行**

##### 4.this与super区别

| 区别点     | this                                                   | super                                    |
| ---------- | ------------------------------------------------------ | ---------------------------------------- |
| 访问属性   | 访问本类中的属性，如果本类没有此属性则从父类中继续查找 | 直接访问父类中的属性                     |
| 调用方法   | 访问本类中的方法，如果本类没有此方法则从父类中继续查找 | 直接访问父类中的方法                     |
| 调用构造器 | 调用本类构造器，必须放在构造器的首行                   | 调用父类构造器，必须放在子类构造器的首行 |

#### 3.static

在java类中，可用static修饰属性、方法、代码块、内部类。

被修饰后的成员具备的提点：

- 随着**类的加载而加载**
- **优先于对象存在**
- 修饰的成员，被**所有对象所共享**
- 访问权限允许时，可不创建对象，**直接被类调用**

内存解析：

**栈**：局部变量；**堆**：new出来的结构，对象、数组； **方法区**：类的加载信息、静态变量、常量。

要点：

- 没有对象的实例时，可以用`类名.属性`, `类名. 方法名()`的形式访问由static修饰的类属性和类方法
- 在static 方法内部只能访问类的`static 修饰的属性或方法`，不能访问类的非static的结构
- 因为不需要实例就可以访问static 方法，因此**static方法内部不能有this，也不能有super**
- **static修饰的方法不能被重写**，因为static方法属于类

#### 4.final

在Java中声明**类、变量和方法**时，可使用关键字final来修饰，表示“最终的”。static final：全局常量。

- final标记的**类不能被继承**：如String类、System类、StringBuffer类，简称为"太监类"
- final标记的**方法不能被子类重写**
- final标记的**变量(成员变量或局部变量)即称为常量，名称大写，且只能被赋值一次**，如同圣旨

final标记的成员变量必须在声明时或在每个构造器中或代码块中**显式赋值**，然后才能使用。
`final double MY_PI = 3.14`

示例：

```java
public final class Test {
    public static int totalNumber = 5;
    public final int ID;
    
    public Test() {
    	ID = ++totalNumber; // 可在构造器中给final修饰的“变量”赋值
    }
    public static void main(String[] args) {
    	Test t = new Test();
    	System.out.println(t.ID);
    	final int I = 10;
    	final int J;
    	J = 20;
    	J = 30; // 非法
    }
}
```

#### 5.package

package语句作为Java**源文件的第一条语句**，指明该文件中定义的类所在的包.

格式:  `package 顶层包名. 子包名 ;`

包对应于文件系统的目录，package 语句中，用 “.”  来指明包( 目录) 的层次.

包通常用**小写单词**标识。通常使用所在公司域名的倒置.

#### 6.import

为使用定义在不同包中的Java类，需用import语句来引入指定包层次下所需要的类或全部类。**import语句告诉编译器到哪里去寻找类**

语法格式:  `import 包名.类名;`

-   import语句在**包的声明和类的声明**之间
-   可以使用`java.util.* `的方式，一次性导入util包下所有的类或接口
-   如果导入的类或接口是**java.lang包**下的，或者是**当前包**下的，则可以省略此import语句
-   如果在代码中使用不同包下的同名的类。那么就需要使用**类的全类名**的方式指明调用的是哪个类
-   已经导入java.a包下的类,要使用a包的子包下的类，仍然需要导入

### 2.常见问题

#### 1.方法重载与重写

##### 1.方法重载(overload)

- 重载的概念

  在**同一个类中**，允许存在一个以上的**同名方法**，只要它们的**参数个数或者参数类型不同**即可

- 重载的特点

  **与返回值类型无关**，只看参数列表，且参数列表必须不同(参数个数或参数类型)。**调用时，根据方法参数列表的不同来区别**

- 重载示例

  ```java
  // 方法重载：同一个类中，方法名相同，参数列表不同，与返回值无关
  // 返回两个整数的和
  int add(int x,int y){return x+y;}
  // 返回三个整数的和
  int add(int x,int y,int z){return x+y+z;}
  // 返回两个小数的和
  double add(double x,double y){return x+y;}
  ```

##### 2.方法重写(override)

在子类中可以根据需要对**从父类中继承来的方法进行改造**，也称为方法的重置、覆盖。**在程序执行时，子类的方法将覆盖父类的方法。**

要求：

- 访问权限：子类不能小于父类 (>=)，不能重写private修饰的方法
- 返回值类型：子类不能大于父类 (<=)
- 方法名称、参数列表：子类与父类必须相同 (==)
- 方法抛出的异常：子类不能大于父类 (<=)
- 必须都为非static的方法。static方法不是重写，static方法属于类，子类无法覆盖父类的方法

```java
// 方法重写：发生在子父类中，方法名、参数列表相同，其他要求参考上面
public class Person {
    public String name;
    public int age;
    public String getInfo() {
    	return "Name: "+ name + "\n" +"age: "+ age;
    }
}
public class Student extends Person {	// 继承
    public String school;
    public String getInfo() { //方法重写
    	return "Name: "+ name + "\nage: "+ age + "\nschool: "+ school;
    }
    
    public static void main(String args[]) {
        // 继承
    	Student s1=new Student();
    	s1.name="Bob";
    	s1.age=20;
    	s1.school="school2";
    	System.out.println(s1.getInfo()); //Name:Bob age:20 school:school2
        // 重写：多态
        Person p1=new Person();
        // 调用Person类的getInfo()方法
        p1.getInfo();
        // 调用Student类的getInfo()方法
        s1.getInfo();
        // 这是一种“多态性”：同名的方法，用不同的对象来区分调用的是哪一个方法。
    }
}
```

#### 2.四种访问权限修饰符

Java权限修饰符public、protected、(缺省)、private置于 `类的成员` 定义前，用来限定**对象对该类成员**的访问权限。

| 修饰符       | 类内部  | 同一个包 | 不同包的子类 | 同一个工程 |
| --------- | ---- | ---- | ------ | ----- |
| private   | √    |      |        |       |
| default   | √    | √    |        |       |
| protected | √    | √    | √      |       |
| public    | √    | √    | √      | √     |

对于`class的权限修饰`只可以用**public和default(缺省)**。
public类可以在任意地方被访问；default类只可以被同一个包内部的类访问。

#### 3.子类对象实例化过程

- this(...)和super(...)不能同时在一个构造器中
- this(...)和super(...)只能作为构造器中第一句
- 每个构造器的第一行默认都隐含有`super()`语句，当显示声明时，隐式会自动被覆盖

无论通过那个构造器创建子类对象，需要保证先初始化父类。

目的：当子类继承父类后，”继承“父类中所有的属性和方法，因此子类有必要知道父类如何为对象进行初始化。

**子类对象在实例化时，会默认先去调用父类中的无参构造方法，之后再调用子类本身的相应构造方法。**

```java
class Creature {
	public Creature() {
		System.out.println("Creature无参数的构造器");
	}
}
class Animal extends Creature {
    public Animal(String name) {
    	System.out.println("Animal带一个参数的构造器，该动物的name为" + name);
    }
    public Animal(String name, int age) {
    	this(name);
    	System.out.println("Animal带两个参数的构造器，其age为" + age);
    }
}
public class Wolf extends Animal {
    public Wolf() {
        super("灰太狼", 3);
        System.out.println("Wolf无参数的构造器");
    }
    public static void main(String[] args) {
        new Wolf();
    }
}

/*
Creature无参数的构造器
Animal带一个参数的构造器，该动物的name为灰太狼
Animal带两个参数的构造器，其age为3
Wolf无参数的构造器
*/
```

#### 4.Object类的使用

Object类是所有java类的根父类，如果在类的声明中未使用extends关键字指明其父类，则默认父类为java.lang.Object类。

主要方法：

| 方法名称                          | 类型 | 描述           |
| --------------------------------- | ---- | -------------- |
| public Object()                   | 构造 | 构造器         |
| public boolean equals(Object obj) | 普通 | 对象比较       |
| public int hashCode()             | 普通 | 取得Hash码     |
| public String toString()          | 普通 | 对象打印时调用 |

##### 1.toString()方法

toString()方法在Object类中定义，其返回值是String类型，返回**类名和它的引用地址**，若重写toString()方法，返回**字符串的值**；基本数据类型转换为String类型时，调用了对应**包装类的toString()方法**。

在进行String与其它类型数据连接操作时，自动调用toString()方法

```java
Date d = new Date();
System.out.println("now="+now);	// 相当于
System.out.println("now"+now.toString());

char[] arr = new char[] { 'a', 'b', 'c' };
System.out.println(arr);    // abc
int[] arr1 = new int[] { 1, 2, 3 };
System.out.println(arr1);   // [I@22927a81
double[] arr2 = new double[] { 1.1, 2.2, 3.3 };
System.out.println(arr2);   // [D@78e03bb5
```

#### 5.包装类的使用

 针对八种基本数据类型定义相应的引用类型—包装类（Wrapper封装类）

| 基本数据类型 | 包装类    |
| ------------ | --------- |
| byte         | Byte      |
| short        | Short     |
| int          | Integer   |
| long         | Long      |
| float        | Float     |
| double       | Double    |
| boolean      | Boolean   |
| char         | Character |

Byte、Short、Integer、Long、Floate、Double父类都是Number;

- 装箱：基本数据类型  -->  包装类

  通过包装类的构造器实现：int i = 300; Integer i = new Integer(i);

  还可以通过字符串参数构造包装类对象：Float f = new Float("1.23");

- 拆箱：包装类  -->  基本数据类型

  调用包装类的`.xxxValue()`方法

- JDK1.5之后，支持自动装箱，自动拆箱

**字符串与基本数据类型之间转换：**

- String  -->  基本数据类型

  通过包装类的`parseXxx(String s)`静态方法：`Float f = Float.parseFloat("12.3");`

  通过包装类的构造器：`int i = new Integer("12");`

- 基本数据类型  -->  String

  调用字符串重载的valueOf()方法：`String str = String.valueOf(2.34f);`

```java
// 包装类的运用
Integer i = new Integer(1);
Integer j = new Integer(1);
System.out.println(i == j); // false
Integer m = 1;
Integer n = 1;
System.out.println(m == n); // true
Integer x = 128;
Integer y = 128;
System.out.println(x == y); // false
```

JVM会自动维护八种基本类型的常量池，int常量池中初始化-128~127的范围，所以当为Integer i=127时，在自动装箱过程中是取自常量池中的数值，而当Integer i=128时，128不在常量池范围内，所以在自动装箱过程中需new 128，所以地址不一样。

故：**Integer类型的数据最好用equals方法进行比较**。

#### 6.理解main方法的语法

- 访问权限必须是public：由于java虚拟机需要调用类的main()方法
- 方法必须是static：java虚拟机在执行main()方法时不必创建对象
- 接受一个String类型的数组参数：数组中保存执行java命令时传递给所运行的类的参数
- 方法体中必须创建类的一个实例对象，才能通过这个对象访问类中非静态成员

```java
// main方法的编译、运行与变量名是否为args无关
public class Something {
    public static void main(String[] something_to_do) {
    	System.out.println("Do something ...");
    }
}
```

#### 7.抽象类与接口

##### 1.抽象类(abstract)

- 用abstract关键字来修饰一个类，这个类叫做抽象类

- 用abstract来修饰一个方法，该方法叫做抽象方法

  抽象方法：只有方法的声明，没有方法的实现。以分号结束。如：`public abstract void talk();`

- **含有抽象方法的类必须被声明为抽象类**

- **抽象类不能被实例化**。抽象类是用来被继承的，**抽象类的子类必须重写父类的抽象方法，并提供方法体**。若没有重写全部的抽象方法，仍为抽象类

- 不能用abstract修饰变量、代码块、构造器

- 不能用abstract修饰私有方法、静态方法、final的方法、final的类

```java
abstract class A {
	abstract void m1();
    public void m2() {
    	System.out.println("A类中定义的m2方法");
    }
}
class B extends A {
	void m1() {
		System.out.println("B类中定义的m1方法");
	}
}
public class Test {
    public static void main(String args[]) {
        A a = new B();
        a.m1();
        a.m2();
    }
}
```

应用：**模板方法设计模式(TemplateMethod)**

抽象类体现的就是一种模板模式的设计，抽象类作为多个子类的通用模板，子类在抽象类的基础上进行扩展、改造，但子类总体上会保留抽象类的行为方式。

当功能内部一部分实现是确定的，一部分实现是不确定的。这时可以把不确定的部分暴露出去，让子类去实现。

```java
abstract class Template {
    public final void getTime() {
        long start = System.currentTimeMillis();
        code();
        long end = System.currentTimeMillis();
        System.out.println("执行时间是：" + (end - start));
    }
    public abstract void code();	// 不确定的部分暴露出去
}
class SubTemplate extends Template {
    public void code() {
        for (int i = 0; i < 10000; i++) {
        	System.out.println(i);
        }
    }
}
```

示例：

```java
//抽象类的应用：模板方法的设计模式
public class TemplateMethodTest {
	public static void main(String[] args) {
		BankTemplateMethod btm = new DrawMoney();
		btm.process();

		BankTemplateMethod btm2 = new ManageMoney();
		btm2.process();
	}
}
abstract class BankTemplateMethod {
	// 具体方法
	public void takeNumber() {
		System.out.println("取号排队");
	}

	public abstract void transact(); // 办理具体的业务 //钩子方法

	public void evaluate() {
		System.out.println("反馈评分");
	}

	// 模板方法，把基本操作组合到一起，子类一般不能重写
	public final void process() {
		this.takeNumber();
		this.transact();// 像个钩子，具体执行时，挂哪个子类，就执行哪个子类的实现代码
		this.evaluate();
	}
}

class DrawMoney extends BankTemplateMethod {
	public void transact() {
		System.out.println("我要取款！！！");
	}
}

class ManageMoney extends BankTemplateMethod {
	public void transact() {
		System.out.println("我要理财！我这里有2000万美元!!");
	}
}
```

##### 2.接口(interface)

接口就是规范，定义的是一组规则，体现了现实世界中“如果你是/要...则必须能...”的思想。**继承是一个"是不是"的关系，而接口实现则是 "能不能"的关系。**

**接口的本质是契约，标准，规范**，就像我们的法律一样。制定好后大家都要遵守。

**接口(interface)是抽象方法和常量值定义的集合。**

接口的特点：

- 接口中的所有成员变量都默认是由public static final修饰的
- 接口中的所有抽象方法都默认是由public abstract修饰的
- 接口中没有构造器
- 接口采用多继承机制

```java
public interface Runner {
    public static final int ID = 1;
    public abstract void start();
    public abstract void run();
    public abstract void stop();
}
```

**理解：**

- 定义java类的语法格式：先写extends，后写implements

  `class SubClass extends SuperClass implements InterfaceA{ }`

- 一个类可以实现多个接口，接口也可以继承其它接口

- 接口和类是并列关系，或者可以理解为一种特殊的类。从本质上讲，**接口是一种特殊的抽象类**，这种抽象类中只包含**常量和方法的定义**(JDK7.0及之前)，而没有**变量和方法的实现**

  ```java
  // 实现类SubAdapter必须给出接口SubInterface以及父接口MyInterface中所有方法的实现。否则，SubAdapter仍需声明为abstract的
  interface MyInterface {
      String s=“MyInterface”;
      public void absM1();
  }
  interface SubInterface extends MyInterface {	// 接口继承接口
  	public void absM2();
  }
  public class SubAdapter implements SubInterface {
      public void absM1(){System.out.println(“absM1”);}
      public void absM2(){System.out.println(“absM2”);}
  }
  ```

**应用：**

**代理模式(Proxy)**是Java开发中使用较多的一种设计模式。代理设计就是**为其他对象提供一种代理以控制对这个对象的访问**。

```java
// 应用场景：安全代理、远程代理、延迟加载
interface Network {
	public void browse();
}

// 被代理类
class RealServer implements Network {
    @Override
    public void browse() {
        System.out.println("真实服务器上网浏览信息");
    }
}

// 代理类
class ProxyServer implements Network {
    private Network network;
    public ProxyServer(Network network) {
    	this.network = network;
    }
    public void check() {
    	System.out.println("检查网络连接等操作");
    }
    public void browse() {
        check();
        network.browse();
    }
}

public class ProxyDemo {
    public static void main(String[] args) {
        Network net = new ProxyServer(new RealServer());
        net.browse();
    }
}
```

##### 3.抽象类与接口对比

| 区别点       | 抽象类                                   | 接口                                        |
| ------------ | ---------------------------------------- | ------------------------------------------- |
| 定义         | 包含抽象方法的类                         | 主要是抽象方法和全局常量的集合              |
| 组成         | 构造方法、抽象方法、普通方法、常量、变量 | 常量、抽象方法、(jdk8.0:默认方法、静态方法) |
| 使用         | 子类继承抽象类(extends)                  | 子类实现接口(implements)                    |
| 关系         | 抽象类可以实现多个接口                   | 接口不能继承抽象类，但允许继承多个接口      |
| 常见设计模式 | 模板方法                                 | 简单工厂、工厂方法、代理模式                |
| 对象         | 通过对象的多态性产生实例化对象           | 通过对象的多态性产生实例化对象              |
| 局限         | 抽象类有单继承的局限                     | 接口没有此局限                              |
| 实际         | 作为一个模板                             | 是作为一个标准或是表示一种能力              |
| 选择         | 优先使用接口，因为避免单继承的局限       | 优先使用接口，因为避免单继承的局限          |

##### 4.java8中关于接口的改进

Java 8中，你可以为接口添加**静态方法**和**默认方法**。

**静态方法**：使用 **static 关键字**修饰。**可以通过接口直接调用静态方法，并执行其方法体**。

**默认方法**：默认方法使用 **default 关键字**修饰。可以通过**实现类对象来调用**。我们在已有的接口中提供新方法的同时，还保持了与旧版本代码的兼容性。比如：java 8 API中对Collection、List、Comparator等接口提供了丰富的默认方法。

- 一个**接口**定义了一个**默认方法**，另外一个**接口**定义了一个**同名同参数的方法**(不管是否是默认方法)，实现类同时实现这两个接口时，会出现**接口冲突**。**实现类必须覆盖接口中同名同参数的方法**，来解决冲突。
- 若一个**接口**中定义了一个**默认方法**，而**父类**中也定义了一个**同名同参数的非抽象方法**，则不会出现冲突问题。因为此时遵守： **类优先原则**。接口中具有相同名称和参数的默认方法会被忽略。

```java
interface Filial {// 孝顺的
    default void help() {
    	System.out.println("老妈，我来救你了");
	}
}
interface Spoony {// 痴情的
    default void help() {
    	System.out.println("媳妇，别怕，我来了");
	}
}

class Man implements Filial, Spoony {
    @Override
    public void help() {	// 实现类必须覆盖接口中同名同参数的方法
        System.out.println("我该怎么办呢？");
        Filial.super.help();
        Spoony.super.help();
    }
}
```

#### 8.成员变量赋值的执行顺序

属性赋值的位置及先后顺序：1 --> 2 --> 3 --> 4

1. 默认初始化：声明时成员变量的默认初始化，`int age;`默认初始化为0。
2. 显式初始化、多个初始化块依次被执行(同级别下按先后顺序执行)：给属性赋值，`int age = 6;`
3. 构造器中初始化：`Person person = new Person(6);`
4. 通过“对象.属性“或“对象.方法”的方式可多次赋值：`person.age = 6; person.setAge(6);`


#### 9.instanceof操作符

`x instanceof A` ：检验x 是否为类A 的对象，返回值为boolean。x属于类A及其子类对象。

#### 10.对象类型转换

##### 1.基本数据类型

- 自动类型转换：小的数据类型可以自动转换成大的数据类型。如long g=20; double d=12.0f。
- 强制类型转换：可以把大的数据类型强制转换成小的数据类型。如 float f=(float)12.0; int a=(int)1200L。

##### 2.对java对象的强制类型转换称为造型

- 从子类到父类的类型转换可以自动进行 (向上转型：多态)
- 从父类到子类的类型转换必须通过造型( 强制类型转换) 实现 (向下转型：使用instanceof进行判断)
- **无继承关系的引用类型间的转换是非法的**

#### 11.equals方法与==操作符

##### 1.==操作符

基本类型比较：比较数据值

引用类型比较：比较地址 (是否指向同一个对象)

##### 2.equals()

只能比较引用类型，比较是否指向同一个对象；

特例：对File、String、Date及包装类是比较内容(重写了Object类的equals方法)

##### 3.重写equals的原则

- 对称性：x.equals(y) -> true；y.equals(x) -> true
- 自反性：x.equals(x) -> true
- 传递性：x.equals(y) -> true；y.equals(z) -> true；z.equals(x) -> true
- 一致性：x.equals(y) -> true；只要内容不变，不管重复x.equals(y)多少次，返回都是"true"










