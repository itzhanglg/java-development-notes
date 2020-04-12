参考链接:

-   [SQL教程](https://www.w3school.com.cn/sql/index.asp)
-   [MySQL教程](https://www.runoob.com/mysql/mysql-tutorial.html)

### 一.创建数据库语法

```sql
--用master数据库
use master
--判断数据库是否存在，若存在则删除
if exists (select * from sysdatabases where name='数据库名称')
drop database 数据库名称
--创建数据库
create database 数据库名称
on(					--指定主文件的属性
	name='',		--文件的逻辑名称(数据之间的相互关系)
	filename='',	--文件的物理名称(数据存储的路径)
	size='',		--文件的初始大小
	maxsize='',		--文件的最大值
	filegrowth=''	--文件的增长方式
)
log on(				--指定日志文件的属性
	name='',		--文件的逻辑名称
	filename='',	--文件的物理名称
	size='',		--文件的初始大小
	filegrowth=''	--文件的增长方式
)
--查看数据库
exec sp_helpdb 数据库名称;
--修改数据库名称
alter database 数据库名称
modify name='';
--修改数据库文件
alter database 数据库名称
modify file(		
	name='',
	filename='',
	size='',
	filegrowth=''
)
```

### 二.创建数据表

```sql
--指定用什么数据库
use 数据库名称
--判断数据表是否存在，若存在则删除
if exists (select * from sysobjects where name='数据表名称')
drop table 数据表名称
--创建数据表
create table 数据表名称(
  	--一般不直接添加约束
	字段名称 字段类型 字段特征(默认值DF_、标识列、主键PK_、关系FK_、索引UQ_、检查CK_、非空)		
)

--为字段添加约束
alter table 数据表名称
add constraint 约束名称 约束(指定字段) [references 主表(主键或唯一键)];
--删除约束
alter table 数据表名称
drop constraint 约束名称;

--增加外键约束时，设置级联删除、级联更新  on delete cascade;
[ ON DELETE { NO ACTION | CASCADE | SET NULL | SET DEFAULT } ]  删除
[ ON UPDATE { NO ACTION | CASCADE | SET NULL | SET DEFAULT } ]  更新
--on--在做何种操作的时候做相应的处理
--NO ACTION--不做任何操作：该报错就报错，可以删除就删除
--CASCADE：级联：删除主表，对应的从表数据也会删除，更新也一样
--SET NULL：如果删除主表的记录，那么对应的从表记录的字段值会被设置为null,前提是从表的这个字段的值可以是null
--SET DEFAULT :删除主表记录，从表的记录的对应字段值设置为默认值，前提是你之前为这个字段设置了默认值

--修改字段的类型及约束
alter table 数据表名称
alter column 字段名称 字段类型 约束;
--添加字段
alter table 数据表名称
add 字段名称 字段类型 [not null];	
--删除某个字段
alter table 数据表名称
drop column 字段名称;
--修改字段名称
exec sp_rename '数据表名称.原字段名称','新字段名称','column';
```

### 三.添加数据

```sql
--一次添加一行数据,实参与形参 参数类型 参数个数 参数顺序 都必须一致
insert into 表名(形参)		
values(实参)

--1.默认约束可以不指定列名
--2.自增不能显示的添加
--3.允许为null值的字段可直接设置null值，也可不指定列名

--一次添加多行数据时添加顺序不是按照书写的顺序添加的，而是按照第一个字段的首a-z字母或者数字从小到大添加的
--一次添加多行数据
insert into 表名(形参)
select 实参 union
select 实参;
--或者
insert into 表名(形参)
values(实参),(实参);

--从别的表中copy数据
insert into 表1(形参)	
select 实参 from 表2;

--将从数据源选择的数据插入到新表中并显示，新表是系统创建的，之前必须不存在
select * into 新表 from 数据源;
--将从数据源选择后得到的结果集添加到目的表中， 目的表必须存在
insert into 目的表 select * from 数据源;
```

### 四.删除数据

```sql
--标识列不会从标识种子开始，可以回滚(恢复数据)
delete from 表名;
--条件与查询语句中条件一样：关系、逻辑运算符、like、in、between...
delete from 表名 where 条件;
--标识列会从标识种子开始
truncate table 表名;
```

### 五.修改数据

```sql
--条件与查询语句中条件一样：关系运算符、逻辑运算符、like、in、between...
update 表名 set 字段=值,... where 条件
```

#### 1.运算符和内置函数

关系运算符: `= < > >= <= != <>` 
逻辑运算符: `not / and / or / is null / is not null` 

内置函数:

-   字符串函数:
    -   `CharIndex('ab','cdab')` 	返回"3"		--返回"ab"在"cdab"中的位置,将返回第一个字母的位置
      -   `Len('我是谁')` 返回"3"		--返回字符串的长度
        -   `substring('abd',2,2)` 返回"bd"--从第二个位置开始截取长度为2的字符串
-   日期函数:
    -   `GetDate()` 	返回当前日期
      -   `DateName(DW,'2009-09-09')` 返回"星期三"	以字符串形式返回某个日期指定的部分
      -   `DateDiff(dd,'2009-09-09','2010-10-09')` 返回两个日期之间的间隔,yy表示年,mm表示月,dd表示日
        -   `DatePart(DW,'2009-09-09')` 返回"4"	以整数形式返回某个日期指定的部分(美国:星期天为第一天,12月为第一个月)
-   数学函数:
    -   `Abs(-1)` 		求绝对值
      -   `Ceiling(24.1)` 大于24.1的最小整数
        -   `Floor(24.1)` 小于24.1的最大整数
    -   `round(18.6)`          四舍五入一个正数或者负数，结果为一定长度的值
-   系统函数
    -   `Convert(varchar(3),123)`   返回"123"	转换数据类型
      -   `DataLength("12中国")` 返回"6"		返回任何数据类型的字节数,汉字为2个字节

### 六.查询数据

#### 1.基本语法

```sql
select top|distinct 值/[percent] */表名.字段 别名,... ['上海' as 城市],...   from 表名 别名   where 条件   group by 字段    having 条件   order by 字段别名
```

1.  `top 值/[percent]`: 一般与order by连用 ; 用百分比时结果集是取小数的ceiling; 限制行是最后一步做的 查询、排序、限制行
2.  `distinct 字段`: 过滤掉重复记录
3.  `表名.字段`: 这种方式内部查询更快 ;  若group by了则select 字段：满足记录中值是**一对一关系** 
4.  `字段 as 别名`; `字段 别名`; `别名=字段`;  --三种方式都可以
5.  可以添加**常量列**: `'上海' as 城市`
6.  where条件：
    -   关系(> < >= <= != <>)
    -   逻辑运算符(and/ or/ not)
    -   `字段 [not] in(具体值1、具体值2)/(子查询)`: 使用in代表一个具体的值范围,in要求指定的范围的数据类型一致; 可以使用子查询
    -   `字段 [not] between 数值1/日期1 and 数值2/日期2`: 如果是数值或日期的范围判断可以使用between...and
    -   `字段 [not] like '张%'、'张_'、'1[1-8]'、'[^21-35]'  --'1[1-8]'表示11-18`  : `[^21-35]` 表示除了1,2,3,5之外的  (通配符只有在模糊查询里才有效)
        -   **%**: 代表任意个任意字符
        -   **_**: 代表任意的单个字符
        -   **[]**: 代表一个具体的值范围中的任意一个字符  `[0-9][a-z]` 只匹配一个字符
        -   **[^]**: 代表不在指定的范围之内,放在[]里面才有这个意义  --取反值 只有MSSQL Server支持，其他MSDB用not like
7.  `group by` ：对指定列的数据进行分组，将该列具有相同值的行划为一组
8.  `having条件` ：**字段要么被分组、要么被聚合**, 条件：与where条件一样
    -   where子句与having子句区别：
        -   where子句与单个的行有关，having子句与组有关
        -   where子句不能直接以聚合函数作为搜索条件，而having可以以聚合函数作为搜索条件，但having不能使用字段别名
        -   where子句与having子句执行顺序不同
9.  `order by 字段 [desc]...`: 默认是asc,可以给多个字段排序

#### 2.聚合函数

聚合函数是对筛选后结果集的某些字段的一组数据进行统计分析.

```sql
--聚合函数:参数一般是字段
select max(字段),min(字段)...  /  select count(*),max(字段) from 表名 where 条件
```

-   `max(数值、字符串、日期)` :求指定数据范围中的最大值:可以对任意类型进行聚合，如果是非数值么就按值的拼音进行排序
-   `min(数值、字符串、日期)` :求指定数据范围中的最小值:可以对任意类型进行聚合，如果是非数值么就按值的拼音进行排序
-   `avg(数值)` :求指定数据范围中的平均值,它只能对数值进行聚合,不能对日期进行聚合
-   `sum(数值)` :求指定数据范围中的和,它只能对数值进行聚合,不能对日期进行聚合
-   `count(*/一个字段)` :求满足条件的记录数,与字段没有任何关系 ; 计算记录数或指定列的非空值数目
    -   空格字符默认是最小的
    -   字符串：字符 a-z ; 汉字 全拼拼音 ; 字母顺序越后值越大
    -   日期：日期越早值越小

#### 3.分组查询

查询语句7个关键字：

```sql
select [top] from [where] [group by] [having] [order by]
```

查询语句的顺序：

```sql
5.select 7.top 字段列表 1.from  表列表 2.where 源数据筛选条件 3.group by 分组统计字段列表 4.having 对分组统计结果集做筛选 6.order by 得到最终结果集之后的数据重排
```

1.先from获取数据源、2.再where筛选数据源、3.再group by对数据源进行分组、4.再having对分组统计的结果集做筛选、[聚合函数对结果集进行聚合分析]、5.再select对筛选后的结果集显示、6.再order by对最终结果集的数据重排、7.最后top再对结果集限制行再显示

疑难解答：

-   where中为什么不能使用聚合函数(聚合函数不能作为where子句的直接搜索条件)？  --需要先筛选出数据源再使用聚合函数对结果集进行聚合分析再进行显示
-   having中为什么不能使用student表中的列？  --分组后的结果集只针对分组后的结果字段，会忽略掉student表中其他字段
-   having中为什么不能使用别名？  --先对分组统计的结果集做筛选，在进行显示
-   order by中为什么能使用别名？  --先对最终结果集进行显示，在对数据进行重排
-   没分组时就是1个组，也可以使用having筛选，只不过没有实际意义
-   没分组时不能对count和classid同时进行显示，数据源筛选后count是一个值，而classid是多个值，筛选后结果集中每个字段值必须是一对一关系. 单个的字段值不能应用于组，having子句中的列必须是组列
-   分组一般是题中出现查询'每个'或'各个'...字样时分组

#### 4.连接查询(表连接)

表连接:

1.  交叉连接(笛卡尔积): `from 结果集 cross join 结果集`
2.  内连接(按条件关系匹配满足条件的): `from 结果集 inner join 结果集 on 条件`  --on 条件(有主外键关系的字段)
3.  外连接(恢复没有匹配的): `from 结果集 left|right|full join 结果集 on 条件` 

语法注意: 网上很多都是这样写的，不建议下面这种写法

```sql
--国际组织ANSI-SQL 89版语法(当时还没有外连接)
select * from Employee, Title;		--交叉连接
select * from Employee t1, Title t2 where t1.titleId=t2.titleId;	--内连接
```

注意问题:

1.  连接查询是将多个表按相应条件拼接成一个结果表

    -   可以select * from 表连接 ;  将所有字段全部显示
    -   也可以 select 字段 from 表连接 ; 若选择重名字段，必须指明其所在表; 不是重名的字段可以不指定

2.  表自连接时必须为表取别名，否则产生中间表后，会因为表同名而产生错误

3.  内连接实现的查询可以使用where来实现  --第五点的说明

4. where子句替换having子句的情况：当实现相同功能时可以替换:having中使用组列进行筛选时可以用where替换，使用聚合函数筛选时不可以用where替换

    -   `where StuNo = 6` ;   筛选源数据中StuNo为6的行(相当于StuNo为6的组)  --这种查询更快，毕竟比下面少一条语句
    -   `group by StuNo having StuNo = 6`;将StuNo分组，再筛选StuNo为6的组

    5.  `from 结果集1 ... join 结果集2 ... join ... on 连接条件`--on 不能使用where替换，会出现语法错误

      `from 结果集1，结果集2，... where 连接条件`	--where 不能使用on替换

#### 5.union运算符和类型转换函数

##### 1.union运算符

union运算符: union可以合并多个结果集

使用有两个前提和一个注意:

-   合并的结果集的列数必须完全一致
-   合并的多个结果集的对应列的类型需要一致(可以相互转换)
-   结果集的列名只与第一个结果集有关(注意)

union 与 union all 区别:

-   union：做了distinct操作，过滤重覆记录
-   union all：不做distinct操作，它的合并效率更高，因为没有必须去判断结果记录是否重复

##### 2.类型转换函数

-   cast(源数据 as 目标类型): `print '我的总成绩是：'+cast(200 as varchar(30))`
-   convert(目标类型，源数据, 格式):
    -   `print '我的总成绩是：'+convert(char(3),200)` 
    -   `select convert(char(30),GETDATE(),121)` 

#### 6.子查询

子查询: 在一个查询中，一个查询的结果作为另一个查询的条件，那么这个查询称为子查询，这个使用条件的查询称为外部查询.

标量子查询与多值子查询:

-   标量子查询：单值子查询(查询结果是单个值)  where 字段=(单值子查询)
-   多值子查询：多值子查询(查询结果是多个值)  where 字段 in(多值子查询)

独立子查询与相关子查询:

-   独立子查询：字段与外部查询没有关系，可以独立查询出结果

-   相关子查询：条件中某个字段与外部查询表中字段有联系，不能独立查询出结果

    `子查询中 where 字段 = 外表名.字段(多值时也可以)` 





