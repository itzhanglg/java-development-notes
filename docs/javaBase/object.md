### 一.面向对象

#### 1.面向过程与面向对象

POP与OOP都是一种思想，面向对象是相对于面向过程而言的。面向过程，强调的是`功能行为，以函数为最小单位，考虑怎么做`。面向对象，将功能封装进对象，强调`具备了功能的对象，以类/对象为最小单位，考虑谁来做`。

程序员从面向过程的**执行者**转化成了面向对象的**指挥者。**

面向对象分析问题的思路和步骤：

- 选择问题所针对的**现实世界中的实体**
- 从实体中寻找解决问题相关的**属性和功能**
- 把抽象的实体用计算机语言进行描述，形成计算机世界中**类的定义**
- 将**类实例化成计算机世界中的对象**

#### 2.类与对象

- 类(Class)和对象(Object)是面向对象的核心概念

  - 类是对一类事物的描述，是**抽象的、概念上的定义**
  - 对象是**实际存在的该类事物的每个个体**，因而也称为实例(instance)

- 常见类的成员

  - 属性：对应类中的成员变量(考虑修饰符、属性类型、属性名、初始化值)
  - 行为：对应类中的成员方法(考虑修饰符、返回值类型、方法名、形参)

- 类的语法格式

  ```java
  /*
  修饰符 class 类名 {
      属性声明;
      方法声明;
  }
  说明：修饰符public：类可以被任意访问；类的正文要用{ }括起来
  */

  public class Person {
  	private int age;	// 属性声明
      public void showAge(int i) {	// 方法声明
      	age = i;
      }
  }
  ```

#### 3.对象的创建和使用

创建对象语法：`类名 对象名 = new 类名();`使用`对象名.对象成员`的方式访问对象成员(包括属性和方法)。

如果创建了一个类的多个对象，对于类中定义的属性，**每个对象都拥有各自的一套副本，且互不干扰**。

匿名对象：不定义对象的句柄，而直接调用这个对象的方法。这样的对象叫做匿名对象。例：`new Person().show();`；经常将**匿名对象作为实参传递给一个方法调用**。

1. 类的访问机制：
   - 在一个类中的访问机制：类中的方法可以直接访问类中的成员变量(static方法访问非static，编译不通过)
   - 在不同类中的访问机制：先创建要访问类的对象 ， 再用对象访问类中定义的成员


2. 对象的生命周期：
   - **当对象失去所有的引用**（没有变量再指向它了（没有变量在存储它的地址）- 相当于失联了，我们无法再使用它了）-- 就是死亡了;（垃圾回收器 并不是立刻进行回收）
     - `Person p1 = new Person();	// person对象被p1引用`
   - `Person p2 =  p1;    // 将p2指向p1所引用的对象`
   - `p1 = null;    // p1为null，但p2还是引用了person对象，person对象不会被回收`

3. 内存解析：
   - `堆（Heap）`：**存放对象实例**，几乎所有的对象实例都在这里分配内存。这一点在Java虚拟机规范中的描述是：**所有的对象实例以及数组都要在堆上分配**。
   - `栈（Stack）`：**存放局部变量**，局部变量表存放了各种基本数据类型、对象引用(对象在堆内存放的首地址)。**方法执行完，自动释放**。
   - `方法区（Method Area）`：存放已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

### 二.类的成员

#### 1.属性

##### 1.语法格式：

`修饰符 数据类型 属性名 = 初始化值；`

- 常见的权限修饰符有：private、缺省、protected、public。其他：static、final
- 数据类型：基本数据类型、引用数据类型
- 属性名：属于标识符，满足命名规则和规范

##### 2.分类：

- 成员变量：方法体外，类体内声明的变量
  - 实例变量（不以static修饰）
  - 类变量（以static修饰）
- 局部变量：方法体内部声明的变量
  - 形参（方法、构造器中定义的变量）
  - 方法局部变量（方法内定义）
  - 代码块局部变量（代码块内定义）
- 相同：都有生命周期； 异：局部变量除形参外，均需显式初始化

|        | 成员变量                         | 局部变量                  |
| ------ | ---------------------------- | --------------------- |
| 声明的位置  | 声明在类中                        | 方法形参或内部、代码块内、构造器内等    |
| 修饰符    | private、public、static、final等 | 不能用权限修饰符修饰，可以用final修饰 |
| 初始化值   | 有默认初始化值                      | 没有默认初始化值，必须显式赋值，方可使用  |
| 内存加载位置 | 堆空间 或 静态域内                   | 栈空间                   |

##### 3.对象属性的默认初始化赋值：

当一个对象被创建时，会对其中各种类型的成员变量自动进行初始化赋值。

| 成员变量类型  | 初始值                  |
| ------- | -------------------- |
| byte    | 0                    |
| short   | 0                    |
| int     | 0                    |
| long    | 0L                   |
| float   | 0.0F                 |
| double  | 0.0                  |
| char    | 0 或写为：'\u0000'(表现为空) |
| boolean | false                |
| 引用类型    | null                 |

#### 2.方法

##### 1.概念：

方法是类或对象行为特征的抽象，用来完成某个功能操作；将功能封装为方法的目的是，可以实现代码重用，简化代码；Java里的方法不能独立存在，所有的方法必须定义在类里。

##### 2.格式：

```java
修饰符 返回值类型 方法名 (参数类型 形参1，参数类型 形参2，...) {
	方法体程序代码；
	return 返回值；
}
// 修饰符：public, 缺省,private, protected 等
// 返回值类型：
// 	   没有返回值：void
//	   有返回值：与方法体中"return 返回值"搭配使用

int num = getNum(3,5);	// 为x,y分配内存并传值
public int getNum(int x,int y){
    return x*y;
}	// 方法执行完后, x,y被释放
```

##### 3.注意：

- 方法通过方法名被调用，且只有被调用才会执行，方法被调用一次，就会执行一次。
- 没有具体返回值的情况，返回值类型用关键字void表示，那么方法体中可以不必使用return语句。如果使用，仅用来结束方法。
- 方法中只能调用方法或属性，不可以在方法内部定义方法

##### 4.个数可变的形参：

JavaSE 5.0 中提供了Varargs(variable number of arguments)机制，允许直接定义能和多个实参相匹配的形参。

```java
//JDK 5.0以前：采用数组形参来定义方法，传入多个同一类型变量
public static void test(int a ,String[] books);
//JDK5.0：采用可变个数形参来定义方法，传入多个同一类型变量
public static void test(int a ,String…books);
```

说明：

- 声明格式：`方法名(参数的类型名 ...参数名)`
- 可变参数：方法参数部分指定类型的参数个数是可变多个：**0个，1个或多个**
- 可变个数形参的方法与同名的方法之间，彼此构成重载
- 可变参数方法的使用与方法参数部分使用数组是一致的
- 方法的参数部分有可变形参，需要**放在形参声明的最后**
- 在一个方法的形参位置，最多**只能声明一个可变个数形参**

##### 5.方法参数的值传递机制(重要)：

Java里方法的参数传递方式只有一种：**值传递**。 即将实际参数值的副本（复制品）传入方法内，而参数本身不受影响。

- 形参是**基本数据类型**：将实参基本数据类型变量的“**数据值**”传递给形参
- 形参是**引用数据类型**：将实参引用数据类型变量的“**地址值**”传递给形参

基本数据类型的参数传递：

```java
// 基本数据类型传递数据值
// main方法与change方法中x变量不是同一个地址
public static void main(String[] args) {
    int x = 5;	// 地址如：0x1122 --> 5
    System.out.println("修改之前x = " + x);// 5
    // x是实参
    change(x);	// 0x1122 --> 5
    System.out.println("修改之后x = " + x);// 5
}

public static void change(int x) {	// 地址如：0x3344 --> 5
    System.out.println("change:修改之前x = " + x);// 5
    x = 3;	// 0x3344 --> 3
    System.out.println("change:修改之后x = " + x);// 3
}
```

引用数据类型的参数传递：

```java
// 引用数据类型传递引用值
public static void main(String[] args) {
    Person obj = new Person();	// 0x1122
    obj.age = 5;	// 0x1122  age->5
    System.out.println("修改之前age = " + obj.age);// 5
    // 操作同一个引用
    change(obj);	// 0x1122  age->3
    System.out.println("修改之后age = " + obj.age);// 3
    // 操作不同的引用
    change2(obj);	// 0x1122  age->5
    System.out.println("修改之后age = " + obj.age);// 5
}

// change 操作同一引用
public static void change(Person obj) {	// 0x1122
    System.out.println("change:修改之前age = " + obj.age);  // 0x1122  age->5
    obj.age = 3;	// 0x1122  age->3
    System.out.println("change:修改之后age = " + obj.age);	// 0x1122  age->3
}

// change2 操作不同的引用
public static void change2(Person obj) {// 0x1122
    obj = new Person();	// 0x3344
    System.out.println("change:修改之前age = " + obj.age);	// 0
    obj.age = 3;	// 0x3344  age->3
    System.out.println("change:修改之后age = " + obj.age);	// 3
}
```

示例：

```java
//传递数据值，操作不同的地址
main(){
    int m = 10;
    int n = 20;
    v.swap(m,n);
    sysout(m,n);	//m=10,n=20
}
swap(int m ,in n){
    int temp = m;
    m = n;
    n = temp;
    sysout(m,n);	//m=20,n=10
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191210195320488.png)

```java
// 传递引用值，操作同一地址
main(){
    Data data = new Data();
    data.m = 10;
    data.n = 20;
    v.swap(data);
    sysout(data.m,data.n);	//m=20,n=10
}
swap(Data data){
    int temp = data.m;	//temp=10
    data.m = data.n;	//m=20
    data.n = temp;		//n=10
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191210195348321.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

##### 6.递归方法：

**递归方法：一个方法体内调用它自身**。

方法递归包含了一种隐式的循环，它会重复执行某段代码，但这种重复执行无须循环控制。

递归一定要向已知方向递归，否则这种递归就变成了无穷递归，类似于死循环。

示例：

```java
// 计算1-100所有自然数的和
int total = sum(100);
public int sum(int num){	// num=100
    if(num == 1){
    	return 1;
    }else{
    	return num + sum(num - 1);
    }
}
```

#### 3.构造器

##### 1.特征

- 与类相同的名称
- 不声明返回值类型(与声明为void不同)
- 不能被static、final、synchronized、abstract、native修饰
- 不能有return语句返回值

##### 2.作用

- 创建对象
- 给对象进行初始化

##### 3.语法格式

```java
修饰符 类名 (参数列表){
    初始化语句;
}

// 示例
public class Animal {
    private int legs;	// 封装属性
    public Animal(int i){	// 构造器
        legs = i;
    }
}
```

##### 4.分类

- 隐式无参构造器(系统默认提供)
- 显式定义一个或多个构造器(无参、有参)

注意：

- **每个类至少有一个构造器**(系统默认提供的无参构造器)
- **默认构造器的修饰符与所属类的修饰符一致**
- **一旦显式定义了构造器，则系统不再提供默认构造器**
- **一个类可以创建多个重载的构造器**
- **父类的构造器不可被子类继承**

##### 5.构造器重载

构造器一般用来创建对象的同时初始化对象。

构造器重载使得对象的创建更加灵活，方便创建各种不同的对象。

```java
public class Person{
    // 一旦显式定义了构造器，则系统不再提供默认构造器
    public Person(String name, int age, Date d) {this(name,age);…}
    public Person(String name, int age) {…}
    public Person(String name, Date d) {…}
    public Person(){…}	//若需要无参的构造器，需显式定义
}
```

#### 4.代码块



#### 5.内部类



### 三.OOP特征

#### 1.封装与隐藏

程序设计追求“高内聚，低耦合”。
**高内聚 ：类的内部数据操作细节自己完成，不允许外部干涉；**
**低耦合 ：仅对外暴露少量的方法用于使用。**

**隐藏对象内部的复杂性，只对外公开简单的接口**。便于外界调用，从而提高系统的可扩展性、可维护性。通俗的说，把该隐藏的隐藏起来，该暴露的暴露出来。这就是封装性的设计思想。

使用者对类内部定义的属性( 对象的成员变量) 的直接操作会导致数据的错误、混乱或 安全性 问题。

Java中通过将数据声明为私有的(private)，再提供公共的（public）方法:getXxx() 和setXxx()实现对该属性的操作，以实现下述目的：**使用者只能通过事先定制好的方法来访问数据，可以方便地加入控制逻辑，限制对属性的不合理操作**。

```java
class Animal {
    private int legs;// 将属性legs定义为private，只能被Animal类内部访问
    public void setLegs(int i) { // 在这里定义方法 eat() 和 move()
        if (i != 0 && i != 2 && i != 4) {
        	System.out.println("Wrong number of legs!");
        	return;
        }
        legs = i;
    }
    public int getLegs() {
    	return legs;
    }
}
public class Zoo {
    public static void main(String args[]) {
        Animal xb = new Animal();
        xb.setLegs(4); // xb.setLegs(-1000);
        //xb.legs = -1000; // 非法
        System.out.println(xb.getLegs());
    }
}
```

#### 2.继承性

#### 3.多态性



### 四.关键字

#### 1.this

##### 1.理解

-   this 可以调用类的属性、方法和构造器
-   它在方法内部使用，即这个方法所属对象的引用(**谁调用就是谁的引用**)
-   它在构造器内部使用，表示该构造器正在初始化的对象

##### 2.this调用属性与方法

```java
class Person{ // 定义Person类
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

#### 3.static

#### 4.final

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

### 五.其他

#### 1.方法重载与重写

##### 1.方法重载(overload)

- 重载的概念

  在同一个类中，允许存在一个以上的**同名方法**，只要它们的**参数个数或者参数类型不同**即可

- 重载的特点

  **与返回值类型无关**，只看参数列表，且参数列表必须不同(参数个数或参数类型)。**调用时，根据方法参数列表的不同来区别**

- 重载示例

  ```java
  // 方法重载：方法名相同，参数列表不同，与返回值无关
  // 返回两个整数的和
  int add(int x,int y){return x+y;}
  // 返回三个整数的和
  int add(int x,int y,int z){return x+y+z;}
  // 返回两个小数的和
  double add(double x,double y){return x+y;}
  ```

##### 2.方法重写



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

#### 4.Object类的使用

#### 5.包装类的使用

#### 6.理解main方法的语法

#### 7.抽象类与接口

#### 8.属性赋值过程

属性赋值的位置及先后顺序：1 --> 2 --> 3 --> 4

1. 默认初始化：声明时默认初始化为0，`int age;`
2. 显式初始化：给属性赋值，`int age = 6;`
3. 构造器中初始化：`Person person = new Person(6);`
4. 通过“对象.属性“或“对象.方法”的方式赋值：`person.age = 6; person.setAge(6);`


