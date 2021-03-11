### 习题

搭建一个MySQL高可用架构集群环境（4台主机，1主、2从、1 MHA）

1. 首先实现一主两从的同步复制功能（采用半同步复制机制）

 2. 然后采用MHA实现主机出故障，从库能自动切换功能

 3. MHA高可用搭建后，在主库新建商品表进行效果测试

 4. 在拉勾业务中职位表相当于电商系统的商品表，投递表相当于电商系统的订单表。职位表我们采用垂直拆分方法分为position（职位描述表）和 position_detail（职位详情表），表结构结构如下：

 5. **position：id(int)、name(varchar)、salary(varchar)、city(varchar)**

 6. **position_detail：id(int)、pid(int)、description(text)**

 7. 作业需要提交集群环境搭建手册和效果演示视频

    手册：包含环境软件版本和架构介绍、环境安装过程、操作的问题和注意事项等。

    视频：仅录制环境介绍和效果演示。

作业资料说明：
1、提供资料：说明文档、验证及讲解视频。
2、讲解内容包含：题目分析、实现思路、环境介绍。
3、说明文档包含：
  环境软件版本、架构介绍
  环境安装过程（各个配置加注释）
  操作过程中遇到的问题
  操作注意事项
4、效果视频验证：
  4.1集群环境
  	4台
	  1主，2从，1MHA
	   一主两从的半同步复制功能
	   MHA实现主机出故障，从库能自动切换功能
  4.2环境介绍
	介绍涉及的各个软件的版本
    介绍各个机器对应角色&作用&ip地址
  4.3主库新建商品表
  添加数据，演示半同步复制
    先查询主从库数据
    添加后，再次查询主从库数据
    查看log，是否显示半同步
  4.4主机出故障，从库能自动切换功能
    关闭 主服务器，查看MHA服务器是否正常切换
	App1:mysql Master failower 之前主ip(之前主ip:3306) to 切换后ip(切换后ip:3306) succeeded
	在切换后的主节点机器上查看状态展示效果
	在切换后的主节点机器数据库添加数据，分别查询主从库数据一致

### 作业说明

#### 环境软件版本、架构介绍

##### 环境版本

| 环境&软件                 | 版本   |
| ------------------------- | ------ |
| 虚拟机&VMware Workstation | 10.0   |
| 服务器&CentOS             | 7.8    |
| 数据库&Mysql              | 5.7.28 |
| 远程连接&Xshell           | 6      |
| 远程文件传输&WinSCP、Xftp | 6      |

##### 架构介绍

架构如图所示，4台机器的IP和角色如下表：

| 机器名称     | IP             | 角色         | 权限         |
| ------------ | -------------- | ------------ | ------------ |
| Mysql_Master | 192.168.91.105 | 数据库Master | 可读写、主库 |
| Mysql_Slave1 | 192.168.91.106 | 数据库Slave  | 只读、从库   |
| Mysql_Slave2 | 192.168.91.107 | 数据库Slave  | 只读、从库   |
| Mysql_MHA    | 192.168.91.108 | MHA Manager  | 高可用监控   |

架构图：

![image-20200830005042763](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200830005042763.png)

#### 环境安装过程（各个配置加注释）

参考下文中的MySQL5.7的安装、主从复制实战、半同步复制实战。

#### 操作过程中遇到的问题

**1.四台服务器ssh互通问题**

1.在每台服务器上都执行ssh-keygen -t rsa生成密钥对:

> ssh-keygen -t rsa

2.在每台服务器上生成密钥对后，将公钥复制到需要无密码登陆的服务器上：举例如192.168.91.105，192.168.91.106，192.168.91.107，192.168.91.108这四台服务器需要做相互免密码登陆，在每台服务器生成密钥对后，在每台服务器上执行ssh-copy-id命令，将公钥复制到其它三台服务器上:

> ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.91.106
>
> ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.91.107
>
> ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.91.108

以上命令，可以自动将公钥添加到名为authorized_keys的文件中，在每台服务器都执行完以上步骤后就可以实现多台服务器相互无密码登陆了。

**2.检测MySQL主从复制问题**

在MHA Manager服务器上执行：

> masterha_check_repl --conf=/etc/mha/app1.cnf

报如下错：

```
[root@atzlg4 ~]# masterha_check_repl --conf=/etc/mha/app1.cnf
Sun Aug 30 00:15:38 2020 - [info] Reading default configuration from /etc/masterha_default.cnf..
Sun Aug 30 00:15:38 2020 - [info] Reading application default configuration from /etc/mha/app1.cnf..
Sun Aug 30 00:15:38 2020 - [info] Reading server configuration from /etc/mha/app1.cnf..
Sun Aug 30 00:15:38 2020 - [info] MHA::MasterMonitor version 0.58.
Sun Aug 30 00:15:39 2020 - [error][/usr/share/perl5/vendor_perl/MHA/Server.pm, ln180] Got MySQL error when connecting 192.168.91.106(192.168.91.106:3306) :1130:Host '192.168.91.108' is not allowed to connect to this MySQL server, but this is not a MySQL crash. Check MySQL server settings.
Sun Aug 30 00:15:39 2020 - [error][/usr/share/perl5/vendor_perl/MHA/ServerManager.pm, ln301]  at /usr/share/perl5/vendor_perl/MHA/ServerManager.pm line 297.
Sun Aug 30 00:15:44 2020 - [error][/usr/share/perl5/vendor_perl/MHA/ServerManager.pm, ln309] Got fatal error, stopping operations
Sun Aug 30 00:15:44 2020 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln427] Error happened on checking configurations.  at /usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm line 329.
Sun Aug 30 00:15:44 2020 - [error][/usr/share/perl5/vendor_perl/MHA/MasterMonitor.pm, ln525] Error happened on monitoring servers.
Sun Aug 30 00:15:44 2020 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!
```

还未解决。。。。。。

1.猜测有可能是克隆导致的问题。（Manager服务器，IP：192.168.91.108是由Slave，IP：192.168.91.106克隆过来的，当时106上在课程视频实战时已经按照MySQL进行相关半同步实战测试了），后面在虚拟机上安装几台干净的centos环境来测试。

2.百度 [masterha_check_repl --conf=/etc/mha/app1.cnf检查错误](https://www.cnblogs.com/orcl-2018/p/13264032.html) 显示原因：MHA配置文件监控用户mha的密码错误。

3.启源导师的作业课件文档中问题解决办法也尝试过，也没有解决。

**3.mysql-lib-5.17与mysql-server-5.6冲突**

在CentOS 6.8系统下已经安装了mysql5.7.21服务和客户端，MHA的Node依赖于perl-DBD-MySQL，所以要先安装perl-DBD-MySQL，安装时报错。rpm -qa|grep mysql时是mysql-server-5.6的服务。

解决办法：安装**MySQL-shared-compat**这个包即可，rpm -ihv MySQL-shared-compat-5.6.40-1.el6.x86_64.rpm

参考地址：[mysql-libs-5.1.73和MySQL-server-5.6.37冲突](https://blog.csdn.net/debimeng/article/details/80320303)

**4.相关错误总结**

- [MHA之错误总结](https://blog.csdn.net/q936889811/article/details/80077344)

#### 操作注意事项

Mysql主库与从库安装semi时分master和slave。

```sql
#在线添加
mysql> INSTALL PLUGIN validate_password SONAME 'validate_password.dll';
 
#在线卸载
mysql> UNINSTALL PLUGIN validate_password;
```

linux在线下载node节点：wget https://github.com/yoshinorim/mha4mysql-node/releases/download/v0.58/mha4mysql-node-0.58-0.el7.centos.noarch.rpm

#### 参考链接

- [mysql的MHA高可用](https://www.lagou.com/lgeduarticle/74994.html)
- [MySQL高可用之MHA部署](https://blog.51cto.com/14154700/2472806)
- [MySQL高可用MHA原理及测试](https://juejin.im/post/6844904099075325960)
- [搭建mysql主从集群半同步复制MHA高可用监控基于LinuxCentos7](https://blog.csdn.net/centryFrom22th/article/details/106947348)



### CentOS7安装MySQL5.7

安装MySQL、关闭防火墙（MySQL之间进行通信）

1.使用rpm安装mysql5.7.28：

> wget https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar

2.查看是否安装mariadb：

> rpm -qa|grep mariadb

删除mariadb：

> rpm -e mariadb-libs-5.5.60-1.el7_5.x86_64 --nodeps

3.解压MySQL到指定文件夹：

> tar -xvf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar -C /home/mysql

安装rpm：按照先后顺序依次安装

```
rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-devel-5.7.28-1.el7.x86_64.rpm
```

成功效果：

```
警告：mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-libs-compat-5.7.2################################# [100%]
```

4.实例化mysql：

> mysqld --initialize --user=mysql

查看临时密码：

> cat /var/log/mysqld.log

获取临时密码：

> grep 'temporary password' /var/log/mysqld.log

5.服务开机自启动：

> systemctl start mysqld.service
>
> 查看mysql服务启动状态：
>
> systemctl status mysqld.service

6.登陆mysql：mysql -uroot -p（密码为mysqld.log中的临时密码）

修改密码：

> set password=password('root');

退出重新登陆：mysql -uroot -proot

7.关闭防火墙：

```
systemctl stop iptables		// 关闭iptables
systemctl stop firewalld	// 关闭firewalld
systemctl disable firewalld.service	 // 关闭firewalld自启动
```

### 主从复制实战

一主一从：异步复制，可能存在下列问题：

- 主库宕机后，数据可能丢失；
- 从库只有一个SQL Thread，主库写压力大，复制很可能延时

#### 主库配置

编写配置文件：vim /etc/my.cnf

```
# log_bin
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
# 指定需要同步的数据库
#binlog-do-db=lagou
#
```

重启mysql服务：systemctl restart mysqld

登陆myql后进行从库授权：

```sql
grant replication slave on *.* to 'root'@'%' identified by 'root';
grant all privileges on *.* to 'root'@'%' identified by 'root';
flush privileges;
show master status;	  #显示主库信息，有binlog文件名称
```

#### 从库配置

编写配置文件：vim /etc/my.cnf

```
# log_bin
# 设置serverId
server-id=2
# 设置中继日志的文件名称
relay_log=mysql-relay-bin
# 设置数据库只读
read_only=1
#
```

重启mysql服务：systemctl restart mysqld

登陆mysql后查看从库信息：

```
show slave status;	//显示从库信息，没启动slave时，信息为空
// 同步主库，指定主库信息
change master to master_host='192.168.91.105',master_port=3306,master_user='root',master_password='root',master_log_file='mysql-bin.000001',master_log_pos=869;
start slave;	// 启动主从同步
show slave status \G;	// 查看从库状态信息，信息不为空
	# Slave_IO_Running: Yes
    # Slave_SQL_Running: Yes
```

#### 效果演示

主库mysql中：

```
show databases;
create database lagou;
use lagou;
create table dept(dno int primary key, dname varchar(20))engine=innodb charset=utf8;
show tables;
insert into dept values(1,'java');
```

从库mysql中：

```
show databases;
use lagou;
show tables;
select * from dept;
```

多个从库时可以采用 mysqldump 工具将主库的数据库进行备份导入从库后再进行主从复制。

```
mysqldump --help  // 查看命令

mysqldump --all-databases > mysql_backup_all.sql -uroot -p	// 输入密码
```

### 半同步复制实战

半同步复制主要用来解决数据一致性问题（降低数据丢失的概率）。

在一主一从异步复制基础上添加半同步复制，需要在主库和从库上安装semi插件。

#### 主库安装semi

登陆MySQL后

```sql
select @@have_dynamic_loading;  # 查看是否可以动态加载插件
show plugins;  # 显示安装了哪些插件
install plugin rpl_semi_sync_master soname 'semisync_master.so';  # 主库安装semi插件
show variables like '%semi%';  # 显示主库semi参数
set global rpl_semi_sync_master_enabled=1;  # 主库开启semi，默认OFF
set global rpl_semi_sync_master_timeout=1000;  # 指定semi延迟时间，默认10000ms
```

参数信息：

```
mysql> show variables like '%semi%';
+-------------------------------------------+------------+
| Variable_name                             | Value      |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled              | OFF        |
| rpl_semi_sync_master_timeout              | 10000      |
| rpl_semi_sync_master_trace_level          | 32         |
| rpl_semi_sync_master_wait_for_slave_count | 1          |
| rpl_semi_sync_master_wait_no_slave        | ON         |
| rpl_semi_sync_master_wait_point           | AFTER_SYNC |
+-------------------------------------------+------------+
```

#### 从库安装semi

登陆MySQL后

```sql
install plugin rpl_semi_sync_slave soname 'semisync_slave.so';  # 从库安装semi
show variables like '%semi%';  # 显示从库semi参数，与主库不同
set global rpl_semi_sync_slave_enabled=1;  # 开启从库semi
stop slave;  # 停止从库
start slave;  # 启动从库
```

参数信息：

```
mysql> show variables like '%semi%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | OFF   |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+
```

#### 效果演示

主库添加数据：

```sql
use lagou;
insert into dept values(2,'h5');
```

从库查询数据：

```sql
use lagou;
select * from dept;
```

#### 查看semi半同步复制是否启动

一：查看semi参数：show variables like '%semi%';

二：查看日志文件：cat /var/log/mysqld.log 

```
2020-08-25T21:36:49.606717Z 6 [Note] Semi-sync replication initialized for transactions.
2020-08-25T21:36:49.606789Z 6 [Note] Semi-sync replication enabled on the master.
2020-08-25T21:36:49.607471Z 0 [Note] Starting ack receiver thread
2020-08-25T21:39:57.516664Z 7 [Note] While initializing dump thread for slave with UUID <8fa5839e-e70e-11ea-abfb-000c296e1242>, found a zombie dump thread with the same UUID. Master is killing the zombie dump thread(3).
2020-08-25T21:39:57.516988Z 7 [Note] Start binlog_dump to master_thread_id(7) slave_server(2), pos(mysql-bin.000001, 1521)
2020-08-25T21:39:57.517130Z 7 [Note] Start semi-sync binlog_dump to slave (server_id: 2), pos(mysql-bin.000001, 1521)
2020-08-25T21:39:57.517489Z 3 [Note] Stop asynchronous binlog_dump to slave (server_id: 2)
```

### 并行复制实战

并行复制主要解决数据延迟问题，加快主从复制过程，减少主从复制延迟。

#### 主库配置

登陆mysql修改binlog_group的参数信息，可以使用set命令修改，也可以到my.cnf配置文件中去配置修改指定参数信息。

```sql
show variables like '%binlog_group%';
set global binlog_group_commit_sync_delay=1000;  # 设置每组提交事务的延迟
set global binlog_group_commit_sync_no_delay_count=100;  # 设置每组提交的事务数
```

参数信息：

```
mysql> show variables like '%binlog_group%';
+-----------------------------------------+-------+
| Variable_name                           | Value |
+-----------------------------------------+-------+
| binlog_group_commit_sync_delay          | 0     |
| binlog_group_commit_sync_no_delay_count | 0     |
+-----------------------------------------+-------+
```

#### 从库配置

从库上的relay_log的回放。

登陆mysql后查看slave和relay_log的参数信息：

查看slave：`show variables like '%slave%';`

```
slave_parallel_type             | DATABASE
slave_parallel_workers          | 0
// ......
```

设置参数前需要先关闭slave：stop slave;

```sql
set global slave_parallel_type='LOGICAL_CLOCK';  # 设置并行复制基于组
set global slave_parallel_workers=8;  # 设置并行复制线程数
```

查看relay_log：`show variables like '%relay_log%';`

```
mysql> show variables like '%relay_log%';
+---------------------------+--------------------------------------+
| Variable_name             | Value                                |
+---------------------------+--------------------------------------+
| max_relay_log_size        | 0                                    |
| relay_log                 | mysql-relay-bin                      |
| relay_log_basename        | /var/lib/mysql/mysql-relay-bin       |
| relay_log_index           | /var/lib/mysql/mysql-relay-bin.index |
| relay_log_info_file       | relay-log.info                       |
| relay_log_info_repository | FILE                                 |
| relay_log_purge           | ON                                   |
| relay_log_recovery        | OFF                                  |
| relay_log_space_limit     | 0                                    |
| sync_relay_log            | 10000                                |
| sync_relay_log_info       | 10000                                |
+---------------------------+--------------------------------------+
```

编写mysql配置文件my.cnf：`vim /etc/my.cnf`，设置参数信息，有些参数是只读的，无法使用set命令更改。

```
# log_bin
# 设置serverId
server-id=2
# 设置中继日志的文件名称
relay_log=mysql-relay-bin
# 设置数据库只读
read_only=1

# 从库并行复制基于组
slave-parallel-type=LOGICAL_CLOCK
# 并行复制线程数
slave-parallel-workers=8
# master和relaylog信息库基于table
master_info_repository=TABLE
relay_log_info_repository=TABLE
# 设置中继日志可以覆盖写入
relay_log_recovery=1
#
```

重启mysql服务：`systemctl restart mysqld`

可再登陆mysql后查看slave和relaylog的参数信息，是否修改成功。

#### 效果演示

主库添加数据：

```sql
use lagou;
insert into dept values(3,'ui');
```

从库查看数据：

```sql
use lagou;
select * from dept;
use performance_schema;
select * from replication_applier_status_by_worker;
```

可以看到worker开始工作了：

```
mysql> select * from replication_applier_status_by_worker;
+--------------+-----------+-----------+---------------+-----------------------+-------------------+--------------------+----------------------+
| CHANNEL_NAME | WORKER_ID | THREAD_ID | SERVICE_STATE | LAST_SEEN_TRANSACTION | LAST_ERROR_NUMBER | LAST_ERROR_MESSAGE | LAST_ERROR_TIMESTAMP |
+--------------+-----------+-----------+---------------+-----------------------+-------------------+--------------------+----------------------+
|              |         1 |        28 | ON            | ANONYMOUS             |                 0 |                    | 0000-00-00 00:00:00  |
|              |         2 |        30 | ON            |                       |                 0 |                    | 0000-00-00 00:00:00  |
|              |         3 |        31 | ON            |                       |                 0 |                    | 0000-00-00 00:00:00  |
|              |         4 |        32 | ON            |                       |                 0 |                    | 0000-00-00 00:00:00  |
|              |         5 |        33 | ON            |                       |                 0 |                    | 0000-00-00 00:00:00  |
|              |         6 |        34 | ON            |                       |                 0 |                    | 0000-00-00 00:00:00  |
|              |         7 |        35 | ON            |                       |                 0 |                    | 0000-00-00 00:00:00  |
|              |         8 |        36 | ON            |                       |                 0 |                    | 0000-00-00 00:00:00  |
+--------------+-----------+-----------+---------------+-----------------------+-------------------+--------------------+----------------------+
8 rows in set (0.00 sec)
```

### 读写分离实战

#### proxy服务器配置

将单独的服务器作为mysql-proxy服务器。

下载mysql-proxy-0.8.5-linux-el6-x86-64bit.tar.gz：

> wget https://cdn.mysql.com/archives/mysql-proxy/mysql-proxy-0.8.5-linux-el6-x86-64bit.tar.gz

解压mysql-proxy：

> tar -xzvf mysql-proxy-0.8.5-linux-el6-x86-64bit.tar.gz

编辑mysql-proxy配置文件：vim /etc/mysql-proxy.cnf

```
[mysql-proxy]
user=root
admin-username=root
admin-password=root
proxy-address=192.168.91.107:4040
proxy-backend-addresses=192.168.91.105:3306
proxy-read-only-backend-addresses=192.168.91.106:3306
proxy-lua-script=/home/mysql-proxy-0.8.5-linux-el6-x86-64bit/share/doc/mysql-proxy/rw-splitting.lua
log-file=/var/log/mysql-proxy.log
log-level=debug
daemon=true
keepalive=true
```

修改mysql-proxy.cnf文件的权限：chmod 660 /etc/mysql-proxy.cnf，可读写

修改lua脚本：将最小空闲连接数改为1，方便测试效果查看。默认为4

> vim mysql-proxy-0.8.5-linux-el6-x86-64bit/share/doc/mysql-proxy/rw-splitting.lua

rw-splitting.lua文件修改：

```
-- connection pool
if not proxy.global.config.rwsplit then
        proxy.global.config.rwsplit = {
                min_idle_connections = 1,	-- 默认为4
                max_idle_connections = 8,

                is_debug = false
        }
end
```

#### 测试效果

使用navicat连接mysql-proxy，和连接mysql一样，proxy服务器IP：192.168.91.107，端口：4040，用户名和密码都为root。关闭从库后，在navicat客户端中使用insert语句添加数据后到主库上查看是否写入了数据。在navicat查数据时还是从库的数据。

### 双主复制实战

两台主库互为主从。

#### 主库A配置

编写配置文件：vim /etc/my.cnf

```
# log_bin
# 开启logbin功能
log-bin=mysql-bin
# 设置serverId
server-id=1
# 每次执行写操作都与磁盘同步
sync-binlog=1
# 指定不需要同步的数据库
binlog-ignore-db=performance_schema
binlog-ignore-db=information_schema
binlog-ignore-db=sys
# 指定需要同步的数据库
#binlog-do-db=lagou

# 开启中继日志
relay_log=mysql-relay-bin
log_slave_updates=1

# 1,3,5,7...
# 双主双写需要指定初始值
auto_increment_offset=1
# 自增id步长
auto_increment_increment=2
#
```

重启mysql服务：systemctl restart mysqld

登陆myql后进行授权：

```sql
grant replication slave on *.* to 'root'@'%' identified by 'root';
grant all privileges on *.* to 'root'@'%' identified by 'root';
flush privileges;
show master status;	  #显示主库信息，有binlog文件名称
```

master参数信息：

```
mysql> show master status;
+------------------+----------+--------------+-------------------------------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB                          | Executed_Gtid_Set |
+------------------+----------+--------------+-------------------------------------------+-------------------+
| mysql-bin.000002 |      154 |              | performance_schema,information_schema,sys |                   |

```

从库配置：指定主库的相关信息，服务器IP，端口号，用户名，密码，日志文件，开始地址。

```sql
change master to master_host='192.168.91.107',master_port=3306,master_user='root',master_password='root',master_log_file='mysql-bin.000001',master_log_pos=884;

start slave;
show slave status \G;
```

#### 主库B配置

编写配置文件：vim /etc/my.cnf

```
# 开启logbin功能
log-bin=mysql-bin
# 设置serverId
server-id=3
# 每次执行写操作都与磁盘同步
sync-binlog=1
# 指定不需要同步的数据库
binlog-ignore-db=performance_schema
binlog-ignore-db=information_schema
binlog-ignore-db=sys

# 开启中继日志
relay_log=mysql-relay-bin
log_slave_updates=1

# 2,4,6,8...
# 双主双写需要指定初始值
auto_increment_offset=2
# 自增id步长
auto_increment_increment=2
#
```

重启mysql服务：systemctl restart mysqld

登陆myql后进行授权：

```sql
grant replication slave on *.* to 'root'@'%' identified by 'root';
grant all privileges on *.* to 'root'@'%' identified by 'root';
flush privileges;
show master status;	  #显示主库信息，有binlog文件名称
```

master参数信息：

```
mysql> show master status;
+------------------+----------+--------------+-------------------------------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB                          | Executed_Gtid_Set |
+------------------+----------+--------------+-------------------------------------------+-------------------+
| mysql-bin.000001 |      884 |              | performance_schema,information_schema,sys |                   |

```

从库配置：指定主库的相关信息，服务器IP，端口号，用户名，密码，日志文件，开始地址。

```sql
change master to master_host='192.168.91.105',master_port=3306,master_user='root',maaster_password='root',master_log_file='mysql-bin.000002',master_log_pos=154;

start slave;
show slave status \G;
```

#### 效果演示

主库A：

```sql
create database mymaster1;
use mymaster1;
create table test1(id int primary key auto_increment,name varchar(20))engine=innodb charset=utf8;
insert into test1(name) values('a');
insert into test1(name) values('b');
select * from test1;
```

效果：

```
mysql> select * from test1;
+----+------+
| id | name |
+----+------+
|  1 | a    |
|  3 | b    |
+----+------+
```

主库B：

```sql
show databases;
use mymaster1;
show tables;
insert into test1(name) values('1');
select * from test1;
```

效果：

```
mysql> select * from test1;
+----+------+
| id | name |
+----+------+
|  1 | a    |
|  3 | b    |
|  4 | 1    |
+----+------+
```

