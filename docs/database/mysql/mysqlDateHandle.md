在工作业务中常会使用mysql的日期函数,下面做一下关于这方面的sql整理:

#### 1.内置日期函数

-   now(): 返回当前的日期和时间
-   unix_timestamp(): 返回自`1970-01-01 00:00:00`到当前时间的秒数差(时间戳)
-   unix_timestamp(指定时间): 返回自`1970-01-01 00:00:00`到指定时间的秒数差(时间戳)
-   from_unixtim(时间字段): 将unix(时间戳)转为datetime日期
-   date_format(时间字段, format): 以不同的格式显示日期或时间数据

补充: 字符串截取函数

-   left('example.com', 3): `exa`, 从最左边开始截取
-   right('example.com', 3): `com`, 从最右边开始截取
-   substring('example.com', 4): `mple.com`,从第4个字符位置开始取,索引为1开始
-   substring('example.com', 4, 2): `mp`,从第四个字符位置开始取2个字符
-   substring('example.com', -4): `.com`,从右边开始数第4个字符位置开始取
-   substring('example.com', -4, 2): `.c`,从右边开始数第4个字符位置开始取2个
-   substring_index('www.example.com', '.', 2): `www.example`, 截取第二个 '.' 之前的所有字符
-   substring_index('www.example.com', '.', -2): `example.com` , 截取倒数第二个 '.' 之后的所有字符
-   substring_index('www.example.com', '.coc', 1): `www.example.com`, 没有查找到指定的值就返回整个字符串

#### 2.查询每日,每周,每月,每季度,每年相关的sql语句

查询2019年每天有多少条记录:

```sql
select date_format(时间字段,'%Y-%m-%d') days,count(*) from 表名 where date_format(时间字段,'%Y')='2019' group by days
--或
select count(*),DATE(时间字段名) from 表名 where YEAR(时间字段名)='2019' group by DAY(时间字段名)
```

查询8月份每周有多少条记录:

```sql
select date_format(时间字段,'%Y%u') weeks,count(*) from 表名 where date_format(时间字段,'%m')='08' group by weeks
--或
select count(*),WEEK(时间字段名) from 表名 where MONTH(时间字段名)='8' group by WEEK(时间字段名) 
```

查询2019年周一到周五每天总共有多少条记录:

```sql
select count(*),DAYNAME(时间字段名) from 表名 where YEAR(时间字段名)='2019' group by DAYNAME(时间字段名)
```

查询本周有多少条记录:

```sql
select count(*) from 表名 where MONTH(时间字段名) = MONTH(CURDATE()) and WEEK(时间字段名) = WEEK(CURDATE())
```

查询2019年每月有多少条记录:

```sql
select date_format(时间字段,'%Y%m') months,count(*) from 表名 where date_format(时间字段,'%Y')='2019' group by months
--或
select count(*),MONTH(时间字段名) from 表名 where YEAR(时间字段名)='2019' group by MONTH(时间字段名)
```

查询本月有多少条记录:

```sql
select count(*) from 表名 where MONTH(时间字段名)=MONTH(CURDATE()) and YEAR(时间字段名) = YEAR(CURDATE())
```

查询2019年每季度有多少条记录:

```sql
select count(*), concat(year(时间字段),floor((date_format(时间字段, '%m')+2)/3)) quarters from 表名 group by quarters
--或
select count(*),QUARTER(时间字段名) from 表名 where YEAR(时间字段名)='2019' group by QUARTER(时间字段名)
```

查询每年有多少条记录:

```sql
select date_format(时间字段, '%Y') years,count(*) from 表名 group by years
--或
select count(*),YEAR(时间字段名) from 表名 group by YEAR(时间字段名)
```

#### 3.时间段查询

查询当天的所有记录:

```sql
select * from 表名 where date(时间字段名) = date(now())
--或
select * from 表名 where to_days(时间字段) = to_days(now())
```

查询n天内所有记录:

```sql
select * from 表名 where to_days(now()) - to_days(时间字段名) <= n
```

查询一周内的所有记录:

```sql
select * from 表名 where date_sub(curdate(), interval 7 day) <= date(时间字段名)
```

查询一个月内的所有记录:

```sql
select * from 表名 where date_sub(curdate(), interval 1 month) <= date(时间字段名)
```

查询'08-06'到'12-12'时间端内所有记录:

```sql
select * from 表名 where date_format(时间字段名,'%m-%d') >= '08-06' and date_format(时间字段名,'%m-%d') <= '12-12'
```

#### 4.date_format()中format可以使用的格式

-   格式	描述
-   %a	缩写星期名
-   %b	缩写月名
-   %c	月，数值
-   %D	带有英文前缀的月中的天
-   %d	月的天，数值（00-31）
-   %e	月的天，数值（0-31）
-   %f	微秒
-   %H	小时（00-23）
-   %h	小时（01-12）
-   %I	小时（01-12）
-   %i	分钟，数值（00-59）
-   %j	年的天（001-366）
-   %k	小时（0-23）
-   %l	小时（1-12）
-   %M	月名
-   %m	月，数值（00-12）
-   %p	AM 或 PM
-   %r	时间，12-小时（hh:mm:ss AM 或 PM）
-   %S	秒（00-59）
-   %s	秒（00-59）
-   %T	时间, 24-小时（hh:mm:ss）
-   %U	周（00-53）星期日是一周的第一天
-   %u	周（00-53）星期一是一周的第一天
-   %V	周（01-53）星期日是一周的第一天，与 %X 使用
-   %v	周（01-53）星期一是一周的第一天，与 %x 使用
-   %W	星期名
-   %w	周的天（0=星期日, 6=星期六）
-   %X	年，其中的星期日是周的第一天，4 位，与 %V 使用
-   %x	年，其中的星期一是周的第一天，4 位，与 %v 使用
-   %Y	年，4 位
-   %y	年，2 位



参考链接:

-   [MySQL中按周,月,季,年分组统计](https://blog.csdn.net/xie8409959/article/details/82663899)
-   [mysql按日,周,月,年统计sql整理,实现报表统计可视化](https://blog.csdn.net/u010543785/article/details/52354957)
-   [MySQL按天,周,月,时间端统计](https://blog.csdn.net/qq_28056641/article/details/78306870)

