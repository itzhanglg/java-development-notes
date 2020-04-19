
###  一.面向对象

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
   	 `Person p1 = new Person();	// person对象被p1引用`
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

作用：对java类或对象进行初始化

代码块只能被static修饰，static修饰的称为静态代码块，没有使用static修饰的，为非静态代码块。

##### 1.静态代码块：

- 可以有输出语句，不可以调用非static属性和方法
- 若有多个静态代码块，按照从上到下的顺序依次执行
- 静态代码块的执行要优先于非静态代码块
- **静态代码块随着类的加载而加载，且只执行一次**

##### 2.非静态代码块：

- 可以有输出语句，静态和非静态都可以调用
- 若有多个非静态代码块，按照从上到下的顺序执行
- **每次创建对象的时候，都会执行一次。且优先于构造器执行**

```java
class Person {
    public static int total;
    static {
    	total = 100;
    	System.out.println("in static block!");
    }
}
public class PersonTest {
    public static void main(String[] args) {
    	System.out.println("total = " + Person.total);
    	System.out.println("total = " + Person.total);
    }
}

// in static block
// total=100
// total=100
```

#### 5.内部类

在Java中，允许一个类的定义位于另一个类的内部，前者称为**内部类**，后者称为**外部类**。

Inner class一般用在定义它的类或语句块之内，在外部引用它时必须给出**完整的名称**。Inner class的名字不能与包含它的外部类类名相同。

##### 1.成员内部类 (static成员内部类和非static成员内部类) 

内部类可以声明为private、protected；也可以声明为static、abstract、final的；

可以在内部定义属性、方法、构造器等结构

注意：

- 非static的成员内部类中的成员不能声明为static的
- 外部类访问成员内部类的成员，需要“内部类.成员”或“内部类对象.成员”的方式
- 成员内部类可以直接使用外部类的所有成员，包括私有的数据
- 当想要在外部类的静态成员部分使用内部类时，可以考虑内部类声明为静态的

```java
public class Outer {
    private int s = 111;
    public class Inner {
        private int s = 222;
        public void mb(int s) {
            System.out.println(s); // 局部变量s
            System.out.println(this.s); // 内部类对象的属性s
            System.out.println(Outer.this.s); // 外部类对象属性s
        }
    }
    
    public static void main(String args[]) {
        Outer a = new Outer();
        Outer.Inner b = a.new Inner();
        b.mb(333);
    }
}
```

##### 2.局部内部类 (不谈修饰符)

声明：

```java
class 外部类{
	方法(){	// 方法
		class 局部内部类{
		}
	}
	{		// 代码块
        class 局部内部类{
        }
    }
}
```

特点：

- 只能在声明它的方法或代码块中使用，而且是先声明后使用。除此之外的任何地方都不能使用该类。
- 局部内部类可以使用外部类的成员，包括私有的。
- **局部内部类可以使用外部方法的局部变量，但是必须是final的**。由局部内部类和局部变量的声明周期不同所致。
- 局部内部类不能使用static修饰，因此也不能包含静态成员

##### 3.匿名内部类

匿名内部类不能定义任何静态成员、方法和类，只能创建匿名内部类的一个实例。一个匿名内部类一定是在new的后面，用其隐含实现一个接口或实现一个类。

格式：

```java
new 父类构造器(实参列表) | 实现接口() {
	// 匿名内部类的类体部分
}
```

特点：

- 匿名内部类必须**继承父类或实现接口**
- 匿名内部类**只能有一个对象**
- 匿名内部类对象只能使用**多态形式引用**

```java
interface A{
	public abstract void fun1();
}
public class Outer{
    public static void main(String[] args) {
        new Outer().callInner(
            new A(){
                //接口是不能new但此处比较特殊是子类对象实现接口，只不过没有为对象取名
                public void fun1() {
                    System.out.println(“implement for fun1");
                }
        	}
        ); // 两步写成一步了
    }
    public void callInner(A a) {
    	a.fun1();
    }
}
```

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

##### 1.概念

多个类中存在相同属性和行为时，将这些内容抽取到单独一个类中，那么多个类无需再定义这些属性和行为，只要继承那个类即可。

此处的**多个类称为子类( 派生类)，单独的这个类称为父类(基类或超类)**。可以理解为:“子类 is a 父类”。

类继承语法规则：`class Subclass extends SuperClass{}`

##### 2.作用

减少了代码冗余，提高了代码的复用性；

有利于功能的扩展；

让类与类之间产生了关系，提供了多态的前提。

##### 3.理解

- 子类继承了父类，继承了父类的属性和方法，可以使用父类中定义的方法和属性，也可以创建新的数据和方法

- 继承的关键字用的是"extends"，子类不是父类的子集，而是对父类的"扩展"

- **子类不能直接访问父类中私有的(private)的成员变量和方法**，可以通过setter、getter和暴露接口来访问

- **java只支持单继承和多层继承(传递性)，不允许多重继承**。一个子类只能有一个父类，一个父类可以派生出多个子类

  ```java
   class SubDemo extends Demo{ }	// ok
   class SubDemo extends Demo1,Demo2...	// error
  ```

#### 3.多态性

##### 1.概念

对象的多态性：父类的引用指向子类的对象  -->  可以直接应用在抽象类和接口上

java引用遍历有两个类型：编译时类型和运行时类型。**编译时类型由声明该变量时使用的类型决定，运行时类型由实际赋给该变量的对象决定**。简称：编译时，看左边；运行时，看右边。

- 若编译时类型和运行时类型不一致 ， 就出现了对象的多态性(Polymorphism)
- 多态情况下 ， **“ 看左边 ” ： 看的是父类的引用**（父类中不具备子类特有的方法）
  **“ 看右边 ” ： 看的是子类的对象**（实际运行的是子类重写父类的方法）

##### 2.对象的多态

在java中，子类的对象可以替代父类的对象使用

- 一个变量只能有一种确定的数据类型
- 一个引用类型变量可能指向(引用)多种不同类型的对象

```java
Person p = new Student();
Object o = new Person();  // Object类型的变量o，指向Person类型的对象
o = new Student();  // Object类型的变量o，指向Student类型的对象
```

**子类可看做是特殊的父类 ， 所以父类类型的引用可以指向子类的对象：向上转型(upcasting)。**

方法声明的形参类型为父类类型，可以使用子类的对象作为实参调用该方法。

**一个引用类型变量如果声明为父类的类型，但实际引用的是子类对象，那么该变量就不能再访问子类中添加的属性和方法。**

```java
Student m = new Student();
m.school = “pku”; // 合法,Student 类有school 成员变量
m.read();	// 合法，Student 类中有read 方法
Person e = new Student();
e.school = “pku”; // 非法,Person 类没有school 成员变量
e.read();	// 非法，Person 类中没有read 方法，编译时错误
```

**属性是在编译时确定的**，编译时e 为Person 类型，没有school 成员变量，因而编译错误。

虚拟方法调用(多态情况下)：子类中定义了与父类同名同参数的方法，在多态情况下，将此时父类的方法称为**虚拟方法，父类根据赋给它的不同子类对象，动态调用属于子类的该方法**。这样的方法调用在编译期是无法确定的。

```java
Person e = new Student();
e.getInfo(); // 调用Student 类的getInfo()
```

编译时类型和运行时类型：编译时e 为Person 类型，而方法的调用是在运行时确定的，所以调用的是Student 类的getInfo() 方法。——动态绑定

**小结：**

- **子类可看做是特殊的父类 ， 所以父类类型的引用可以指向子类的对象：向上转型(upcasting)。**
- **父类引用指向子类对象，该变量就不能再访问子类中新添加的属性和方法，编译时错误。**
- **父类引用指向子类对象，子父类出现同名同参数方法，方法调用是运行时确定的，调用的是子类中的方法。**

##### 3.总结

前提：**需要存在继承或实现关系、有方法的重写**

成员方法：

- 编译时：要查看**引用变量所声明的类**中是否有所调用的方法
- 运行时：调用实际**new的对象所属的类**中的重写方法

成员变量：**不具备多态性，只看引用变量所声明的类**

##### 4.示例

```java
public class FieldMethodTest {

    public static void main(String[] args){
        Sub s = new Sub();
        System.out.println(s.count);    // 20
        s.display();    // 20
        Base b = s;	// 指向同一个对象
        System.out.println(b == s); // true
        System.out.println(b.count);// 10	成员变量不具有多态性
        b.display();    // 20	多态，调用指向对象的方法
    }

}

class Sub extends Base {
    int count = 20;
    public void display() {
        System.out.println(this.count);
    }
}

class Base {
    int count = 10;
    public void display() {
        System.out.println(this.count);
    }
}
```

- 子类重写了父类方法，就意味着子类里定义的方法彻底覆盖了父类里的同名方法，系统将不可能把父类里的方法转移到子类中
- 对于实例变量则不存在这样的现象，即使子类里定义了与父类完全相同的实例变量，这个实例变量依然不可能覆盖父类中定义的实例变量










