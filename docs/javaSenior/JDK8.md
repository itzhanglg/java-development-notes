

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

- 示例：`(o1,o2) -> Integer.compare(o1,o2);`

- -> : Lambada操作符或箭头操作符
- 符号左边: Lambda形参列表 （其实就是接口中的抽象方法的**形参列表**）
- 符号右边: lambda体 （其实就是重写的抽象方法的**方法体**）

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
            if (p.test(s)) {
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

Employee类：

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

#### 1.Stream 概述

**Stream 是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列**。使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来**并行执行**操作。简言之，Stream API 提供了一种高效且易于使用的**处理数据**的方式。

Stream 和 Collection 集合的区别：Collection 是一种静态的**内存数据结构**，主要**面向内存**，存储在内存中。而 Stream 是有关**对数据的运算**，主要是**面向 CPU**，通过 CPU 实现计算。

**说明：**

- Stream 自己不会存储元素。
- Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。
- Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。

**执行流程：**

- Stream实例化：指定一个数据源(Collection子类、数组，Map不支持)，获取一个流
- 中间操作：一个中间操作链，对数据源的数据进行处理，返回Stream本身
- 终止操作：一旦执行终止操作，就执行中间操作链，返回一特定类型的计算结果。之后，不会再被使用

#### 2.Stream 创建方式

- **通过集合** -- Java8 中的 Collection 接口提供了两个获取流的方法：
  - `default Stream<E> stream()` : **返回一个顺序流**
  - `default Stream<E> parallelStream()` : 返回一个并行流
- **通过数组** -- Java8 中的 Arrays 的静态方法 stream() 可以获取数组流：
  - `static <T> Stream<T> stream(T[] array)` : 返回一个流
- **通过Stream的of()** -- 可以调用Stream类静态方法 of(), 通过显示值创建一个流。它可以接收任意数量的参数。
  - `public static<T> Stream<T> of(T... values)` : 返回一个流
- **创建无限流** -- 可以使用静态方法 Stream.iterate() 和 Stream.generate()，创建无限流。
  - `public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)`
  - `public static<T> Stream<T> generate(Supplier<T> s)`

示例：

```java
// 通过集合：串行流
List<Employee> employees = EmployeeData.getEmployees();
Stream<Employee> stream = employees.stream();
stream.forEach(System.out::println);
// 通过集合：并行流
Stream<Employee> parallelStream = employees.parallelStream();
parallelStream.forEach(System.out::println);

// 通过数组
IntStream stream1 = Arrays.stream(new int[]{1, 2, 3, 4, 5});
stream1.forEach(e -> System.out.print(e+" "));

// 通过Stream的of()
Stream<Integer> integerStream = Stream.of(1, 2, 3, 4, 5);
integerStream.forEach(e -> System.out.print(e+" "));

// 创建无限流
// 迭代
Stream.iterate(1, e -> e*2).limit(6).forEach(e -> System.out.print(e+" "));
// 生成
Stream<Double> generate = Stream.generate(Math::random);
generate.limit(6).forEach(e -> System.out.print(e+" "));
```

EmployeeData.java

```java
public class EmployeeData {
	
	public static List<Employee> getEmployees(){
		List<Employee> list = new ArrayList<>();
		
		list.add(new Employee(1001, "Jack", 16, 6006.36));
		list.add(new Employee(1002, "Mary", 30, 3006.80));
		list.add(new Employee(1003, "Tick", 26, 7686.36));
		list.add(new Employee(1004, "Tom", 48, 6666.16));
		list.add(new Employee(1005, "Tom", 60, 6666.16));
		list.add(new Employee(1006, "Tom", 60, 6666.16));
		
		return list;
	}
}
```

Employee：

```java
public class Employee implements Comparable{
    
	private int id;
	private String name;
	private int age;
	private double salary;
    
    // 构造方法省略
    // set、get方法省略
    // toString方法省略
    // hashcode、equals方法省略
    
    @Override
	public int compareTo(Object o) {
		if (o instanceof Employee) {
			Employee e = (Employee)o;
			int compare = Double.compare(this.salary, e.salary);
			if(compare != 0){
				return compare;
			}else {
				return this.name.compareTo(e.name);
			}
		}
		throw new RuntimeException("传入的数据类型不一致！");
	}
    
}
```

#### 3.Stream 中间操作

一个中间操作链，对数据源的数据进行处理，返回Stream本身。

- 筛选与切片
  - `filter(Predicate p)`：**接收 Lambda ， 从流中排除某些元素**
  - `limit(long maxSize)`：使其元素不超过给定数量
  - `skip(long n)`：跳过元素，返回一个扔掉了前 n 个元素的流
  - `distinct(Predicate p)`：通过流所生成元素的 hashCode() 和 equals() 去除重复元素
- 映射
  - `map(Function f)`：**接收一个函数作为参数，将元素转换成其他形式或提取信息，该函数会被应用到每个元素上，并将其映射成一个新的元素，返回的值包装在 Optional 中**
  - `flatMap(Function f)`：**函数作为参数，并对值调用这个函数，然后直接返回结果，返回的值是解除包装的值**
- 排序
  - `sorted()`：**产生一个新流，其中按自然顺序排序**
  - `sorted(Comparator com)`：**产生一个新流，其中按比较器顺序排序**

示例：

```java
List<Employee> list = EmployeeData.getEmployees();
// 薪水大于6000的员工信息
list.stream().filter(e -> e.getSalary()>6000).forEach(System.out::println);

// 元素不超过3个
list.stream().limit(3).forEach(System.out::println);

// 跳过元素，返回一个扔掉了前 n 个元素的流
list.stream().skip(2).forEach(System.out::println);

// 根据元素的hashcode() 和 equals() 去除重复元素
list.stream().distinct().forEach(System.out::println);

// 将员工信息的姓名都转为大写
list.stream().map(e -> e.getName().toUpperCase()).forEach(e -> System.out.print(e+" "));

// 将员工按照薪水排序，再按照名称排序 -- 自然排序(Employee需要实现Comparable接口)
list.stream().sorted().forEach(System.out::println);
// 将员工按照薪水排序，再按照名称排序 -- 定制排序
list.stream().sorted((e1,e2) -> {
    int compare = Double.compare(e1.getSalary(), e2.getSalary());
    if (compare != 0){
        return compare;
    }else {
        return e1.getName().compareTo(e2.getName());
    }
}).forEach(System.out::println);
```

**map(Function f) 与 flatMap(Function f) 区别：**

```java
// map方法:对值应用(调用)作为参数的函数，然后将返回的值包装在 Optional 中
Stream<Stream<Character>> streamStream = list.stream().map(StreamAPITest::fromStringToStream);
streamStream.forEach(s -> s.forEach(System.out::println));

// 函数作为参数，并对值调用这个函数，然后直接返回结果，返回的值是解除包装的值
Stream<Character> characterStream = list.stream().flatMap(StreamAPITest::fromStringToStream)；
characterStream.forEach(System.out::println);

// 将字符串中的多个字符构成的集合转换为对应的Stream的实例
public static Stream<Character> fromStringToStream(String str){ //aa
    ArrayList<Character> list = new ArrayList<>();
    for(Character c : str.toCharArray()) {
        list.add(c);
    }
    return list.stream();
}
```



#### 4.Stream 终止操作

终端操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如：List、Integer，甚至是 void 。**流进行了终止操作后，不能再次使用**。

- 匹配与查找
  - `allMatch(Predicate p)`：**检查是否匹配所有元素**
  - `anyMatch(Predicate p)`：检查是否至少匹配一个元素
  - `noneMatch(Predicate p)`：检查是否没有匹配的元素
  - `findFirst()`：返回第一个元素
  - `count()`：**返回流中元素的总个数**
  - `forEach(Consumer c)`：**内部迭代**
  - `max(Comparator c)、max(Comparator c)`：返回流中最大值、最小值
- 归约
  - `reduce(T iden, BinaryOperator b)`：可以将流中元素反复结合起来，得到一个值。返回 T
  - `reduce(BinaryOperator b)`：**可以将流中元素反复结合起来，得到一个值**。返回 Optional<T>
  - map 和 reduce 的连接通常称为 map-reduce 模式
- 收集
  - `collect(Collector c)`：**将流转换为其他形式**。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法

示例：

```java
List<Employee> list = EmployeeData.getEmployees();
// 是否所有的员工的年龄都大于18 -- 是否匹配所有元素
boolean b = list.stream().allMatch(e -> e.getAge() > 18);
System.out.println(b);  // false
// 是否存在员工的工资大于7000 -- 是否至少匹配一个元素
boolean b1 = list.stream().anyMatch(e -> e.getSalary() > 7000);
System.out.println(b1); // true
// 是否存在员工姓“J” -- 是否没有匹配的元素
boolean b2 = list.stream().noneMatch(e -> e.getName().startsWith("J"));
System.out.println(b2); // false
// 查找第一个元素
Optional<Employee> first = list.stream().findFirst();
System.out.println(first);  // Optional[Employee{id=1001, name='Jack', age=16, salary=6006.36}]
// 流中元素总个数
long count = list.stream().count();
System.out.println(count);  // 6
// 返回最高的工资员工信息
Optional<Employee> max = list.stream()
    .max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
System.out.println(max);    // Optional[Employee{id=1004, name='Tick', age=26, salary=7686.36}]
// 返回最低的工资员工信息
Optional<Double> min = list.stream().map(Employee::getSalary).min(Double::compareTo);
System.out.println(min);    // Optional[3006.8]

// 内部迭代
list.stream().forEach(System.out::println);
// 集合的遍历操作 -- Collection接口实现了Iterable，Iterable扩展了forEach的默认方法
list.forEach(System.out::println);

// 计算所有员工工资的总和
Optional<Double> reduce = list.stream().map(Employee::getSalary).reduce((e1, e2) -> e1 + e2);
System.out.println(reduce.get()); // 36698.0

// 查找工资大于7000的员工，结果返回为一个List或Set
list.stream().filter(e -> e.getSalary() > 7000)
    .collect(Collectors.toList()).forEach(System.out::println);
```

#### 5.forEach 集合中的使用

**Collection 接口实现了 Iterable 接口，而 Iterable 接口在 Java 8开始具有一个新的 API：**

```java
public interface Collection<E> extends Iterable<E>

public interface Iterable<T> {
	// 对 Iterable的每个元素执行给定的操作，直到所有元素都被处理或动作引发异常。
	// forEach形参可用Lambda表达式或方法引用
	default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
}
```

**任何类型Collection的可迭代 - 列表，集合，队列 等都具有使用forEach的相同语法。**

```java
// List
List<String> strings = Arrays.asList("Jack", "Tome", "Mary");
strings.forEach(System.out::println);
// Set
HashSet<String> strings1 = new HashSet<>(Arrays.asList("Jack", "Tome", "Mary"));
strings1.forEach(System.out::println);
// Queue
Queue<String> strings2 = new ArrayDeque<>(Arrays.asList("Jack", "Tome", "Mary"));
strings2.forEach(System.out::println);
```

Map没有实现Iterable接口，但它**提供了自己的forEach 变体，它接受BiConsumer**。

```java
HashMap<Integer, String> map = new HashMap<>();
map.put(1,"Jack");
map.put(2,"Tome");
map.put(3,"Mary");
map.forEach((key, value) -> System.out.println(key+"="+value));
map.entrySet().forEach(entry -> System.out.println(entry.getKey()+"="+entry.getValue()));
```

### 七.Optional类

Optional<T> 类(java.util.Optional) 是一个**容器类，它可以保存类型T的值，代表这个值存在。或者仅仅保存null，表示这个值不存在**。原来用 null 表示一个值不存在，现在 Optional 可以更好的表达这个概念。并且可以避免空指针异常。

- 创建Optiona类l对象：
  - `Optional.of(T t)` : 创建一个 Optional 实例，t必须非空；
  - `Optional.empty()` : 创建一个空的 Optional 实例
  - `Optional.ofNullable(T t)` ：**t可以为null**
- 判断容器中是否包含对象：
  - `boolean isPresent()` : **判断是否包含对象**
  - `void ifPresent(Consumer<? super T> consumer)` ：**如果有值，就执行Consumer接口的实现代码，并且该值会作为参数传给它。**
- 获取Optional类的对象：
  - `T get()` : 如果调用对象包含值，返回该值，否则抛异常
  - `T orElse(T other)` ：**如果有值则将其返回，否则返回指定的other对象**
  - `T orElseGet(Supplier<? extends T> other)` ：如果有值则将其返回，否则返回由Supplier接口实现提供的对象
  - `T orElseThrow(Supplier<? extends X> exceptionSupplier)` ：**如果有值则将其返回，否则抛出由Supplier接口实现提供的异常。**
- 中间操作：
  - `Optional<T> filter(Predicate<? super T> predicate)`：过滤值
  - `Optional<U> map(Function<? super T, ? extends U> mapper)`：转换值。**接收一个函数作为参数，将元素转换成其他形式或提取信息，该函数会被应用到每个元素上，并将其映射成一个新的元素，返回的值包装在 Optional 中**
  - `Optional<U> flatMap(Function<? super T, Optional<U>> mapper)`：转换值。**函数作为参数，并对值调用这个函数，然后直接返回结果，返回的值是解除包装的值**

示例：

```java
public static String getName(User u) {
    if (u == null || u.name == null)
        return "Unknown";
    return u.name;
}
// 改写
public static String getName(User u) {
    return Optional
        .ofNullable(u)
        .map(user -> user.name)
        .orElse("Unknown");
}

// 为空则不打印
string.ifPresent(System.out::println);

// 检验参数的合法性
public void setName(String name) throws IllegalArgumentException {
	this.name = Optional
        .ofNullable(name)
        .filter(User::isNameValid)
        .orElseThrow(() -> new IllegalArgumentException("Invalid username..."));
}

// Optional类的链式方法
User user = new User("Jack", "123");
String result = Optional.ofNullable(user)
  .flatMap(User::getAddress)
  .flatMap(Address::getCountry)
  .map(Country::getIsocode)
  .orElse("default");

public class User {
    private Address address;
    public Optional<Address> getAddress() {
        return Optional.ofNullable(address);
    }
    // ...
}
public class Address {
    private Country country;
    public Optional<Country> getCountry() {
        return Optional.ofNullable(country);
    }
    // ...
}
```

### 参考链接

-   [Java 8 新特性总结](https://snailclimb.gitee.io/javaguide/#/docs/java/What's%20New%20in%20JDK8/Java8Tutorial)
-   [Java 8的新特性—终极版](https://blog.csdn.net/yczz/article/details/50896975)
-   [理解、学习与使用 JAVA 中的 OPTIONAL](https://www.cnblogs.com/zhangboyu/p/7580262.html)