## 知识点

![image-20210220230023943](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210220230023943.png)

<!-- more -->

MySQL索引原理知识点如上思维导图所示，索引类型、索引原理、索引分析和优化、查询优化等。

## 一.索引类型

索引可以提升查询速度,会影响where查询,以及order by排序。MySQL索引类型如下：

- 从索引存储结构划分：B Tree索引、Hash索引、FULLTEXT全文索引、R Tree索引；
- 从应用层次划分：普通索引、唯一索引、主键索引、复合索引
- 从索引键值类型划分：主键索引、辅助索引（二级索引）
- 从数据存储和索引键值逻辑关系划分：聚集索引（聚簇索引）、非聚集索引（非聚簇索引）

### 1.普通索引

**最基本的索引类型，基于普通字段建立的索引，没有任何限制**。创建普通索引方法如下：

- `create index <索引的名字> on tablename (fieldname);`
- `alter table tablename add index [索引的名字] (fieldname);`
- `create table tablename ([...]，index [索引的名字] (fieldname) );`

### 2.唯一索引

**索引字段值必须唯一，但允许有空值。在创建或修改表时追加唯一约束，会自动创建对应的唯一索引**。

创建唯一索引的方法如下：

- `CREATE UNIQUE INDEX <索引的名字> ON tablename (字段名);`
- `ALTER TABLE tablename ADD UNIQUE INDEX [索引的名字] (字段名);`
- `CREATE TABLE tablename ( [...], UNIQUE [索引的名字] (字段名) ;`

### 3.主键索引

它是**一种特殊的唯一索引，不允许有空值。在创建或修改表时追加主键约束即可，每个表只能有一个主键**。

创建主键索引的方法如下：

- `CREATE TABLE tablename ( [...], PRIMARY KEY (字段名) );`
- `ALTER TABLE tablename ADD PRIMARY KEY (字段名);`

### 4.复合索引

用户可以**在多个列上建立索引，这种索引叫做组复合索引**（组合索引）。复合索引可以代替多个单一索引，相比多个单一索引复合索引所需的开销更小。

索引同时有两个概念叫做窄索引和宽索引，**窄索引是指索引列为1-2列的索引，宽索引也就是索引列超过2列的索引**。

创建组合索引的方法如下：

- `CREATE INDEX <索引的名字> ON tablename (字段名1，字段名2...);`
- `ALTER TABLE tablename ADD INDEX [索引的名字] (字段名1，字段名2...);`
- `CREATE TABLE tablename ( [...], INDEX [索引的名字] (字段名1，字段名2...) );`

复合索引使用注意事项：

- 何时使用复合索引，要**根据where条件建索引**，注意不要过多使用索引，过多使用会对更新操作效率有很大影响。
- 如果表已经建立了(col1，col2)，就没有必要再单独建立（col1）；如果现在有(col1)索引，如果查
    询需要col1和col2条件，可以**建立(col1,col2)复合索引**，对于查询有一定提高。

### 5.全文索引

**对于大量的文本数据检索，使用全文索引，查询速度会比like快很多倍**。从MySQL 5.6开始MyISAM和InnoDB存储引擎均支持。创建全文索引的方法如下：

- `CREATE FULLTEXT INDEX <索引的名字> ON tablename (字段名);`
- `ALTER TABLE tablename ADD FULLTEXT [索引的名字] (字段名);`
- `CREATE TABLE tablename ( [...], FULLTEXT KEY [索引的名字] (字段名) ;`

**全文索引有自己的语法格式，使用 `match` 和 `against` 关键字**，比如：

```sql
select * from user
where match(name) against('aaa');
```

**全文索引使用注意事项**：

- 全文索引必须在字符串、文本字段上建立；
- 全文索引字段值必须在最小字符和最大字符之间的才会有效。（innodb：3-84；myisam：4-84）；
- 全文索引字段值要进行切词处理，按syntax字符进行切割，例如b+aaa，切分成b和aaa；
- 全文索引匹配查询，默认使用的是等值匹配，例如a匹配a，不会匹配ab,ac。如果想匹配可以在布
    尔模式下搜索a*。

```sql
select * from user
where match(name) against('a*' in boolean mode);
```

## 二.索引原理

### 1.二分查找法

二分查找法也叫作折半查找法，它是在**有序数组中查找指定数据的搜索算法**。它的**优点是等值查询、范围查询性能优秀，缺点是更新数据、新增数据、删除数据维护成本高**。

1. 首先定位left和right两个指针；
2. 计算(left+right)/2；
3. 判断除2后索引位置值与目标值的大小比对；
4. 索引位置值大于目标值就-1，right移动；如果小于目标值就+1，left移动。

### 2.Hash结构

Hash结构由Hash表来实现的，是根据**键值 <key,value> 存储数据的结构**；Hash索引可以方便的提供等值查询，对于范围查询就需要全表扫描；Hash结构在MySQL中主要应用在Memory原生的Hash索引 、InnoDB 自适应哈希索引。

![image-20210220230610346](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210220230610346.png)

InnoDB自适应哈希索引是为了提升查询效率，InnoDB存储引擎会监控表上各个索引页的查询，**当InnoDB注意到某些索引值访问非常频繁时，会在内存中基于B+Tree索引再创建一个哈希索引，使得内存中的 B+Tree 索引具备哈希索引的功能**，即能够快速定值访问频繁访问的索引页。

InnoDB自适应哈希索引：在使用Hash索引访问时，一次性查找就能定位数据，等值查询效率要优于B+Tree。

InnoDB自适应哈希索引的功能，用户只能选择开启或关闭功能，无法进行人工干涉。

```sql
show engine innodb status \G;
show variables like '%innodb_adaptive%';
set global innodb_adaptive_hash_index=0;  # 关闭自适应hash索引; 1为开启
```

### 3.B+Tree结构

MySQL数据库索引采用的是B+Tree结构。下面分别介绍下B-Tree结构和B+Tree结构。

B-Tree结构：

- **索引值和data数据分布在整棵树结构**中；
- 每个节点可以**存放多个索引值及对应的data数据**；
- 树节点中的多个**索引值从左到右升序排列**。

![image-20210220231602022](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210220231602022.png)

B树的搜索：从根节点开始，对节点内的索引值序列采用二分法查找，如果命中就结束查找。没有命中会进入子节点重复查找过程，直到所对应的的节点指针为空，或已经是叶子节点了才结束。

B+Tree结构：

- **非叶子节点不存储data数据，只存储索引值**，这样便于存储更多的索引值；
- **叶子节点包含了所有的索引值和data数据**；
- **叶子节点用指针连接，提高区间的访问性能**。

![image-20210220232020701](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210220232020701.png)

B+树进行范围查找时，只需要查找定位两个节点的索引值，然后**利用叶子节点的指针进行遍历**即可。而B树需要遍历范围内所有的节点和数据，显然B+Tree效率高。

### 4.聚簇索引和辅助索引

- 聚簇索引和非聚簇索引：B+Tree的叶子节点存放**主键索引值和行记录**就属于**聚簇索引**；如果索引值和行记录分开存放就属于非聚簇索引。

- 主键索引和辅助索引：B+Tree的叶子节点存放的是**主键字段值就属于主键索引**；如果存放的是非主键值就属于辅助索引（二级索引）。

在InnoDB引擎中，**主键索引采用的就是聚簇索引结构存储**。

聚簇索引（聚集索引）：InnoDB的聚簇索引就是按照主键顺序构建 B+Tree结构。**B+Tree的叶子节点就是行记录，行记录和主键值紧凑地存储在一起**。 这也意味着 InnoDB 的主键索引就是数据表本身，它按主键顺序存放了整张表的数据，**占用的空间就是整个表数据量的大小**。通常说的主键索引就是聚集索引。

InnoDB表必须要有聚簇索引：

1. 若表定义了主键，主键索引就是聚集索引；
2. 若没有定义主键，第一个非空unique作为聚集索引；
3. 都没有则会建一个隐藏的row-id作为聚集索引

辅助索引：也叫作二级索引，是根据索引列构建 B+Tree结构。但在 B+Tree 的**叶子节点中只存了索引列和主键的信息**。二级索引占用的空间会比聚簇索引小很多， 通常创建辅助索引就是为了提升查询效率。**一个表InnoDB只能创建一个聚簇索引，但可以创建多个辅助索引**。

![image-20210220233318402](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210220233318402.png)

非聚簇索引：与InnoDB表存储不同，MyISAM数据表的**索引文件和数据文件是分开的**，被称为非聚簇索引结构。

![image-20210220233405224](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210220233405224.png)

## 三.索引分析与优化

### 1.explain命令

MySQL提供的explain命令可以对select语句进行分析，并输出select执行的详细信息，可供开发人员针对性优化。如

```sql
EXPLAIN SELECT * from user WHERE id < 3;
```

![image-20210220234023906](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210220234023906.png)

下面一一为上图中显示的信息进行说明：

#### 1.1 select_type

select_type表示**查询的类型**。常见的值如下：

- SIMPLE ： 表示查询语句不包含子查询或union；
- PRIMARY：表示此查询是最外层的查询；
- UNION：表示此查询是UNION的第二个或后续的查询；
- DEPENDENT UNION：UNION中的第二个或后续的查询语句，使用了外面查询结果；
- UNION RESULT：UNION的结果；
- SUBQUERY：SELECT子查询语句；
- DEPENDENT SUBQUERY：SELECT子查询语句依赖外层查询的结果。

如下面例子：

```sql
explain select * from user where id < 3 \G;  //SIMPLE
explain select * from user where id=1 union select * from user where id=2 \G;  //PRIMARY & UNION & UNION RESULT
explain select * from user where id=(select max(id) from user) \G;  //PRIMARY & SUBQUERY
explain select * from user u1 where id=(select max(u2.id) from user u2 where u1.age=18) \G;  //PRIMARY & DEPENDENT SUBQUERY
```

#### 1.2 type :star:

type表示**存储引擎查询数据采用的方式**。可以判断出查询是全表扫描还是基于索引的部分扫描。常用属性值如下，从上至下效率依次增强：

- ALL：表示**全表扫描**，性能最差。
- index：表示**基于索引的全表扫描**，先扫描索引再扫描全表数据。
- range：表示**使用索引范围查询**。使用>、>=、<、<=、in等等。
- ref：表示使用**非唯一索引进行单值查询**。
- eq_ref：一般情况下出现在**多表join查询**，表示前面表的每一个记录，都只能匹配后面表的一行结果。
- const：表示**使用主键或唯一索引**做等值查询，常量查询。
- NULL：表示不用访问表，速度最快。

例子如下：

```sql
explain select * from user where age=18 \G;  //ALL(有多条结果记录)
explain select * from user where age=18 order by id \G;  //index
explain select * from user where age=22 \G;  //ref(只有一条结果记录)
explain select * from user where id>1 \G;  //range
explain select * from user where id=1 \G;  //const
explain select now() \G;  //NULL
explain select * from user u,score s where u.id=s.user_id \G;  //s.ALL & u.eq_ref
```

#### 1.3 Extra :star:

Extra表示很多额外的信息。各种操作会在Extra提示相关信息，常见几种如下：
- Using where：表示查询需要**通过索引回表查询**数据。
- Using index：表示查询需要**通过索引**，索引就可以满足所需数据。
- Using filesort：表示查询出来的结果需要**额外排序**，数据量小在内存，大的话在磁盘，因此有Using filesort建议优化。
- Using temprorary：查询使用到了**临时表**，一般出现于**去重、分组**等操作。

例子如下：

```sql
explain select * from user where age>18 \G;  //Using where
explain select age from user where age>18 \G;  //Using where; Using index
explain select age from user order by age \G;  //Using index
explain select * from user order by age \G;  //Using filesort
drop index age_1 on user;
explain select distinct(age) from user \G;  //Using temporary
explain select * from user group by age \G;  //Using temporary; Using filesort
```

#### 1.4 key :star:

key表示查询时真正使用到的索引，显示的是索引名称。

#### 1.5 rows :star:

MySQL查询优化器会根据统计信息，估算SQL要查询到结果需要扫描多少行记录。

#### 1.6 key_len :star:

key_len 表示**查询使用索引的字节数量**。可以判断是否全部使用了组合索引，key_len的计算规则如下：

```java
字符串类型：
	- 字符串长度跟字符集有关：latin1=1、gbk=2、utf8=3、utf8mb4=4
    - char(n)：n*字符集长度
    - varchar(n)：n * 字符集长度 + 2字节
数值类型：
    - TINYINT：1个字节
    - SMALLINT：2个字节
    - MEDIUMINT：3个字节
    - INT、FLOAT：4个字节
    - BIGINT、DOUBLE：8个字节
时间类型：
    - DATE：3个字节
    - TIMESTAMP：4个字节
    - DATETIME：8个字节
字段属性：
	- NULL属性占用1个字节，如果一个字段设置了NOT NULL，则没有此项
```

例子如下：

```sql
desc user;
show create table user;
explain select * from user where age=22 \G;  //5 int(11) & NULL  4+1
explain select * from user where name='a' \G;  //63 varchar(20) & NULL  20*3+2+1
```

#### 1.7 possible_keys

possible_keys 表示**查询时能够使用到的索引**。注意并不一定会真正使用，显示的是索引名称。

### 2.回表查询

回表查询：通常情况下，需要扫码两遍索引树。先通过辅助索引定位主键值，然后再通过聚簇索引定位行记录；

聚簇索引的叶子节点存储**行记录**，辅助索引的叶子节点存储的是主键值和**索引字段值**；explain的输出结果Extra字段为 `Using where`时，回表查询。

### 3.覆盖索引

覆盖索引：只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快；

explain的输出结果Extra字段为 `Using index`时，能够触发索引覆盖；实现索引覆盖最常见的方法就是：**将被查询的字段，建立到组合索引**。

```sql
create index name_age on user(name,age);
explain select id,name,age from user where name='a' \G;  //Using index
```

### 4.最左前缀原则

复合索引使用时遵循最左前缀原则：**查询中使用到最左边的列，那么查询就会使用到索引，如果从索引的第二列开始查找，索引将失效**。

![image-20210221000953291](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210221000953291.png)

```sql
create index age_name on user(age,name);
show index from user;
explain select id from user where age=18 and name='a' \G;  //Using index  age_name
explain select id from user where name='a' and age=18 \G;  //Using index  age_name
-- 查询优化器会采用优化策略得到最优的执行计划，会把 age 和 name的条件顺序进进行优化
```

### 5.索引与排序

MySQL查询支持 `filesort`和 `index`两种方式的排序，filesort是先把结果查出，然后在缓存或磁盘进行排序操作，效率较低。使用index是指利用索引自动实现排序，不需另做排序操作，效率会比较高。

filesort有两种排序算法：双路排序和单路排序。
- 双路排序：需要两次磁盘扫描读取，最终得到用户数据。第一次将排序字段读取出来，然后排序；第二次去读取其他字段数据。
- 单路排序：从磁盘查询所需的所有列数据，然后在内存排序将结果返回。如果查询数据超出缓存 sort_buffer，会导致多次磁盘读取操作，并创建临时表，最后产生了多次IO，反而会增加负担。（**少使用select *；增加`sort_buffer_size`容量和 `max_length_for_sort_data`容量**）
  ```sql
  show variables like '%sort_buffer%';
  ```

Extra属性显示Using filesort，表示使用了**filesort排序方式，需要优化**。会使用filesort方式的排序：
```sql
-- 对索引列同时使用了ASC和DESC
  explain select id from user order by age asc,name desc;   //对应(age,name)索引

-- WHERE子句和ORDER BY子句满足最左前缀，但where子句使用了范围查询（例如>、<、in等）
  explain select id from user where age>10 order by name;   //对应(age,name)索引

-- ORDER BY或者WHERE+ORDER BY索引列没有满足索引最左前列
  explain select id from user order by name;   //对应(age,name)索引

-- 使用了不同的索引，MySQL每次只采用一个索引，ORDER BY涉及了两个索引
  explain select id from user order by name,age; //对应(name)、(age)两个索引

-- WHERE子句与ORDER BY子句，使用了不同的索引
  explain select id from user where name='tom' order by age; //对应(name)、(age)索引

-- WHERE子句或者ORDER BY子句中索引列使用了表达式，包括函数表达式
  explain select id from user order by abs(age);  //对应(age)索引
```

Extra属性显示**Using index时，表示覆盖索引**，也表示所有操作在索引上完成，也可以使用index排序方式，建议大家尽可能采用覆盖索引。会使用index方式的排序：

```sql
-- ORDER BY 子句索引列组合满足索引最左前列
  explain select id from user order by id; //对应(id)、(id,name)索引有效

-- WHERE子句+ORDER BY子句索引列组合满足索引最左前列
  explain select id from user where age=18 order by name; //对应(age,name)索引
```

### 6.like查询、null查询

MySQL在使用like模糊查询时，索引能不能起作用？MySQL在使用Like模糊查询时，索引是可以被使用的，**只有把 `%`字符写在后面才会使用到索引**。

```sql
create index name_1 on user(name);
explain select * from user where name like '%o%' \G;  //不起作用
explain select * from user where name like 'o%' \G;  //起作用
explain select * from user where name like '%o' \G;  //不起作用
```

如果MySQL表的某一列含有NULL值，那么包含该列的索引是否有效？**MySQL可以在含有NULL的列上使用索引，但NULL和其他数据还是有区别的，不建议列上允许为NULL**。

```sql
create index name_age on user(name,age);
show index from user;
explain select * from user where name is null \G;  //Using where; Using index
explain select * from user where name is null and age=18 \G;  //Using where; Using index
```

## 四.查询优化

### 1.慢查询定位

慢查询日志相关参数命令设置：

```sql
-- 查看数据库是否开启了慢查询日志：
  SHOW VARIABLES LIKE 'slow_query_log%'；

-- 开启慢查询日志命令：
  SET global slow_query_log = ON;

-- 修改日志文件名称：
  SET global slow_query_log_file = 'fishleap-slow.log';

-- 记录没有使用索引的查询SQL：前提是slow_query_log的值为ON
  SET global log_queries_not_using_indexes = ON;

-- 指定慢查询的阀值，单位秒：
  SET long_query_time = 10;
```

查看慢查询日志：

- 可以使用文件方式查看，直接使用文本编辑器打开slow.log日志即可：
    ![image-20210221224328752](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210221224328752.png)
    ```sql
    time：日志记录的时间
    User@Host：执行的用户及主机
    Query_time：执行的时间
    Lock_time：锁表时间
    Rows_sent：发送给请求方的记录数，结果数量
    Rows_examined：语句扫描的记录条数
    SET timestamp：语句执行的时间点
    select....：执行的具体的SQL语句
    ```
- 使用mysqldumpslow工具查看，先安装 perl 环境才能使用 mysql 提供的工具查看。
    ![image-20210221224627822](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210221224627822.png)
    ```sh
    - 在 MySQL bin目录下执行下面命令可以查看该使用格式：
    perl mysqldumpslow.pl --help
    
    - 运行命令查看慢查询日志信息：
    perl mysqldumpslow.pl -t 3 -s at D:\javaExt\javaApplet\mysql5.7\data\fishleap-slow.log
    ```

除了使用mysqldumpslow工具，也可以使用第三方分析工具，比如pt-query-digest、mysqlsla等。

### 2.慢查询优化

**慢查询和索引**

MySQL判断一条语句是否为**慢查询**语句，主要依据SQL语句的执行时间，它把当前语句的执行时间跟 `long_query_time` 参数做比较，如果语句的执行时间 > long_query_time，就会把这条执行语句记录到慢查询日志里面。

SQL语句是否使用了索引，可根据SQL语句执行过程中有没有用到表的索引，可通过 **explain命令**分析查看，检查结果中的 `key` 值，是否为NULL。

**查询是否使用索引，只是表示一个SQL语句的执行过程；而是否为慢查询，是由它执行的时间决定的**，也就是说是否使用了索引和是否是慢查询两者之间没有必然的联系。在使用索引时，不要只关注是否起作用，应该关心**索引是否减少了查询扫描的数据行数**；对于一个大表，不止要创建索引，还要考虑索引过滤性，过滤性好，执行速度才会快。

**提升索引的过滤性**

索引过滤性与索引字段、表的数据量、表设计结构都有关系。案例如下：

- 表：student
- 字段：id,name,sex,age
- 造数据： `insert into student (name,sex,age) select name,sex,age from student;`

```sql
-- SQL案例：
  select * from student where age=18 and name like '张%';（全表扫描）

-- 优化1：
  alter table student add index(name); //追加name索引
-- 优化2：
  alter table student add index(age,name); //追加age,name索引（index condition pushdown 优化的效果）
-- 优化3：为user表添加first_name虚拟列，以及联合索引(first_name,age)
  alter table student add first_name varchar(2) generated always as  (left(name, 1)), add index(first_name, age);

  explain select * from student where first_name='张' and age=18;
```

**慢查询原因总结**

- 全表扫描：explain分析type属性all；
- 全索引扫描：explain分析type属性index；
- 索引过滤性不好：靠索引字段选型、数据量和状态、表设计；
- 频繁的回表查询开销：尽量少用select *，使用覆盖索引。

### 3.分页查询优化

**一般性分页**

分页查询使用简单的 limit 子句就可以实现： `SELECT * FROM 表名 LIMIT [offset,] rows`
- 第一个参数指定第一个返回记录行的**偏移量**，注意从0开始；
- 第二个参数指定返回记录行的**最大数目**；
- 如果只给定一个参数，它表示返回最大的记录行数目；

这种分页查询机制，每次都会从数据库第一条记录开始扫描，越往后查询越慢，而且查询的数据越多，也会拖慢总查询速度。

**查看SQL语句的执行时间**

```sh
show variables like 'profiling';
set profiling = 1;	-- 开启profiling
show profiles;	-- 查看sql语句的执行时间
```

**分页中问题**

```sql
-- 如果偏移量固定，返回记录量对执行时间有什么影响？
  select * from user limit 10000,1;
  select * from user limit 10000,10;
```

在查询记录时，返回记录量低于100条，查询时间基本没有变化，差距不大。随着查询记录量越大，所花费的时间也会越来越多。

```sql
-- 如果查询偏移量变化，返回记录数固定对执行时间有什么影响？
  select * from user limit 1,100;
  select * from user limit 10,100;
```

在查询记录时，如果查询记录量相同，偏移量超过100后就开始随着偏移量增大，查询时间急剧的增加。

**分页优化方案**

1.利用覆盖索引优化

```sql
select * from user limit 10000,100;
select id from user limit 10000,100;
```

2.利用子查询优化

```sql
-- 利用子查询优化（使用了id做主键比较(id>=)，并且子查询使用了覆盖索引进行优化）
select * from user limit 10000,100;
select * from user where id>= (select id from user limit 10000,1) limit 100;
```

## 五.高质量SQL建议

查询：

1. 查询SQL尽量使用select具体字段；
2. 查询结果只有一条记录时建议使用 limit 1；
3. 插入数据过多，考虑批量插入（foreach标签）；
4. 字段过多时慎用 distinct 关键字；
5. 删除冗余和重复索引；
6. 删除/修改数据量过大建议分批进行删除；

where子句：

1. 避免where子句中使用 or 来连接条件（union all）；
2. 优化like语句（%放关键字后面），若不走索引可以使用覆盖索引；
3. 避免返回多余的行（条件唯一）；
4. 避免在索引列上使用内置函数；
5. 避免对字段进行表达式操作；
6. 避免使用 != 或 <>（考虑分两条sql写）；
7. 考虑在 where 及 order by 涉及的列上建立索引；
8. 考虑使用默认值代替null；

limit分页：

1. 返回上次查询的最大记录（where id > 10000） limit 10；

表连接：

1. 都满足SQL需求前提下，优先使用inner join，left join左边数据尽量小；

联合索引：

1. 遵循最左匹配原则；



## 推荐链接

- [后端程序员必备：书写高质量SQL的30条建议](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486461&idx=1&sn=60a22279196d084cc398936fe3b37772&chksm=cea24436f9d5cd20a4fa0e907590f3e700d7378b3f608d7b33bb52cfb96f503b7ccb65a1deed&mpshare=1&srcid=&sharer_sharetime=1585317374974&sharer_shareid=8f8f5e325299616b2c29319c72cb6590&from=timeline&scene=2&subscene=1&clicktime=1585321617&enterid=1585321617&key=794d660217fd628feceb36dffc0a1fd8440f1856f2a849a41f3953a8ca13c234ebc12e1412627fcfebf26c4575fc23c289dca7a2fb1596e7cbbdcc024b40c9af21ed8e194d8867462f604b98a3fdde9e&ascene=14&uin=MTU0ODQ4ODg4MQ%3D%3D&devicetype=Windows+10&version=62080079&lang=zh_CN&exportkey=AULCVbziX381bSGrXIFU6OQ%3D&pass_ticket=9bkjJ%2FBGkqHpaG3mhEOBosW%2FAlDjDBYjTfstVeDnbMAABrwGjyXcOsozcDcJlcjO#)
- [MySQL索引原理及慢查询优化](https://tech.meituan.com/2014/06/30/mysql-index.html)
- [Mysql 慢查询优化实践](https://juejin.cn/post/6844903769356894222)



