## 知识点

![image-20210222014608387](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210222014608387.png)

<!-- more -->

## 一.MySQL架构体系

### 1.四层模型

MySQL Server架构自顶向下大致可以分网络连接层、服务层、存储引擎层和系统文件层。

![image-20210221235743343](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210221235743343.png)

**网络连接层**：客户端连接器（Client Connectors）：提供与MySQL服务器建立支持。

**服务层**：

- 系统管理和控制工具（Management Services & Utilities）：例如备份恢复、安全管理、集群管理等；
- 连接池（Connection Pool）：负责存储和管理客户端与数据库的连接，一个线程负责管理一个连接；
- SQL接口（SQL Interface）：接受客户端发送的各种SQL命令，并且返回用户需要查询的结果；
- 解析器（Parser）：将请求的SQL解析生成一个"解析树"，再根据一些MySQL规则进一步检查解析树是否合法；
- 查询优化器（Optimizer）：选取 --》投影 --》联接，将语法检查后的解析树转化成执行计划，然后与存储引擎交互；
- 缓存（Cache&Buffer）：缓存机制由一系列小缓存组成的。

**存储引擎层**：负责MySQL中数据的存储与提取，与底层系统文件进行交互；存储引擎是插件式的，服务器中的查询执行引擎通过接口与存储引擎进行通信，接口屏蔽了不同存储引擎之间的差异。

**系统文件层**：将数据库的数据和日志存储在文件系统之上，并完成与存储引擎的交互，是文件的物理存储层。

- 日志文件：错误日志、通用查询日志、二进制日志、慢查询日志、Undo Log、Redo Log、Relay Log；
- 数据文件：
    - db.opt文件：默认字符集和校验规则；
    - frm文件：表相关的元数据信息；
    - MYD文件：存放MyISAM表的数据；
    - MYI文件：存放MyISAM表的索引相关信息；
    - ibd文件和IBDATA文件：存放InnoDB的数据文件（包括索引）；
    - ibdata1文件：系统表空间数据文件，存储表元数据、Undo日志等；
    - ib_logfile0、ib_logfile1文件：Redo log日志文件。
- 配置文件：my.cnf、my.ini；
- pid文件：在 Unix/Linux 环境下的一个进程文件；
- socket文件：在 Unix/Linux 环境下客户端连接可以不通过，TCP/IP 网络而直接使用 Unix Socket 来连接 MySQL。

### 2.SQL运行机制

MySQL运行机制大概分为以下几步：建立连接、查询缓存、解析器、查询优化器及查询执行引擎。

![image-20210222000243453](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210222000243453.png)

**建立连接**（Connectors&Connection Pool）：客户端与服务器建立通讯，通讯模式采用的是“半双工模式”，可以通过 show processlist; 来查看客户端与服务端连接的线程状态信息，操作等；

**查询缓存**（Cache&Buffer）：客户端发送SQL查询语句，若开启了查询缓存并查询缓存中查看是否有完全相同的SQL语句，若存在则查询结果直接返回给客服端，若没有命中进入解析阶段。查询缓存默认是关闭的，若需要可以开启；

**解析器**（Parser）：解析分为两个阶段，第一阶段解析器将SQL语法进行解析成“解析树”，第二阶段预处理器根据规则进一步检查“解析树”是否合法，再次生成“解析树”，例如表和数据列是否存在，名字和别名是否存在歧义等；

**查询优化器**（Optimizer）：查询优化器根据“解析树”生成最优的执行计划，MySQL有很多的优化策略来生成最优的执行计划，例如等价变换策略，优化count、min、max，limit查询，in查询等；

**查询执行引擎**：查询执行引擎负责执行 SQL 语句，会根据SQL语句中表的存储引擎类型，以及对应的API接口与底层存储引擎缓存或物理文件的交互，得到查询结果并返回；若启动查询缓存，先将查询结果做缓存操作，返回结果过多，采用增量模式返回。

## 二.MySQL存储引擎

### 1.InnoDB和MyISAM对比

1. **事务和外键**：InnoDB支持事务和外键，MyISAM不支持事务和外键(MyISAM更利于查询)；
2. **锁机制**：InnoDB支持行级锁，MyISAM支持表级锁；
3. **索引结构**：InnoDB使用聚簇索引，索引和记录存储在一起，MyISAM使用非聚簇索引，索引和记录分开存储；
4. **存储文件**：InnoDB表对应两个文件，一个.frm表结构文件，一个.ibd数据文件。InnoDB表最大支持64TB；MyISAM表对应三个文件，一个.frm表结构文件，一个MYD表数据文件，一个.MYI索引文件，从MySQL5.0开始默认限制是256TB；
5. **并发存储能力**：InnoDB读写阻塞与隔离级别有关，采用多版本并发控制（MVCC）来；支持高并发；MyISAM使用表锁，写操作并发率低，读操作并不阻塞。

### 2.InnoDB存储结构

InnoDB引擎架构图，主要分为内存结构和磁盘结构两大部分。

![image-20210222001837270](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210222001837270.png)

#### 2.1 内存结构

内存结构主要包括Buffer Pool、Change Buffer、Adaptive Hash Index和Log Buffer四大组件：

- `Buffer Pool`：缓冲池，简称BP。BP以Page页为单位，默认16K，底层采用链表数据结构管理Page；
- `Change Buffer`：写缓冲区，简称CB。当更新一条记录时，该记录在BufferPool存在，直接在BufferPool修改，一次内存操作。如果该记录在BufferPool不存在（没有命中），会直接在ChangeBuffer进行一次内存操作，不用再去磁盘查询数据，避免一次磁盘IO。当下次查询记录时，会先进性磁盘读取，然后再从ChangeBuffer中读取信息合并，最终载入BufferPool中；
- `Adaptive Hash Index`：自适应哈希索引，用于优化对BP数据的查询；
- `Log Buffer`：日志缓冲区，用来保存要写入磁盘上log文件（Redo/Undo）的数据，日志缓冲区的内容定期刷新到磁盘log文件中。

#### 2.2 磁盘结构

InnoDB磁盘主要包含Tablespaces，InnoDB Data Dictionary，Doublewrite Buffer、Redo Log和Undo Logs。

- 表空间（Tablespaces）：用于存储表结构和数据。表空间又分为系统表空间、独立表空间、通用表空间、临时表空间、Undo表空间等多种类型；
- 数据字典（InnoDB Data Dictionary）：用于查找表、索引和表字段等对象的元数据。元数据物理上位于InnoDB系统表空间中；
- 双写缓冲区（Doublewrite Buffer）：位于系统表空间，是一个存储区域。在BufferPage的page页刷新到磁盘真正的位置前，会先将数据存在Doublewrite 缓冲区；
- 重做日志（Redo Log）：一种基于磁盘的数据结构，用于在崩溃恢复期间更正不完整事务写入的数据。MySQL以循环方式写入重做日志文件，记录InnoDB中所有对Buffer Pool修改的日志。默认情况下，重做日志在磁盘上由两个名为ib_logfile0和ib_logfile1的文件物理表示；
- 回滚日志（Undo Logs）：在事务开始之前保存的被修改数据的备份，用于例外情况时回滚事务。撤消日志存在于系统表空间、撤消表空间和临时表空间中。

#### 2.3 新版本结构演变

![image-20210222001339393](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210222001339393.png)

MySQL5.7版本：

- 将 Undo日志表空间从共享表空间 ibdata 文件中分离出来，可以在安装 MySQL 时由用户自行指定文件大小和数量。
- 增加了 temporary 临时表空间，里面存储着临时表或临时查询结果集的数据。
- Buffer Pool 大小可以动态修改，无需重启数据库实例。

MySQL8.0版本：

- 将InnoDB表的数据字典和Undo都从共享表空间ibdata中分离出来；
- temporary 临时表空间也可以配置多个物理文件，而且均为 InnoDB 存储引擎并能创建索引；
- 用户可以设置表空间，每个表空间对应多个物理文件，每个表空间可以给多个表使用，但一个表只能存储在一个表空间中；
- 将Doublewrite Buffer从共享表空间ibdata中也分离出来了。

### 3.InnoDB线程模型

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20210222002010930.png" alt="image-20210222002010930" style="zoom:50%;" />

InnoDB线程模型后台线程有IO Thread、Purge Thread、Page Cleaner Thread、和Master Thread。

- `IO Thread`：在InnoDB中使用了大量的AIO（Async IO）来做读写处理：
    - read thread ： 负责读取操作，将数据从磁盘加载到缓存page页。4个；
    - write thread：负责写操作，将缓存脏页刷新到磁盘。4个；
    - log thread：负责将日志缓冲区内容刷新到磁盘。1个；
    - insert buffer thread ：负责将写缓冲内容刷新到磁盘。1个。
- `Purge Thread`：事务提交之后，其使用的undo日志将不再需要；
- `Page Cleaner Thread`：将脏数据刷新到磁盘，脏数据刷盘后相应的redo log也就可以覆盖，即可以同步数据，又能达到redo log循环使用的目的。会调用write thread线程处理；
- `Master Thread`：Master thread是InnoDB的主线程，负责调度其他各线程，优先级最高。作用是将缓冲池中的数据异步刷新到磁盘 ，保证数据的一致性。包含：脏页的刷新（page cleaner thread）、undo页回收（purge thread）、redo日志刷新（log thread）、合并写缓冲等。内部有两个主处理，分别是每隔1秒和10秒处理。

### 4.InnoDB数据文件

InnoDB数据文件**存储结构**：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20210222002527036.png" alt="image-20210222002527036" style="zoom:50%;" />

分为一个ibd数据文件Tablespace（表空间） -->Segment（段）-->Extent（区）-->Page（页）-->Row（行）。

- 表空间：用于存储多个ibd数据文件，用于存储表的记录和索引。一个文件包含多个段；
- 段：用于管理多个Extent，分为数据段（Leaf node segment）、索引段（Non-leaf node segment）、回滚段（Rollback segment）；
- 区：一个区固定包含64个连续的页，大小为1M；
- 页：用于存储多个Row行记录，大小为16K；
- 行：包含了记录的字段值，事务ID（Trx id）、滚动指针（Roll pointer）、字段指针（Field pointers）等信息。

Page是文件最基本的单位，无论何种类型的page，都是由page header，page trailer和page body组成。

InnoDB文件**存储格式**：

- File文件格式（File_Format）：
    - Antelope: 最原始的InnoDB文件格式，它支持两种行格式：COMPACT和 REDUNDANT；
    - Barracuda: 新的文件格式。它支持InnoDB的所有行格式，包括新的行格式：COMPRESSED 和 DYNAMIC。
- Row行格式（Row_format）：InnoDB存储引擎支持四种行格式：REDUNDANT、COMPACT、DYNAMIC和COMPRESSED。

## 三.MySQL日志文件

MySQL日志系统：WAL(Write-Ahead Logging)，**先写日志，再写磁盘**。这里先对MySQL日志文件做下初步了解，后续文章会专门详细介绍。

### 1.Undo Log

概念：数据库事务开始之前，会将修改的记录存放到Undo日志里，当事务回滚时或数据库崩溃时，利用Undo日志，撤销未提交事务对数据库产生的影响。

Undo Log属于**逻辑日志，记录一个变化过程**。例如执行一个delete，undolog会记录一个insert；执行一个update，undolog会记录一个相反的update。Undo log采用**段的方式管理和记录**。

**实现事务的原子性**：事务处理过程中，如果出现了错误或者用户执行了 ROLLBACK 语句，MySQL 可以利用 Undo Log 中的备份将数据恢复到事务开始之前的状态。

**实现多版本并发控制**：事务未提交之前，Undo Log保存了未提交之前的版本数据，Undo Log 中的数据可作为数据旧版本快照供其他并发事务进行快照读。

### 2.Redo Log

概念：以恢复操作为目的，在数据库发生意外时重现操作；随着事务操作的执行，就会生成Redo Log，在事务提交时会将产生Redo Log写入Log Buffer，并不是随着事务的提交就立刻写入磁盘文件。

Redo Log 文件内容是以**顺序循环的方式写入文件**，写满时则回溯到第一个文件，进行覆盖写。

MySQL在执行更新语句的时候，在服务层进行语句的解析和执行，在引擎层进行数据的提取和存储；同时在服务层对binlog进行写入，在InnoDB内进行redo log的写入。

在对redo log写入时有两个阶段(prepare/commit))的提交，一是binlog写入之前prepare状态的写入，二是binlog写入之后commit状态的写入。单阶段提交会导致原先数据库的状态和被恢复后的数据库的状态不一致。两阶段的提交就是为了避免上述的问题，使得binlog和redo log中保存的信息是一致的。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20210222003611509.png" alt="image-20210222003611509" style="zoom:70%;" />

### 3.Binlog

Binlog以事件形式记录，还包括语句所执行的消耗时间。作用可以用来主从复制和数据恢复。

- 主从复制：在主库中开启Binlog功能，主库可以把Binlog传递给从库，从库可以拿到Binlog后实现数据恢复到主从数据一致性。
- 数据恢复：通过mysqlbinlog工具来恢复数据

文件结构：binlog文件中记录的是对数据库的各种修改操作，用来表示修改操作的数据结构是Log event。Log event结构包括 事件开始的执行时间、指明该事件的类型、服务器的server ID、该事件的长度等。

写入机制：

- 根据记录模式和操作触发event事件生成log event（事件触发执行机制）。
- 将事务执行过程中产生log event写入缓冲区，每个事务线程都有一个缓冲区 Log Event保存在一个binlog_cache_mngr数据结构中，在该结构中有两个缓冲区，一个是 stmt_cache，用于存放不支持事务的信息；另一个是trx_cache，用于存放支持事务的信息。
- 事务在提交阶段会将产生的log event写入到外部binlog文件中。不同事务以串行方式将log event写入binlog文件中，所以一个事务包含的log event信息在 binlog 文件中是连续的，中间不会插入其他事务的log event

### 4.Redolog和Binlog区别 :star:

- Redo Log是属于InnoDB引擎功能，Binlog是属于MySQL Server自带功能，并且是以二进制文件记录。
- Redo Log属于物理日志，记录该数据页更新状态内容，Binlog是逻辑日志，记录更新过程。
- Redo Log日志是循环写，日志空间大小是固定，Binlog是追加写入，写完一个写下一个，不会覆盖使用。
- Redo Log作为服务器异常宕机后事务数据自动恢复使用，Binlog可以作为主从复制和数据恢复使用。Binlog没有自动crash-safe能力。

### 5.Relay Log

Relay Log主要用于主从复制。

## 四.事务

事务就是要保证一组数据库操作，要么全部成功，要么全部失败；在MySQL中事务支持是在引擎层实现的。

### 1.ACID特性

- 原子性（Atomicity）：语句要么全部执行，要么全部不执行；主要基于 `undo log`日志实现；
- 持久性（Durability）：保证事务提交后不会因为宕机等原因导致数据丢失；主要基于 `redo log`日志实现；
- 隔离性（Isolation）：保证事务执行尽可能不受其他事务影响；InnoDB默认的隔离级别是RR，RR的实现主要基于**锁机制、数据的隐藏列、undo log和类next-key lock机制**；
- 一致性（Consistency）：事务追求的最终目标，一致性的实现既需要数据库层面的保障，也需要应用层面的保障；通过回滚，以及恢复，和在并发环境下的隔离做到一致性。

### 2.并发读写问题

MySQL并发读写问题主要有赃读、不可重复读和 幻读。

- 更新丢失：当两个或多个事务更新同一行记录，会产生更新丢失现象：回滚覆盖和提交覆盖；
- 赃读（dirty read）：一个事务读取到了另一个事务修改但未提交的数据；
- 不可重复读（non-repeatable read）：一个事务中多次读取同一行记录不一致，后面读取的跟前面读取的不一致；
- 幻读（phantom read）：在事务A中按照某个条件先后两次查询数据库，两次查询结果的行数不同。

**赃读和不可重复读**的区别：前者读到的是其他事务未提交的数据，后者读到的是其他事务已提交的数据；

**不可重复读与幻读**的区别：前者是数据变了（修改），后者是数据的行数变了（添加或删除）。

### 3.事务隔离级别

锁是数据库实现并发控制的基础，**事务隔离性是采用锁来实现，对相应操作加不同的锁，就可以防止其他事务同时对数据进行读写操作**；用户建议首先选择使用隔离级别，当选用的隔离级别不能解决并发问题或需求时，才有必要在开发中手动的设置锁。

MySQL事务隔离级别有读未提交、已提交读、可重复读和可串行化。

- 读未提交RU（read uncommitted）：一个事务还没提交时，它做的变更就能被别的事务看到。
- 已提交读RC（read committed）：一个事务提交之后，它做的变更才会被其他事务看到。可避免脏读；Oracle、SQLServer默认隔离级别为读已提交。
- 可重复读RR（repeatable read）：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。可避免脏读,不可重复读；MySQL默认隔离级别为可重复读。
- 可串行化（serializable）：对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。都可避免. 级别最高,效率最低。

**已提交读跟可重复读**在实现上最大的**差异**就在于：已提交读(RC)每次select都会生成一个**快照**，没有**间隙锁**；可重复读(RR)只有在第一次会生成一个快照（视图），有间隙锁。

### 4.事务演进

#### 4.1 演进

1.排队：序列化执行所有的事务单元，数据库某个时刻只处理一个事务操作。

2.排他锁：涉及到相同的数据项时，先进入的事务独占数据项以后，其他事务被阻塞，等待前面的事务释放锁。

3.读写锁：读写锁，可以让读和读并行，而读和写、写和读、写和写这几种之间还是要加排它锁。

4.MVCC：基于数据的隐藏列和回滚日志实现：行ID、事务ID、回滚指针。

#### 4.2 MVCC

多版本控制MVCC（Multi-Version Concurrency Control），也就是Copy on Write的思想。MVCC除了支持读和读并行，还支持读和写、写和读的并行，但为了保证一致性，写和写是无法并行的。

![image-20210222010217735](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210222010217735.png)

多版本并发控制支持RC和RR两种隔离级别，实现了读和写并行，写和写是无法并行的。在同一时刻，不同事务可以读取到不同版本的数据;

在 MVCC 并发控制中，读操作可以分为两类: 快照读（Snapshot Read）与当前读 （Current Read）。

- **快照读**：读取的是记录的快照版本（有可能是历史版本），不用加锁。（select）
- **当前读**：读取的是记录的最新版本，并且当前读返回的记录，都会加锁，保证其他事务不会再并发修改这条记录。（select... for update 或lock in share mode，insert/delete/update）

如何生成的多版本？每次事务修改操作之前，都会在Undo日志中记录修改之前的数据状态和事务号，该备份记录可以用于其他事务的读取，也可以进行必要时的数据回滚。若想进一步解决写写冲突，可以采用乐观锁或悲观锁。

MVCC是通过**数据的隐藏列和回滚日志**（undo log），实现多个版本数据的共存；在实现MVCC时，每一行的数据中会额外保存几个隐藏的列，比如**当前行的隐含ID、事务ID和指向undo log的回滚指针**；
- 用排他锁锁定该行；记录 Redo log；
- 把该行修改前的值复制到 Undo log，即图中下面的行；
- 修改当前行的值，填写事务编号，使回滚指针指向 Undo log 中修改前的行。

除了这些隐藏列以外，实际上每条记录的记录头信息中还会存储一个标志位，标志该记录是否删除。

RR可以避免幻读（加锁）：通过next-key lock机制实现的；next-key lock实际上就是行锁的一种，只不过它不只是会锁住当前行记录的本身，还会锁定一个范围。虽然能够避免幻读现象，但是却没有达到可串行化的级别。**普通查询语句没有加锁，是不能避免幻读的**。

## 五.锁机制

InnoDB引擎锁机制是基于**索引实现的记录锁定**。

### 1.锁的分类

在 MySQL中锁有很多不同的分类。

从**操作的粒度**可分为表级锁、行级锁和页级锁。

- **表级锁**：每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB、BDB 等存储引擎中。
- **行级锁**：每次操作锁住一行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高。应用在InnoDB 存储引擎中。
- **页级锁**：每次锁定相邻的一组记录，锁定粒度界于表锁和行锁之间，开销和加锁时间界于表锁和行锁之间，并发度一般。应用在BDB 存储引擎中。

从**操作的类型**可分为读锁和写锁。

- **读锁**（S锁）：共享锁，针对同一份数据，多个读操作可以同时进行而不会互相影响。
- **写锁**（X锁）：排他锁，当前写操作没有完成前，它会阻断其他写锁和读锁。

IS锁、IX锁：意向读锁、意向写锁，属于表级锁，S和X主要针对行级锁。在对表记录添加S或X锁之前，会先对表添加IS或IX锁。

S锁：事务A对记录添加了S锁，可以对记录进行读操作，不能做修改，其他事务可以对该记录追加S锁，但是不能追加X锁，需要追加X锁，需要等记录的S锁全部释放。

X锁：事务A对记录添加了X锁，可以对记录进行读和修改操作，其他事务不能对记录做读和修改操作。

从**操作的性能**可分为乐观锁和悲观锁。

- **乐观锁**：一般的实现方式是对记录数据版本进行比对，在数据更新提交的时候才会进行冲突检测，如果发现冲突了，则提示错误信息。乐观锁可使用版本字段（version）或使用时间戳（Timestamp）来实现
- **悲观锁**：在对一条数据修改的时候，为了避免同时被其他人修改，在修改数据之前先锁定，再修改的控制方式。共享锁和排他锁是悲观锁的不同实现，但都属于悲观锁范畴。

### 2.行锁原理

在InnoDB引擎中，我们可以使用行锁和表锁，其中行锁又分为共享锁和排他锁。**InnoDB行锁是通过对索引数据页上的记录加锁实现的**，主要实现算法有 3 种：Record Lock、Gap Lock 和 Next-key Lock。

- RecordLock锁：锁定**单个行**记录的锁。（记录锁，RC、RR隔离级别都支持）;
- GapLock锁：**间隙锁**，锁定索引记录间隙，确保索引记录的间隙不变。（范围锁，RR隔离级别支持）;
- Next-key Lock 锁：**记录锁和间隙锁组合**，同时锁住数据，并且锁住数据前后范围。（记录锁+范围锁，RR隔离级别支持）。

在RR隔离级别，InnoDB对于记录加锁行为都是先采用Next-Key Lock，但是当SQL操作含有唯一索引时，Innodb会对Next-Key Lock进行优化，降级为RecordLock，仅锁住索引本身而非范围。

- `select ... from` 语句：InnoDB引擎采用MVCC机制实现非阻塞读，所以对于**普通的select语句，InnoDB不加锁**。
- `select ... from lock in share mode`语句：**追加了共享锁**，InnoDB会使用Next-Key Lock锁进行处理，如果扫描发现唯一索引，可以降级为RecordLock锁。
- `select ... from for update`语句：**追加了排他锁**，InnoDB会使用Next-Key Lock锁进行处理，如果扫描发现唯一索引，可以降级为RecordLock锁。
- `update ... where` 语句：InnoDB会使用Next-Key Lock锁进行处理，如果扫描发现唯一索引，可以降级为RecordLock锁。
- `delete ... where` 语句：InnoDB会使用Next-Key Lock锁进行处理，如果扫描发现唯一索引，可以降级为RecordLock锁。
- `insert`语句：InnoDB会在将要插入的那一行设置一个排他的RecordLock锁。

下面以 `update t1 set name=‘XX’ where id=10`操作为例，举例子分析下 InnoDB 对不同索引的**加锁行为**，以RR隔离级别为例。

- 主键加锁：仅在id=10的主键索引**记录**上加X锁；
- 唯一键加锁：先在唯一**索引**id上加X锁，然后在id=10的主键索引**记录**上加X锁；
- 非唯一键加锁：对满足id=10条件的**记录和主键分别加X锁**，然后在(6,c)-(10,b)、(10,b)-(10,d)、(10,d)-(11,f) **范围分别加Gap Lock**；
- 无索引加锁：表里**所有行和间隙都会加X锁**。（当没有索引时，会导致全表锁定，因为InnoDB引擎锁机制是基于索引实现的记录锁定）。

### 3.死锁产生与解决方案



