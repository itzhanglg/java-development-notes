### 一.字符串相关的类
#### 1.String及常用方法
##### 1.1 String的特性
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
##### 1.2 String对象的创建
String的实例化方式：
- 通过字面量定义的方式
- 通过new + 构造器的方式

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

##### 1.3 String使用陷阱
关于拼接字符串后两个变量地址是否相等。
- 常量与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量。
- 只要其中有一个是变量，结果就在堆中
- 如果拼接的结果调用intern()方法，返回值就在常量池中

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

##### 1.4 String常用方法
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

##### 1.5 string类与其他结构之间的转换
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

#### 2.StringBuffer



#### 3.StringBuilder



### 二.日期时间API
#### 1.JDK8之前
##### 1.1 System静态方法


##### 1.2 Date类


##### 1.3 Calendar类


##### 1.4 SimpleDataFormat类



#### 2.JDK8
##### 2.1 LocalDate、LocalTime、LocalDateTime


##### 2.2 Instant


##### 2.3 DateTimeFormatter


##### 2.4 其它类



### 三.java比较器
#### 1.Comparable接口



#### 2.Comparator接口



### 四.System类



### 五.Math类



### 六.BigInteger与BigDecimal





