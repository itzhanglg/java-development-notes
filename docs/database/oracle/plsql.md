### 一.基本语法

1.plsql是oracle对sql语言的过程扩展,在sql命令语言中增加了过程处理语句，(如分支,循环等),使sql语言具有过程处理能力。

语法：

```plsql
		declare
			说明部分(变量说明,光标申明,例外说明)
			变量名 数据类型(长度);	-- 直接用":="赋值
			变量名 表.字段%type;	-- 引用型变量(在sql中使用into来赋值)
			变量名 表%rowtype;		-- 记录型变量(在sql中使用into来赋值)
		begin
			语句序列(DML语句)...
		exception
			例外处理语句
		end;
		/
```

2.常量和变量的定义：

说明变量(char,varchar2,date,number,boolean,long)

```plsql
var1 char(15);
married boolean := true;	":=" --> "=";  "=" --> "==";
psal number(7,2);	
```

my_name emp.ename%type;		引用型变量

emp_rec emp%rowtype;		记录型变量

引用型变量:

```plsql
			declare
				emprec emp.ename%type;
			begin
				select ename into emprec from emp where empno=7369;
				dbms_output.put_line(emprec);
			end;
			/
```

记录型变量:

```plsql
			declare
				emprec emp%rowtype;
			begin
				select  * into emprec from emp where empno = 7369;
				dbms_output.put_line(emprec.ename || '' || emprec.sal);
			end;
			/
```

3.if语句：

```plsql
		set serveroutput on;	-- 定义输出;打开语句
		accept num prompt '请输入一个数字';		-- 接受键盘输入,变量num是地址值
		-------------------------------------------------------
		declare
			pnum number := &num;	-- 保存输入的数字(指针)
		begin
			if pnum = 1 then
				dbms_output.put_line('我是1');
			end if;
		end;
		/
		-------------------------------------------------------
		declare
			pnum number := &num;
		begin
			if pnum = 1 then
				dbms_output.put_line('我是1');
			else 
				dbms_output.put_line('我不是1');
			end if;
		end;
		/
		-------------------------------------------------------
		declare
			pnum number := &num;
		begin
			if pnum = 1 then
				dbms_output.put_line('我是1');
			elsif pnum =2 then
				dbms_output.put_line('我是2');
			else
				dbms_output.put_line('其它数字');
			end if;
		end;
		/
```

4.loop循环：

```plsql
		declare
			step number := 1;
		begin
			while step <= 10
			loop
				dbms_output.put_line(step);
				step := step + 1;
			end loop;
		end;
		/
		-------------------------------------------------------
		declare
			step number := 1;
		begin
			loop
				exit when step > 10;
				dbms_output.put_line(step);
				step := step + 1;
			end loop;
		end;
		/
		-------------------------------------------------------
		declare
			step number := 1;
		begin
			for step in 1..10
			loop
				dbms_output.put_line(step);
			end loop;
		end;
		/
```

5.游标(光标cursor)：游标可以存储查询返回的多条数据(ResultSet)。

语法：

```plsql
cursor 游标名 [(参数名 数据类型,参数名 数据类型,...)] is select 语句;
cursor c1 is select ename from emp;
```

步骤：

- 打开游标: open c1; (打开游标执行查询)
- 取一行游标的值: fetch c1 into pename; (取一行到变量中)
- 关闭游标: close c1; (关闭游标释放资源)
- 游标的结束方式: exit when c1%notfound;
- 注意:上面的pjob必须与emp表中的ename列类型一致;定义:pjob emp.ename%type;

示例：

```plsql
			declare
				cursor pc is select * from emp;
				pemp emp%rowtype;
			begin
				open pc;
					loop
						fetch pc into pemp;
						exit when pc%notfound;
						dbms_output.put_line(pemp.empno||' '||pemp.ename);
					end loop;
				close pc;
			end;
			/
			-------------------------------------------------------
			declare
				cursor pc(dno emp.deptno%type) is select empno from emp where deptno=dno;
				pno emp.empno%type;
			begin
				open pc(20);
					loop
						fetch pc into pno;
						exit when pc%notfound;
						update emp set sal=sal+1000 where empno=pno;
					end loop;
				close pc;
			end;
			/
```

6.例外

系统定义异常：

- no_data_found (没有找到数据)
- too_many_rows (select...into语句匹配多个行)
- zero_divide (被零除)
- timeout_on_resource (在等待资源时发生超时)

自定义异常：	

- no_data exception;
- 若遇到异常我们要抛出raise no_data;

### 二.存储过程及函数

#### 1.存储过程

存储过程是一组为了完成特定功能的sql语句集,经编译后存储在数据库中,用户通过指定存储。过程的名字并给出参数来执行它。

语法：

```plsql
		create [or replace] procedure 过程名[(参数名 in/out 数据类型)]
		as/is
		begin
			plsql子程序体;
		end [过程名];
```

调用：

```plsql
		--方法一:
		set serveroutput on;
		exec raisesalary(7839);
		--方法二:
		set serveroutput on;
		begin
			-- call the procedure
			addSal(eno => 7839);
			commit;
		end;
```

#### 2.存储函数

函数为一命名的存储程序,可带参数,并返回一计算值.函数说明要指定函数名,参数类型及结果值的类型,必须有一个return子句.

语法：

```plsql
		create or replace function 函数名(Name in type,Name out type) 
		return 数据类型
		is/as 结果变量 数据类型;
		begin
			...
			return(结果变量);
		end [函数名];
```

调用：

```plsql
		declare
			v_sal number;
		begin
			v_sal := queryEmpSalary(7839);
			dbms_output.put_line('salary is:'||v_sal);
		end;
		/
```

存储过程和存储函数的区别：函数可以有一个返回值,过程没有返回值；但过程和函数都可以通过out指定一个或多个输出参数。

使用存储过程/存储函数的时机：若只有一个返回值,用存储函数;否则,就用存储过程。

#### 3.包和包体

```plsql
	--查询某个部门中所有员工的所有信息
	--申明包结构
	create or replace package mypackage 
	as
		type empcursor is ref cursor;
		procedure queryEmpList(dno in number, empList out empcursor);
	end mypackage;
	--创建包体
	create or replace package body mypackage
	as
		procedure queryEmpList(dno in number,empList out empcursor)
		as
		begin
			open empList for select * from emp where deptno=dno;
		end queryEmpList;
	end mypackage;
```

### 三.触发器

#### 1.基本概念

触发器是一个与表相关联的,存储的pl/sql程序.每当一个特定的数据操作语句(insert,update,delete)在指定的表上发出时,oracle自动地执行触发器中定义的语句序列.

触发器作用：

- 数据确认：例:员工涨后的工资不能少于涨前的工资
- 实施复杂的安全性检查：例:禁止在非工作时间插入新员工
- 做审计,跟踪表上所做的数据操作等
- 数据的备份和同步

触发器的类型：

- 语句级触发器：在指定的操作之前或之后执行一次,不管这条语句影响了多少行。
- 行级触发器:(for each row)：触发语句作用的每一条记录都被触发.在行级触发器中使用old和new伪记录变量,识别值的状态。

语法：

```plsql
		create [or replace] trigger 触发器名
		{before|after}
		{delete|insert|update[of 列名]}
		on 表名
		[for each row[when(条件)]]
		declare
			...
		begin
			plsql块
		end 触发器名
```

在触发器中触发语句与伪记录变量的值:

```sql
	触发语句			:old					:new
	insert				所有字段都是空(null)	将要插入的数据
	update				更新以前该行的值		更新后的值
	delete				删除以前该行的值		所有字段都是空(null)
```

示例：

```plsql
	create trigger firsttrigger
	after insert
	on emp
	declare
	begin
	  dbms_output.put_line('成功插入新员工');
	end;
	/

	insert into emp(empno,ename,sal,deptno) values(1001,'Tom',3000,10);
```

审计：强制审计、标准审计(配置)、基于值的审计、细粒度审计、管理员审计。

#### 2.使用序列,触发器来模拟mysql中自增效果

2.1 创建序列

```plsql
		--创建表
		create table testuser
		(
			id number(6) not null ,
			name varchar2(30) not null primary key		--字符串可以作为主键
		);
		--创建序列
		create sequence user_seq 
		increment by 1
		start with 1
		minvalue 1
		maxvalue 9999999999999
		nocache order;
```

2.2 创建自增的触发器

```plsql
		create or replace trigger user_trigger
		before insert
		on testuser
		for each row
		begin
			select user_seq.nextval into :new.id from dual;
		end;
		/
```

2.3 测试效果

```plsql
		insert into testuser(name) values('aa');
		insert into testuser(name) values('bb');
```

