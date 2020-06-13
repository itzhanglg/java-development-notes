## 一.Java集合框架概述

集合、数组都是对多个数据进行存储操作的结构，简称Java容器。此时的存储，主要指的是内存层面的存储，不涉及到持久化的存储（.txt, .jpg, .avi，数据库中）。Java 集合就像一种容器，可以动态地把多个对象的引用放入容器中。

**1.数组在内存存储方面的特点：**

- 数组初始化以后，长度就确定了
- 数组声明的类型，就决定了进行元素初始化时的类型

**2.数组在存储数据方面的弊端：**

- 数组初始化以后，长度就不可变了，不便于扩展
- 数组中提供的属性和方法少，不便于进行添加、删除、插入等操作，且效率不高。同时无法直接获取数组中实际元素的个数
- 数组存储的数据是有序的、可以重复的。对于无序、不可重复的需求，不能满足。

**3.Java集合分为Collection和Map两种关系：**

- **Collection接口**：单列集合，用来存储一个一个的对象
  - List：存储有序的、可重复的数据。  --> “动态”数组
    - `ArrayList、LinkedList、Vector`
  - Set：存储无序的、不可重复的数据   --> 高中讲的“集合”
    - `HashSet、LinkedHashSet、TreeSet`
- **Map接口**：双列集合，用来存储一对(key - value)一对的数据   --> 高中函数：y = f(x)
  - `HashMap、LinkedHashMap、TreeMap、Hashtable、Properties`

## 二.Collection接口

Collection 接口是 List、Set 和 Queue 接口的父接口，该接口里定义的方法。既可用于操作 Set 集合，也可用于操作 List 和 Queue 集合。

### 1.常用的API
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191220135919258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

### 2.抽象方法

删除、包含相关方法底层都是调用元素类型对象**重写的equals方法(比较内容)**来判断集合中是否有要删除或包含的元素。向集合中添加obj数据时，要求obj所在类要重写equals()。

- 增：
  - `boolean add(Object o)`：**添加元素到集合**
  - `boolean addAll(Collection c)`：将指定集合中的所有元素添加到此集合
- 删：
  - `boolean remove(Object o)`：**删除找到的第一个元素(equals)**
  - `boolean removeAll(Collection c)`：从当前集合中删除公共元素(equals)(差集)
  - `boolean retainAll(Collection c)`：从当前集合中删除非公共元素(equals)(交集)
  - `void clear()`：删除所有元素
- 查：
  - `iterator()`：**返回迭代器对象，用于遍历**
  - `for(集合元素的类型 局部变量 : 集合对象)`：**增强for循环，用于遍历**
  - `int size()`：**查询有效元素个数**
  - `hashCode()`：**查询当前对象的哈希值**
- 判断：
  - `boolean contains(Object o)`：**是否包含某个元素(equals)**
  - `boolean containsAll(Collection c)`：是否包含某个集合的所有元素(equals)
  - `boolean isEmpty()`：**判断集合size==0** 
  - `boolean equals(Object o)`：**集合是否相等(比较集合是否相等：元素及顺序)**
- 转换：
  - `Object[] toArray()`：**集合 --> 数组**
  - `T[] toArray(T[] a) ` : **集合 --> 数组**
  - `(Arrays)public static List<T> asList(T... a)`：**数组 --> 集合**

### 3.注意

**获取长度的区分:**

-   java 中的**length属性**是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.
-   java 中的**length()方法**是针对字符串String说的,如果想看这个字符串的长度则用到 length()这个方法.
-   java 中的**size()方法**是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看!

**向Collection接口的实现类的对象中添加obj数据时，要求obj所在类要重写equals():**

- 在判断时会调用obj对象所在类的equals()方法：equals方法默认比较地址，需要重写来比较内容。String、File、Date、包装类默认重写了equals方法
- equals(Object obj)：要想返回true，需要当前集合和形参集合的元素都相同(包括元素的顺序)
- 调用Arrays类的静态方法asList()：基本数据类型数组会被当做一个对象，需要使用包装类对象数组当形参

```java
Collection coll = new ArrayList();
coll.add(123);
coll.add(new Person("Jerry",20));
coll.add(new String("Tom"));
coll.add(false);

// 在判断时会调用obj对象所在类的equals()方法：equals方法默认比较地址，需要重写来比较内容
// 1.contains(Object obj):判断当前集合中是否包含obj
boolean contains = coll.contains(123);
System.out.println(contains);	// true
// String、File、Date、包装类重写了equals方法
System.out.println(coll.contains(new String("Tom")));	// true
// Person没有重写equals方法时：false；重写了equals方法：true
System.out.println(coll.contains(new Person("Jerry",20)));

// coll2集合中：元素内容与coll集合相同，但顺序不同
Collection coll2 = new ArrayList();
coll2.add(new Person("Jerry",20));
coll2.add(new String("Tom"));
coll2.add(123);
coll2.add(false);
// 2.equals(Object obj):要想返回true，需要当前集合和形参集合的元素都相同(包括元素的顺序)。
System.out.println(coll.equals(coll1));  // false

// 3.集合 --> 数组：toArray()
Object[] arr = coll.toArray();
for(int i = 0;i < arr.length;i++){
    System.out.println(arr[i]);
}
// 若集合中都是相同类型的元素,若都为Integer类型
// 第一种方式(最常用)
Integer[] integer = arrayList.toArray(new Integer[0]);
// 第二种方式(容易理解)
Integer[] integer1 = new Integer[arrayList.size()];
arrayList.toArray(integer1);
// 抛出异常，java不支持向下转型,讲Object数组转为Integer数组
//Integer[] integer2 = new Integer[arrayList.size()];
//integer2 = arrayList.toArray();
System.out.println();

// 4.数组 --> 集合:调用Arrays类的静态方法asList()
List<String> list = Arrays.asList(new String[]{"AA", "BB", "CC"});
System.out.println(list);	// [AA, BB, CC]
// 当是int[]时，会当成一个int数组对象
List arr1 = Arrays.asList(new int[]{123, 456});
System.out.println(arr1.size());	// 1
// 包装类数组时，会当成多个元素
List arr2 = Arrays.asList(new Integer[]{123, 456});
System.out.println(arr2.size());	// 2
```

Person类：重写equals()方法

```java
// Person对象重写equals方法
public class Person {
    String name;
    int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age &&
                Objects.equals(name, person.name);
    }
}
```

## 三.集合遍历

### 1.Iterator接口

Iterator对象称为迭代器(设计模式的一种)，主要用于**遍历 Collection 集合中的元素**。

Collection接口继承了java.lang.Iterable接口，该接口有一个iterator()方法，那么所有实现了Collection接口的集合类都有一个**iterator()方法，用以返回一个实现了Iterator接口的对象**。

- 内部的方法：`hasNext()`：**判断是否还有下一个元素** 和  `next()`：**指针下移并将指向的元素返回**
- 集合对象每次调用iterator()方法都得到一个**全新的迭代器对象，默认游标都在集合的第一个元素之前**
- 内部定义了remove(),可以**在遍历的时候，删除集合中的元素**。此方法不同于集合直接调用remove()

示例：

```java
Collection coll = new ArrayList();
coll.add(123);
coll.add(new Person("Jerry",20));
coll.add(new String("Tom"));

Iterator iterator = coll.iterator();
//hasNext():判断是否还有下一个元素
while(iterator.hasNext()){
    //next():1.指针下移 2.将下移以后集合位置上的元素返回
    System.out.print(iterator.next()+" ");	//123 Person{name='Jerry', age=20} Tom
}

//错误方式：
//集合对象每次调用iterator()方法都得到一个全新的迭代器对象，默认游标都在集合的第一个元素之前。
while (coll.iterator().hasNext()){	// 死循环
    System.out.print(coll.iterator().next());	// 123 123 123 ...
}

//遍历过程中删除集合中"Tom"元素
Iterator iterator = coll.iterator();
while (iterator.hasNext()){
    //iterator.remove();	//指针没有下移就remove()会报IllegalStateException
    Object obj = iterator.next();
    if("Tom".equals(obj)){
        iterator.remove();
    }
}
//删除"Tom"之后再遍历集合
iterator = coll.iterator();
while (iterator.hasNext()){
    System.out.println(iterator.next());	//123 Person{name='Jerry', age=20}
}
```

### 2.增强for循环

遍历集合：`for(集合元素的类型 局部变量 : 集合对象)`

```java
Collection coll = new ArrayList();
coll.add(123);
coll.add(new Person("Jerry",20));
coll.add(new String("Tom"));
coll.add(false);

//for(集合元素的类型 局部变量 : 集合对象)
//内部仍然调用了迭代器。
for(Object obj : coll){
    System.out.println(obj);
}
```

遍历数组：`for(数组元素的类型 局部变量 : 数组对象)`

```java
int[] arr = new int[]{1,2,3,4,5,6};
//for(数组元素的类型 局部变量 : 数组对象)
for(int i : arr){
    System.out.println(i);
}
```

注意：数组赋值时不能使用增强for循环

```java
String[] arr = new String[]{"MM","MM","MM"};

//    //方式一：普通for赋值
//    for(int i = 0;i < arr.length;i++){
//        arr[i] = "GG";
//    }

//方式二：增强for循环
for(String s : arr){
    s = "GG";	// s是局部变量，存储在栈中，并不是原数组的引用
}

for(int i = 0;i < arr.length;i++){
    System.out.print(arr[i]+" ");	// MM MM MM
}
```

## 四.List接口

List集合类中 **元素有序、且可重复**，集合中的每个元素都有其对应的**顺序索引**。

JDK API中List接口的实现类常用的有：**ArrayList、LinkedList和 Vector**。

LIst除了从**Collection集合继承的方法**外,List集合里添加了一些**根据索引来操作集合元素的方法**.

插:

-   `void add(int index, Object ele)` :  在 index 位置插入ele 元素
-   `boolean addAll(int index, Collection eles)` :  从index 位置开始将eles 中的所有元素添加进来

删:

-   `Object remove(int index) ` : **移除指定index 位置的元素，并返回此元素**

改:

-   `Object set(int index, Object ele) ` : **设置指定index 位置的元素为ele**

查:

-   `Object get(int index) ` : **获取指定index 位置的元素**
-   `int indexOf(Object obj) `: 返回obj 在集合中首次出现的位置,如果不存在，返回-1.
-   `int lastIndexOf(Object obj) ` : 返回obj 在当前集合中末次出现的位置,如果不存在，返回-1.
-   `List subList(int fromIndex, int toIndex) ` : 返回从fromIndex 到toIndex位置的左闭右开区间的子集合

### 1.ArrayList

ArrayList 是 List 接口的**主要实现类**,本质上,ArrayList是对象引用的一个"变长"数组,容量能动态增长.

ArrayList 的JDK1.8 之前与之后的实现区别？

-   JDK1.7：ArrayList像饿汉式，直接创建一个**初始容量为10的数组**
-   JDK1.8：ArrayList像懒汉式，一开始创建一个长度为0的数组，当**添加第一个元素时再创建一个初始容量为10的数组**

说一说ArrayList的扩容机制(1.8):

- 构造方法: **以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10**。
- 扩容机制: **ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右(oldCapacity为偶数就是1.5倍，否则是1.5倍左右)**

区分List中remove(int index)和remove(Object obj) ?

```java
List list = new ArrayList();
list.add(1);	// 自动装箱
list.add(2);
list.add(3);
updateList(list);
System.out.println(list);	// [1, 3]
// 判断list是否包含3,contains没有索引的形参
System.out.println(list.contains(3));	// true

private void updateList(List list) {
  // list.remove(2);	// 2代表索引,先根据索引
  list.remove(new Integer(2));	// 2代表对象
}
```

更多内容详看 : 源码学习 -- >  [ArrayList](http://itzlg.gitee.io/java-development-notes/#/docs/javaSenior/collection/source)

### 2.LinkedList

**双向链表**，**内部没有声明数组**，而是**定义了Node类型的first和last，用于记录首末元素。定义内部类Node，作为LinkedList中保存数据的基本结构。**

Node除了保存数据，还定义了两个变量：**prev变量记录前一个元素的位置; next变量记录下一个元素的位置.**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019122518054592.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

```java
	// 双向链表
    private static class Node<E> {    // 内部类：只有当前类需要使用，外面类不需要使用时
        E item;        // 泛型，obj对象数据
        Node<E> next;    // 下一个元素节点地址：指向下一个元素的指针
        Node<E> prev;    // 上一个元素节点地址：指向上一个元素的指针

        // 元素节点分为三部分：上节点地址，obj对象数据，下节点地址
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

对于 **频繁的插入或删除元素**的操作，建议使用LinkedList类，效率较高.

新增方法:

-    `void addFirst(Object obj)` 
-    ` void addLast(Object obj)` 
-    `Object getFirst()` 
-    `Object getLast()` 
-    `Object removeFirst()` 
-    `Object removeLast()` 

更多内容详看 : 源码学习 -- > LinkedList

### 3.Vector

Vector 是一个古老的集合，JDK1.0就有了。大多数操作与ArrayList相同，区别之处在于Vector是线程安全的.

在各种list中，最好把**ArrayList作为默认选择。当插入、删除频繁时，使用LinkedList**；Vector总是比ArrayList慢，所以尽量避免使用。

更多内容详看 : 源码学习 --> ArrayList --> Vector

### 4.三者联系与区别

联系:<div style="color:red;">

-   都实现了List接口, 存储有序的、可重复的数据
-   add自定义类时,该类需要重写equals方法(contains,remove等方法以equals为标准判断的)</div>

区别:<div style="color:red;">

-   ArrayList: 底层使用**动态数组结构**; **线程不安全**,效率高; 使用无参构造函数创建对象时,默认创建**空列表**,添加第一个元素时,分配10个大小的空间,每次扩容为原来的**1.5倍**;对于**随机访问**get和set操作效率高于LinkedList
-   LinkedList: 底层使用**双向链表结构**; **线程不安全**; 对于频繁的**插入, 删除**操作效率比ArrayList高
-   Vector: 底层使用**动态数组结构**; **线程安全**,效率低; 默认创建**10个大小**的数组,每次扩容为原来的**2倍**</div>

## 五.Set接口

Set接口是Collection的子接口，set接口没有提供额外的方法，使用的都是Collection中声明过的方法。存储**无序的、不可重复**的数据，类似于高中数学中的"集合"。

以HashSet为例：

- **无序性**：不等于随机性。存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的哈希值决定的
- **不可重复性**：保证添加的元素按照equals()判断时，不能返回true.即：相同的元素只能添加一个

向Set(主要指：HashSet、LinkedHashSet)中添加的数据，其所在的类一定要**重写hashCode()和equals()**，重写的hashCode()和equals()尽可能保持一致性：**相等的对象必须具有相等的散列码**。 Set 判断两个对象是否相同不是使用 == 运算符，而是根据 **equals() 方法**。

HashSet和TreeSet是Set接口的实现类，LinkedHashSet是HashSet的子类。

- `HashSet`：作为Set接口的**主要实现类；线程不安全的；可以存储null值**
- `LinkedHashSet`：作为HashSet的子类；遍历其内部数据时，可以**按照添加的顺序遍历**；对于频繁的**遍历操作**，LinkedHashSet效率高于HashSet
- `TreeSet`：可以按照添加**对象的指定属性，进行排序**

### 1.HashSet

#### 1.1 概述

HashSet 是 Set接口的主要实现，**HashSet 按Hash算法来存储集合中的元素**，因此具有很好的存取、查找、删除性能。底层是 **数组+链表结构**。数组**初始容量为16**，当如果使用率超过0.75，（16 * 0.75=12）就会扩大容量为**原来的2倍**。（16扩容为32，依次为64,128....等）。

特点：不能保证元素的排列顺序、不是线程安全的、集合元素可以是null。

判断两个元素相等的标准：两个对象通过 **hashCode() 方法比较相等**，并且两个对象的 **equals() 方法**返回值也相等。

对于**存放在Set容器中的对象**， 对应的**类一定要重写equals() 和hashCode(Object obj) 方法，以实现对象相等规则** 。即： “**相等的对象必须具有相等的散列码**” 。

#### 1.2 添加元素过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180604760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

**向HashSet中添加元素的过程(重要)：**<div style="color:red;">

首先调用元素a所在类的hashCode()方法，计算元素a的哈希值，此哈希值接着通过某种算法计算出在HashSet**底层数组中的存放位置**（即为：索引位置），判断**数组此位置**上是否已经有元素：

- 如果此位置上没有其他元素，则元素a添加成功；
- 如果此位置上有其他元素b(或以链表形式存在的多个元素），则比较元素a与元素b的**hash值**：
  - 如果hash值不相同，则元素a添加成功；
  - 如果hash值相同，进而需要调用元素a所在类的**equals()方法**：
    - equals()返回true,元素a添加失败；
    - equals()返回false,则元素a添加成功。</div>

元素a 与已经存在指定索引位置上数据以**链表的方式存储**。可以用"七上八下"来形容。

- JDK7：元素a放到数组中，指向原来的元素
- JDK8：原来的元素在数组中，指向元素a

如果两个元素的 equals() 方法返回 true，但它们的 hashCode() 返回值不相等，hashSet 将会把它们存储在不同的位置，但依然可以添加成功。

示例：

```java
//其中Person 类中重写了hashCode() 和equal() 方法
HashSet set = new HashSet();
Person p1 = new Person(1001,"AA");
Person p2 = new Person(1002,"BB");

set.add(p1);
set.add(p2);
// [Person{id=1002, name='BB'}, Person{id=1001, name='AA'}]
System.out.println(set);

p1.name = "CC";
set.remove(p1);	// 由于p1的name变化导致hash值变化了，去查找对应的存储位置是null的
// [Person{id=1002, name='BB'}, Person{id=1001, name='CC'}]
System.out.println(set);    

set.add(new Person(1001,"CC"));	// p1原先的hash值是由"AA"算出来的
// [Person{id=1002, name='BB'}, Person{id=1001, name='CC'}, 
// Person{id=1001, name='CC'}]
System.out.println(set);

set.add(new Person(1001,"AA")); // hash值与p1相同，但equals时是false的
// [Person{id=1002, name='BB'}, Person{id=1001, name='CC'}, 
// Person{id=1001, name='CC'}, Person{id=1001, name='AA'}]
System.out.println(set);
```

图示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180620804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 1.3 HashSet如何检查重复

当你把对象加入`HashSet`时，HashSet会先计算对象的`hashcode`值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用`equals（）`方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。

**hashCode（）与equals（）的相关规定：**

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个equals方法返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. 综上，equals方法被覆盖过，则hashCode方法也必须被覆盖
5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。

**==与equals的区别**

1. ==是判断两个变量或实例是不是指向同一个内存空间 equals是判断两个变量或实例所指向的内存空间的值是不是相同
2. ==是指对内存地址进行比较 equals()是对字符串的内容进行比较
3. ==指引用是否相同 equals()指的是值是否相同

#### 1.4 重写方法原则

一般用idea自动生成的重写方法hashCode和equals方法就可以了，复写的hashCode方法有31这个数字。

重写hashCode方法：

- 同一个对象多次调用 hashCode() 方法应该返回相同的值
- 当两个对象的 equals() 方法比较返回 true 时，这两个对象的 hashCode()方法的返回值也应相等
- 对象中用作 equals() 方法比较的 Field，都应该用来计算 hashCode 值

重写equals方法：

- 当改写equals方法时，总要改写hashCode方法；
- 相等的对象必须具有相等的散列码
- 参与计算hashCode 的对象的属性也应该参与到equals() 中进行计算

#### 1.5 示例

```java
Set set = new HashSet();
set.add(456);
set.add(123);
set.add(123);
set.add("AA");
set.add("CC");
set.add(new User("Tom",12));
set.add(new User("Tom",12));
set.add(129);

// 若不重写hashCode方法，则用父类Object的hashCode方法，hash值是随机生成的
// 先判断hash值，hash值不同，两个user对象都会添加到集合中
Iterator iterator = set.iterator();
while(iterator.hasNext()){
    // AA	CC	129	456	123	User{name='Tom', age=12}
    System.out.print(iterator.next()+"\t");
}
```

Person类：

```java
public class User implements Comparable{
    private String name;
    private int age;

    public User() {
    }
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        System.out.println("User equals()....");
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        User user = (User) o;

        if (age != user.age) return false;
        return name != null ? name.equals(user.name) : user.name == null;
    }
    @Override
    public int hashCode() { //return name.hashCode() + age;
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        return result;
    }

    //按照姓名从大到小排列,年龄从小到大排列
    @Override
    public int compareTo(Object o) {
        if(o instanceof User){
            User user = (User)o;
//            return -this.name.compareTo(user.name);
            int compare = -this.name.compareTo(user.name);
            if(compare != 0){
                return compare;
            }else{
                return Integer.compare(this.age,user.age);
            }
        }else{
            throw new RuntimeException("输入的类型不匹配");
        }

    }
}
```

使用HashSet去除List重复数字值：

```java
public List duplicateList(List list) {
    HashSet set = new HashSet();
    set.addAll(list);
    return new ArrayList(set);
}
```

### 2.LinkedHashSet

LinkedHashSet 是 HashSet 的子类。根据**元素的 hashCode 值来决定元素的存储位置**，在添加数据的同时，每个数据还维护了**两个引用，记录此数据前一个数据和后一个数据**。对于**频繁的遍历操作，效率高于HashSet**。

底层结构：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180638165.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

示例：

```java
Set set = new LinkedHashSet();
set.add(456);
set.add(123);
set.add(123);
set.add("AA");
set.add("CC");
set.add(new User("Tom",12));
set.add(new User("Tom",12));
set.add(129);

// 打印输出是按照元素添加的顺序，原因是由于每个数据还维护了两个引用
// 存储位置是由hash值决定的
Iterator iterator = set.iterator();
while(iterator.hasNext()){
    //456	123	AA	CC	User{name='Tom', age=12}	129	
    System.out.print(iterator.next()+"\t");
}
```

### 3.TreeSet

TreeSet 是 SortedSet 接口的实现类，TreeSet 可以确保**集合元素处于排序状态**。TreeSet底层使用 **红黑树结构**存储数据。有序，查询速度比List快。向TreeSet中**添加的数据**，要求是**相同类的对象**。

红黑树介绍：[平衡查找树之红黑树](https://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html)

结构：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225180648282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

TreeSet 两种排序方法： 自然排序和 定制排序。默认情况下，TreeSet 采用自然排序。

- 自然排序中，比较两个对象是否相同的标准为：compareTo()返回0.不再是equals().
- 定制排序中，比较两个对象是否相同的标准为：compare()返回0.不再是equals().

#### 3.1 自然排序

 TreeSet 会调用集合元素的 compareTo(Object obj) 方法来比较元素之间的大小关系，然后将集合元素按升序(默认情况)排列。

如果试图把一个对象添加到 TreeSet 时，则该对象的类必须**实现 Comparable接口**。实现 Comparable 的类必须实现 compareTo(Object obj) 方法，两个对象即通过compareTo(Object obj) 方法的返回值来比较大小。

示例：

```java
TreeSet set = new TreeSet();

set.add(new User("Tom",12));
set.add(new User("Jerry",32));
set.add(new User("Jim",2));
set.add(new User("Mike",65));
set.add(new User("Jack",33));
set.add(new User("Jack",56));

Iterator iterator = set.iterator();
while(iterator.hasNext()){
    System.out.println(iterator.next());
}
//        User{name='Tom', age=12}
//        User{name='Mike', age=65}
//        User{name='Jim', age=2}
//        User{name='Jerry', age=32}
//        User{name='Jack', age=33}
//        User{name='Jack', age=56}

// User类
public class User implements Comparable {
    //按照姓名从大到小排列,年龄从小到大排列
    @Override
    public int compareTo(User u) {
        int compare = -this.name.compareTo(u.name);
        if(compare != 0){
            return compare;
        }else {
            return Integer.compare(this.age,u.age);
        }
    }
}
```

#### 3.2 定制排序

定制排序，通过**Comparator接口**来实现。需要重写compare(T o1,T o2)方法。将实现Comparator接口的实例作为**形参传递给TreeSet的构造器**。向TreeSet中只能添加**类型相同**的对象。否则发生ClassCastException异常。

示例：

```java
Comparator com = new Comparator() {
    //按照年龄从小到大排列
    @Override
    public int compare(User u1, User u2) {
        return Integer.compare(u1.getAge(),u2.getAge());
    }
};

TreeSet set = new TreeSet(com);
set.add(new User("Tom",12));
set.add(new User("Jerry",32));
set.add(new User("Jim",2));
set.add(new User("Mike",65));
set.add(new User("Mary",33));
set.add(new User("Jack",33));
set.add(new User("Jack",56));

Iterator iterator = set.iterator();
while(iterator.hasNext()){
    System.out.println(iterator.next());
}
```

## 六.Map接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191220135937272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

1.Map与Collection并列存在。用于保存具有**映射关系的数据：key-value**。key和value可以是任何引用类型的数据。 Map接口的常用实现类：**HashMap、TreeMap、LinkedHashMap和**
**Properties**。

- `HashMap`：作为Map的**主要实现类；线程不安全的，效率高；可以存储null的key和value**。
- `LinkedHashMap`：保证在遍历map元素时，可以按照**添加的顺序实现遍历**。原因：在原有的HashMap底层结构基础上，添加了一对指针，指向前一个和后一个元素。对于**频繁的遍历操作，此类执行效率高于HashMap**。
- `TreeMap`：保证按照添加的key-value对进行排序，实现排序遍历。此时考虑**key的自然排序或定制排序**。底层使用**红黑树**。
- `Hashtable`：作为古老的实现类；**线程安全的，效率低；不能存储null的key或value**。
- `Properties`：常用来**处理配置文件**。**key和value都是String类型**。

**2.Map常用方法：**

- 添加：
  - `Object put(Object key,Object value)`：**将指定key-value添加到(或修改)当前map对象中**
  - `void putAll(Map m)`：将m中的所有key-value对存放到当前map中
- 删除：
  - `Object remove(Object key)`：**移除指定key的key-value对，并返回value**
  - `void clear()`：清空当前map中的所有数据
- 修改：
  - `Object put(Object key,Object value)`：将指定key-value添加到(或修改)当前map对象中
- 查询：
  - `Object get(Object key)`：**获取指定key对应的value**
  - `int size()`：**返回map中key-value对的个数**
- 判断：
  - `boolean containsKey(Object key)`：**是否包含指定的key**
  - `boolean containsValue(Object value)`：是否包含指定的value
  - `boolean isEmpty()`：**判断当前map是否为空**
  - `boolean equals(Object obj)`：**判断当前map和参数对象obj是否相等**
- 遍历：
  - `Set keySet()`：**返回所有key构成的Set集合**
  - `Collection values()`：**返回所有value构成的Collection集合**
  - `Set entrySet()`：**返回所有key-value对构成的Set集合**

### 1.HashMap

HashMap是Map接口的主要实现类。允许使用null键和null值。

#### 1.1 结构理解

- `key`：**无序的、不可重复的，使用Set存储所有的key  -->  key所在的类要重写equals()和hashCode()**
- `value`：**无序的、可重复的，使用Collection存储所有的value  -->  value所在的类要重写equals()**
- `键值对`：key-value构成了一个Entry对象
- `entry`：**无序的、不可重复的，使用Set存储所有的entry**

即判断两个key相等的标准：hashCode相等且equals相等；判断value相等的标准：equals相等。

#### 1.2 重要常量

- `DEFAULT_INITIAL_CAPACITY` : 默认容量：16
- `DEFAULT_LOAD_FACTOR`：默认加载因子：0.75
- `threshold`：扩容的临界值(容量*填充因子)：16 * 0.75 => 12
- `TREEIFY_THRESHOLD`：Bucket(桶)中链表长度大于该默认值，转化为红黑树：8
- `MIN_TREEIFY_CAPACITY`：桶中的Node被树化时最小的hash表容量：64

#### 1.3 底层实现

##### JDK8之前：

- `new HashMap()`：创建了一个长度为**16的Entry[] table数组**
- **数组+链表**(形成链表时：**新的元素指向旧的元素**)

JDK1.8 之前 `HashMap` 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**。**HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突**。

**所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法, 换句话说使用扰动函数之后可以减少碰撞**。

JDK1.8的hash方法源码:

```java
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^ ：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

所谓 **“拉链法”** 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

##### JDK8之后:

推荐阅读:  [Java 8系列之重新认识HashMap](https://zhuanlan.zhihu.com/p/21673805)

 JDK1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。TreeMap、TreeSet以及JDK1.8之后的HashMap底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。

- `new HashMap()`：**没有创建**一个长度为16的数组，首次**调用put()方法**，底层创建成都为**16的 Node[] 数组**
- **数组+链表+红黑树**(形成链表时：**旧的元素指向新的元素**)
- 当数组的某一个索引位置上的元素以**链表形式存在的数据个数(大于8) 且当前数组的长度(大于64)**时，此索引位置上的**所有数据改为使用红黑树存储**

**创建时不指定容量大小,HashMap默认初始化为16,每次扩容为原来的2倍;创建时指定容量初始值,HashMap总是使用2的幂作为哈希表的大小,会将其扩充为2的幂次方大小(`tableSizeFor()`方法保证).**

下面这个方法保证了 HashMap 总是使用2的幂作为哈希表的大小:

```java
/**
  * Returns a power of two size for the given target capacity.
  */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

##### HashMap的长度为什么是2的幂次方:

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。

经过hash方法得到的散列码是不能直接拿来用的,一般要先对数组的长度取模,得到的余数用来存放的位置.数组的下标的计算方法是`(n-1)&hash`(n代表数组长度). **取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是2的 n 次方；）。”** 并且 **采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方**.

补充:

与运算(&),或运算(|),异或运算(^): 参加运算的两个对象,按二进制位进行运算.

- &: 两个同时为1，结果为1，否则为0; (0&0=0；0&1=0；1&0=0；1&1=1;)
- |: 参加运算的两个对象，一个为1，其值为1; (0|0=0； 0|1=1； 1|0=1；  1|1=1;)
- ^: 参与运算的两个对象的二进制位值不同，则该位结果为1，否则为0; (0^0=0； 0^1=1； 1^0=1；  1^1=0;)

#### 1.4 添加元素的过程(JDK7)

<div style="color:red;">

向HashMap中**添加entry1(key，value)**，需要首先**计算entry1中key的哈希值**(根据key所在类的hashCode()计算得到)，此**哈希值**经过处理以后，得到在底层**Entry[]数组中要存储的位置i**。

- 如果**位置i**上没有元素，则entry1直接添加成功。
- 如果位置i上已经存在entry2(或还有链表存在的entry3，entry4)，则需要通过循环的方法，依次比较entry1中key和其他的entry。
  - 如果彼此**hash值**不同，则直接添加成功。
  - 如果hash值相同，继续比较二者是否**equals**。
    - 如果返回值为true，则使用entry1的value去**替换**equals为true的entry的value。
    - 如果遍历一遍以后，发现所有的equals返回都为false,则entry1仍可添加成功。entry1**指向原有**的entry元素。</div>

#### 1.5 数组扩容

JDK7：

在不断的添加过程中，会涉及到扩容问题，**当超出临界值12(且要存放的位置非空)**时，扩容。默认的扩容方式：**扩容为原来容量的2倍，并将原有的数据复制过来**。

HashMap数组扩容后，原数组中的数据必须**重新计算其在新数组中的位置，并放进去，这就是resize方法**。比较消耗性能。

JDK8：

扩容：当HashMap中元素个数超过16 * 0.75=12（这个值就是代码中的threshold值，也叫做临界值）的时候，就把数组的大小扩展为 2*16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，并放进去。

树形化：当HashMap中的其中**一个链的对象个数如果达到了8个**，此时如果**数组capacity没有达到64**，那么HashMap会**先扩容解决**，如果**已经达到了64**，那么这个**链会变成树**，结点类型由Node变成TreeNode类型。

#### 1.6 负载因子值

- 负载因子的大小决定了HashMap的数据密度
- 负载因子越大密度越大，发生碰撞的几率越高，数组中的链表越容易长,造成查询或插入时的**比较次数增多**，性能会下降
- 负载因子越小，就越容易触发扩容，数据密度也越小，意味着发生碰撞的几率越小，数组中的链表也就越短，查询和插入时比较的次数也越小，性能会更高。但是会**浪费一定的内存空间**。而且经常扩容也会影响性能
- 按照语言参考及研究经验，会将负载因子设置为 **0.7~0.75**，此时平均检索长度接近于常数

#### 1.7 HashMap与HashSet 的区别

如果你看过 `HashSet` 源码的话就应该知道：HashSet 底层就是基于 HashMap 实现的。（HashSet 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 HashSet 自己不得不实现之外，其他方法都是直接调用 HashMap 中的方法。

| HashMap                          | HashSet                                                      |
| -------------------------------- | ------------------------------------------------------------ |
| 实现了Map接口                    | 实现Set接口                                                  |
| 存储键值对                       | 仅存储对象                                                   |
| 调用 `put（）`向map中添加元素    | 调用 `add（）`方法向Set中添加元素                            |
| HashMap使用键（Key）计算Hashcode | HashSet使用成员对象来计算hashcode值，对于两个对象来说hashcode可能相同，所以equals()方法用来判断对象的相等性， |

#### 1.8 示例

```java
Map map = new HashMap();
map.put("AA",123);
map.put(45,123);
map.put("BB",56);
map.put(null,76);

System.out.println(map.get(45));	// 123

Object value = map.remove("AA");
System.out.println(value);	// 123 

boolean isExist = map.containsKey("BB");
System.out.println(isExist);	// true
isExist = map.containsValue(123);
System.out.println(isExist);	// true

map.clear();
System.out.println(map.isEmpty());	// true

//遍历时：通常使用增强for循环
//遍历所有的key集：keySet()
Set set = map.keySet();
Iterator iterator = set.iterator();
while(iterator.hasNext()){
    System.out.println(iterator.next());
}
//遍历所有的value集：values()
Collection values = map.values();
for(Object obj : values){
    System.out.println(obj);
}
//遍历所有的key-value
//方式一：entrySet()
Set entrySet = map.entrySet();
Iterator iterator1 = entrySet.iterator();
while (iterator1.hasNext()){
    Object obj = iterator1.next();
    //entrySet集合中的元素都是entry
    Map.Entry entry = (Map.Entry) obj;
    System.out.println(entry.getKey() + "---->" + entry.getValue());
}
//方式二：
Set keySet = map.keySet();
Iterator iterator2 = keySet.iterator();
while(iterator2.hasNext()){
    Object key = iterator2.next();
    Object value = map.get(key);
    System.out.println(key + "=====" + value);
}
```

更多内容详看 : 源码学习 -- > HashMap

### 2.LinkedHashMap

LinkedHashMap 是 HashMap 的子类。

在HashMap存储结构的基础上，**使用了一对双向链表来记录添加元素的顺序**。迭代遍历时：**顺序与添加顺序一致**。

```java
// HashMap内部类：Node
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}

// LinkedHashMap内部类：Entry 继承 Node
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;	// 前驱节点和后继节点：记录添加元素的顺序
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

### 3.TreeMap

TreeMap底层使用**红黑树结构**存储数据。可以保证所有的 Key-Value 对处于**有序状态**。向TreeMap中添加key-value，要求**key必须是由同一个类创建的对象**。

按照key进行排序：

- 自然排序：所有key必须**实现Comparable接口**，所有key应是同一个类的对象，否则抛出ClassCastException
- 定制排序：TreeMap构造器中**传入一个Comparator对象**，该对象负责对所有key进行排序，此时不需要Key所在类实现Comparable接口

**判断两个key相等的标准：两个key通过compareTo()方法或者compare()方法返回0**

自然排序示例：

```java
TreeMap map = new TreeMap();
User u1 = new User("Tom",23);
User u2 = new User("Jerry",32);
User u3 = new User("Jack",20);
User u4 = new User("Rose",18);

map.put(u1,98);
map.put(u2,89);
map.put(u3,76);
map.put(u4,100);

// 遍历打印
Set<User> keys = map.keySet();
for (User key : keys) {
    System.out.print(key + "：" + map.get(key)+"   ");

}

// User类：实现 Comparable接口
public class User implements Comparable{
	private String name;
    private int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    //按照姓名从大到小排列,年龄从小到大排列
    @Override
    public int compareTo(User u) {
        if(this.name.equals(u.name)){
            return Integer.compare(this.age,U.age);
        }else{
            return -this.name.compareTo(u.name);
        }
    }
}    
```

定制排序示例：

```java
TreeMap map = new TreeMap(new Comparator() {
    @Override
    public int compare(User u1, User u2) {
        return Integer.compare(u1.getAge(), u2.getAge());
    }
});

User u1 = new User("Tom",23);
User u2 = new User("Jerry",32);
User u3 = new User("Jack",20);
User u4 = new User("Rose",18);

map.put(u1,98);
map.put(u2,89);
map.put(u3,76);
map.put(u4,100);

// 遍历打印
Set<User> keys = map.keySet();
for (User key : keys) {
    System.out.print(key + "：" + map.get(key)+"   ");
}
```

### 4.Hashtable与Properties

#### 4.1 HashMap与Hashtable的区别

1.Hashtable比较古老了，JDK1.0就提供了。**Hashtable实现原理和HashMap相同**，功能相同。判断两个key或value值相等的标准也一致。

HashMap和Hashtable的区别:

1. **线程是否安全：** HashMap 是非线程安全的，HashTable 是线程安全的；HashTable 内部的方法基本都经过`synchronized` 修饰。（如果你要保证线程安全的话就使用 ConcurrentHashMap 吧！）；
2. **效率：** 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。另外，HashTable 基本被淘汰，不要在代码中使用它；
3. **对Null key 和Null value的支持：** HashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null。。但是在 HashTable 中 put 进的键值只要有一个 null，直接抛出 NullPointerException。
4. **初始容量大小和每次扩充容量大小的不同 ：** ①创建时如果不指定容量初始值，Hashtable 默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。②创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 HashMap 会将其扩充为2的幂次方大小（HashMap 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 HashMap 总是使用2的幂作为哈希表的大小,后面会介绍到为什么是2的幂次方。
5. **底层数据结构：** JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。

总结：

- **HashMap线程不安全，Hashtable线程安全, HashMap要比HashTable效率要高一点**
- **HashMap允许使用null作为key或value，Hashtable不允许**
- **创建时不指定容量大小,HashMap默认初始化为16,每次扩容为原来的2倍,Hashtable默认初始化大小为11,每次扩容为原来的2n+1;创建时指定容量初始值,HashMap总是使用2的幂作为哈希表的大小,会将其扩充为2的幂次方大小(`tableSizeFor()`方法保证),Hashtable会直接使用给定的大小**
- **JDK1.8以后的HashMap解决hash冲突时,当链表长度大于阈值8且数组长度大于64时,将当前索引位置的链表转为红黑树,Hashtable没有这样的机制**

#### 4.2 Properties的使用

2.Properties 类是 Hashtable 的子类，该对象用于**处理配置文件**。由于属性文件里的 key、value 都是字符串类型，所以 Properties 里的 **key和 value 都是字符串类型**。

存取数据时，建议使用`setProperty(String key,String value)`方法和`getProperty(String key)`方法：

```java
Properties pros = new Properties();

FileInputStream fis = new FileInputStream("jdbc.properties");
pros.load(fis);//加载流对应的文件

String name = pros.getProperty("name");
System.out.println("name = " + name);
```

## 七.Collections工具类

Collections 是一个操作 Collection 和 Map 等集合的工具类。Collections 中提供了一系列**静态的方法**对集合元素进行**排序、查询和替换**等操作，还提供了对集合对象设置不可变、对集合对象**实现同步控制**等方法。

参考链接: [Collections工具类和Arrays工具类常见方法](https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/java/basic/Arrays,CollectionsCommonMethods.md)

### 1.排序操作

| 方法                     | 描述                                    |
| ---------------------- | ------------------------------------- |
| reverse(List)          | 反转 List 中元素的顺序                        |
| shuffle(List)          | 对 List 集合元素进行随机排序                     |
| sort(List)             | 根据元素的自然顺序对指定 List 集合元素按升序排序           |
| sort(List, Comparator) | 根据指定的 Comparator 产生的顺序对 List 集合元素进行排序 |
| swap(List, int, int)   | 将指定 list 集合中的 i 处元素和 j 处元素进行交换        |

示例：

```java
List list = new ArrayList();
list.add(56);
list.add(12);
list.add(34);
list.add(89);
list.add(-53);
list.add(0);
list.add(34);
// 反转
Collections.reverse(list);
System.out.println(list);
// 随机排序
Collections.shuffle(list);
System.out.println(list);
// 两个索引位置的元素进行交换
Collections.swap(list,1,2);
System.out.println(list);

// Collections自然排序和定制排序方法底层都还是调用的 Arrays.sort(a, (Comparator) c)方法
// 自然排序
Collections.sort(list);
System.out.println(list);

// List接口中sort默认方法部分源码(JDK8中接口新特性)
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();	// 将集合转为数组
    Arrays.sort(a, (Comparator) c);	// 调用Arrays的sort方法
    
    ...
}
```

### 2.查找、替换操作

| 方法                                       | 描述                               |
| ---------------------------------------- | -------------------------------- |
| Object max(Collection)                   | 根据元素的自然顺序，返回给定集合中的最大元素           |
| Object max(Collection，Comparator)        | 根据 Comparator 指定的顺序，返回给定集合中的最大元素 |
| Object min(Collection)                   |                                  |
| Object min(Collection，Comparator)        |                                  |
| int frequency(Collection，Object)         | 返回指定集合中指定元素的出现次数                 |
| void copy(List dest,List src)            | 将src中的内容复制到dest中                 |
| boolean replaceAll(List list，Object oldVal，Object newVal) | 使用新值替换List 对象的所有旧值               |

示例：

```java
// 元素出现次数
int frequency = Collections.frequency(list, 34);
System.out.println(frequency);

// 复制
//报异常：IndexOutOfBoundsException("Source does not fit in dest")
// List dest = new ArrayList();
// Collections.copy(dest,list);
//正确的：目标集合size大小 大于等于 原集合size大小
List dest = Arrays.asList(new Object[list.size()]);
System.out.println(dest.size());//list.size();
Collections.copy(dest,list);

// Collections中void copy(List dest,List src)方法源码
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    int srcSize = src.size();
    // 若原集合size大小 大于 目标集合size大小(并不是集合初始化容量大小)，则抛异常
    if (srcSize > dest.size())	
        throw new IndexOutOfBoundsException("Source does not fit in dest");

    ...
}
```

### 3.同步控制操作

Collections 类中提供了多个 `synchronizedXxx()` 方法，该方法可使将**指定集合包装成线程同步的集合**，从而可以解决多线程并发访问集合时的线程安全问题。

```java
List list = new ArrayList();
list.add(56);
list.add(12);
//返回的list2即为线程安全的List
List list2 = Collections.synchronizedList(list);
```



参考链接: [JavaGuide--Java集合面试题总结](https://snailclimb.gitee.io/javaguide-interview/#/./docs/b-2Java%E9%9B%86%E5%90%88)