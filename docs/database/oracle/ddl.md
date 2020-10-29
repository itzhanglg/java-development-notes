### 一.创建和管理表

#### 1.创建表

先去看下Oracle有哪些数据类型：

```
varchar2(size) 可变长字符数据
char(size) 定长字符数据
number(p,s) 可变长数值数据
date 日期型数据
rowid 行地址(伪列)	如：select rowid,empno,ename,sal from emp;
```

1.1 手动创建表

```sql
create table test1  
(tid number,tname varchar2(20));
```

1.2 使用子查询创建表

```sql
-- 创建表：保存20号部门的员工
create table emp20
as
select * from emp where deptno=20;

-- 创建表：员工号 姓名 月薪 年薪 部门名称
create table empinfo
as
select e.empno,e.ename,e.sal,e.sal*12 annsal,d.dname
from emp e,dept d
where e.deptno = d.deptno;
```

#### 2.修改表

增加新列,修改列,删除列,重命名列,重命名表：

2.1 增加新列

```sql
alter table test1 add photo blob;
```

2.2 修改列

```sql
alter table test1 modify tname varchar2(30);
```

2.3 删除列

```sql
alter table test1 drop column photo;
```

2.4 重命名列

```sql
alter table test1 rename column tname to username;
```

2.5 重命名对象(执行rename语句改变表,视图,序列或同义词的名称;必须是对象的拥有者)

```sql
rename test1 to test2;
```

#### 3.删除表

3.1 删除表:放在oracle回收站  注意:管理员没有回收站

- 数据和结构都被删除;
- 所有正在运行的相关事务被提交;
- 所有相关索引被删除;
- drop table语句不能回滚,但是可以闪回;

例如：

```sql
drop table test1;
```

查看回收站的表:需要加上双引号

```sql
select * from "BIN$dOIikvhJSGybgRBwrH7Zjg==$0";
```

闪回删除 --> 从回收站还原数据

```sql
flashback table test1 to before drop;
```

3.2 删除表：不经过回收站

```sql
drop table test1 purge;
```

3.3 清空表：删除表中所有的数据;释放表的存储空间;不能回滚;

```sql
truncate table test;
```

3.4 删除表数据(可回滚)

```sql
delete from test1;
```

3.5 查看回收站

```sql
show recyclebin;
```

3.6 清空回收站

```sql
purge recyclebin;
```

#### 4.数据完整性(约束)

oracle中约束类型：

```
主键约束(primary key)
非空约束(not null)
唯一约束(unique)
外键约束(foreign key)	-- 违反完整性约束条件
检查性约束(check)
default值：值,表达式和sql语句都可以作为默认值,类型必须和该列类型一致
如：gender number(1) default 1
```

外键关联注意：

```
foreign key:在子表中,定义了一个表级的约束;
references:指定表和父表中的列
on delete cascade:当删除父表时,级联删除子表记录
on delete set null:将子表的相关依赖记录的外键值置为null

外键引用的一定是主表的主键;
删表时一定先删子表数据再删主表数据;
也可以强制删除drop table orders cascade constraint;
也可以将子表关联数据设置为null,再删主表(on delete set null);
也可使用级联删除(on delete cascade);
```

check约束中可以使用下面的表达式：

```
引用currval,nextval,level和rownum伪列
调用sysdate,uid,user和userenv函数
另一个表的查询记录
```

创建约束的时机：

```sql
-- 1.创建表的时候,同时创建约束:
create table student
(
    sid number(10) constraint student_id_pk primary key,
    sname varchar2(20) constraint student_name_nn not null,
    gender varchar2(2) constraint student_gender_ck check (gender in ('男','女')),
    email varchar2(30) constraint student_email_uq unique
    constraint student_email_nn not null,
    deptno number constraint student_deptno_fk references dept(deptno) on delete set null
);

-- 2.表结构创建完成后
create table student2
(
    sid number(10),
    sname varchar2(20) constraint student_name_nn not null,
    gender varchar2(2),
    email varchar2(30) constraint student_email_nn not null,						   
    deptno number,
    constraint student_id_pk primary key(sid),		
    constraint student_gender_ck check (gender in ('男','女')),
    constraint student_email_uq unique(email),
    constraint student_deptno_fk foreign key(deptno) references dept(deptno) on delete set null
);

insert into student values(1,'Tom','男','tom@126.com',10);
```

### 二.其它数据库对象

首先介绍下sys管理员，赋予权限：

```sql
grant create view to scott;  -- 视图权限
grant select on hr.employees to scott;  -- 查询权限
grant create synonym to scott;  -- 同义词权限
```

#### 1.视图(简化复杂查询)

1.1 什么是视图

视图就是封装了一条复杂查询的语句，视图理解为存储起来的select语句。视图是一个虚表，视图建立在已有表的基础上，视图赖以建立的这些表称为基表。视图向用户提供基表数据的另一种表现形式。

1.2 视图的优点

```
限制数据访问；
简化复杂查询；
提供数据的相互独立；
同样的数据可以有不同的显示方式；
但:视图不能提高性能。
```

1.3 语法

```sql
create [or replace] view 视图名
as subquery
[with read only]  -- 只能做查询操作
[with check option]
```

1.3 创建视图示例

```sql
-- 注:不建议通过视图对表中数据进行修改,因为会受到很多限制.
create or replace view empvd20
as select * from emp t where t.deptno=20;
```

1.4 其它视图操作

```sql
-- 修改视图
create or replace view 子句
-- 屏蔽DML操作
with read only 屏蔽对视图的DML操作
-- 删除视图：删除视图只是删除视图的定义,并不会删除基表的数据
drop view 视图名;
```

#### 2.序列(自增)

Oracle完成自增只能依靠序列完成,并且oracle将序列值装入内存可以提高访问效率。

语法：

```sql
create sequence 序列名称
[increment by n]  -- 步长
[start with n]  -- 起始点
[cache n | nocache]  -- 缓存量
```

序列操作(伪列)：

- nextval：取得序列的下一个内容；
- currval：取得序列的当前内容。

序列可能产生裂缝的原因：回滚、系统异常、多个表共用一个序列。

修改序列：

```sql
-- 修改初始值只能通过删除序列之后重建序列的方法实现
alter sequence 序列名
increment by ***
maxvalue ***
nocache;		
```

删除序列：

```sql
drop sequence 序列名;
```

示例：

```sql
create sequence myseq;
create table testseq (tid number,tname varchar2(20));
insert into testseq values(myseq.nextval,'aaa');
commit;  -- 没提交时可回滚,会产生裂缝
```

#### 3.索引(提高查询效率)

索引是用来加速数据存取的数据对象，查询语句中不需要指定使用哪个索引，由oracle管理系统进行维护和使用。通过指针加速oracle服务器的查询速度，通过快速定位数据的方法，减少磁盘I/O。

单列索引：基于单个列所建立的索引。

```sql
create index 索引名 on 表名(列名)
```

复合索引：基于两个列或多个列的索引.同一张表可以有多个索引,但列的组合必须不同。

```sql
create index empidx on emp(ename,job);
create index empidx on emp(job,ename);
```

删除索引：

```sql
drop index 索引名;
```

示例：

```sql
-- SQL的执行计划
explain plan for select * from emp where deptno=10;
select * from table(dbms_xplan.display);

--------------------------------------------------------------------------
| Id  | Operation         | Name | Rows  | Bytes | Cost (%CPU)| Time     |
--------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |      |     3 |   114 |     3   (0)| 00:00:01 |
|*  1 |  TABLE ACCESS FULL| EMP  |     3 |   114 |     3   (0)| 00:00:01 |
--------------------------------------------------------------------------

-- 创建目录(索引)
create index myindex on emp(deptno);
explain plan for select * from emp where deptno=10;
select * from table(dbms_xplan.display);
		
--------------------------------------------------------------------------------------
| Id  | Operation                   | Name    | Rows  | Bytes | Cost (%CPU)| Time    |
--------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |         |     3 |   114 |     2   (0)| 00:00:01|
|   1 |  TABLE ACCESS BY INDEX ROWID| EMP     |     3 |   114 |     2   (0)| 00:00:01|
|*  2 |   INDEX RANGE SCAN          | MYINDEX |     3 |       |     1   (0)| 00:00:01|
--------------------------------------------------------------------------------------
```

#### 4.同义词(给对象起别名)

同义词可以很方便的访问其它用户的数据库对象；缩短了对象名字的长度。

语法(私有或公有的):为数据库对象取别名

```sql
create [public] synonym 同义词名
for object;
```

示例：

```sql
-- 为scott.emp起公有的别名
create public synonym emp for scott.emp;
-- 用公有别名查询表
select * from emp;
-- 删除公有的同义词
drop public synonym emp;

-- 在scott用户下查询hr用户的employees表
select count(*) from hr.employees;
-- 为hr.employees起别名
create synonym hremp for hr.employees;
-- 用别名查询表
select count(*) from hremp;
```

