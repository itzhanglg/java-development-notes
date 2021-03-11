### 作业习题

采⽤Sharding-JDBC实现c_order表分库分表+读写分离：

1.基于user_id对c_order表进⾏数据分⽚

2.分别对master1和master2搭建⼀主⼆从架构

![image-20200906072144944](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200906072144944.png)

3.基于master1和master2主从集群实现读写分离

4.c_order建表SQL如下：

```sql
CREATE TABLE `c_order`(
 `id` bigint(20) NOT NULL AUTO_INCREMENT,
 `is_del` bit(1) NOT NULL DEFAULT 0 COMMENT '是否被删
除',
 `user_id` int(11) NOT NULL COMMENT '⽤户id',
 `company_id` int(11) NOT NULL COMMENT '公司id',
 `publish_user_id` int(11) NOT NULL COMMENT 'B端⽤户id',
 `position_id` int(11) NOT NULL COMMENT '职位ID',
 `resume_type` int(2) NOT NULL DEFAULT 0 COMMENT '简历类型：
0附件 1在线',
 `status` varchar(256) NOT NULL COMMENT '投递状态 投递状态
WAIT-待处理 AUTO_FILTER-⾃动过滤 PREPARE_CONTACT-待沟通 REFUSE-拒绝
ARRANGE_INTERVIEW-通知⾯试',
 `create_time` datetime NOT NULL COMMENT '创建时间',
 `update_time` datetime NOT NULL COMMENT '处理时间',
 PRIMARY KEY (`id`),
 KEY `index_userId_positionId` (`user_id`, `position_id`),
 KEY `idx_userId_operateTime` (`user_id`, `update_time`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
```

### 环境介绍

1.环境介绍涉及的各个软件的版本

| 环境&软件                 | 版本   |
| ------------------------- | ------ |
| 虚拟机&VMware Workstation | 10.0   |
| 服务器&CentOS             | 7.8    |
| 数据库&Mysql              | 5.7.28 |
| 远程连接&Xshell           | 6      |
| 远程文件传输&WinSCP、Xftp | 6      |

2.介绍各个机器对应角色&作用&ip地址

架构如图所示，6台机器的IP和角色如下表：

| 机器名称 | IP             | 角色         | 权限         |
| -------- | -------------- | ------------ | ------------ |
| master1  | 192.168.91.105 | 数据库Master | 负责写、主库 |
| slave1   | 192.168.91.106 | 数据库Slave  | 只读、从库   |
| slave2   | 192.168.91.107 | 数据库Slave  | 只读、从库   |
| master2  | 192.168.91.108 | 数据库Master | 负责写、主库 |
| slave3   | 192.168.91.109 | 数据库Slave  | 只读、从库   |
| slave4   | 192.168.91.110 | 数据库Slave  | 只读、从库   |

架构图：

![image-20200906212603659](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200906212603659.png)

- 每个库上都有c_order表，即对 c_order 表进行分库操作，并没有在同一个库中进行分表操作
- 分库分表是对数据进行分片操作，对数据进行存储扩展和读写扩展，每个库中的数据记录不一致，根据分片配置进行路由
- 主从复制架构是对数据进行读写分离，实现高可用和读写扩展，读操作不用加锁，每个库中的数据记录一致，根据SQL语义进行路由

### 实现思路

#### 1.主库配置操作

vim /etc/my.cnf，每个库的sever-id都是唯一的。

```
# 开启logbin功能
log_bin=mysql-bin
# 设置serverId
server-id=1
# 每次执行写操作都与磁盘同步
sync-binlog=1
# 指定不需要同步的数据库
binlog-ignore-db=performance_schema
binlog-ignore-db=information_schema
binlog-ignore-db=sys
binlog-ignore-db=mysql
# 指定需要同步的数据库
#binlog-do-db=lagou

# 开启中继日志
relay_log=mysql-relay-bin
log_slave_updates=1
relay_log_purge=0

# 设置semi参数,自动开启半同步复制
#rpl_semi_sync_master_enabled=ON
#rpl_semi_sync_master_timeout=1000
```

重启mysql服务：systemctl restart mysqld

登陆myql后进行从库授权：

```sql
grant replication slave on *.* to 'root'@'%' identified by 'root';
grant all privileges on *.* to 'root'@'%' identified by 'root';
flush privileges;
show master status;	  #显示主库信息，有binlog文件名称
```

#### 2.从库配置操作

vim /etc/my.cnf

```
# log_bin
log_bin=mysql-bin
# 设置serverId
server-id=2
binlog_format=ROW
sync-binlog=1
binlog-ignore-db=information_schema
binlog-ignore-db=mysql
binlog-ignore-db=performance_schema
binlog-ignore-db=sys
# 设置中继日志的文件名称
relay_log=mysql-relay-bin
log_slave_updates=1
relay_log_purge=0
# 设置数据库只读
read_only=1

# 自动开启半同步复制
#rpl_semi_sync_slave_enabled=ON

# 从库并行复制基于组
#slave-parallel-type=LOGICAL_CLOCK
# 并行复制线程数
#slave-parallel-workers=8
# master和relaylog信息库基于table
#master_info_repository=TABLE
#relay_log_info_repository=TABLE
# 设置中继日志可以覆盖写入
#relay_log_recovery=1
```

重启mysql服务：systemctl restart mysqld

登陆mysql后查看从库信息：

```
grant all privileges on *.* to 'root'@'%' identified by 'root';
flush privileges;

show slave status;	//显示从库信息，没启动slave时，信息为空
// 同步主库，指定主库信息
change master to master_host='192.168.91.105',master_port=3306,master_user='root',master_password='root',master_log_file='mysql-bin.000010',master_log_pos=1381;
start slave;	// 启动主从同步
show slave status \G;	// 查看从库状态信息，信息不为空
	# Slave_IO_Running: Yes
    # Slave_SQL_Running: Yes
```

#### 3.核心配置代码和实现

引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
</dependency>
```

application-sharding-master-slaves.properties配置文件：

```properties
# datasource
spring.shardingsphere.datasource.names=master1,slave1,slave2,master2,slave3,slave4

spring.shardingsphere.datasource.master1.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.master1.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.master1.jdbc-url=jdbc:mysql://192.168.91.105:3306/lagou?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.shardingsphere.datasource.master1.username=root
spring.shardingsphere.datasource.master1.password=root

spring.shardingsphere.datasource.slave1.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.slave1.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.slave1.jdbc-url=jdbc:mysql://192.168.91.106:3306/lagou?useSSL=false
spring.shardingsphere.datasource.slave1.username=root
spring.shardingsphere.datasource.slave1.password=root

spring.shardingsphere.datasource.slave2.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.slave2.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.slave2.jdbc-url=jdbc:mysql://192.168.91.107:3306/lagou?useSSL=false
spring.shardingsphere.datasource.slave2.username=root
spring.shardingsphere.datasource.slave2.password=root

spring.shardingsphere.datasource.master2.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.master2.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.master2.jdbc-url=jdbc:mysql://192.168.91.108:3306/lagou?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.shardingsphere.datasource.master2.username=root
spring.shardingsphere.datasource.master2.password=root

spring.shardingsphere.datasource.slave3.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.slave3.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.slave3.jdbc-url=jdbc:mysql://192.168.91.109:3306/lagou?useSSL=false
spring.shardingsphere.datasource.slave3.username=root
spring.shardingsphere.datasource.slave3.password=root

spring.shardingsphere.datasource.slave4.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.slave4.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.slave4.jdbc-url=jdbc:mysql://192.168.91.110:3306/lagou?useSSL=false
spring.shardingsphere.datasource.slave4.username=root
spring.shardingsphere.datasource.slave4.password=root


# sharding-database：分库
spring.shardingsphere.sharding.tables.c_order.database-strategy.inline.sharding-column=user_id
spring.shardingsphere.sharding.tables.c_order.database-strategy.inline.algorithm-expression=master$->{user_id % 2 + 1}
spring.shardingsphere.sharding.tables.c_order.actual-data-nodes=master$->{1..2}.c_order


# master-slave：读写
spring.shardingsphere.sharding.master-slave-rules.master1.master-data-source-name=master1
spring.shardingsphere.sharding.master-slave-rules.master1.slave-data-source-names=slave1, slave2
spring.shardingsphere.sharding.master-slave-rules.master2.master-data-source-name=master2
spring.shardingsphere.sharding.master-slave-rules.master2.slave-data-source-names=slave3, slave4
spring.shardingsphere.masterslave.load-balance-algorithm-type=ROUND_ROBIN

# id：主键生成策略
spring.shardingsphere.sharding.tables.c_order.key-generator.column=id
spring.shardingsphere.sharding.tables.c_order.key-generator.type=SNOWFLAKE
```

### 开发问题

1.java.lang.NullPointerException at dao.ShardingDatabaseTest.addTest(ShardingDatabaseTest.java:28)

在positionRepository.save(position);调用时没有@Resource进行注入private PositionRepository positionRepository;



2.Caused by: java.lang.IllegalStateException: no database route info;

org.springframework.dao.InvalidDataAccessApiUsageException: no database route info; nested exception is java.lang.IllegalStateException: no database route info

由于进入分表策略的时候逻辑表匹配不到物理真实的表，导致报错，不知道锁定那张表。

解决：看一下自己的分片策略，是否能够匹配到真实的表。

可能是由于分片策略导致访问不到真实数据库。



3.Caused by: org.hibernate.HibernateException: The database returned no natively generated identity value

实体类设置id属性值时，实体类中id属性又添加了主键自增策略（`@GeneratedValue(strategy = GenerationType.IDENTITY)`），在执行中并没有生成值，将其注释掉就可以了。

若实体类没有设置id属性值，实体类中需要将主键自增策略注解添加上，这样就可以自动根据使用UUID或SNOWFLAKE。



4.Caused by: java.lang.RuntimeException: Invalid `org.apache.shardingsphere.spi.keygen.ShardingKeyGenerator` SPI type `SNOWFLAK`.
配置文件中 `spring.shardingsphere.sharding.tables.position.key-generator.type=SNOWFLAKE` id生成策略名称指定错误。



5.自定义主键生成策略时，在generateKey方法里new SnowflakeShardingKeyGenerator 对象来获取key时，生成的数据都会路由到一个数据库里，需要把new SnowflakeShardingKeyGenerator 的对象作为全局变量。

```java
@Override
public Comparable<?> generateKey() {
    System.out.println("执行自定义主键生成器类MyLagouId");
    return new SnowflakeShardingKeyGenerator().generateKey();
}

// 正确代码
// 需要作为全局变量,不然所有数据都会路由到同一个库
private SnowflakeShardingKeyGenerator snow = new SnowflakeShardingKeyGenerator();
@Override
public Comparable<?> generateKey() {
    System.out.println("执行自定义主键生成器类MyLagouId");
    return snow.generateKey();
}
```



6.java.lang.NullPointerException: Cannot invoke method mod() on null object

```properties
# sharding-database:分片键+分片算法(对2取模)
spring.shardingsphere.sharding.tables.position.database-strategy.inline.sharding-column=id
spring.shardingsphere.sharding.tables.position.database-strategy.inline.algorithm-expression=ds$->{id % 2}

# pid就是id的值,分片算法中也要时pid,当是id是,会报上面的异常
spring.shardingsphere.sharding.tables.position_detail.database-strategy.inline.sharding-column=pid
spring.shardingsphere.sharding.tables.position_detail.database-strategy.inline.algorithm-expression=ds$->{pid % 2}
```



7.Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Unknown column 'pwd' in 'field list'

可能是由于数据源的端口号或其它配置导致数据库连接到其他库，但其他库中的表却没有pwd等逻辑字段。



8.Caused by: org.apache.shardingsphere.underlying.common.exception.ShardingSphereException: Can't find datasource type!

配置了datasource.names=ds0,ds1,但没有配置ds1的数据源绑定信息,导致不能查找到数据源.



9.Caused by: java.lang.IllegalArgumentException: Unresolvable class definition for class [org.apache.shardingsphere.shardingjdbc.spring.boot.shadow.SpringBootShadowRuleConfigurationProperties]

导入xa事务依赖时,写入了版本导致的错误,去掉版本即可.

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-transaction-xa-core</artifactId>
    <!-- <version>4.0.0-RC2</version> -->
</dependency>
```

分布式事务注解:

```java
@Transactional  // 开启事务
// @ShardingTransactionType(TransactionType.XA)
public void xaTest(){
    TransactionTypeHolder.set(TransactionType.XA);  // 指定分布式事务模型
    //......
}
```



10.Factory method 'shardingDataSource' threw exception：Host '192.168.91.1' is not allowed to connect to this MySQL server"

可能是由于服务器上没有进行远程访问用户授权导致的。解决办法：

```sql
grant all privileges on *.* to 'root'@'%' identified by 'root';
flush privileges;
```

### 视频验证

**视频最后讲解错误部分：查数据是在 slave1 和 slave3 或其它从库上查的，真实sql有显示，口误说成从 master1 和 master2 上查的了**。

集群环境

* 6台

* 主master1负责写，从slave1 slave2负责读

* 主master2负责写，从slave3 slave4负责读

代码

* 主要类&方法&参数&返回值及代码行标注注释

* 基于user_id对c_order表进⾏数据分⽚

* 基于master1和master2主从集群实现读写分离

运行效果

* 项目各个类作用介绍，重点代码行介绍，启动

* 演示基于user_id对c_order表进⾏数据分⽚，基于master1和master2主从集群实现读写分离

* 添加数据，通过Navicat展示数据分片成功，通过代码控制台真实sql展示数据分别在master1和master2进行写入

* 查询数据，通过代码控制台真实sql展示数据是通过4个从节点查询的







