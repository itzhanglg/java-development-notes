### 一.Java8新特性简介

Java8 是Oracle于**2014年3月**发布的一个重要版本，其API在现存的接口上引入了非常多的新方法. 自从**2017年9月** 21日 Java9 正式发布之时，Oracle 就宣布今后会按照**每六个月**一次的节奏进行更新. Oracle 将以**三年**为周期发布长期支持版本(long term support).

这篇文章主要从以下几点介绍 Java8 新特性,其它点没有涉及到的可以阅读参考链接中的文章.

- 面向对象：接口的增强(静态方法、默认方法)
- 常用类:  新的时间和日期API(LocalDataTime、Instant、DataTimeFormat等)
- 集合：ArrayList、HashMap实例化数组容量和底层原理等改变
- 函数式接口：Lambda表达式、方法引用、构造器引用
- Stream API：并行流、串行流
- Optional类：最大化减少空指针异常


由于在**面向对象, 常用类, 集合** 这三篇文章中分别介绍了前三点, 这里就不再一一介绍了.这里再进行补充一下接口的默认方法与静态方法.

### 二.接口中的默认方法与静态方法

#### 1.概述

在Java8以前，接口中只能有**抽象方法**(public abstract 修饰的方法)跟**全局静态常量**(public static final 常量  )；但是在Java8中，允许接口中包含具有具体实现的方法，该方法称为 “默认方法”，**默认方法使用 `default` 关键字修饰，其次，Java8中，接口中还允许添加静态方法。**

#### 2.默认方法

比如:List接口中sort()方法:

```java
public interface List<E> extends Collection<E> {
    
    // ...其他成员
        
    // 默认方法
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
}
```

可以看到，这个新增的sort方法有方法体，由default修饰符修饰，这就是接口的默认方法。默认方法不是static的,必须由接口的实例来调用.

**和抽象类的区别:**

-   一个类只能继承一个抽象类；但是一个类可以实现多个接口。
-   抽象类有实例变量，而接口只能有类变量

**解决冲突:**

随着默认方法在Java8中的引入，有可能出现一个类继承了多个签名一样的方法。这种情况下，类会选择使用哪一个函数呢？为解决这种多继承关系，Java8提供了下面三条规则：

1.  **类中的方法优先级最高**，类或父类中声明的方法的优先级高于接口中任何声明为默认方法的优先级。
2.  如果第一条无法判断，那么**子接口的优先级更高**：方法签名相同时，优先选择拥有最具体实现的默认方法的接口， 即如果B继承了A，那么B就比A更加具体。
3.  最后，如果还是无法判断，**继承了多个接口的类必须通过 显式覆盖 和调用期望的方法**， 显式地选择使用哪一个默认方法的实现。

**示例:**

```java
// 以下几个接口和类不在一个文件中,只是为了方便观看和书写
public interface A {
    default void hello() {
        System.out.println("hello from A");
    }
}
public interface B extends A {	// A的子接口
    default void hello() {
        System.out.println("hello from B");
    }
}
public class C implements A {	// A的实现类

}
public class D implements A {	// A的实现类,覆盖了A的默认方法
    public void hello() {
        System.out.println("hello from D");
    }
}
public interface E {
    default void hello() {
        System.out.println("hello from E");
    }
}

public class F implements A, B {
    public static void main(String[] args) {
        new F().hello();	// hello from B	-- 子接口优先级更高
    }
}
// C未覆盖A的默认方法,子接口优先级更高
public class G extends C implements A, B {
    public static void main(String[] args) {
        new G().hello();	// hello from B	-- 子接口优先级更高
    }
}
// D覆盖了A的默认方法,类优先级更高
public class H extends D implements A, B {
    public static void main(String[] args) {
        new H().hello();	// hello from D	-- 类优先级更高
    }
}

//由于编译器无法识别A还是E的实现更加具体，所以会抛出编译错误
//可以在I中覆盖hello()方法并在方法内显示的选择调用A还是B的方法
public class I implements A, E {
    public static void main(String[] args) {
        new I().hello();	//编译报错
    }
}
//修改类I,覆盖hell()方法并显示的选择调用接口的方法
public class I implements A, B {
    public void hello() {
        // 显式地选择调用接口B中的方法
        // 同理，要调用接口A中的方法，可以这样：A.super.hello()
        B.super.hello();
    }
    public static void main(String[] args) {
        new C().hello();	// 输出 hello from B
    }
}
```

#### 2.静态方法

静态方法使用 `static` 修饰.下面是示例:

```java
public interface MyInterface {
    // ...其它成员
	
    // 默认方法
    default void method() {
        System.out.println("默认方法...");
    }
    // 静态方法
    public static void say() {
        System.out.println("静态方法...");
    }
}

// 静态方法调用方式
MyInterface.say();	// 接口名.静态方法名()
```

### 三.Lambda表达式

#### 1.概述

Lambda表达式（也称为闭包）是Java 8中最大和最令人期待的语言改变。它允许我们**将函数当成参数传递给某个方法，或者把代码本身当作数据处理.** java8之前只能使用 **匿名内部类** 代替 Lambda 表达式. Lambda表达式类似于前端 [ES6新特性](https://www.cnblogs.com/itzlg/p/11854386.html#autoid-3-4-0) 中 [箭头函数](https://www.cnblogs.com/itzlg/p/11300774.html). Lambda表达式本质是 **作为函数式接口的实例**. 函数式接口下文会介绍.

**格式:**

-   -> : Lambada操作符或箭头操作符
-   符号左边: Lambda形参列表 （其实就是接口中的抽象方法的**形参列表**）
-   符号右边: lambda体 （其实就是重写的抽象方法的**方法体**）

**使用:**

-   符号左边: lambda **形参列表的参数类型可以省略**(类型推断); 如果lambda形参列表只有**一个参数**, 其一对`()` 也可以省略
-   符号右边: lambda **体应该使用一对{}包裹**; 如果lambda体只有**一条执行语句**(可能是return语句), 省略这一对`{}` 和`return` 关键字

**说明:**

-   **类型推断**: Lambda 表达式的类型依赖于上下文环境, 是由编译器推断出来的. 表达式中无需指定类型, 程序依然可以编译. Lambda表达式有返回值，返回值的类型也由编译器推理得出。
-   Lambda表达式可以引用类成员和局部变量（会将这些变量隐式得转换成**final**的）

#### 2.作用域

##### 4.1 访问局部变量

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(5);     // 6
```

变量num可以不用声明为final (隐性的具有final的语义), num 必须不可被后面的代码修改.

##### 4.2 访问字段和静态变量

与局部变量相比,我们对lambda表达式中的实例字段和静态变量都有读写访问权限.该行为和匿名对象是一致的。

```java
class LambdaTest {
    static int outerStaticNum;
    int outerNum;

    void testScopes() {
        Converter<Integer, String> stringConverter1 = (from) -> {
            outerNum = 16;
            return String.valueOf(from);
        };

        Converter<Integer, String> stringConverter2 = (from) -> {
            outerStaticNum = 60;
            return String.valueOf(from);
        };
    }
}
```

##### 4.3 访问接口中默认方法

接口定义了一个默认方法，可以从包含匿名对象的每个实例访问该方法。 这不适用于lambda表达式。即**无能从 lambda 表达式中访问默认方法**, 会无法编译。

#### 3.示例

```java
// Lambda表达式
public class LambdaTest {
  
    @Test
    public void test1() {
        // 匿名内部类
        Runnable r1 = new Runnable(){
            @Override
            public void run() {
                System.out.println("好好学习");
            }
        };
        // 并没有启动线程，启动线程必须调用start()方法
        r1.run();

        // Lambda表达式：类似于前端ES6中的箭头函数
        Runnable r2 = () -> System.out.println("天天向上");
        r2.run();
    }

    @Test
    public void test2() {
        // 匿名内部类
        Comparator<Integer> c1 = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return Integer.compare(o1, o2);
            }
        };
        System.out.println(c1.compare(12, 32)); // -1

        // Lambda表达式：类似于前端ES6中的箭头函数
        Comparator<Integer> c2 = (o1, o2) -> Integer.compare(o1, o2);
        System.out.println(c2.compare(16, 12)); // 1

        // 方法引用
        Comparator<Integer> c3 = Integer :: compare;    //类名.静态方法
        System.out.println(c3.compare(16, 16)); // 0

        Comparator<Integer> c4 = Integer :: compareTo;  //类名(对象).非静态方法
        System.out.println(c4.compare(16, 11)); // 1
    }
}
```

不管是**抽象类还是接口，都可以通过匿名内部类的方式访问**。不能通过抽象类或者接口直接创建对象。对于上面通过匿名内部类方式访问接口，理解：**一个内部类实现了接口里的抽象方法并且返回一个内部类对象，之后我们让接口的引用来指向这个对象**。**以前用匿名实现类表示的现在都可以用Lambda表达式来写**。

### 四.函数式(Functional)接口

#### 1.概述

函数接口指的是只有一个函数的接口，这样的接口可以隐式转换为Lambda表达式。`java.lang.Runnable`和`java.util.concurrent.Callable`是函数式接口的最佳例子。Java 8 提供了一个特殊的注解**@FunctionalInterface**进行声明,虚拟机会自动判断该接口是否为函数式接口。java8 函数式接口都在 `java.util.function` 包里。**Lambda表达式就是一个函数式接口的实例**。

**说明:**

- 接口中只能有一个抽象方法
- 可以有静态方法和默认方法
- 可以使用注解 `@FunctionalInterface` 标记
- 默认方法可以被重写

#### 2.内置函数式接口

JDK 1.8 API包含许多内置函数式接口。 其中一些接口在老版本的 Java 中是比较常见的比如： `Comparator` 或`Runnable`，这些接口都增加了`@FunctionalInterface`注解以便能用在 lambda 表达式上。四大核心函数式接口:

-   消费型接口 `Consumer<T>     void accept(T t)` : 对单个输入参数执行的操作
-   供给型接口 `Supplier<T>     T get()` : 产生给定泛型类型的结果
-   函数型接口 `Function<T,R>   R apply(T t)` :接受一个参数并生成结果
-   断言型接口 `Predicate<T>    boolean test(T t)` : 接受一个传入类型,返回一个布尔值

**Predicate源码** : 包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如与，或，非）：

```java
package java.util.function;
import java.util.Objects;

@FunctionalInterface
public interface Predicate<T> {

    // 该方法是接受一个传入类型,返回一个布尔值.此方法应用于判断.
    boolean test(T t);

    //and方法与关系型运算符"&&"相似，两边都成立才返回true
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    // 与关系运算符"!"相似，对判断进行取反
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    //or方法与关系型运算符"||"相似，两边只要有一个成立就返回true
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
   // 该方法接收一个Object对象,返回一个Predicate类型.
   // 此方法用于判断第一个test的方法与第二个test方法相同(equal).
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
```

示例:

```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

**Function源码** : 默认方法可用于将多个函数链接在一起（compose, andThen）：

```java
package java.util.function;
import java.util.Objects;

@FunctionalInterface
public interface Function<T, R> {

    //将Function对象应用到输入的参数上，然后返回计算结果。
    R apply(T t);
    //将两个Function整合，并返回一个能够执行两个Function对象功能的Function对象。
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    // 
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

示例:

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);
backToString.apply("123");     // "123"
```

#### 3.示例

```java
// 函数式接口
public class FunctionalTest {

    @Test
    public void test1(){
        // 消费money
        spendMoney(60.6, money -> System.out.println("开心的购物了"+money));
    }

    public void spendMoney(Double money, Consumer<Double> c){
        c.accept(money);
    }


    @Test
    public void test2(){
        List<String> strList = Arrays.asList("ab","ac","ad","bc","bd","cd");

        List<String> result = filterStr(strList, str -> str.contains("a"));
        System.out.println(result); // [ab, ac, ad]
    }

    //根据给定的规则，过滤集合中的字符串。此规则由Predicate的方法决定
    public List<String> filterStr(List<String> list, Predicate<String> p){
        ArrayList<String> arraylist = new ArrayList<>();
        for (String s : list) {
            if (p.test(s)){
                arraylist.add(s);
            }
        }
        return arraylist;
    }

}
```

### 五.方法引用与构造器引用

#### 1.方法引用

##### 1.1 概述

方法引用使得开发者可以直接引用现存的方法、Java类的构造方法或者实例对象。**方法引用本质上就是Lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法**，可以认为是Lambda表达式的一个语法糖。

**格式**: `类(或对象) :: 方法名`

**情况**: 

-   `对象::非静态方法` , `类::静态方法`

    实现接口的**抽象方法**的参数列表和返回值类型，必须与**方法引用的方法**的参数列表和返回值类型保持一致

-   `类::非静态方法`

    函数式接口**抽象方法**的 第一个参数 是需要引用方法的**调用者**, 并且 第二个参数 是需要引用方法的**参数**( 或无参数)   

##### 1.2 示例

```java
// 方法引用
public class MethodRefTest {

    @Test
    public void test1(){
        Consumer<String> c1 = str -> System.out.println(str);   // Lambda表达式
        c1.accept("好好学习");  // 好好学习

        // Consumer中的void accept(T t)
        // PrintStream中的void println(T t)
        // 当函数接口中抽象方法形参列表与方法体中调用的方法形参列表相同时，可用方法引用
        Consumer<String> c2 = System.out :: println;    // 对象 :: 非静态方法
        c2.accept("天天向上");  // 天天向上
    }

    @Test
    public void test2(){
        Function<Double, Long> f1 = d -> Math.round(d); // Lambda表达式
        System.out.println(f1.apply(5.20));  // 5

        // Function中的R apply(T t)
        // Math中的Long round(Double d)
        Function<Double, Long> f2 = Math :: round;  // 类 :: 静态方法
        System.out.println(f2.apply(13.14));  // 13
    }

    @Test
    public void test3(){
        BiPredicate<String, String> b1 = (s1, s2) -> s1.equals(s2); // Lambda表达式
        System.out.println(b1.test("abc", "bcd"));  // false

        // BiPredicate中的boolean test(T t1, T t2);
        // String中的boolean t1.equals(t2)
        // 当方法体中用 函数接口中抽象方法形参列表中第一个形参 调用 
      	// 一个形参列表与函数接口中抽象方法除第一个形参以外列表相同的方法时
        BiPredicate<String, String> b2 = String :: equals; // 类 :: 非静态方法
        System.out.println(b2.test("bcd", "bcd"));  // true
    }

}
```

#### 2.构造器引用

##### 2.1 概述

**构造器引用**: 和方法引用类似，函数式接口的抽象方法的形参列表和构造器的形参列表一致。抽象方法的返回值类型即为构造器所属的类的类型。

格式: `类名 :: new` 

**数组引用**: 可以把数组看做是一个特殊的类，则写法与构造器引用一致。

格式: `类名[] :: new`

**要求**: 构造器参数列表要与接口中抽象方法的**参数列表一致**且方法的**返回值**即为构造器对应类的对象或数组

##### 2.2 示例

```java
public class ConstructorRefTest {

    @Test
    public void test1(){
        Supplier<Employee> s1 = () -> new Employee();
        System.out.println(s1.get());   // Employee{id=0, name='null', age=0, salary=0.0}

        Supplier<Employee> s2 = Employee :: new;
        System.out.println(s2.get());   // Employee{id=0, name='null', age=0, salary=0.0}
    }

    @Test
    public void test2(){
        Function<Integer, Employee> f1 = id -> new Employee(id);
        System.out.println(f1.apply(6));    // Employee{id=6, name='null', age=0, salary=0.0}

        Function<Integer, Employee> f2 = Employee :: new;
        System.out.println(f2.apply(8));    // Employee{id=8, name='null', age=0, salary=0.0}
    }

    @Test
    public void test3(){
        BiFunction<Integer, String, Employee> f1 = (id, name) -> new Employee(id, name);
        System.out.println(f1.apply(6, "Jack"));    // Employee{id=6, name='Jack', age=0, salary=0.0}

        BiFunction<Integer, String, Employee> f2 = Employee :: new;
        System.out.println(f2.apply(8, "Tom"));     // Employee{id=8, name='Tom', age=0, salary=0.0}
    }

    @Test
    public void test4(){
        Function<Integer, String[]> f1 = lenght -> new String[lenght];
        System.out.println(Arrays.toString(f1.apply(3)));   // [null, null, null]

        Function<Integer, String[]> f2 = String[] :: new;
        System.out.println(Arrays.toString(f2.apply(5)));   // [null, null, null, null, null]
    }

}
```

Employee类:

```java
public class Employee {

	private int id;
	private String name;
	private int age;
	private double salary;

	// get, set方法省略...

	public Employee() {
		System.out.println("Employee().....");
	}
	public Employee(int id) {
		this.id = id;
		System.out.println("Employee(int id).....");
	}
	public Employee(int id, String name) {
		this.id = id;
		this.name = name;
      System.out.println("int id, String name.....");
	}
	public Employee(int id, String name, int age, double salary) {
		this.id = id;
		this.name = name;
		this.age = age;
		this.salary = salary;
	}

	@Override
	public String toString() {
		return "Employee{" + "id=" + id + ", name='" + name + '\'' + ", age=" + age + ", salary=" + salary + '}';
	}

	@Override
	public boolean equals(Object o) {
		if (this == o)
			return true;
		if (o == null || getClass() != o.getClass())
			return false;
		Employee employee = (Employee) o;
		if (id != employee.id)
			return false;
		if (age != employee.age)
			return false;
		if (Double.compare(employee.salary, salary) != 0)
			return false;
		return name != null ? name.equals(employee.name) : employee.name == null;
	}

	@Override
	public int hashCode() {
		int result;
		long temp;
		result = id;
		result = 31 * result + (name != null ? name.hashCode() : 0);
		result = 31 * result + age;
		temp = Double.doubleToLongBits(salary);
		result = 31 * result + (int) (temp ^ (temp >>> 32));
		return result;
	}
}
```

### 六.Stream API





### 七.Optional类





参考链接:

-   [Java 8 新特性总结](https://snailclimb.gitee.io/javaguide/#/docs/java/What's%20New%20in%20JDK8/Java8Tutorial)
-   [Java 8的新特性—终极版](https://blog.csdn.net/yczz/article/details/50896975)