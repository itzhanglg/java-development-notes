### 1.JDK8之前日期API
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

| 构造器             | 描述                |
| --------------- | ----------------- |
| Date()          | 创建一个对应当前时间的Date对象 |
| Date(long date) | 创建指定毫秒数的Date对象    |

**常用方法：**

| 方法         | 描述                                       |
| ---------- | ---------------------------------------- |
| toString() | 把此 Date 对象转换为以下形式的 String： dow mon dd hhmmss zzz yyyy |
| getTime()  | 获取当前Date对象对应的毫秒数。（时间戳）                   |

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

| 方法                                      | 描述                           |
| --------------------------------------- | ---------------------------- |
| public SimpleDateFormat(String pattern) | 该构造方法可以用参数pattern指定的格式创建一个对象 |
| public String format(Date date)         | 格式化时间对象date                  |
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

| 方法                                       | 描述            |
| ---------------------------------------- | ------------- |
| **Calendar.getInstance()**               | 获取Calendar实例  |
| public int **get(int field)**            | 获取指定部位的时间信息   |
| public void **set(int field,int value)** | 设置指定部位的时间信息   |
| public void **add(int field,int amount)** | 修改指定部位的时间信息   |
| public final Date **getTime()**          | 日历类---> Date  |
| public final void **setTime(Date date)** | Date ---> 日历类 |

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

### 2.JDK8日期API

Calendar面临的问题：

**可变性**：像日期和时间这样的类应该是不可变的。
**偏移性**：Date中的年份是从1900开始的，而月份都从0开始。
**格式化**：格式化只对Date有用，Calendar则不行。
此外，它们也不是线程安全的；不能处理闰秒等

Java 8 吸收了 Joda-Time 的精华，以一个新的开始为 Java 创建优秀的 API。新的 java.time 中包含了所有关于本地日期（LocalDate）、本地时间（LocalTime）、本地日期时间（LocalDateTime）、时区（ZonedDateTime）和持续时间（Duration）的类。

**API：**

| 包                  | 描述            |
| ------------------ | ------------- |
| java.time          | 包含值对象的基础包     |
| java.time.chrono   | 提供对不同的日历系统的访问 |
| java.time.format   | 格式化和解析时间和日期   |
| java.time.temporal | 包括底层框架和扩展特性   |
| java.time.zone     | 包含时区支持的类      |

#### 2.1 LocalDate、LocalTime、LocalDateTime

这三个类的实例都是**不可变的对象**。LocalDate代表IOS格式(yyyy-MM-dd)的日期；LocalTime表示一个时间；LocalDateTime是用来表示日期和时间的。

LocalDateTime使用频率偏高，**用法类似于Calendar**。

| 方法                                       | 描述                             |
| ---------------------------------------- | ------------------------------ |
| now()                                    | 静态方法，根据当前时间创建对象                |
| of()                                     | 静态方法，根据指定日期/时间创建对象             |
| getDayOfMonth() / getDayOfYear()         | 获得月份天数(1-31) /获得年份天数(1-366)    |
| getDayOfWeek()                           | 获得星期几(返回一个 DayOfWeek 枚举值)      |
| getMonth()                               | 获得月份, 返回一个 Month 枚举值           |
| getMonthValue() / getYear()              | 获得月份(1-12) /获得年份               |
| getHour() / getMinute() / getSecond()    | 获得当前对象对应的小时、分钟、秒               |
| withDayOfMonth() / withDayOfYear() / withMonth() / withYear() | 将月份天数、年份天数、月份、年份修改为指定的值并返回新的对象 |
| plusDays() / plusWeeks() / plusMonths() / plusYears() / plusHours() | 向当前对象添加几天、几周、几个月、几年、几小时        |
| minusMonths() / minusWeeks() / minusDays() / minusYears() / minusHours() | 从当前对象减去几月、几周、几天、几年、几小时         |

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

| 方法                            | 描述                                       |
| ----------------------------- | ---------------------------------------- |
| now()                         | 静态方法，返回默认UTC时区的Instant类的对象               |
| ofEpochMilli(long epochMilli) | 静态方法，返回在1970-01-01 00 00 00基础上加上指定毫秒数之后的Instant类的对象 |
| atOffset(ZoneOffset offset)   | 结合即时的偏移来创建一个 OffsetDateTime              |
| toEpochMilli()                | 返回1970-01-01 00 00 00到当前时间的毫秒数，即为时间戳     |

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

| 方法                         | 描述                                 |
| -------------------------- | ---------------------------------- |
| ofPattern(String pattern)  | 静态方法，返回一个指定字符串格式的DateTimeFormatter |
| format(TemporalAccessor t) | 格式化一个日期、时间，返回字符串                   |
| parse(CharSequence text)   | 将指定格式的字符序列解析为一个日期、时间               |

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








