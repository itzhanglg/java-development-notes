
## 一.java比较器

Java中的对象，正常情况下，只能进行比较：==  或  != 。不能使用 > 或 < 的

- 但是在开发场景中，我们需要对多个对象进行排序，言外之意，就需要比较对象的大小。
- 如何实现？使用两个接口中的任何一个：Comparable 或 Comparator

Java实现对象数组排序的方式有两种：

- 自然排序：java.lang.Comparable (**实现类的对象在任何位置都可以比较大小**)
- 定制排序：java.util.Comparator (**属于临时性的比较**)

### 1.Comparable接口

Comparable接口强行对实现它的每个类的对象进行整体排序。这种排序被称为类的**自然排序**。

1.像**String、包装类**等实现了Comparable接口，重写了**compareTo(obj)方法**，给出了比较两个对象大小的方式，进行**从小到大**的排列。

2.对于**自定义类**来说，如果需要排序，我们可以让自定义类**实现Comparable接口**，重写**compareTo(obj)方法**。在compareTo(obj)方法中**指明如何排序**。

3.实现Comparable接口的对象列表（和数组）可以通过 `Collections.sort` 或`Arrays.sort`进行自动排序。实现此接口的对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

4.重写compareTo(obj)的规则：

- 如果当前对象this大于形参对象obj，则返回正整数
- 如果当前对象this小于形参对象obj，则返回负整数
- 如果当前对象this等于形参对象obj，则返回零

5.Comparable的典型实现：(默认都是从小到大排列)

- String：按照字符串中字符的Unicode值进行比较
- Character：按照字符的Unicode值来进行比较
- 数值类型对应的包装类以及BigInteger、BigDecimal：按照它们对应的数值大小进行比较
- Boolean：true 对应的包装类实例大于 false 对应的包装类实例
- Date、Time等：后面的日期时间比前面的日期时间大

6.示例：

```java
// 1.String类已经实现了Comparable接口
String[] arr = new String[]{"AA","CC","KK","MM","GG","JJ","DD"};
// 调用Arrays工具类进行自动排序
Arrays.sort(arr);
// [AA, CC, DD, GG, JJ, KK, MM]
System.out.println(Arrays.toString(arr));

// ------------------------------------------------------------------
// 2.自定义Goods类
Goods[] arr = new Goods[5];
arr[0] = new Goods("lenovoMouse",34);
arr[1] = new Goods("dellMouse",43);
arr[2] = new Goods("xiaomiMouse",12);
arr[3] = new Goods("huaweiMouse",65);
arr[4] = new Goods("microsoftMouse",43);

Arrays.sort(arr);
System.out.println(Arrays.toString(arr));
// [Goods{name='xiaomiMouse', price=12.0},
// Goods{name='lenovoMouse', price=34.0},
// Goods{name='microsoftMouse', price=43.0},
// Goods{name='dellMouse', price=43.0},
// Goods{name='huaweiMouse', price=65.0}]

// ------------------------------------------------------------------
// 自定义Goods类，实现Comparable接口，重写compareTo()方法
public class Goods implements Comparable{
    private String name;
    private double price;
    public Goods(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    //指明商品比较大小的方式:按照价格从低到高排序,再按照产品名称从高到低排序
    @Override
    public int compareTo(Goods g) {
        if(this.price == g.price){
            return -this.name.compareTo(g.name); // 添加符号则从高到低
        }else{
            return Double.compare(this.price,g.price);
        }
    }
}
```

### 2.Comparator接口

1.背景：当元素的类型**没有实现java.lang.Comparable接口**而又不方便修改代码，或者实现了java.lang.Comparable接口的排序规则不适合当前的操作，那么可以考虑使用 Comparator 的对象来排序。

2.**重写compare(Object o1,Object o2)方法**，比较o1和o2的大小：如果方法返回正整数，则表示o1大于o2；如果返回0，表示相等；返回负整数，表示o1小于o2。

3.示例：

```java
// 1.String类数组
String[] arr = new String[]{"AA","CC","KK","MM","GG","JJ","DD"};
Arrays.sort(arr,new Comparator<String>(){	// 匿名对象
    //按照字符串从大到小的顺序排列
    @Override
    public int compare(String s1, String s2) {
        return -s1.compareTo(s2);
    }
});
System.out.println(Arrays.toString(arr));   // [MM, KK, JJ, GG, DD, CC, AA]

// ------------------------------------------------------------------
// 2.自定义类数组
Goods[] arr = new Goods[6];
arr[0] = new Goods("lenovoMouse",34);
arr[1] = new Goods("dellMouse",43);
arr[2] = new Goods("xiaomiMouse",12);
arr[3] = new Goods("huaweiMouse",65);
arr[4] = new Goods("huaweiMouse",224);
arr[5] = new Goods("microsoftMouse",43);

Arrays.sort(arr, new Comparator<Goods>() {	// 匿名对象
    //指明商品比较大小的方式:按照产品名称从低到高排序,再按照价格从高到低排序
    @Override
    public int compare(Goods g1, Goods g2) {
        if(g1.getName().equals(g2.getName())){
            return -Double.compare(g1.getPrice(),g2.getPrice());
        }else{
            return g1.getName().compareTo(g2.getName());
        }
    }
});

System.out.println(Arrays.toString(arr));
// [Goods{name='dellMouse', price=43.0},
// Goods{name='huaweiMouse', price=224.0},
// Goods{name='huaweiMouse', price=65.0},
// Goods{name='lenovoMouse', price=34.0},
// Goods{name='microsoftMouse', price=43.0},
// Goods{name='xiaomiMouse', price=12.0}]

// ------------------------------------------------------------------
// 自定义Goods类，不实现Comparable接口
public class Goods {
    private String name;
    private double price;
    public Goods(String name, double price) {
        this.name = name;
        this.price = price;
    }
}
```

## 二.System类

System类代表系统，系统级的很多属性和控制方法都放置在该类的内部。该类位于java.lang包。

由于该类的构造器是private的，所以无法创建该类的对象，也就是无法实例化该类。其**内部的成员变量和成员方法都是static的**。

成员变量：

| 成员变量 | 描述           |
| ---- | ------------ |
| in   | 标准输入流(键盘输入)  |
| out  | 标准输出流(显示器)   |
| err  | 标准错误输出流(显示器) |

成员方法：

| 成员方法                            | 描述                              |
| ------------------------------- | ------------------------------- |
| native long currentTimeMillis() | 返回当前的计算机时间(1970-01-01 00-00-00) |
| void exit(int status)           | 退出程序。其中status的值为0代表正常退出，非零代表    |
| void gc()                       | 请求系统进行垃圾回收(不一定是立刻回收)            |
| String getProperty(String key)  | 获得系统中属性名为key的属性对应的值             |

系统中常见的属性名以及属性的作用：

| 属性名          | 属性说明        |
| ------------ | ----------- |
| java.version | java运行时环境版本 |
| java.home    | java安装目录    |
| os.name      | 操作系统的名称     |
| os.version   | 操作系统的版本     |
| user.name    | 用户的账户名称     |
| user.home    | 用户的主目录      |
| user.dir     | 用户的当前工作目录   |

## 三.Math类

java.lang.Math 提供了一系列静态方法用于科学计算。其方法的参数和返回值类型一般为double 型。

| 方法                                       | 描述                        |
| ---------------------------------------- | ------------------------- |
| abs                                      | 绝对值                       |
| sqrt                                     | 平方根                       |
| pow(double a, double b)                  | a的b次幂                     |
| log                                      | 自然对象                      |
| exp                                      | e为底指数                     |
| max(double a, double b) / min(double a, double b) | a和b的最大值/最小值               |
| random()                                 | 返回[0.0, 1.0)的随机数          |
| long round(double a)                     | double 型数据a转换为long型(四舍五入) |
| toDegrees(double angrad)                 | 弧度—> 角度                   |
| toRadians(double angdeg)                 | 角度—> 弧度                   |

## 四.BigInteger与BigDecimal

### 1.BigInteger类

 java.math包的**BigInteger 可以表示不可变的任意精度的整数**。BigInteger 提供所有 Java 的基本整数操作符的对应物，并提供 java.lang.Math 的所有相关方法。

构造方法：

- `BigInteger(String val)`：根据字符串构建BigInteger对象

常用方法：

| 方法                                       | 描述                                       |
| ---------------------------------------- | ---------------------------------------- |
| BigInteger add(BigInteger val)           | 返回其值为 (this + val) 的 BigInteger          |
| BigInteger subtract(BigInteger val)      | 返回其值为 (this - val) 的 BigInteger          |
| BigInteger multiply(BigInteger val)      | 返回其值为 (this * val) 的 BigInteger          |
| BigInteger divide(BigInteger val)        | 返回其值为 (this / val) 的 BigInteger。整数相除只保留整数部分 |
| BigInteger remainder(BigInteger val)     | 返回其值为 (this % val) 的 BigInteger          |
| BigInteger[] divideAndRemainder(BigInteger val) | 返回包含 (this / val) 后跟(this % val) 的两个 BigInteger 的数组 |

### 2.BigDecimal类

 一般的Float类和Double类可以用来做科学计算或工程计算，但在商业计算中，要求数字精度比较高，故用到java.math.BigDecimal 类 。BigDecimal对象是**不可变**的。

**BigDecimal类支持不可变的、任意精度的有符号十进制定点数。**

**常用构造方法：**

- `public BigDecimal(double val)`：根据double构建BigDecimal对象(有精确度缺失)
- `public BigDecimal(String val)`：根据字符串构建BigDecimal对象(**建议采用**)

**常用方法：**

| 方法                                       | 描述       |
| ---------------------------------------- | -------- |
| public BigDecimal add(BigDecimal augend) | 加        |
| public BigDecimal subtract(BigDecimal subtrahend) | 减        |
| public BigDecimal multiply(BigDecimal multiplicand) | 乘        |
| public BigDecimal divide(BigDecimal divisor, int scale, int roundingMode) | 除        |
| public String toString()                 | 数值转换成字符串 |
| public double doubleValue()              | 以双精度数返回  |
| public BigDecimal setScale(int newScale, int roundingMode) | 格式化小数点   |

**注意**：

- `setScale(int newScale, int roundingMode)`

  由于BigDecimal对象是**不可变**的，所以此方法的调用不会导致原始对象被修改。 `setScale`返回具有适当比例的对象; 返回的**对象可能会被新分配( 值发生改变)，也可能不会被新分配( 值没有发生改变:如13.16格式化2位小数)**。

- **BigDecimal(String val)构造是靠谱的**，BigDecimal(“57.3”)就是妥妥的等于57.3，建议采用这种方式；

  BigDecimal(double val)等构造不太靠谱，在精确度上有缺失。

  ```java
  float a = 57.3f;
  BigDecimal decimalA = new BigDecimal(a);
  System.out.println(decimalA);   // 57.299999237060546875
  double b = 57.3;
  BigDecimal decimalB = new BigDecimal(b);
  System.out.println(decimalB);   // 57.2999999999999971578290569595992565155029296875
  double c = 57.3;
  BigDecimal decimalC = new BigDecimal(Double.toString(c));
  System.out.println(decimalC);   // 57.3
  double d = 57.3;
  BigDecimal decimalD = BigDecimal.valueOf(d);
  System.out.println(decimalD);   // 57.3
  ```

**示例**：

```java
BigDecimal bd = new BigDecimal("12435.351");
BigDecimal bd2 = new BigDecimal("11");
// System.out.println(bd.divide(bd2));  // 当不能整除时，必须给出小数位精确位，不然会报异常
// ROUND_HALF_UP：表示四舍五入模式
System.out.println(bd.divide(bd2, BigDecimal.ROUND_HALF_UP));  // 1130.486
System.out.println(bd.divide(bd2, 12, BigDecimal.ROUND_HALF_UP));  // 1130.486454545455

// 四舍五入
BigDecimal b = new BigDecimal("111231.5585");
// 调用setScale方法格式化小数点
double result = b.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();
System.out.println(result);  //111231.56

// 保留两位小数
double num = 13.154256
//方式一
DecimalFormat df1 = new DecimalFormat("0.00");
String str = df1.format(num);
System.out.println(str);  //13.15
//方式二
// #.00 表示两位小数 #.0000四位小数
DecimalFormat df2 =new DecimalFormat("#.00");
String str2 =df2.format(num);
System.out.println(str2);  //13.15
//方式三
//%.2f %.：表示小数点前任意位数；   2：表示两位小数格式后的结果	f：表示浮点型
String result = String.format("%.2f", num);
System.out.println(result);  //13.15
```










