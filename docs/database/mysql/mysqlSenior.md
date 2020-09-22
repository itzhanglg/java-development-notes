## 一.前言

基础：

- DDL、DML、DQL、TCL
- 主键、外键、非空、唯一等约束
- 索引、事务概念和基本使用

高级：

- 架构原理和存储机制
- 索引存储机制和工作原理
- 事务和锁工作原理
- 集群架构及相关原理
- 海量数据处理实战
- 第三方工具实战

MySQL起源和分支：

发展：

- 96：开发MySQL1.0
- 99-20：MySQL AB公司，引入BDB引擎
- 01：MySQL集成MyISAM和InnoDB两种主力引擎
- 05：MySQL5.0发布，加入游标、存储过程和触发器
- 10：MySQL5.5发布，默认引擎更换为InnoDB
- 13：MySQL5.6发布，提供全文索引
- 15：MySQL5.7发布
- 16：MySQL8.0发布

MySQL应用架构演变：

用户请求--》（nginx+tomcat）应用层--》（分布式SOA+微服务）服务层--》（数据存储能力）存储层

单机单库--》主从架构--》分库分表--》云数据库（云计算）（saas存储作为服务，提供对外接口，节约成本）

## 二.MySQL架构原理

### MySQL体系架构

网络连接层（JDBC等）

核心服务层（连接池（线程池）、SQL接口、解析器（解析sql，检查校验语法）、查询优化器（生成执行计划）、缓存（提升查询结果的响应））

存储引擎层（MyISAM、InnoDB，可插拔（插件式），针对表指定存储引擎，不是针对于库）

系统文件层（日志、数据、配置、pid文件、socket文件等）

- **日志文件**
  - 错误日志：show variables like '%log_error%';
  - 通用查询日志：show variables like '%general%';
  - 二进制日志：记录对数据库执行的更改操作，用于数据库恢复和主从复制；log_bin、binlog、show binary logs
  - 慢查询日志：所有执行时间超时的查询sql，默认10s；slow_query、long_query_time

- **配置文件**：my.cnf、my.ini
- **数据文件**：

  - db.opt文件：字符集和校验规则
  - frm文件：表相关的元数据（meta）信息，表结构
  - MYD文件：存储MyISAM表的数据
  - MYI文件：存储MyISAM表的索引
  - ibd文件和IBDATA文件：存储InnoDB的数据（包括索引）
  - ibdata1文件：undo log等
  - ib_logfile0、ib_logfile1文件：Redo log日志文件

- pid文件：进程id
- socket文件：socket通讯

show variables like '%datadir%';

### MySQL运行机制

客户端--》查询缓存

查询缓存--》解析器--》解析树--》预处理器--》新解析树

新解析树--》查询优化器--》执行计划

执行计划--》查询执行引擎--》存储引擎

1.建立连接（Connectors&Connection Pool）：通过客户端/服务器通信协议与MySQL建立连接。MySQL客户端与服务端的通信方式是"半双工"。对于每一个MySQL连接，都有一个线程状态来标识这个连接正在做什么。

通讯机制：

- 全双工：能同时发送和接受数据
- 半双工：某一时刻，要么发送数据，要么接受数据，不能同时
- 单工：只能发送数据或接受数据

线程状态：show processlist;  // 查看用户正在运行的线程信息，root用户能查看所有线程，其它用户只能看自己的。

- Id：
- User：
- Host：
- db：
- Command：
- Time：
- State：
- Info：

2.查询缓存（Cache&Buffer）：MySQL可优化查询的地方，若开启了查询缓存且在查询缓存过程中查询到完全相同的SQL语句，则将查询结果返回给客户端；否则会由解析器进行语法语义解析，生成“解析树”。

- 即时开启了查询缓存，下列SQL也不能缓存

  - 查询语句使用SQL_NO_CACHE
  - 查询的结果大于query_cache_limit设置
  - 查询中有一些不确定的参数，比如now()

- show variables like '%query_cache%';  //查看查询缓存是否启用，空间大小，限制等
- show status like 'Qcache%'; //查看更详细的缓存参数，可用缓存空间，缓存块，缓存多少等

3.解析器（Parser）：对客户端发送的SQL进行语法解析，生成“解析树”。预处理器根据规则进一步检查“解析树”是否合法，再次生成“解析树”。

4.查询优化器（Optimizer）：根据“解析树”生成最优的执行计划。优化策略分为两类：静态优化（编译时优化）、动态优化（运行时优化）。

- 等价变换策略
- 优化count、min、max等函数
- 提前终止查询
- in的优化

5.查询执行引擎负责执行 SQL 语句，此时查询执行引擎会根据 SQL 语句中表的存储引擎类型，以及对应的API接口与底层存储引擎缓存或者物理文件的交互，得到查询结果并返回给客户端。如果开启了查询缓存，先将查询结果做缓存操作；返回结果过多，采用增量模式返回。

#### 运行机制总结

客户端与服务器建立通讯，通讯模式采用的是“半双工模式”，可以通过 show processlist; 来查看客户端与服务端连接的状态信息，操作等；

客户端发送SQL查询语句，若开启了查询缓存并查询缓存中查看是否有完全相同的SQL语句，若存在则查询结果直接返回给客服端，若没有命中进入解析阶段。查询缓存默认是关闭的，若需要可以开启；

解析分为两个阶段，第一阶段解析器将SQL语法进行解析成“解析树”，第二阶段预处理器根据规则进一步检查“解析树”是否合法，再次生成“解析树”，例如表和数据列是否存在，名字和别名是否存在歧义等；

查询优化器根据“解析树”生成最优的执行计划，MySQL有很多的优化策略来生成最优的执行计划，例如等价变换策略，优化count、min、max，limit查询，in查询等；

查询执行引擎负责执行 SQL 语句，会根据SQL语句中表的存储引擎类型，以及对应的API接口与底层存储引擎缓存或物理文件的交互，得到查询结果并返回；若启动查询缓存，先将查询结果做缓存操作，返回结果过多，采用增量模式返回。

### MySQL存储引擎

MySQL存储引擎负责MySQL中的数据存储和提取，是与文件打交道的子系统。使用 show engines 命令查看当前数据库支持的引擎信息。

在5.5版本之前默认采用MyISAM存储引擎，从5.5开始采用InnoDB存储引擎。

- InnoDB：支持事务，具有提交，回滚和崩溃恢复能力，事务安全（增删改查）
- MyISAM：不支持事务和外键，访问速度快（查）

#### InnoDB和MyISAM对比

InnoDB和MyISAM是使用MySQL时最常用的两种引擎类型：

- 事务和外键：InnoDB支持事务和外键，MyISAM不支持事务和外键
- 锁机制：InnoDB支持行级锁，MyISAM支持表级锁
- 索引结构：InnoDB使用聚簇索引，索引和记录存储在一起，MyISAM使用非聚簇索引，索引和记录分开存储
- 存储文件：InnoDB表对应两个文件，一个.frm表结构文件，一个.ibd数据文件。InnoDB表最大支持64TB；MyISAM表对应三个文件，一个.frm表结构文件，一个MYD表数据文件，一个.MYI索引文件。从MySQL5.0开始默认限制是256TB

InnoDB具有很好的事务特性，行级锁对高并发有很好的适应能力，适用于数据更新较为频繁的场景，数据一致性要求较高，硬件设备内存较大，可以利用InnoDB较好的缓存能力来提高内存利用率，减少磁盘IO。

#### InnoDB存储结构

从MySQL 5.5版本开始默认使用InnoDB作为引擎，它擅长处理事务，具有自动崩溃恢复的特性。

InnoDB引擎主要分为**内存结构和磁盘结构**两大部分。

##### InnoDB内存结构

内存结构主要包括Buffer Pool、Change Buffer、Adaptive Hash Index和Log Buffer四大组件。

1.Buffer Pool：缓冲池，简称BP。BP以Page页为单位，**默认16K**，底层采用链表数据结构管理Page。(查操作)

- Page管理机制： show engine innodb status \G;  // 查看innoDB的Page管理机制Page根据状态分为三种类型：

    - free page：空闲page，未被使用
    - clean page：被使用page，数据没有被修改过
    - dirty page：脏页，被使用page，页中数据和磁盘数据不一致

- 三组page类型，InnoDB通过三种链表结构维护和管理：

    - free list：空闲缓冲区，管理free page
    - flush list：需要刷新到磁盘的缓冲区，管理dirty page
    - lru list：表示正在使用的缓冲区，管理clean page和dirty page，缓冲区以midpoint为基点，前面链表称为new列表区，存放经常访问的数据，占63%；后面的链表称为old列表区，存放使用较少数据，占37%。
- 改进型LRU算法维护
    普通LRU：末尾淘汰法，新数据从链表头部加入，释放空间时从末尾淘汰
    改性LRU：链表分为new和old两个部分，加入元素时并不是从表头插入，而是从中间midpoint位置插入，如果数据很快被访问，那么page就会向new列表头部移动，如果数据没有被访问，会逐步向old尾部移动，等待淘汰
- Buffer Pool配置参数
    show variables like '%innodb_page_size%'; //查看page页大小   select @@innodb_page_size/1024;
    show variables like '%innodb_old%'; //查看lru list中old列表参数
    show variables like '%innodb_buffer%'; //查看buffer pool参数
    建议：将innodb_buffer_pool_size设置为总内存大小的60%-80%，innodb_buffer_pool_instances可以设置为多个，这样可以避免缓存争夺。

2.Change Buffer：写缓冲区，简称CB。当更新一条记录时，该记录在BufferPool存在，直接在BufferPool修改，一次内存操作。如果该记录在BufferPool不存在（没有命中），会直接在ChangeBuffer进行一次内存操作，不用再去磁盘查询数据，避免一次磁盘IO。当下次查询记录时，会先进性磁盘读取，然后再从ChangeBuffer中读取信息合并，最终载入BufferPool中。

如果在索引设置唯一性，在进行修改时，InnoDB必须要做唯一性校验，因此必须查询磁盘，做一次IO操作。会直接将记录查询到BufferPool中，然后在缓冲池修改，不会在ChangeBuffer操作。

3.Adaptive Hash Index：自适应哈希索引，用于优化对BP数据的查询。

4.Log Buffer：日志缓冲区，用来保存要写入磁盘上log文件（Redo/Undo）的数据，日志缓冲区的内容定期刷新到磁盘log文件中。主要是用于记录InnoDB引擎日志，在DML操作时会产生Redo和Undo日志。LogBuffer空间满了，会自动写入磁盘。可以通过将innodb_log_buffer_size参数调大，减少磁盘IO频率。

show variables like '%innodb_log%';  // 查看log参数   select @@innodb_log_file_size/1024/1024
show variables like '%innodb_flush_log%';

innodb_flush_log_at_trx_commit参数控制日志刷新行为，默认为1。0 ： 每隔1秒写日志文件和刷盘操作（写日志文件LogBuffer-->OS cache，刷盘OScache-->磁盘文件），最多丢失1秒数据1：事务提交，立刻写日志文件和刷盘，数据不丢失，但是会频繁IO操作2：事务提交，立刻写日志文件，每隔1秒钟进行刷盘操作

##### InnoDB磁盘结构

InnoDB磁盘主要包含Tablespaces，InnoDB Data Dictionary，Doublewrite Buffer、Redo Log和Undo Logs。

表空间（Tablespaces）：用于存储表结构和数据。表空间又分为系统表空间、独立表空间、通用表空间、临时表空间、Undo表空间等多种类型；

- System Tablespace（系统表空间）：存储的数据和索引，被多个表共享   。该空间的数据文件通过参数
    innodb_data_file_path控制，（ibdata1）
    show variables like '%innodb_data_file_path%';

- File-Per-Table Tablespaces（独立表空间）：默认开启。（每张表有ibd文件），当innodb_file_per_table选项开启时，表将被创建于表空间中。否则，innodb将被创建于系统表空间中。
    show variables like '%innodb_file_per_table%';

- General Tablespaces（通用表空间）：通过create tablespace语法创建的共享表空间。
    use database;
    create table t1(c1 int primary key) tablespace ts1;

- Undo Tablespaces（撤销表空间）：撤销表空间由一个或多个包含Undo日志文件组成。InnoDB使用的undo表空间由innodb_undo_tablespaces配置选项控制，默认为0。0表示使用系统表空间ibdata1;大于0表示使用undo表空间undo_001、undo_002等。
    show variables like '%innodb_undo_tablespaces%';

- Temporary Tablespaces（临时表空间）：分为session temporary tablespaces 和global temporary tablespace两种。mysql服务器正常关闭或异常终止时，临时表空间将被移除，每次启动时会被重新创建。

数据字典（InnoDB Data Dictionary）：用于查找表、索引和表字段等对象的元数据。元数据物理上位于InnoDB系统表空间中。

双写缓冲区（Doublewrite Buffer）：位于系统表空间，是一个存储区域。在BufferPage的page页刷新到磁盘真正的位置前，会先将数据存在Doublewrite 缓冲区。在大多数情况下，默认情况下启用双写缓冲区，要禁用Doublewrite 缓冲区，可以将innodb_doublewrite设置为0。使用Doublewrite 缓冲区时建议将`innodb_flush_method` 设置为O_DIRECT。

- show variables like '%innodb_doublewrite%';
- show variables like '%innodb_flush_method%';  这个参数控制着innodb数据文件及redo log的打开、
    刷写模式。有三个值：fdatasync(默认)，O_DSYNC，O_DIRECT。

重做日志（Redo Log）：一种基于磁盘的数据结构，用于在崩溃恢复期间更正不完整事务写入的数据。MySQL以循环方式写入重做日志文件，记录InnoDB中所有对Buffer Pool修改的日志。默认情况下，重做日志在磁盘上由两个名为ib_logfile0和ib_logfile1的文件物理表示。

撤销日志（Undo Logs）：在事务开始之前保存的被修改数据的备份，用于例外情况时回滚事务。撤消日志存在于系统表空间、撤消表空间和临时表空间中。

##### 新版本结构演变

![image-20200812044727737](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200812044727737.png)

MySQL5.7版本：

- 将 Undo日志表空间从共享表空间 ibdata 文件中分离出来，可以在安装 MySQL 时由用户自行指定文件大小和数量。
- 增加了 temporary 临时表空间，里面存储着临时表或临时查询结果集的数据。
- Buffer Pool 大小可以动态修改，无需重启数据库实例。

MySQL8.0版本：

- 将InnoDB表的数据字典和Undo都从共享表空间ibdata中分离出来
- temporary 临时表空间也可以配置多个物理文件，而且均为 InnoDB 存储引擎并能创建索引
- 用户可以设置表空间，每个表空间对应多个物理文件，每个表空间可以给多个表使用，但一个表只能存储在一个表空间中
- 将Doublewrite Buffer从共享表空间ibdata中也分离出来了

#### InnoDB线程模型

show engine innodb status \G; 查看innodb引擎状态信息

IO Thread：在InnoDB中使用了大量的AIO（Async IO）来做读写处理。在InnoDB1.0版本之前共有4个IO Thread，分别是write，read，insert buffer和log thread，后来版本将read thread和write thread分别增大到了4个，一共有10个了。

- read thread ： 负责读取操作，将数据从磁盘加载到缓存page页。4个
- write thread：负责写操作，将缓存脏页刷新到磁盘。4个
- log thread：负责将日志缓冲区内容刷新到磁盘。1个
- insert buffer thread ：负责将写缓冲内容刷新到磁盘。1个

Purge Thread：事务提交之后，其使用的undo日志将不再需要。

- show variables like '%innodb_purge_threads%';

Page Cleaner Thread：将脏数据刷新到磁盘，脏数据刷盘后相应的redo log也就可以覆盖，即可以同步数据，又能达到redo log循环使用的目的。会调用write thread线程处理。

- show variables like '%innodb_page_cleaners%';

Master Thread：Master thread是InnoDB的主线程，负责调度其他各线程，优先级最高。作用是将缓冲池中的数据异步刷新到磁盘 ，保证数据的一致性。包含：脏页的刷新（page cleaner thread）、undo页回收（purge thread）、redo日志刷新（log thread）、合并写缓冲等。内部有两个主处理，分别是每隔1秒和10秒处理。

- show variables like '%innodb_max_dirty_pages_pct%';

#### InnoDB数据文件

##### InnoDB文件存储结构

InnoDB数据文件存储结构：分为一个ibd数据文件 -->Segment（段）-->Extent（区）-->Page（页）-->Row（行）。

- Tablespace：表空间，用于存储多个ibd数据文件，用于存储表的记录和索引。一个文件包含多个段。
- Segment：段，用于管理多个Extent，分为数据段（Leaf node segment）、索引段（Non-leaf node segment）、回滚段（Rollback segment）。
- Extent：区，一个区固定包含64个连续的页，大小为1M。
- Page：页，用于存储多个Row行记录，大小为16K。包含很多种页类型，比如数据页，undo页，系统页，事务数据页，大的BLOB对象页。
- Row：行，包含了记录的字段值，事务ID（Trx id）、滚动指针（Roll pointer）、字段指针（Field pointers）等信息。

Page是文件最基本的单位，无论何种类型的page，都是由page header，page trailer和page body组成。

##### InnoDB文件存储格式

先 use table 命令，再通过 SHOW TABLE STATUS 命令。一般情况下，如果row_format为REDUNDANT、COMPACT，文件格式为Antelope；如果row_format为DYNAMIC和COMPRESSED，文件格式为Barracuda。

也可以通过 information_schema 查看指定表的文件格式。``select * from information_schema.innodb_sys_tables;`

select * from information_schema.innodb_sys_tables where name='nacos_config/users';

![image-20200812055506991](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200812055506991.png)

**File文件格式**（File_Format）：目前InnoDB只支持两种文件格式：Antelope 和 Barracuda。

- Antelope: 最原始的InnoDB文件格式，它支持两种行格式：COMPACT和 REDUNDANT，MySQL 5.6及其以前版本默认格式为Antelope。
- Barracuda: 新的文件格式。它支持InnoDB的所有行格式，包括新的行格式：COMPRESSED 和 DYNAMIC。

通过 `innodb_file_format`  配置参数可以设置InnoDB文件格式，之前默认值为Antelope，5.7版本开始改为Barracuda。

**Row行格式**（Row_Format）：InnoDB存储引擎支持四种行格式：REDUNDANT、COMPACT、DYNAMIC和COMPRESSED。DYNAMIC和COMPRESSED新格式引入的功能有：数据压缩、增强型长列数据的页外存储和大索引前缀。

每个表的数据分成若干页来存储，每个页中采用B树结构存储；如果某些字段信息过长，无法存储在B树节点中，这时候会被单独分配空间，此时被称为溢出页，该字段被称为页外列。

在创建表和索引时，文件格式都被用于每个InnoDB表数据文件（其名称与*.ibd匹配）。修改文件格式的方法是重新创建表及其索引，最简单方法是对要修改的每个表使用以下命令：

>  ALTER TABLE 表名 ROW_FORMAT=格式类型;

#### Undo Log

##### Undo Log介绍

Undo：意为撤销或取消，以撤销操作为目的，返回指定某个状态的操作。

Undo Log：数据库事务开始之前，会将修改的记录存放到Undo日志里，当事务回滚时或数据库崩溃时，利用Undo日志，撤销未提交事务对数据库产生的影响。

Undo Log产生和销毁：Undo Log在事务开始前产生；事务在提交时，并不会立刻删除undolog，innodb会将该事务对应的undo log放入到删除列表中，后面会通过后台线程purge thread进行回收处理。Undo Log属于逻辑日志，记录一个变化过程。例如执行一个delete，undolog会记录一个insert；执行一个update，undolog会记录一个相反的update。

Undo Log存储：undo log采用段的方式管理和记录。在innodb数据文件中包含一种rollback segment回滚段，内部包含1024个undo log segment。可以通过下面一组参数来控制Undo log存储。

> show variables like '%innodb_undo%';

##### Undo Log作用

- 实现事务的原子性
- 实现多版本并发控制（MVCC）

事务A手动开启事务，执行更新操作，首先会把更新命中的数据备份到 Undo Buffer 中。事务B手动开启事务，执行查询操作，会读取 Undo 日志数据返回，进行快照读。

#### Redo Log和Binlog

##### Redo Log日志

介绍：

Redo：顾名思义就是重做。以恢复操作为目的，在数据库发生意外时重现操作。

Redo Log：指事务中修改的任何数据，将最新的数据备份存储的位置（Redo Log），被称为重做日志。

Redo Log 的生成和释放：随着事务操作的执行，就会生成Redo Log，在事务提交时会将产生Redo Log写入Log Buffer，并不是随着事务的提交就立刻写入磁盘文件。等事务操作的脏页写入到磁盘之后，Redo Log 的使命也就完成了，Redo Log占用的空间就可以重用（被覆盖写入）。

工作原理：

Redo Log 是为了实现事务的持久性而出现的产物。防止在发生故障的时间点，尚有脏页未写入表的 IBD 文件中，在重启 MySQL 服务的时候，根据 Redo Log 进行重做，从而达到事务的未入磁盘数据进行持久化这一特性。

写入机制：

Redo Log 文件内容是以顺序循环的方式写入文件，写满时则回溯到第一个文件，进行覆盖写。

write pos 和 checkpoint 之间还空着的部分，可以用来记录新的操作。如果 write pos 追上 checkpoint，表示写满，这时候不能再执行新的更新，得停下来先擦掉一些记录，把 checkpoint推进一下。

相关配置参数：

每个InnoDB存储引擎至少有1个重做日志文件组（group），每个文件组至少有2个重做日志文件，默认为ib_logfile0和ib_logfile1。可以通过下面一组参数控制Redo Log存储：

> show variables like '%innodb_log%';

Redo Buffer 持久化到 Redo Log 的策略，可通过 Innodb_flush_log_at_trx_commit 设置：Buffer Pool  --》 Log Buffer  --》 OS Cache  --》 日志文件

- 0：每秒提交 Redo buffer ->OS cache -> flush cache to disk，可能丢失一秒内的事务数据。由后台Master线程每隔 1秒执行一次操作。
- 1（默认值）：每次事务提交执行 Redo Buffer -> OS cache -> flush cache to disk，最安全，性能最差的方式。
- 2：每次事务提交执行 Redo Buffer -> OS cache，然后由后台Master线程再每隔1秒执行OS cache -> flush cache to disk 的操作。

一般建议选择取值2，因为 MySQL 挂了数据没有损失，整个服务器挂了才会损失1秒的事务提交数据。

##### Binlog日志

Binlog：日志是以事件形式记录，还包括语句所执行的消耗时间。

- 主从复制：在主库中开启Binlog功能，主库可以把Binlog传递给从库，从库可以拿到Binlog后实现数据恢复到主从数据一致性。
- 数据恢复：通过mysqlbinlog工具来恢复数据。

主从采用ROW文件记录模式。

Binlog文件结构：

MySQL的binlog文件中记录的是对数据库的各种修改操作，用来表示修改操作的数据结构是Log event。不同的修改操作对应的不同的log event。比较常用的log event有：Query event、Rowevent、Xid event等。binlog文件的内容就是各种Log event的集合。

Binlog文件中Log event结构如下图所示：事件开始的执行时间、指明该事件的类型、服务器的server ID、该事件的长度等。

Binlog写入机制：

- 根据记录模式和操作触发event事件生成log event（事件触发执行机制）。
- 将事务执行过程中产生log event写入缓冲区，每个事务线程都有一个缓冲区 Log Event保存在一个binlog_cache_mngr数据结构中，在该结构中有两个缓冲区，一个是 stmt_cache，用于存放不支持事务的信息；另一个是trx_cache，用于存放支持事务的信息。
- 事务在提交阶段会将产生的log event写入到外部binlog文件中。不同事务以串行方式将log event写入binlog文件中，所以一个事务包含的log event信息在 binlog 文件中是连续的，中间不会插入其他事务的log event。

Binlog文件操作：

- Binlog状态查看：`show variables like 'log_bin';`
- 开启Binlog功能：`set global log_bin=mysqllogbin;`
- 使用show binlog events命令

  - show binary logs; //等价于show master logs;
  - show master status;
  - show binlog events;
  - show binlog events in 'mysqlbinlog.000001';

- 使用mysqlbinlog命令

  - mysqlbinlog "文件名"
  - mysqlbinlog "文件名" > "test.sql"

- 使用binlog恢复数据

  - //按指定时间恢复 mysqlbinlog --start-datetime="2020-04-25 18:00:00" --stop-datetime="2020-04-26 00:00:00" mysqlbinlog.000002 | mysql -uroot -p1234
  - //按事件位置号恢复 mysqlbinlog --start-position=154 --stop-position=957 mysqlbinlog.000002| mysql -uroot -p1234
  - mysqldump：定期全部备份数据库数据。mysqlbinlog可以做增量备份和恢复操作

- 删除Binlog文件

  - purge binary logs to 'mysqlbinlog.000001';  //删除指定文件
  - purge binary logs before '2020-04-28 00:00:00';  //删除指定时间之前的文件
  - reset master;  //清除所有文件
  - 可以通过设置expire_logs_days参数来启动自动清理功能。默认值为0表示没启用。设置为1表示超出1天binlog文件会自动删除掉。 show variables like '%expire_logs_days%';

Redo Log和Binlog区别：

- Redo Log是属于InnoDB引擎功能，Binlog是属于MySQL Server自带功能，并且是以二进制文件记录。
- Redo Log属于物理日志，记录该数据页更新状态内容，Binlog是逻辑日志，记录更新过程。
- Redo Log日志是循环写，日志空间大小是固定，Binlog是追加写入，写完一个写下一个，不会覆盖使用。
- Redo Log作为服务器异常宕机后事务数据自动恢复使用，Binlog可以作为主从复制和数据恢复使用。Binlog没有自动crash-safe能力。

## 三.MySQL索引原理

### 1.索引类型

索引可以提升查询速度,会影响where查询,以及order by排序。MySQL索引类型如下：

- 从索引存储结构划分：B Tree索引、Hash索引、FULLTEXT全文索引、R Tree索引
- 从应用层次分：普通索引、唯一索引、主键索引、复合索引
- 从索引键值类型划分：主键索引、辅助索引（二级索引）
- 从数据存储和索引键值逻辑关系划分：聚集索引（聚簇索引）、非聚集索引（非聚簇索引）

#### 1.1 普通索引

最基本的索引类型，基于普通字段建立的索引，没有任何限制。创建普通索引方法如下：

- create index <索引的名字> on tablename (fieldname);
- alter table tablename add index [索引的名字] (fieldname);
- create table tablename ([...]，index [索引的名字] (fieldname) );

#### 1.2 唯一索引

索引字段值必须唯一，但允许有空值。在创建或修改表时追加唯一约束，会自动创建对应的唯一索引。

创建唯一索引的方法如下：

- CREATE UNIQUE INDEX <索引的名字> ON tablename (字段名);
- ALTER TABLE tablename ADD UNIQUE INDEX [索引的名字] (字段名);
- CREATE TABLE tablename ( [...], UNIQUE [索引的名字] (字段名) ;

#### 1.3 主键索引

它是一种特殊的唯一索引，不允许有空值。在创建或修改表时追加主键约束即可，每个表只能有一个主键。

创建主键索引的方法如下：

- CREATE TABLE tablename ( [...], PRIMARY KEY (字段名) );
- ALTER TABLE tablename ADD PRIMARY KEY (字段名);

#### 1.4 复合索引

用户可以在多个列上建立索引，这种索引叫做组复合索引（组合索引）。复合索引可以代替多个单一索引，相比多个单一索引复合索引所需的开销更小。

索引同时有两个概念叫做窄索引和宽索引，窄索引是指索引列为1-2列的索引，宽索引也就是索引列超过2列的索引。

创建组合索引的方法如下：

- CREATE INDEX <索引的名字> ON tablename (字段名1，字段名2...);
- ALTER TABLE tablename ADD INDEX [索引的名字] (字段名1，字段名2...);
- CREATE TABLE tablename ( [...], INDEX [索引的名字] (字段名1，字段名2...) );

复合索引使用注意事项：

- 何时使用复合索引，要根据where条件建索引，注意不要过多使用索引，过多使用会对更新操作效率有很大影响。
- 如果表已经建立了(col1，col2)，就没有必要再单独建立（col1）；如果现在有(col1)索引，如果查
    询需要col1和col2条件，可以建立(col1,col2)复合索引，对于查询有一定提高。

#### 1.5 全文索引

使用全文索引，查询速度会比like快很多倍。从MySQL 5.6开始MyISAM和InnoDB存储引擎均支持。创建全文索引的方法如下：

- CREATE FULLTEXT INDEX <索引的名字> ON tablename (字段名);
- ALTER TABLE tablename ADD FULLTEXT [索引的名字] (字段名);
- CREATE TABLE tablename ( [...], FULLTEXT KEY [索引的名字] (字段名) ;

全文索引有自己的语法格式，使用 `match` 和 `against` 关键字，比如：

```sql
select * from user
where match(name) against('aaa');
```

全文索引使用注意事项：

- 全文索引必须在字符串、文本字段上建立
- 全文索引字段值必须在最小字符和最大字符之间的才会有效。（innodb：3-84；myisam：4-84）
- 全文索引字段值要进行切词处理，按syntax字符进行切割，例如b+aaa，切分成b和aaa
- 全文索引匹配查询，默认使用的是等值匹配，例如a匹配a，不会匹配ab,ac。如果想匹配可以在布
    尔模式下搜索a*

```sql
select * from user
where match(name) against('a*' in boolean mode);
```

### 2.索引原理

#### 2.1 二分查找法





#### 2.2 Hash结构

```
show engine innodb status \G;
show variables like '%innodb_adaptive%';
set global innodb_adaptive_hash_index=0; // 关闭自适应hash索引; 1为开启
```





#### 2.3 B+Tree结构





#### 2.4 聚簇索引和辅助索引





### 3.索引分析与优化

### 4.查询优化



添加数据：

insert into slow(name) select name from slow;



### 隔离级别和锁关系

在Read Uncommited级别下,读操作不加S锁。

在Read Commited级别下,读操作需要加S锁,但是在查询语句执行完以后立刻释放S锁。

在Repeatable Read级别下,读操作需要加S锁,但是事务提交之前并不释放S锁,必须等事务结束才释放。

在Serializable级别下,会在Repeatable Read级别的基础上,添加一个范围锁,保证一个事务在两次查询时结果完全相同。



读未提交：一个事务开启后进行update，但未提交。另一个事务前后两次select结果不一致

读已提交：一个事务开启后进行update，已经提交。另一个事务前后两次select结果不一致

可重复读：一个事务开启后进行insert，另一个事务进行前后两次读取结果是一致的，但对新记录update后，再次select结果就不一致了（间隙锁）

可串行化：一个事务开启后进行select，另一个事务进行update后会进入阻塞；读读不会阻塞





数据库允许同一资源存在多个共享锁；但不允许同一资源存在多个排它锁。



查看死锁日志
通过show engine innodb status\G命令查看近期死锁日志信息。
使用方法：1、查看近期死锁日志信息；2、使用explain查看下SQL执行计划（全表查询，记录加锁）



每条数据变更(insert/update/delete)操作都伴随一条undo log的生成,并且回滚日志必须先于数据持久化到磁盘上

读数据：会首先从缓冲池中读取，如果缓冲池中没有，则从磁盘读取在放入缓冲池；
写数据：会首先写入缓冲池，缓冲池中的数据会定期同步到磁盘中；

既然redo log也需要存储，也涉及磁盘IO为啥还用它？

（1）redo log 的存储是顺序存储，而缓存同步是随机操作。

（2）缓存同步是以数据页为单位的，每次传输的数据大小大于redo log。





prepare：redolog写入log buffer，并fsync持久化到磁盘，在redolog事务中记录2PC的XID，在redolog事务打上prepare标识
commit：binlog写入log buffer，并fsync持久化到磁盘，在binlog事务中记录2PC的XID，同时在redolog事务打上commit标识
其中，prepare和commit阶段所提到的“事务”，都是指内部XA事务，即2PC



## 参考链接

MySQL事务日志：

- [超干货！为了让你彻底弄懂MySQL事务日志，我通宵肝出了这份图解！](https://mp.weixin.qq.com/s?src=11&timestamp=1597979478&ver=2535&signature=jn7Rp7vJJXCp62ai*dfmKFouYFFX1MWZTOGUjchXjXOuMkzgb4WrT8qGgpxZIJAP7ckEzzNx39JStG-s1xwHtAEwi*2ElQQQHeL1bWkUIk2adX0GyH1JoXfxdvbhieoO&new=1)
- [事务的基本概念，Mysql事务处理原理](https://mp.weixin.qq.com/s?src=11&timestamp=1597979478&ver=2535&signature=jn7Rp7vJJXCp62ai*dfmKFouYFFX1MWZTOGUjchXjXPmkcOjgcyjREpFDe916IvEiksdEygkhI7q4TKIGFWC07MHMjpChUR2CDQsWDevJXxot7Jr8*Z9wmXV1zBCxGtp&new=1)
- [Mysql事务实现原理](https://www.lagou.com/lgeduarticle/82740.html)





