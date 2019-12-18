## 一.字符串相关的类
### 1.String及常用方法
#### 1.1 String的特性
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204190526872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

String:字符串，使用一对""引起来表示。
1. String声明为`final`的，**不可被继承**
2. String实现了`Serializable`接口：表示字符串是**支持序列化**的。
    实现了`Comparable`接口：表示String可以**比较大小**
3. String内部定义了`final char[] value`用于**存储字符串数据**
4. String:代表**不可变的字符序列**。简称：**不可变性**。
  体现：
  - 当对字符串**重新赋值**时，需要**重写指定内存区域赋值**，不能使用原有的value进行赋值。
  - 当对现有的字符串进行**连接操作**时，也需要**重新指定内存区域赋值**，不能使用原有的value进行赋值。
  - 当调用String的`replace()`方法修改指定字符或字符串时，也需要**重新指定内存区域赋值**，不能使用原有的value进行赋值。
5. 通过**字面量**的方式（区别于new）给一个字符串赋值，此时的**字符串值声明在字符串常量池**中。
6. 字符串常量池中是**不会存储相同内容的字符串**的。

```java
	String s1 = "abc";//字面量的定义方式
	String s2 = "abc";
	s1 = "hello";
	System.out.println(s1 == s2);	// false 比较s1和s2的地址值	
	System.out.println(s1);	// hello
	System.out.println(s2);	// abc	
	System.out.println("*****************");
	
	String s3 = "abc";
	s3 += "def";
	System.out.println(s3);	// abcdef
	System.out.println(s2);	// abc	
	System.out.println("*****************");
	
	String s4 = "abc";
	String s5 = s4.replace('a', 'm');
	System.out.println(s4);	// abc
	System.out.println(s5);	// mbc
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120416002349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)
#### 1.2 String对象的创建
String的实例化方式：
- 通过字面量定义的方式：`String.valueOf()`
- 通过new + 构造器的方式：`new String()`

```java
	//通过字面量定义的方式：此时的s1和s2的数据javaEE声明在方法区中的字符串常量池中。
	String s1 = "javaEE";
	String s2 = "javaEE";
	//通过new + 构造器的方式:此时的s3和s4保存的地址值，是数据在堆空间中开辟空间以后对应的地址值。
	String s3 = new String("javaEE");
	String s4 = new String("javaEE");
	
	System.out.println(s1 == s2);//true
	System.out.println(s1 == s3);//false
	System.out.println(s1 == s4);//false
	System.out.println(s3 == s4);//false
	System.out.println("***********************");
	
	Person p1 = new Person("Tom",12);
	Person p2 = new Person("Tom",12);
	
	System.out.println(p1.name.equals(p2.name));//true
	System.out.println(p1.name == p2.name);//true
	
	p1.name = "Jerry";
	System.out.println(p2.name);//Tom
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204161749776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204162108690.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

**面试题**：String s = new String("abc");方式创建对象，在内存中创建了几个对象？
**两个**: 一个是堆空间中new结构，另一个是char[]对应的常量池中的数据："abc"

#### 1.3 String使用陷阱
关于拼接字符串后两个变量地址是否相等。<div style="color:red;">
- 常量与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量。
- 只要其中有一个是变量，结果就在堆中
- 如果拼接的结果调用intern()方法，返回值就在常量池中</div>

```java
	String s1 = "javaEE";
	String s2 = "hadoop";
	
	String s3 = "javaEEhadoop";
	String s4 = "javaEE" + "hadoop";
	String s5 = s1 + "hadoop";
	String s6 = "javaEE" + s2;
	String s7 = s1 + s2;
	
	final String s8 = "javaEE";//s8:常量
	String s9 = s8 + "hadoop";
	String s11 = s8 + s2;
	System.out.println(s3 == s9);   // true
	System.out.println(s3 == s11);   // false
	
	System.out.println(s3 == s4);//true
	System.out.println(s3 == s5);//false
	System.out.println(s3 == s6);//false
	System.out.println(s3 == s7);//false
	System.out.println(s5 == s6);//false
	System.out.println(s5 == s7);//false
	System.out.println(s6 == s7);//false
	
	String s10 = s6.intern();//返回值得到的s8使用的常量值中已经存在的“javaEEhadoop”
	System.out.println(s3 == s10);//true
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204163431407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)
**面试题：**

```java
	String str = new String("good");
	char[] ch = { 't', 'e', 's', 't' };
	
	public void change(String str, char ch[]) {	// 传递的是引用
	    str = "test ok";	// 给传递过来的地址重新赋值，而不是改变引用重新指向
	    ch[0] = 'b';
	}
	public static void main(String[] args) {
	    StringTest ex = new StringTest();
	    ex.change(ex.str, ex.ch);
	    System.out.println(ex.str);	// good string是不可变的，不能重新赋值
	    System.out.println(ex.ch);	// best 数组是可变的
	}
```

#### 1.4 String常用方法
a.基本方法：

| 方法                                       | 描述                                       |
| ---------------------------------------- | ---------------------------------------- |
| int length()                             | 返回字符串的长度： return value.length            |
| char charAt(int index)                   | 返回某索引处的字符return value[index]             |
| boolean isEmpty()                        | 判断是否是空字符串：return value.length == 0       |
| String toLowerCase()                     | 使用默认语言环境，将 String 中的所有字符转换为小写            |
| String toUpperCase()                     | 使用默认语言环境，将 String 中的所有字符转换为大写            |
| String trim()                            | 返回字符串的副本，忽略前导空白和尾部空白                     |
| boolean equals(Object obj)               | 比较字符串的内容是否相同                             |
| boolean equalsIgnoreCase(String anotherString) | 与equals方法类似，忽略大小写                        |
| String concat(String str)                | 将指定字符串连接到此字符串的结尾。 等价于用“+”                |
| int compareTo(String anotherString)      | 比较两个字符串的大小                               |
| String substring(int beginIndex)         | 返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。 |
| String substring(int beginIndex, int endIndex) | 返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。 |

b.包含：

| 方法                                       | 描述                                    |
| ---------------------------------------- | ------------------------------------- |
| boolean endsWith(String suffix)          | 测试此字符串是否以指定的后缀结束                      |
| boolean startsWith(String prefix)        | 测试此字符串是否以指定的前缀开始                      |
| boolean startsWith(String prefix, int toffset) | 测试此字符串从指定索引开始的子字符串是否以指定前缀开始           |
| boolean contains(CharSequence s)         | 当且仅当此字符串包含指定的 char 值序列时，返回 true       |
| int indexOf(String str)                  | 返回指定子字符串在此字符串中第一次出现处的索引               |
| int indexOf(String str, int fromIndex)   | 返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始      |
| int lastIndexOf(String str)              | 返回指定子字符串在此字符串中最右边出现处的索引               |
| int lastIndexOf(String str, int fromIndex) | 返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索 |

注：indexOf和lastIndexOf方法如果未找到都是返回-1

```java
	String str1 = "hellowworld";
	boolean b1 = str1.endsWith("rld");
	System.out.println(b1);     // true
	
	boolean b2 = str1.startsWith("He");
	System.out.println(b2);     // false
	
	boolean b3 = str1.startsWith("ll",2);
	System.out.println(b3);     // true
	
	String str2 = "wor";
	System.out.println(str1.contains(str2));    // true
	
	System.out.println(str1.indexOf("lol"));    // -1
	
	System.out.println(str1.indexOf("lo",5));   // -1   (3+2)
	
	String str3 = "hellorworld";
	
	System.out.println(str3.lastIndexOf("or")); // 7
	System.out.println(str3.lastIndexOf("or",6));   // 4    (7-1)
	
	//什么情况下，indexOf(str)和lastIndexOf(str)返回值相同？
	//情况一：存在唯一的一个str。情况二：不存在str
```

c.替换：

| 方法                                       | 描述                                       |
| ---------------------------------------- | ---------------------------------------- |
| String replace(char oldChar, char newChar) | 返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的**所有 oldChar** 得到的。 |
| String replace(CharSequence target, CharSequence replacement) | 使用指定的字面值替换序列替换此字符串所有匹配字面值目标序列的子字符串。      |
| String replaceAll(String regex, String replacement) | 使用给定的 replacement 替换此字符串所有匹配给定的正则表达式的子字符串。 |
| String replaceFirst(String regex, String replacement) | 使用给定的 replacement 替换此字符串匹配给定的正则表达式的第一个子字符串。 |

d.匹配:

| 方法                            | 描述                  |
| ----------------------------- | ------------------- |
| boolean matches(String regex) | 告知此字符串是否匹配给定的正则表达式。 |

e.切片：

| 方法                                      | 描述                                       |
| --------------------------------------- | ---------------------------------------- |
| String[] split(String regex)            | 根据给定正则表达式的匹配拆分此字符串。                      |
| String[] split(String regex, int limit) | 根据匹配给定的正则表达式来拆分此字符串，最多不超过limit个，如果超过了，剩下的全部都放到最后一个元素中。 |

```java
	String str1 = "helloworld";
	String str2 = str1.replace('o', 'u');
	System.out.println(str1);   // helloworld
	System.out.println(str2);   // helluwurld
	
	String str3 = str1.replace("he", "do");
	System.out.println(str3);   // dolloworld	
	System.out.println("*************************");
	
	String str = "12hello34world5java7891mysql456";
	//把字符串中的数字替换成,，如果结果中开头和结尾有，的话去掉
	String string = str.replaceAll("\\d+", ",").replaceAll("^,|,$", "");
	System.out.println(string);     // hello,world,java,mysql
	System.out.println("*************************");
	
	str = "12345";
	//判断str字符串中是否全部有数字组成，即有1-n个数字组成
	boolean matches = str.matches("\\d+");
	System.out.println(matches);    // true
	String tel = "0571-4534289";
	//判断这是否是一个杭州的固定电话
	boolean result = tel.matches("0571-\\d{7,8}");
	System.out.println(result);     // true
	System.out.println("*************************");
	
	str = "hello|world|java";
	String[] strs = str.split("\\|");
	for (int i = 0; i < strs.length; i++) {
	    System.out.println(strs[i]);    // hello world java
	}
	System.out.println();   
	str2 = "hello.world.java";
	String[] strs2 = str2.split("\\.");
	for (int i = 0; i < strs2.length; i++) {
	    System.out.println(strs2[i]);   // hello world java
	}
```

#### 1.5 string类与其他结构之间的转换
a.String 与基本数据类型、包装类之间的转换
- String --> 基本数据类型、包装类：调用`包装类的静态方法`：parseXxx(str)
- 基本数据类型、包装类 --> String:调用String重载的`valueOf(xxx)`

```java
	String str1 = "123";
//  int num = (int)str1;	// 错误的
    int num = Integer.parseInt(str1);

    String str2 = String.valueOf(num);	// "123"
    String str3 = num + "";

    System.out.println(str1 == str3);	// false
```

b.String 与 char[]之间的转换
- String --> char[] : 调用String的`toCharArray()`
- char[] --> String : 调用String的`构造器`

```java
	String str1 = "abc123";	
	char[] charArray = str1.toCharArray();
	for (int i = 0; i < charArray.length; i++) {
	    System.out.println(charArray[i]);
	}
	
	char[] arr = new char[]{'h','e','l','l','o'};
	String str2 = new String(arr);
	System.out.println(str2);
```

c.String 与 byte[]之间的转换
- **编码**：String --> byte[] : 调用String的`getBytes()`
  编码：字符串 -->字节数组  (看得懂 --->看不懂的二进制数据)
- **解码**：byte[] --> String : 调用String的`构造器`
  解码：编码的逆过程，字节数组 --> 字符串 （看不懂的二进制数据 ---> 看得懂）
- 说明：解码时，要求解码使用的字符集必须与编码时使用的字符集一致，否则会出现乱码。

```java
	String str1 = "abc123中国";
	byte[] bytes = str1.getBytes();//使用默认的字符集，进行编码。
	System.out.println(Arrays.toString(bytes));	
	byte[] gbks = str1.getBytes("gbk");//使用gbk字符集进行编码。
	System.out.println(Arrays.toString(gbks));
	
	System.out.println("******************");
	
	String str2 = new String(bytes);//使用默认的字符集，进行解码。
	System.out.println(str2);	
	String str3 = new String(gbks);
	System.out.println(str3);//出现乱码。原因：编码集和解码集不一致！	
	String str4 = new String(gbks, "gbk");
	System.out.println(str4);//没有出现乱码。原因：编码集和解码集一致！
```

### 2.StringBuffer与StringBuilder

#### 2.1 概念

StringBuffer与StringBuilder都是可变的字符序列，都可以对字符串内容进行增删，不会产生新的对象。两者非常相似，很多方法与String相同。

下面是StringBuffer、StringBuilder及AbstractStringBuilder的部分源码：

```java
// StringBuffer 
* @since  JDK1.0
 */
 public final class StringBuffer extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence {
    
    private transient char[] toStringCache;

    static final long serialVersionUID = 3388685877147921107L;

    public StringBuffer() {
        super(16);	// AbstractStringBuilder(int capacity)
    }
    public StringBuffer(int capacity) {
        super(capacity);
    }
    public StringBuffer(String str) {
        super(str.length() + 16);	// AbstractStringBuilder(int capacity)
        append(str);
    }
}

// -------------------------------------------------------------------------
// StringBuilder
 * @since  1.5
 */
public final class StringBuilder extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence {

    static final long serialVersionUID = 4383685877147921099L;

    public StringBuilder() {
        super(16);	// AbstractStringBuilder(int capacity)
    }
    public StringBuilder(int capacity) {
        super(capacity);
    }
    public StringBuilder(String str) {
        super(str.length() + 16);	// AbstractStringBuilder(int capacity)
        append(str);
    }
}

// -------------------------------------------------------------------------
// AbstractStringBuilder
abstract class AbstractStringBuilder implements Appendable, CharSequence {
	// 没有final声明,value可以不断扩容
    char[] value;
	// 记录有效字符的个数
    int count;

    AbstractStringBuilder() {
    }
    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
 
 	// 容量扩展机制
 	public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);// 有效字符个数+附加字符长度
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
    // append(String str)：当str==null时
    private AbstractStringBuilder appendNull() {
        int c = count;
        ensureCapacityInternal(c + 4);
        final char[] value = this.value;
        value[c++] = 'n';
        value[c++] = 'u';
        value[c++] = 'l';
        value[c++] = 'l';
        count = c;
        return this;
    }
    // 容量扩充
 	private void ensureCapacityInternal(int minimumCapacity) {
        if (minimumCapacity - value.length > 0) {
            value = Arrays.copyOf(value,
                    newCapacity(minimumCapacity));// 创建新的容量
        }
    }
    private int newCapacity(int minCapacity) {
        int newCapacity = (value.length << 1) + 2;// (count+len)*2+2
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
            ? hugeCapacity(minCapacity)
            : newCapacity;
    }
}
```

**源码分析：**

- `String str = new String();  //char[] value = new char[0];`
- `String str1 = new String("abc");  //char[] value = new char[]{'a','b','c'};`
- `StringBuffer sb1 = new StringBuffer();  //char[] value = new char[16];创建了一个长度是16的数组。`
- `StringBuffer sb2 = new StringBuffer("abc");  //char[] value = new char["abc".length() + 16];`

**问题：**

- `System.out.println(sb1.length()); //0；System.out.println(sb2.length());//3`
- 扩容问题:如果要添加的数据底层数组盛不下了，那就需要扩容底层的数组。**默认情况下，扩容为原来容量的2倍 + 2，同时将原有数组中的元素复制到新的数组中**。
- 指导意义：开发中建议大家使用：`StringBuffer(int capacity) 或 StringBuilder(int capacity)`

#### 2.2 构造器及常用方法

**构造器：**

| 构造器                   | 描述                         |
| ------------------------ | ---------------------------- |
| StringBuffer()           | 初始容量为16 的字符串缓冲区  |
| StringBuffer(int size)   | 构造指定容量的字符串缓冲区   |
| StringBuffer(String str) | 将内容初始化为指定字符串内容 |

**常用方法：**

| 方法                                                 | 描述                           |
| ---------------------------------------------------- | ------------------------------ |
| StringBuffer append(xxx)                             | 增                             |
| StringBuffer delete(int start,int end)               | 删                             |
| public void setCharAt(int n ,char ch)                | 改                             |
| StringBuffer replace(int start, int end, String str) | 改：把[start,end)位置替换为str |
| public char charAt(int n )                           | 查                             |
| StringBuffer insert(int offset, xxx)                 | 插                             |
| public int length()                                  | 长度                           |
| for() + charAt() / toString()                        | 遍历                           |

注意：

```java
String str = null;
StringBuffer sb = new StringBuffer();
sb.append(str);	// 当添加null时，会返回"null"字符
System.out.println(sb.length());//4
System.out.println(sb);//"null"

StringBuffer sb1 = new StringBuffer(str);// 创造对象时为null，则是空对象
System.out.println(sb1);//java.lang.NullPointerException
```

### 3.String, Stringbuffer, StringBuilder区别
<div style="color:red;">

相同点：

- 都是final类，都不能被继承；
- 都是采用的char[]存储。

异同点：
1. String是不可变的字符序列(char[] 是final修饰的)，每次对String对象改变，都会返回一个新的String对象；StringBuffer、StringBuilder是可变的字符序列(char[]没有final修饰)，不会产生新的对象；
2. StringBuffer是线程安全的，效率低；StringBuilder是线程不安全的，效率高；StringBuilder > StringBuffer > String。
3. StringBuffer和StringBuilder扩容都是value长度的2倍+2。

</div>

## 二.日期时间API
### 1.JDK8之前
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191218113633652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

java.util.Date、java.sql.Date、SimpleDateFormat、Calendar、GregorianCalendar

#### 1.1 System静态方法

**java.lang.System**类提供的`public static long currentTimeMillis()`用来返回**当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差**

```java
long time = System.currentTimeMillis();
//返回当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差。
//称为时间戳
System.out.println(time);
```

#### 1.2 Date类

java.util.Date类表示特定的瞬间，精确到毫秒。子类java.sql.Date类。

**构造器：**

| 构造器          | 描述                           |
| --------------- | ------------------------------ |
| Date()          | 创建一个对应当前时间的Date对象 |
| Date(long date) | 创建指定毫秒数的Date对象       |

**常用方法：**

| 方法       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| toString() | 把此 Date 对象转换为以下形式的 String： dow mon dd hhmmss zzz yyyy |
| getTime()  | 获取当前Date对象对应的毫秒数。（时间戳）                     |

示例：

```java
//构造器一：Date()：创建一个对应当前时间的Date对象
Date date1 = new Date();
System.out.println(date1.toString());//Sat Feb 16 16:35:31 GMT+08:00 2019
System.out.println(date1.getTime());//1550306204104

//构造器二：创建指定毫秒数的Date对象
Date date2 = new Date(155030620410L);
System.out.println(date2.toString());
```

**java.sql.Date类：**

实例化：`new java.sql.Date(long date)`

util.Date与sql.Date相互转换：

sql.Date --> util.Date：子类对象转父类对象自动转型(多态)

util.Date --> sql.Date：`new java.sql.Date( (new Date()).getTime() )`

示例：

```java
//创建java.sql.Date对象
java.sql.Date date3 = new java.sql.Date(35235325345L);
System.out.println(date3);//1971-02-13

//如何将java.util.Date对象转换为java.sql.Date对象
//情况一：
//  Date date4 = new java.sql.Date(2343243242323L);
//  java.sql.Date date5 = (java.sql.Date) date4;
//情况二：
Date date6 = new Date();
java.sql.Date date7 = new java.sql.Date(date6.getTime());
```

#### 1.3 SimpleDataFormat类

SimpleDateFormat类对日期Date类的格式化和解析：

- 格式化：日期 --> 字符串
- 解析：字符串 --> 日期

相关构造器及方法：

| 方法                                    | 描述                                              |
| --------------------------------------- | ------------------------------------------------- |
| public SimpleDateFormat(String pattern) | 该构造方法可以用参数pattern指定的格式创建一个对象 |
| public String format(Date date)         | 格式化时间对象date                                |
| public Date parse(String source)        | 从给定字符串的开始解析文本，以生成一个日期        |

示例：

```java
// SimpleDateFormat sdf1 = new SimpleDateFormat("yyyyy.MMMMM.dd GGG hh:mm aaa");
SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

//格式化
String format1 = sdf1.format(date);
System.out.println(format1);//2019-02-18 11:48:27

//解析:要求字符串必须是符合SimpleDateFormat识别的格式(通过构造器参数体现),否则，抛异常
Date date2 = sdf1.parse("2020-02-18 11:48:27");
System.out.println(date2);
```


#### 1.4 Calendar类

 Calendar是一个抽象基类，主用用于完成日期字段之间相互操作的功能。

构造器及常用方法：

**int field**：静态属性(YEAR、MONTH、DAY_OF_WEEK、HOUR_OF_DAY 、MINUTE、SECOND)等

| 方法                                      | 描述                   |
| ----------------------------------------- | ---------------------- |
| **Calendar.getInstance()**                | 获取Calendar实例       |
| public int **get(int field)**             | 获取指定部位的时间信息 |
| public void **set(int field,int value)**  | 设置指定部位的时间信息 |
| public void **add(int field,int amount)** | 修改指定部位的时间信息 |
| public final Date **getTime()**           | 日历类---> Date        |
| public final void **setTime(Date date)**  | Date ---> 日历类       |

**注意：**

- **获取月份时**：一月是0，二月是1，以此类推，12月是11
- **获取星期时**：周日是1，周二是2 ， 。。。。周六是7

**示例：**

```java
//1.实例化
//方式一：创建其子类（GregorianCalendar）的对象
//方式二：调用其静态方法getInstance()
Calendar calendar = Calendar.getInstance();
//  System.out.println(calendar.getClass());

//2.常用方法
//get()
int days = calendar.get(Calendar.DAY_OF_MONTH);
System.out.println(days);
System.out.println(calendar.get(Calendar.DAY_OF_YEAR));

//set()
//calendar可变性
calendar.set(Calendar.DAY_OF_MONTH,22);
days = calendar.get(Calendar.DAY_OF_MONTH);
System.out.println(days);

//add()
calendar.add(Calendar.DAY_OF_MONTH,-3);
days = calendar.get(Calendar.DAY_OF_MONTH);
System.out.println(days);

//getTime():日历类---> Date
Date date = calendar.getTime();
System.out.println(date);

//setTime():Date ---> 日历类
Date date1 = new Date();
calendar.setTime(date1);
days = calendar.get(Calendar.DAY_OF_MONTH);
System.out.println(days);
```

### 2.JDK8

Calendar面临的问题：

**可变性**：像日期和时间这样的类应该是不可变的。
**偏移性**：Date中的年份是从1900开始的，而月份都从0开始。
**格式化**：格式化只对Date有用，Calendar则不行。
此外，它们也不是线程安全的；不能处理闰秒等

Java 8 吸收了 Joda-Time 的精华，以一个新的开始为 Java 创建优秀的 API。新的 java.time 中包含了所有关于本地日期（LocalDate）、本地时间（LocalTime）、本地日期时间（LocalDateTime）、时区（ZonedDateTime）和持续时间（Duration）的类。

**API：**

| 包                 | 描述                       |
| ------------------ | -------------------------- |
| java.time          | 包含值对象的基础包         |
| java.time.chrono   | 提供对不同的日历系统的访问 |
| java.time.format   | 格式化和解析时间和日期     |
| java.time.temporal | 包括底层框架和扩展特性     |
| java.time.zone     | 包含时区支持的类           |

#### 2.1 LocalDate、LocalTime、LocalDateTime

这三个类的实例都是**不可变的对象**。LocalDate代表IOS格式(yyyy-MM-dd)的日期；LocalTime表示一个时间；LocalDateTime是用来表示日期和时间的。

LocalDateTime使用频率偏高，**用法类似于Calendar**。

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| now()                                                        | 静态方法，根据当前时间创建对象                               |
| of()                                                         | 静态方法，根据指定日期/时间创建对象                          |
| getDayOfMonth() / getDayOfYear()                             | 获得月份天数(1-31) /获得年份天数(1-366)                      |
| getDayOfWeek()                                               | 获得星期几(返回一个 DayOfWeek 枚举值)                        |
| getMonth()                                                   | 获得月份, 返回一个 Month 枚举值                              |
| getMonthValue() / getYear()                                  | 获得月份(1-12) /获得年份                                     |
| getHour() / getMinute() / getSecond()                        | 获得当前对象对应的小时、分钟、秒                             |
| withDayOfMonth() / withDayOfYear() / withMonth() / withYear() | 将月份天数、年份天数、月份、年份修改为指定的值并返回新的对象 |
| plusDays() / plusWeeks() / plusMonths() / plusYears() / plusHours() | 向当前对象添加几天、几周、几个月、几年、几小时               |
| minusMonths() / minusWeeks() / minusDays() / minusYears() / minusHours() | 从当前对象减去几月、几周、几天、几年、几小时                 |

示例：

```java
//now():获取当前的日期、时间、日期+时间
LocalDate localDate = LocalDate.now();
LocalTime localTime = LocalTime.now();
LocalDateTime localDateTime = LocalDateTime.now();

System.out.println(localDate);  // 2019-12-18
System.out.println(localTime);  // 10:14:29.600
System.out.println(localDateTime);  // 2019-12-18T10:14:29.600

//of():设置指定的年、月、日、时、分、秒。没有偏移量
LocalDateTime localDateTime1 = LocalDateTime.of(2020, 10, 6, 13, 23, 43);
System.out.println(localDateTime1); // 2020-10-06T13:23:43


//getXxx()：获取相关的属性
System.out.println(localDateTime.getDayOfMonth());  // 18
System.out.println(localDateTime.getDayOfWeek());   // WEDNESDAY
System.out.println(localDateTime.getMonth());   // DECEMBER
System.out.println(localDateTime.getMonthValue());  // 12
System.out.println(localDateTime.getMinute());  // 14

//体现不可变性
//withXxx():设置相关的属性
LocalDate localDate1 = localDate.withDayOfMonth(22);
System.out.println(localDate);  // 2019-12-18
System.out.println(localDate1); // 2019-12-22

LocalDateTime localDateTime2 = localDateTime.withHour(4);
System.out.println(localDateTime);  // 2019-12-18T10:14:29.600
System.out.println(localDateTime2); // 2019-12-18T04:14:29.600

LocalDateTime localDateTime3 = localDateTime.plusMonths(3);
System.out.println(localDateTime3); // 2020-03-18T10:14:29.600

LocalDateTime localDateTime4 = localDateTime.minusDays(6);
System.out.println(localDateTime4); // 2019-12-12T10:14:29.600
```

#### 2.2 Instant

Instant：**时间线上的一个瞬时点，不需要任何上下文信息**。这可以用来记录应用程序中的事件时间戳。**用法类似于 java.util.Date 类**。

年月日时分秒是面向人类的一个时间模型，时间线中一点是面向机器的另一个时间模型。在Java中，也是从1970年开始，但以毫秒为单位。

1秒 = 1000毫秒 =10^6 微妙=10^9纳秒

| 方法                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| now()                         | 静态方法，返回默认UTC时区的Instant类的对象                   |
| ofEpochMilli(long epochMilli) | 静态方法，返回在1970-01-01 00 00 00基础上加上指定毫秒数之后的Instant类的对象 |
| atOffset(ZoneOffset offset)   | 结合即时的偏移来创建一个 OffsetDateTime                      |
| toEpochMilli()                | 返回1970-01-01 00 00 00到当前时间的毫秒数，即为时间戳        |

**时间戳是指格林威治时间1970 年01 月01 日00 时00 分00 秒( 北京时间1970 年01 月01**
**日08 时00 分00 秒) 起至现在的总秒数(东八区)**。

示例：

```java
//now():获取本初子午线对应的标准时间
Instant instant = Instant.now();
System.out.println(instant);//2019-12-18T02:42:50.214Z

//添加时间的偏移量
OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
System.out.println(offsetDateTime);//2019-12-18T10:42:50.214+08:00

//toEpochMilli():获取自1970年1月1日0时0分0秒（UTC）开始的毫秒数  ---> Date类的getTime()
long milli = instant.toEpochMilli();
System.out.println(milli);  // 1576636970214

//ofEpochMilli():通过给定的毫秒数，获取Instant实例  -->Date(long millis)
Instant instant1 = Instant.ofEpochMilli(1576636970214L);
System.out.println(instant1);   // 2019-12-18T02:42:50.214Z
```

#### 2.3 DateTimeFormatter

java.time.format.DateTimeFormatter类：格式化与解析日期或时间类。**用法类似于SimpleDateFormat类**。DateTimeFormatter提供了三种格式化方法：

- 预定义的标准格式。静态常量：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE;ISO_LOCAL_TIME

  `DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;`

- 本地化相关的格式。静态方法：ofLocalizedDateTime(FormatStyle.LONG)、ofLocalizedDate(FormatStyle.MEDIUM)。

  ofLocalizedDateTime()：

  - FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT :适用于LocalDateTime

    `DateTimeFormatter formatter1 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);`

  ofLocalizedDate()：

  - FormatStyle.FULL / FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT : 适用于LocalDate

    `DateTimeFormatter formatter2 = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM);`

- **自定义的格式(常用)**。如：ofPattern(“yyyy-MM-dd hh mm ss”)

| 方法                       | 描述                                                |
| -------------------------- | --------------------------------------------------- |
| ofPattern(String pattern)  | 静态方法，返回一个指定字符串格式的DateTimeFormatter |
| format(TemporalAccessor t) | 格式化一个日期、时间，返回字符串                    |
| parse(CharSequence text)   | 将指定格式的字符序列解析为一个日期、时间            |

示例：

```java
//重点：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)
DateTimeFormatter formatter3 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
//格式化
String str4 = formatter3.format(LocalDateTime.now());
System.out.println(str4);// 2019-12-18 11:08:46 没有偏移量

//解析
TemporalAccessor accessor = formatter3.parse("2019-12-18 11:08:46");
System.out.println(accessor);
// {SecondOfMinute=46, NanoOfSecond=0, MicroOfSecond=0, MinuteOfHour=8, MilliOfSecond=0,
//  HourOfAmPm=11},ISO resolved to 2019-12-18
```

#### 2.4 其它API

- ZoneId ：该类中包含了所有的时区信息，一个时区的ID，如 Europe/Paris

- ZonedDateTime ：一个在ISO-8601日历系统时区的日期时间，如 2007-12-03T10 15 30+01 00 Europe/Paris。

  - 其中每个时区都对应着ID，地区ID都为“{区域}/{城市}”的格式，例如：Asia/Shanghai等

- Clock ：使用时区提供对当前即时、日期和时间的访问的时钟。

- 持续时间：Duration，用于计算两个“时间”间隔

- 日期间隔：Period，用于计算两个“日期”间隔

- TemporalAdjuster : 时间校正器。有时我们可能需要获取例如：将日期调整到“下一个工作日”等操作。

- TemporalAdjusters : 该类通过静态方法

  (firstDayOfXxx()/lastDayOfXxx()/nextXxx())提供了大量的常用TemporalAdjuster 的实现


## 三.java比较器

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

## 四.System类

System类代表系统，系统级的很多属性和控制方法都放置在该类的内部。该类位于java.lang包。

由于该类的构造器是private的，所以无法创建该类的对象，也就是无法实例化该类。其**内部的成员变量和成员方法都是static的**。

成员变量：

| 成员变量 | 描述                   |
| -------- | ---------------------- |
| in       | 标准输入流(键盘输入)   |
| out      | 标准输出流(显示器)     |
| err      | 标准错误输出流(显示器) |

成员方法：

| 成员方法                        | 描述                                              |
| ------------------------------- | ------------------------------------------------- |
| native long currentTimeMillis() | 返回当前的计算机时间(1970-01-01 00-00-00)         |
| void exit(int status)           | 退出程序。其中status的值为0代表正常退出，非零代表 |
| void gc()                       | 请求系统进行垃圾回收(不一定是立刻回收)            |
| String getProperty(String key)  | 获得系统中属性名为key的属性对应的值               |

系统中常见的属性名以及属性的作用：

| 属性名       | 属性说明           |
| ------------ | ------------------ |
| java.version | java运行时环境版本 |
| java.home    | java安装目录       |
| os.name      | 操作系统的名称     |
| os.version   | 操作系统的版本     |
| user.name    | 用户的账户名称     |
| user.home    | 用户的主目录       |
| user.dir     | 用户的当前工作目录 |

## 五.Math类

java.lang.Math 提供了一系列静态方法用于科学计算。其方法的参数和返回值类型一般为double 型。

| 方法                                              | 描述                                 |
| ------------------------------------------------- | ------------------------------------ |
| abs                                               | 绝对值                               |
| sqrt                                              | 平方根                               |
| pow(double a, double b)                           | a的b次幂                             |
| log                                               | 自然对象                             |
| exp                                               | e为底指数                            |
| max(double a, double b) / min(double a, double b) | a和b的最大值/最小值                  |
| random()                                          | 返回[0.0, 1.0)的随机数               |
| long round(double a)                              | double 型数据a转换为long型(四舍五入) |
| toDegrees(double angrad)                          | 弧度—> 角度                          |
| toRadians(double angdeg)                          | 角度—> 弧度                          |

## 六.BigInteger与BigDecimal

### 1.BigInteger类

 java.math包的**BigInteger 可以表示不可变的任意精度的整数**。BigInteger 提供所有 Java 的基本整数操作符的对应物，并提供 java.lang.Math 的所有相关方法。

构造方法：

- `BigInteger(String val)`：根据字符串构建BigInteger对象

常用方法：

| 方法                                            | 描述                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| BigInteger add(BigInteger val)                  | 返回其值为 (this + val) 的 BigInteger                        |
| BigInteger subtract(BigInteger val)             | 返回其值为 (this - val) 的 BigInteger                        |
| BigInteger multiply(BigInteger val)             | 返回其值为 (this * val) 的 BigInteger                        |
| BigInteger divide(BigInteger val)               | 返回其值为 (this / val) 的 BigInteger。整数相除只保留整数部分 |
| BigInteger remainder(BigInteger val)            | 返回其值为 (this % val) 的 BigInteger                        |
| BigInteger[] divideAndRemainder(BigInteger val) | 返回包含 (this / val) 后跟(this % val) 的两个 BigInteger 的数组 |

### 2.BigDecimal类

 一般的Float类和Double类可以用来做科学计算或工程计算，但在商业计算中，要求数字精度比较高，故用到java.math.BigDecimal 类 。BigDecimal对象是**不可变**的。

**BigDecimal类支持不可变的、任意精度的有符号十进制定点数。**

**常用构造方法：**

- `public BigDecimal(double val)`：根据double构建BigDecimal对象(有精确度缺失)
- `public BigDecimal(String val)`：根据字符串构建BigDecimal对象(**建议采用**)

**常用方法：**

| 方法                                                         | 描述             |
| ------------------------------------------------------------ | ---------------- |
| public BigDecimal add(BigDecimal augend)                     | 加               |
| public BigDecimal subtract(BigDecimal subtrahend)            | 减               |
| public BigDecimal multiply(BigDecimal multiplicand)          | 乘               |
| public BigDecimal divide(BigDecimal divisor, int scale, int roundingMode) | 除               |
| public String toString()                                     | 数值转换成字符串 |
| public double doubleValue()                                  | 以双精度数返回   |
| public BigDecimal setScale(int newScale, int roundingMode)   | 格式化小数点     |

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










