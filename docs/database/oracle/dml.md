### 一.基本查询

基本命令：

```sql
SQL> --当前用户
SQL> show user;
SQL> --当前用户下的表
SQL> select * from tab;
SQL> --员工表结构
SQL> desc emp;

SQL> --清屏
SQL> host cls（window）;
SQL> host clear（Linux）;
SQL> --sql语句执行时间
SQL> set timing on
SQL> --关闭sql语句执行时间
SQL> set timing off
SQL> --关闭回显(每条语句的提示)
SQL> set feedback off
SQL> set feedback on(打开回显)
SQL> --执行sql脚本
SQL> @e:\mysource\testdelete.sql

SQL> --排版格式
SQL> --分页行数
SQL> set pagesize 16;
SQL> --显示行宽
SQL> show linesize;
SQL> --设置行宽
SQL> set linesize 120;
SQL> --设置列宽
SQL> col ename for a8;
SQL> col sal for 9999;
SQL> --执行上一条sql语句
SQL> /
```

基本查询：

```sql
SQL> --通过具体列名查询
SQL> select empno,ename,sal
  2  from emp;
SQL> --查询员工信息:员工号 姓名 月薪 年薪
SQL> select empno,ename,sal,sal*12
  2  from emp;

SQL> --1.包含null的表达式都为null
SQL> --查询员工信息:员工号 姓名 月薪 年薪 奖金 年收入
SQL> select empno,ename,sal,sal*12,comm,sal*12+comm
  2  from emp;		(错误写法)
SQL> --nvl(a,b)  nvl2 过滤null值,a为null时,就为b,反则就为a
SQL> select empno,ename,sal,sal*12,comm,sal*12+nvl(comm,0)
  2  from emp;		(正确写法)

SQL> --2.null永远!=null
SQL> select *
  2  from emp
  3  where comm=null;	(错误写法)
未选定行
SQL> select *
  2  from emp
  3  where comm is null;(正确写法)

SQL> --添加别名,ed命令编辑SQL语句 edit (数字,空格时别名必须加引号)
SQL> ed;
已写入 file afiedt.buf
  1  select empno as "员工号",ename "姓名",sal 月薪,sal*12,comm,sal*12+nvl(comm,0)
  2* from emp
SQL> /

SQL> --distinct 去掉重复记录
SQL> select distinct deptno from emp;
SQL> --distinct 作用于后面所有字段(所有字段组合不重复)
SQL> select distinct deptno,job from emp;

SQL> --字符串拼接:concat()函数  ||连接符(用来合成列)
SQL> --dual表:伪表
SQL> select concat('hello','world') from dual;
SQL> select 3+2 from dual;
SQL> --伪列
SQL> select 'hello'||' world' 字符串 from dual;
SQL> select ename||'的薪水是'||sal 信息 from emp;
```

sqlplus命令：

```sql
SQL> --1.c命令修改错误单词 change
SQL> c /form/from	--将form改成from
SQL> /
SQL> --2.ed命令编辑SQL语句 edit
SQL> ed;
已写入 file afiedt.buf
  1  select empno as "员工号",ename "姓名",sal 月薪,sal*12,comm,sal*12+nvl(comm,0)
  2* from emp
SQL> /

--oracle自带的录屏功能（txt文件）
SQL> spool E:\database\oracle\notes\1_基本查询.txt
SQL> spool off
```

sql语句与sqlplus命令：

```sql
-- 1.sql关键字不能缩写
insert 
delete
update
select

-- 2.sqlplus关键字能缩写
c	-- change(改变sql语句中某个词)
ed	-- edit(编辑sql语句)
for	-- format(设置字段格式)
desc	-- describe(表结构)
col 	-- column(字段)
```

### 二.过滤和排序

日期相关：

```sql
SQL> --默认日期格式(DD-MON-RR)(17-11月-1981)(17-11月-81)
SQL> --Oracle会把满足默认日期格式的字符串转化为date类型
SQL> select *
  2  from emp
  3  where hiredate = '17-11月-1981';

SQL> --修改日期默认格式
SQL> select * from v$nls_parameters;
SQL> alter session set NLS_DATE_FORMAT='yyyy-mm-dd';  -- 当前会话有效
SQL> alter system set NLS_DATE_FORMAT='yyyy-mm-dd';	 -- 永久有效
SQL> select *
  2  from emp
  3  where hiredate='1981-11-17';
```

过滤：

```sql
-- 语法：
java: int a = 0;
pl/sql: a number := 0;

-- 1.比较运算符：
=  等于(不是==)  赋值使用 := 符号
>  大于
>= 大于等于
<  小于
<= 小于等于
<> 不等于(也可以是!=)

-- 2.其它比较运算:
between...and...  在两个值之间(包含边界)
in(set)  等于值列表中的一个
like  模糊查询  
  %:代表任意个任意字符 
  _:代表任意的单个字符 
  []:代表一个具体的值范围中的任意一个字符
is null  空值

-- 注:
[not] between and: 1.含有边界 2.小值在前,大值在后
[not] in(set): 在集合中  in(10,20,null)  若集合中含有null,不能使用not in;但可以使用in
like: 转义字符,使用escape标识符选择'%'和'_'符号
SQL> select * from emp where ename like '%\_%' escape '\';

mysql:手动开启事务
oracle:自动开启事务	rollback回滚

-- 3.逻辑运算符:
and  逻辑并
or  逻辑或
not  逻辑否

SQL优化: 2.where解析顺序: 右 --> 左  -- sql执行计划

-- 4.优先级
算术运算符 > 连接符 > 比较符 > is [not] null,like,[not] in > [not] between > not > and > or
```

排序：

```sql
-- order by 后面 + 列,表达式,别名,序号(列名的顺序值)
SQL> --表达式
SQL> select empno,ename,sal,sal*12 
SQL> from emp 
SQL> order by sal*12 desc;
SQL> --别名
SQL> select empno,ename,sal,sal*12 年薪
SQL> from emp 
SQL> order by 年薪 desc;
SQL> --序号(在字段个数内,索引从1开始)
SQL> select empno,ename,sal,sal*12 年薪
SQL> from emp 
SQL> order by 4 desc;
```

order by 作用于后面所有列,先按照第一个列排序,如果相同,则按照第二列排序,以此类推。desc只作用于离他最近的列(asc,desc)。排序后的结果,是否是原来的表?

**小结**

where语句(比较,逻辑运算符)：

```sql
-- 1.非空和空的限制
is [not] null:
SQL> select * from emp where comm is null and not(sal>1500)

-- 2.范围限制
between..and:
in(子查询): 值可以是数字或者字符串
SQl> select * from emp where hiredate between '1-1月-1981' and '31-12月-1981';

-- 3.模糊查询
like中主要使用以下两种通配符:
"%":可以匹配任意长度的内容
"_":可以匹配一个长度的内容
```

order by对结果排序：

```sql
-- 1.排序语法
select *|列名 from 表名 {where 查询条件} order by 列名1 asc|desc,列名2 asc|desc...

-- 2.排序中的空值问题
--当排序时有可能存在null时会出现问题,可以用nulls first,nulls last指定null值显示的位置
SQL> select * from emp order by comm desc nulls last
--原因:oracle中null值最大
```

### 三.单行函数

#### 1.字符函数

1.1 大小写控制函数

```sql
lower 转小写
upper 转大写
initcap 每个单词首字母大写,其余小写
```

1.2 字符控制函数

```sql
concat(a,b,...) 拼接字符串
substr(索引从1开始,子串包含b位)
	substr(a,b) 从a中,第b位开始取后面所有
	substr(a,b,c) 从a中,第b位开始取c的长度
length/lengthb(1个汉字是2个字节)
	length(a) 字符数
	lengthb(a) 字节数
instr
	instr(a,b) 在a中查找b,若存在则返回索引(从1开始),反则返回0
lpad|rpad
	lpad 左填充  lpad('abcd',10,'*') - ******abcd
	rpad 右填充  rpad('abcd',10,'*') - abcd******
trim
	trim 去掉前后指定的字符  trim('H' from 'Hello WorldH') - 'ello world'
replace
	replace('a','b','c') 用c替换a中所有的b  replace('hello','l','*') - he**o
```

#### 2.数字函数

```sql
-- 1.round 四舍五入
	round(45.926,2) - 45.93
	round(45.926,0) - 46
	round(45.926,-1) - 50
	round(45.926,-2) - 0
-- 2.trunc 截断
	trunc(45.926,2) - 45.92
	trunc(45.926,0) - 45
	trunc(45.926,-1) - 40
	trunc(45.926,-2) - 0
-- 3.mod 求余
	mod(1600,300) - 100
```

#### 3.日期函数

日期型数据含有两个值:日期和时间。默认格式:DD-MON-RR (1-1月-2019),输出时是没有时间的。

3.1 数学运算

```plsql
-- a.在日期上加上或减去一个数字结果仍为日期
	select (sysdate-1) 昨天,sysdate 今天,(sysdate+1) 明天 from dual;
	
-- b.两个日期相减返回日期之间相差的天数
	select ename,hiredate,(sysdate-hiredate) 天,(sysdate-hiredate)/7 星期,
	(sysdate-hiredate)/30 月,(sysdate-hiredate)/365 年
	from emp;
	
-- c.两个日期不能相加
```

3.2 日期函数

```plsql
months_between 两个日期相差的月数
	months_between(sysdate,hiredate)
add_months 向指定日期中加上若干月数
	add_months(sysdate,12) / add_months(sysdate,-12)
next_day 指定日期的下一个日期
	next_day(sysdate,'星期五')
last_day 本月的最后一天
	last_day(sysdate)
round 日期四舍五入
	round(sysdate,'month')	看日期是否超过一半
	round(sysdate,'year')	看月份是否超过一半
trunc 日期截断
	trunc(sysdate,'month')
	trunc(sysdate,'year')
```

MySQL与Oracle在日期上的区别：

```sql
mysql：
	date 年月日
	datetime 年月日时分秒
	获取当前时间 now()
oracle：
	date 年月日时分秒
	获取当前时间 sysdate关键字
	to_char(sysdate,'yyyy-mm-dd hh24:mi:ss')将日期按照某种格式转为为相应的字符串
	-- select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') 当前时间 from dual;
	
	天数:(sysdate-date)
	星期:(sysdate-date)/7
	月数:months_between(sysdate,date)
	
	next_day函数的应用:每个星期一自动备份表中的数据
	1.分布式数据库
	2.触发器 快照
```

#### 4.转换函数

数据类型转换：

```plsql
-- a.隐式:oracle自动完成数据类型的转换
varchar2 or char  -->  number
varchar2 or char  -->  date
number  -->  varchar2
date  -->  varchar2

-- b.显示
to_char(date,'format_model')
	yyyy年  mm月  mon几月  day星期几  dd日
	to_char(sysdate,'fmyyyy-mm-dd')	fm去掉补齐的0
	to_char(sysdate,'yyyy-mm-dd hh24:mi:ss" 今天是"day')
to_char(number,'format_model')
	9数字  0零  $美元符  L货币符  .小数点  ,千位符
	to_char(sal,'L9,999.99')
to_number(char[,'format_model'])
	to_number('￥1,200.00','L9,999.99')
to_date(char[,'format_model'])
	to_date('2019-01-01 12:12:12 今天是星期二','yyyy-mm-dd hh24:mi:ss" 今天是"day')
```

#### 5.通用函数

通用函数适用于任何数据类型,同时也适用于空值。

```plsql
-- 通用函数
nvl(a,b)	若a为null,则a赋为b
nvl2(a,b,c)	若a为null,则赋为c;反之则赋为b
	select ename,nvl(comm,0),sal*12+nvl2(comm,comm,0) from emp
nullif(a,b)	若a=b时,返回null;否则返回a
	select nullif('abc','abc') 值 from dual; -- 
	select nullif('abc','abcd') 值 from dual;  -- abc
conalesce(a,b,c,...) 从左到右,找到第一个不为null的值
	select comm,sal,coalesce(comm,sal) from emp;
```

#### 6.条件表达式

在sql语句中使用if-then-else。

实现方式：

- case表达式:SQL99语法			-- case ... end 别名
- decode函数:oracle自己语法		-- decode() 别名

示例：

```plsql
select ename,job,sal 涨前,
	case job when 'PRESIDENT' then sal+1000
		when 'MANAGER' then sal+800
		else sal+400
	end 涨后
from emp;
	
select ename,job,sal 涨前,
	decode(job,'PERSIDENT',sal+1000,'MANAGER',sal+800,sal+400) 涨后
from emp;
	
select ename,sal,
	decode(trunc(sal/2000,0),
		0,0.00,
		1,0.09,
		2,0.20,
		3,0.30,
		4,0.40,
		5,0.42,
		6,0.44,
		  0.45) tax_rate
from emp;
	
case when sal<3000 then *****
	when sal>=3000 and sal<6000 then *****
	else *****
end
```

### 四.分组函数

分组函数作用于一组数据,并对一组数据返回一个值。

常用的分组函数：

```sql
avg()
count(expr)    返回expr不为空的记录总数
count(distinct expr)	返回expr非空且不重复的记录总数
max()
min()
sum()

-- 组函数会忽略空值，nvl函数使分组函数无法忽略空值(comm中存在null)
select sum(sal)/count(*) 一,avg(sal) 二 from emp;
select sum(comm)/count(*) 一,sum(comm)/count(comm) 二,avg(comm) 三 from emp;
select sum(comm)/count(*) 一,sum(comm)/count(nvl(comm,0)) 二,avg(comm) 三 from emp;
```

分组数据：可以使用group by子句将表中的数据分成若干组。多个列的分组:先按照第一个列分组,如果相同,再按第二个列分组,以此类推。

注意：

- 如果使用分组函数，sql只可以把group by分组条件字段和分组函数查询出来，不能有其它字段；
- 如果使用分组函数，不使用group by只可以查询出来分组函数的值。
- 在select列表中所有未包含在组函数中的列都应该包含在group by子句中；
- 包含在group by子句中的列不必包含在select列表中。

过滤分组：使用having过滤分组、行已经被分组、使用了组函数、满足having子句中条件的分组将被显示。

```sql
select column,group_function(column) 
from table
[where condition]
[group by group_by_expression]
[having group_condition]
[order by column];
```

where和having的区别:where不能使用组函数,having子句中可以使用组函数。

嵌套组函数:显示平均工资的最大值  max(avg(sal))。

group by增强版(sqlplus 报表)：

```sql
-- 查询各部门不同工种的工资情况:
select deptno,job,sum(sal) from emp group by deptno,job
+
select deptno,sum(sal) from emp group by deptno
+
select sum(sal) from emp
=
select deptno,job,sum(sal) from emp group by rollup(deptno,job)
-- 显示报表格式:
break on deptno skip 2
break on null
```

### 五.多表查询

#### 1.笛卡尔集

两张表:列数相加,行数相乘得到的笛卡尔集。

> select count(*) from emp e,emp b;	-- 笛卡尔全集的记录数

笛卡尔集会在下面条件下产生：省略连接条件、省略连接条件、所有表中的所有行互相连接。

为了避免笛卡尔集，可以在where加入有效的连接条件。在实际运行环境下，应避免使用笛卡尔全集。

#### 2.oracle的连接条件的类型(在笛卡尔积的基础上)

oracle连接：

```sql
--在where子句中写入连接条件；在表中有相同列时,在列名之前加上表名前缀
select table1.column,table2.column
from table1,table2
where table1.column=table2.column2;
```

表的别名：使用别名可以简化查询,使用表名前缀可以提高执行效率;如果使用了表的别名,则不能再使用表的真名。

连接多个表：连接n个表,至少需要n-1个连接条件。例如:连接三个表,至少需要两个连接条件。

外连接语法：使用外连接可以查询不满足连接条件的数据，外连接的符号是(+)。

```sql
-- 右外连接:当t1.column=t2.column不成立的时候，等号右边的表任然被包含在最后的结果中
select t1.column,t2.column from t1,t2 where t1.column(+)=t2.column;
-- 左外连接:当t1.column=t2.column不成立的时候，等号左边的表任然被包含在最后的结果中
select t1.column,t2.column from t1,t2 where t1.column=t2.column(+);

-- SQL99语法：inner/left/right/full join (内/左/右/全)连接  inner可省略
select t1.column,t2.column
from t1 
inner join t2 on t1.column= t2.column
inner join t3 on t2.column= t3.column;
```

多表连接查询示例：

```sql
--1.等值连接：查询员工信息：员工号  姓名 月薪 部门名称
select e.empno,e.ename,e.sal,d.dname
from emp e,dept d
where e.dept=d.deptno;
		
--2.不等值连接：查询员工信息：员工号  姓名 月薪 工资级别
select e.empno,e.ename,e.sal,s.grade
from emp e,salgrade s
where e.sal between s.losal and s.hisal;

--3.外连接：希望把某些不成立的记录（40号部门），任然包含在最后的结果中 ---> 外连接
select d.deptno 部门号,d.dname 部门名称,count(e.empno) 人数
from emp e,dept d
where e.deptno(+)=d.deptno
group by d.deptno,d.dname;

--4.自连接：通过表的别名，将同一张表视为多张表
--查询员工信息：员工姓名  老板姓名
select e.ename 员工姓名,b.ename 老板姓名
from emp e,emp b
where e.mgr=b.empno;	
--自连接不适合操作大表
select count(*) from emp e,emp b;  -- 笛卡尔全集

--5.层次查询
select
from emp
connect by 上一层的员工=老板
start with 起始条件

select level,empno,ename,mgr
from emp
connect by prior empno=mgr
start with mgr is null		
order by 1;

//start with empno=7839;
```

### 六.子查询

#### 1.概念

oracle中的null：代表的是无意义，或者没有值。

```sql
1.逻辑运算:与null值进行逻辑运算,只有or且另一个为真时才为真,其余不是null就是false
					and null		or null
		true		null			true
		false		false			null
		null		null			null
2.与null值进行比较或者算术运算（<、>、=、!=、+、-、*、/），结果仍是NULL
3.要判定某个值是否为NULL，可以用IS NULL或者IS NOT NULL。
4.可以用一些函数来处理null值，oracle排序中默认null最大
```

子查询：

```sql
--1.单行子查询：只返回一条记录
单行操作符:
	=  >  >=  <  <=  <>
单行子查询中的null值问题:
	子查询不返回任何行时  --  no rows selected
--2.多行子查询：返回了多条记录
多行操作符:
	in 等于列表中的任何一个
	any 和子查询返回的任意一个值比较
	all 和子查询返回的所有值比较
多行子查询中的null值问题:
	not in  <==>  <>all  <==>  !=all  <==>  不等于子查询返回的所有值
	in  <==>  =any  <==>  等于子查询返回的任意一个值
--3.相关子查询：将主查询中的值作为参数传递给子查询
```

其它：

```sql
--1.伪列rownum
1.rownum永远按照默认的顺序生成
2.rownum只能使用< <=；不能使用> >=
--2.表
标准表,索引表
临时表:
	1.create global temporary table  ****
	2.自动：对表进行排序时自动创建临时表
	特点：当事务或者会话结束的时候，表中的数据自动删除
--3.行转列
wm_concat(varchar2) 组函数
col nameslist for a50;
select deptno,wm_concat(ename) nameslist from emp group by deptno;
	
	DEPTNO NAMESLIST
---------- --------------------------------------------------
		10 CLARK,MILLER,KING
		20 SMITH,FORD,ADAMS,SCOTT,JONES
		30 ALLEN,JAMES,TURNER,BLAKE,MARTIN,WARD
```

#### 2.注意问题

1. 子查询要在括号内,将子查询放在比较条件的右侧
2. 合理的书写风格
3. 可以在主查询的where select having from 后面使用子查询
4. 不可以在group by使用子查询
5. 强调from后面的子查询
6. 主查询和子查询可以不是同一张表;只要子查询返回的结果 主查询可以使用 即可
7. 一般不在子查询中排序;但在top-n分析问题中,必须对子查询排序
8. 一般先执行子查询,再执行主查询;但相关子查询例外
9. 单行子查询只能使用单行操作符;多行子查询只能使用多行操作符
10. 子查询中的null

具体问题分析：

3.可以在主查询的where select having from 后面使用子查询

```sql
where:
	select * from emp 
	where sal>(select sal from emp where ename='SCOTT');
select:(select后只能是单行子查询)
	select empno,ename,sal,(select job from emp where empno=7839) 第四列 
	from emp;
having:
	select deptno,min(sal) from emp
	group by deptno
	having min(sal)>(select min(sal) from emp where deptno=20);
```

5.强调from后面的子查询  *****

```sql
select * from (select empno,ename,sal from emp);
select * from (select empno,ename,sal,sal*12 年薪 from emp);
```

6.主查询和子查询可以不是同一张表;只要子查询返回的结果 主查询可以使用 即可

```sql
--查询部门名称是SALES的员工
-- 子查询:
select * from emp 
where deptno=(select deptno from dept where dname='SALES');
-- 多表查询:
select e.*
from emp e,dept d
where e.deptno=d.deptno and d.dname='SALES';
```

9.单行子查询只能使用单行操作符;多行子查询只能使用多行操作符

```sql
--多行子查询:
		--in(在集合中)
		--查询部门名称是SALES和ACCOUNTING的员工
		select * from emp
		where deptno in (select deptno from dept where dname='SALES' or dname='ACCOUNTING');
		select e.* from emp e,dept d
		where e.deptno=d.deptno and (d.dname='SALES' or d.dname='ACCOUNTING');
		
		--any:和集合中的任意一个值比较
		--查询工资比30号部门任意一个员工高的员工信息
		select * from emp 
		where sal > any (select sal from emp where deptno=30);
		select * from emp 
		where sal > (select min(sal) from emp where deptno=30);

		--all:和集合中的所有值比较
		--查询工资比30号部门所有员工高的员工信息
		select * from emp
		where sal > all (select sal from emp where deptno=30);
		select * from emp
		where sal > (select max(sal) from emp where deptno=30);
```

10.子查询中的null

```sql
--多行子查询中的null
	sal not in (a1,a2,..) <==> sal!=a1 and sal!=a2 and sal!=null and sal!=... 
	sal in (a1,a2,...) <==> sal=a1 or sal=a2 or sal=null or sal=...
--查询不是老板的员工信息	-- not in (set) 子查询中不能有null值
	select * from emp where empno not in (select mgr from emp);  --  错误
	select * from emp 
	where empno not in (select mgr from emp where mgr is not null);  --  正确
--查询是老板的员工信息
	select * from emp where empno in (select mgr from emp);
```

7.一般不在子查询中排序;但在top-n分析问题中,必须对子查询排序

```sql
--rownum永远按照默认的顺序生成
		
		--找到员工表中工资最高的前三名
		select rownum,empno,ename,sal
		from (select * from emp order by sal desc)
		where rownum <= 3;
		
			ROWNUM      EMPNO ENAME             SAL
		---------- ---------- ---------- ----------
				 1       7839 KING             5000
				 2       7788 SCOTT            3000
				 3       7902 FORD             3000
				 
--Paging Query：rownum只能使用< <=；不能使用> >=
		
		select *
		from (select rownum r,e1.empno,e1.ename,e1.sal
			  from (select * from emp order by sal) e1
			  where rownum<=8)
		where r>=5;
		
				 R      EMPNO ENAME             SAL
		---------- ---------- ---------- ----------
				 5       7654 MARTIN           1250
				 6       7934 MILLER           1300
				 7       7844 TURNER           1500
				 8       7499 ALLEN            1600
```

8.一般先执行子查询,再执行主查询;但相关子查询例外

```sql
--找到员工表中薪水大于本部门平均薪水的员工
表连接:
	select e.empno,e.ename,e.sal,d.avgsal
	from emp e,(select deptno,avg(sal) avgsal from emp group by deptno) d
	where e.deptno = d.deptno and e.sal>d.avgsal;
相关子查询:将主查询中的值 作为参数传递给子查询
	select empno,ename,sal,(select avg(sal) from emp where deptno=e.deptno) avgsal
	from emp e
	where sal > (select avg(sal) from emp where deptno=e.deptno);
		
			 EMPNO ENAME             SAL     AVGSAL
		---------- ---------- ---------- ----------
			  7499 ALLEN            1600 1566.66667
			  7566 JONES            2975       2175
			  7698 BLAKE            2850 1566.66667
			  7788 SCOTT            3000       2175
			  7839 KING             5000 2916.66667
			  7902 FORD             3000       2175
		  	
	=============================================================================
--统计每年入职的员工个数
	select count(*) total,
		sum(decode(to_char(hiredate,'yyyy'),'1980',1,0)) "1980",
		sum(decode(to_char(hiredate,'yyyy'),'1981',1,0)) "1981",
		sum(decode(to_char(hiredate,'yyyy'),'1982',1,0)) "1982",
		sum(decode(to_char(hiredate,'yyyy'),'1987',1,0)) "1987"
	from emp;
		
			 TOTAL       1980       1981       1982       1987
		---------- ---------- ---------- ---------- ----------
				14          1         10          1          2
```

### 七.集合运算

并集：

- union 运算符返回两个集合去掉重复元素后的所有记录
- union all返回两个集合的所有记录,包括重复的

交集：intersect运算符返回同时属于两个集合的记录。

差集：minus返回属于第一个集合,但不属于第二个集合的记录。

注意问题：

1. 参与运算的各个集合必须列数相同 且类型一致
2. 采用第一个语句的表头作为最后的表头
3. order by永远在最后(放在最后一句查询语句后)
4. 可以使用括号改变集合执行的顺序(那两个语句先集合运算)

示例：

```sql
--查询10和20号部门的员工
	1. select * from emp where deptno=10 or deptno=20;
	2. select * from emp where deptno in (10,20);
	3. 集合运算
		select * from emp where deptno=10
		union
		select * from emp where deptno=20
		
--尽量不要使用集合运算
	select deptno,job,sum(sal) from emp group by deptno,job
    union
    select deptno,to_char(null),sum(sal) from emp group by deptno
    union
    select to_number(null),to_char(null),sum(sal) from emp;
	已用时间:  00: 00: 00.06
	group by增强版:
	select deptno,job,sum(sal) from emp group by rollup(deptno,job);
	已用时间:  00: 00: 00.02		
```

### 八.处理数据

#### 1.插入数据

1.insert into 表名[(列名1,列名2,...)] values(值1,值2,...)。

```sql
--尽量使用列名,不建议省略,字符和日期型数据包含在单引号种
--若省略必须按照表中字段的顺序来插入值,而且若有空的字段则使用null
SQL> insert into emp(empno,ename,sal,deptno) values(1001,'Tom',3000,10);

--一次插入多条数据:
			insert all
			into test values(***,***)
			into test values(***,***)
			select 1 from dual;

			insert into test(id,name)
			select 1,'11' from dual union
			select 2,'11' from dual;

			begin
			insert into test values();
			insert into test values();
			end;
			/
```

2.地址符&：

```sql
SQL> --PreparedStatement pst = "insert into emp(empno,ename,sal,deptno) values(?,?,?,?)";
SQL> --地址符 &
SQL> insert into emp(empno,ename,sal,deptno) values(&empno,&ename,&sal,&deptno);
		
SQL> select empno,ename,sal,&t from emp;
SQL> select * from &t;
```

3.从其他表种拷贝数据:在insert语句中加入子查询。

```sql
--不必书写values子句,子查询的值列表与insert子句中列名对应
SQL> create table emp10 as select * from emp where 1=2;		(创表)
SQL> --一次性将emp中，所有10号部门的员工插入到emp10中
SQL> insert into emp10 select * from emp where deptno=10;	(外部表拷贝数据)
```

4.海量插入数据:

```sql
1、数据泵（PLSQL程序）
	dbms_datapump(程序包)
2、SQL*Loader
3、外部表
```

#### 2.更新数据

```sql
1.全部修改:update 表名 set 列名1=值1,列名2=值2,...
	局部修改:update 表名 set 列名1=值1,列名2=值2,... where 修改条件
	  
2.在update中使用子查询
	update emp set sal=sal+100 
	where deptno in (select deptno from dept where loc='NEW YORK');
```

#### 3.删除数据

```sql
1.delete from 表名 where 删除条件
	  在delete中使用子查询
2.delete与truncate的区别:
		1.delete是DML(可以回滚),truncate是DDL(不可以回滚)
		2.delete可以闪回(flashback),truncate不可以
		3.delect删除可能产生碎片,并且不再释放空间
		4.delete逐条删除,truncate是先摧毁表结构,再重构表结构
		--flashback其实是一种恢复	  
```

注：插入,更新和删除会引起数据的变化,必须考虑数据的完整性。 由于truncate不能回滚，不使用UNDO表空间，所以在大表删除的时候truncate效率会更高，不过如果表很小的话truncate比delete就要慢了。

#### 4.oracle中的事务

1.事务起始标志: 事务中的第一条DML语句(第一条DML语句的执行作为开始)。

事务结束标志：

- 提交： 显式  commit  隐式(自动提交)： exit(事务正常退出) DDL语言 DCL语言
- 回滚： 显式 rollback   隐式(系统异常终了)： 关闭窗口 掉电  宕机

对数据库的变更处理后,必须提交事务才能让数据真正在数据库中变更，同样还可以把事务回滚,若事务提交后则不可以再回滚。

oracle自动开启事务,mysql需要手动开启事务。

2.事务的保存点:

- savepoint a; 在当前事务中创建保存点
- rollback to savepoing a; 回滚到创建的保存点

3.事务的隔离级别和隔离性:

```sql
SQL99支持的4种隔离级别:
			read uncommitted 读未提交数据
			read commited 读已提交数据
			repeatable read 可重复读  (mysql默认)
			serializable 串行化
oracle支持的3种事务隔离级别:
			read committed	(oracle默认)
			serializable
			read only 只读		
```

4.设置事务的隔离级别：`set transaction read only;`  设置事务只读,不可更改数据。

### 九.分析函数

1.over窗口函数：计算基于组的某种聚合值，对每个组返回多行，而聚会函数对于每个组只返回一行。

```sql
over([partition by column] order by order_by_clause)
		partition by column：参考的组列
		order by order_by_clause：需要参与计算的列
```

2.rank函数：计算一个值在一组值的排位，排位是以1开始的连续整数，具有相等值的行排位相同。序数值后跳跃相应的数值，即若两行的序数为1，则没有序数2，下一行的序数为3。

```sql
rank() over([partition by column] order by order_by_clause) rank
		partition by column：参考的组列
		order by order_by_clause：需要参与计算的列
```

3.将rollup函数与group by命令配合使用，可以提供信息汇总功能（类似于“小计”）。

```sql
group by rollup(column)

break on column skip 1; //调整显示格式
```

4.lag分析函数可以在一次查询中取出同一字段的前N行数据，一般与over窗口函数结合使用。

```sql
--其中，N默认为1。	-- lag(empno)
lag(column,N) over(...)
```

5.first_value函数用于返回结果集中排在第一位的值。

```sql
first_value(expr) over(...)

--显示部门中最高工资的员工姓名
first_value(ename) over(partition by deptno order by sal desc) rich_men
```





