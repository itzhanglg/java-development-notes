## 作业习题

业务场景：用户在拉勾网投递简历时，我们会为每次投递的简历生成一份快照，将快照信息存储到

MongoDB中。

功能需求：搭建MongoDB分片集群，模拟简历快照数据进行操作，具体要求如下：

(1) 如图搭建一个分片集群，要求每个分片节点中的复制集含有一个仲裁节点

(2) 使用权限控制，建立要访问的数据库lg_resume，这个账号名字是lagou_gx、密码是abc321

这个账号对数据库有读写权限

(3) 使用SpringBoot 进行访问分片集群，对lg_resume 库中的lg_resume_datas 进行增加和查询操作

## 环境介绍

1.环境介绍涉及的各个软件的版本

| 环境&软件                 | 版本  |
| ------------------------- | ----- |
| 虚拟机&VMware Workstation | 10.0  |
| 服务器&CentOS             | 7.8   |
| 数据库&MongoDB            | 4.1.3 |
| 远程连接&Xshell           | 6     |
| 远程文件传输&WinSCP、Xftp | 6     |

2.介绍各个机器对应角色&作用&ip地址

架构如图所示，1台机器的IP和角色如下表：

| 机器名称          | IP             | 端口              | 角色         | 权限         |
| ----------------- | -------------- | ----------------- | ------------ | ------------ |
| Config Server集群 | 192.168.91.111 | 17011/17013/17015 | 配置服务器   | 提供配置信息 |
| shard1 集群       | 192.168.91.111 | 37011/37013/37015 | 复制集1      | 提供读写     |
|                   |                | 37017             | 仲裁节点     | 裁判         |
| shard2 集群       | 192.168.91.111 | 47011/47013/47015 | 复制集2      | 提供读写     |
|                   |                | 47017             | 仲裁节点     | 裁判         |
| shard3 集群       | 192.168.91.111 | 57011/57013/57015 | 复制集3      | 提供读写     |
|                   |                | 57017             | 仲裁节点     | 裁判         |
| shard4 集群       | 192.168.91.111 | 58011/58013/58015 | 复制集4      | 提供读写     |
|                   |                | 58017             | 仲裁节点     | 裁判         |
| Router Server节点 | 192.168.91.111 | 27017             | 数据分片路由 | 路由         |

架构图：

![image-20200910034247279](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200910034247279.png)

- MongoDB：是一个基于分布式文件存储的数据库。体系结构。数据模型为内嵌或引用，存储引擎为WiredTiger。
    - 存储引擎优势：文档空间分配方式、并发级别、数据压缩、Cache使用、内存使用
    - 存储引擎实现原理： journal日志（WAL）、每60s或Log文件达到2G就checkpoint产生快照文件
- 主从复制master-slave：采用的是 oplog 日志，从服务器会定期从主服务器中获取oplog记录，再执行回放oplog集合是固定大小；主从结构没有自动故障转移功能
- 复制集replica sets：成员包括Primary主节点（读写）,secondary从节点（读）和vote投票节点
    - 高可用、灾难恢复、功能隔离
    - oplog（全量同步和增量同步）+ heartbeat（任意两个节点）+ vote（二阶段过程+多数派协议）
- 分片集群shard-cluster：分片集群由以下3个服务组成
    - 配置服务器：存储所有数据库元信息（路由、分片）的配置
    - 路由服务器：数据库集群的请求入口，负责把应用程序的请求转发到对应的Shard服务器上
    - 复制集：存储数据

## 实现步骤

### Config集群

#### prepare

tar -xvf mongodb-linux-x86_64-4.1.3.tgz

mv mongodb-linux-x86_64-4.1.3 shard-cluster

cd shard-cluster/

mkdir config route shard

#### cd config

mkdir config1 config2 config3 logs

vim config-17011.conf

```
# 数据库文件位置
dbpath=config/config1
#日志文件位置
logpath=config/logs/config1.log
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=0.0.0.0
port = 17011
# 表示是一个配置服务器
configsvr=true
#配置服务器副本集名称
replSet=configsvr

# 数据库文件位置
dbpath=config/config2
#日志文件位置
logpath=config/logs/config2.log
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=0.0.0.0
port = 17013
# 表示是一个配置服务器
configsvr=true
#配置服务器副本集名称
replSet=configsvr

# 数据库文件位置
dbpath=config/config3
#日志文件位置
logpath=config/logs/config3.log
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=0.0.0.0
port = 17015
# 表示是一个配置服务器
configsvr=true
#配置服务器副本集名称
replSet=configsvr
```

cp config-17011.conf config-17013.conf

cp config-17011.conf config-17015.conf

vim config-17013.conf

vim config-17015.conf

启动每个mongod 

./bin/mongod -f config/config-17011.conf

./bin/mongod -f config/config-17013.conf

./bin/mongod -f config/config-17015.conf

然后进入其中一个进行集群配置

./bin/mongo --port 17011

```
use admin

var cfg ={"_id":"configsvr",
"members":[
{"_id":1,"host":"192.168.91.111:17011"},
{"_id":2,"host":"192.168.91.111:17013"},
{"_id":3,"host":"192.168.91.111:17015"}]
};

rs.initiate(cfg)
```

### Shard集群

cd shard/

mkdir shard1 shard2 shard3 shard4

#### cd shard1

mkdir shard1-37011 shard1-37013 shard1-37015 shard1-37017 logs

vim shard1-37011.conf

```
dbpath=shard/shard1/shard1-37011
bind_ip=0.0.0.0
port=37011
fork=true
logpath=shard/shard1/logs/shard1-37011.log
replSet=shard1
shardsvr=true

dbpath=shard/shard1/shard1-37013
bind_ip=0.0.0.0
port=37013
fork=true
logpath=shard/shard1/logs/shard1-37013.log
replSet=shard1
shardsvr=true

dbpath=shard/shard1/shard1-37015
bind_ip=0.0.0.0
port=37015
fork=true
logpath=shard/shard1/logs/shard1-37015.log
replSet=shard1
shardsvr=true

dbpath=shard/shard1/shard1-37017
bind_ip=0.0.0.0
port=37017
fork=true
logpath=shard/shard1/logs/shard1-37017.log
replSet=shard1
shardsvr=true 
```

cp shard1-37011.conf shard1-37013.conf

cp shard1-37011.conf shard1-37015.conf

cp shard1-37011.conf shard1-37017.conf

vim shard1-37013.conf

vim shard1-37015.conf

vim shard1-37017.conf

启动每个mongod 

./bin/mongod -f shard/shard1/shard1-37011.conf

./bin/mongod -f shard/shard1/shard1-37013.conf

./bin/mongod -f shard/shard1/shard1-37015.conf

./bin/mongod -f shard/shard1/shard1-37017.conf

然后进入其中一个进行集群配置

./bin/mongo --port 37011

```
var cfg ={"_id":"shard1",
"protocolVersion" : 1,
"members":[
{"_id":1,"host":"192.168.91.111:37011"},
{"_id":2,"host":"192.168.91.111:37013"},
{"_id":3,"host":"192.168.91.111:37015"},
{"_id":4,"host":"192.168.91.111:37017","arbiterOnly":true}
]
};

rs.initiate(cfg)
rs.status()

rs.addArb("192.168.91.106:37017")
```

#### cd shard2

mkdir shard2-47011 shard2-47013 shard2-47015 shard2-47017 logs

vim shard2-47011.conf

```
dbpath=shard/shard2/shard2-47011
bind_ip=0.0.0.0
port=47011
fork=true
logpath=shard/shard2/logs/shard2-47011.log
replSet=shard2
shardsvr=true

dbpath=shard/shard2/shard2-47013
bind_ip=0.0.0.0
port=47013
fork=true
logpath=shard/shard2/logs/shard2-47013.log
replSet=shard2
shardsvr=true

dbpath=shard/shard2/shard2-47015
bind_ip=0.0.0.0
port=47015
fork=true
logpath=shard/shard2/logs/shard2-47015.log
replSet=shard2
shardsvr=true

dbpath=shard/shard2/shard2-47017
bind_ip=0.0.0.0
port=47017
fork=true
logpath=shard/shard2/logs/shard2-47017.log
replSet=shard2
shardsvr=true
```

cp shard2-47011.conf shard2-47013.conf

cp shard2-47011.conf shard2-47015.conf

cp shard2-47011.conf shard2-47017.conf

vim shard2-47013.conf

vim shard2-47015.conf

vim shard2-47017.conf

启动每个mongod 

./bin/mongod -f shard/shard2/shard2-47011.conf

./bin/mongod -f shard/shard2/shard2-47013.conf

./bin/mongod -f shard/shard2/shard2-47015.conf

./bin/mongod -f shard/shard2/shard2-47017.conf

然后进入其中一个进行集群配置

./bin/mongo --port 47011

```
var cfg ={"_id":"shard2",
"protocolVersion" : 1,
"members":[
{"_id":1,"host":"192.168.91.111:47011"},
{"_id":2,"host":"192.168.91.111:47013"},
{"_id":3,"host":"192.168.91.111:47015"},
{"_id":4,"host":"192.168.91.111:47017","arbiterOnly":true}
]
};

rs.initiate(cfg)
rs.status()
```

#### cd shard3

mkdir shard3-57011 shard3-57013 shard3-57015 shard3-57017 logs

vim shard3-57011.conf

```
dbpath=shard/shard3/shard3-57011
bind_ip=0.0.0.0
port=57011
fork=true
logpath=shard/shard3/logs/shard3-57011.log
replSet=shard3
shardsvr=true

dbpath=shard/shard3/shard3-57013
bind_ip=0.0.0.0
port=57013
fork=true
logpath=shard/shard3/logs/shard3-57013.log
replSet=shard3
shardsvr=true

dbpath=shard/shard3/shard3-57015
bind_ip=0.0.0.0
port=57015
fork=true
logpath=shard/shard3/logs/shard3-57015.log
replSet=shard3
shardsvr=true

dbpath=shard/shard3/shard3-57017
bind_ip=0.0.0.0
port=57017
fork=true
logpath=shard/shard3/logs/shard3-57017.log
replSet=shard3
shardsvr=true
```

cp shard3-57011.conf shard3-57013.conf

cp shard3-57011.conf shard3-57015.conf

cp shard3-57011.conf shard3-57017.conf

vim shard3-57013.conf

vim shard3-57015.conf

vim shard3-57017.conf

启动每个mongod 

./bin/mongod -f shard/shard3/shard3-57011.conf

./bin/mongod -f shard/shard3/shard3-57013.conf

./bin/mongod -f shard/shard3/shard3-57015.conf

./bin/mongod -f shard/shard3/shard3-57017.conf

然后进入其中一个进行集群配置

./bin/mongo --port 57011

```
var cfg ={"_id":"shard3",
"protocolVersion" : 1,
"members":[
{"_id":1,"host":"192.168.91.111:57011"},
{"_id":2,"host":"192.168.91.111:57013"},
{"_id":3,"host":"192.168.91.111:57015"},
{"_id":4,"host":"192.168.91.111:57017","arbiterOnly":true}
]
};

rs.initiate(cfg)
rs.status()
```

#### cd shard4

mkdir shard4-58011 shard4-58013 shard4-58015 shard4-58017 logs

vim shard4-58011.conf

```
dbpath=shard/shard4/shard4-58011
bind_ip=0.0.0.0
port=58011
fork=true
logpath=shard/shard4/logs/shard4-58011.log
replSet=shard4
shardsvr=true

dbpath=shard/shard4/shard4-58013
bind_ip=0.0.0.0
port=58013
fork=true
logpath=shard/shard4/logs/shard4-58013.log
replSet=shard4
shardsvr=true

dbpath=shard/shard4/shard4-58015
bind_ip=0.0.0.0
port=58015
fork=true
logpath=shard/shard4/logs/shard4-58015.log
replSet=shard4
shardsvr=true

dbpath=shard/shard4/shard4-58017
bind_ip=0.0.0.0
port=58017
fork=true
logpath=shard/shard4/logs/shard4-58017.log
replSet=shard4
shardsvr=true
```

cp shard4-58011.conf shard4-58013.conf

cp shard4-58011.conf shard4-58015.conf

cp shard4-58011.conf shard4-58017.conf

vim shard4-58013.conf

vim shard4-58015.conf

vim shard4-58017.conf

启动每个mongod 

./bin/mongod -f shard/shard4/shard4-58011.conf

./bin/mongod -f shard/shard4/shard4-58013.conf

./bin/mongod -f shard/shard4/shard4-58015.conf

./bin/mongod -f shard/shard4/shard4-58017.conf

然后进入其中一个进行集群配置

./bin/mongo --port 58011

```
var cfg ={"_id":"shard4",
"protocolVersion" : 1,
"members":[
{"_id":1,"host":"192.168.91.111:58011"},
{"_id":2,"host":"192.168.91.111:58013"},
{"_id":3,"host":"192.168.91.111:58015"},
{"_id":4,"host":"192.168.91.111:58017","arbiterOnly":true}
]
};

rs.initiate(cfg)
rs.status()
```

### 路由节点

#### prepare

cd route/

mkdir logs

vim route-27017.conf

```
port=27017
bind_ip=0.0.0.0
fork=true
logpath=route/logs/route.log
configdb=configsvr/192.168.91.111:17011,192.168.91.111:17013,192.168.91.111:17015
```

启动路由节点使用 mongos（注意不是mongod）

> ./bin/mongos -f route/route-27017.conf

#### mongos（路由）中添加分片节点

进入路由mongos

./bin/mongo --port 27017

添加分片节点时不能添加 冲载 节点的信息

```
sh.status()   
sh.addShard("shard1/192.168.91.111:37011,192.168.91.111:37013,192.168.91.111:37015,192.168.91.106:37017");
sh.addShard("shard2/192.168.91.111:47011,192.168.91.111:47013,192.168.91.111:47015,192.168.91.106:47017");
sh.addShard("shard3/192.168.91.111:57011,192.168.91.111:57013,192.168.91.111:57015,192.168.91.106:57017");
sh.addShard("shard4/192.168.91.111:58011,192.168.91.111:58013,192.168.91.111:58015,192.168.91.106:58017");
sh.status()
```

正确添加各个节点信息：

```
sh.status()   
sh.addShard("shard1/192.168.91.111:37011,192.168.91.111:37013,192.168.91.111:37015");
sh.addShard("shard2/192.168.91.111:47011,192.168.91.111:47013,192.168.91.111:47015");
sh.addShard("shard3/192.168.91.111:57011,192.168.91.111:57013,192.168.91.111:57015");
sh.addShard("shard4/192.168.91.111:58011,192.168.91.111:58013,192.168.91.111:58015");
sh.status()
```

#### 开启数据库和集合分片(指定片键)

继续使用 mongos 完成分片开启和分片大小设置

```
为数据库开启分片功能
sh.enableSharding("lagou_resume")
为指定集合开启分片功能
sh.shardCollection("lagou_resume.lagou_resume_datas",{"片键字段名如 name":索引说明})

sh.shardCollection("lagou_resume.lagou_resume_datas",{"name":"hashed"})
```

#### 向集合中插入数据测试

通过路由循环向集合中添加数

```
use  lagou_resume;
for(var i=1;i<= 1000;i++){
  db.lagou_resume_datas.insert({"name":"test"+i,
    salary:(Math.random()*20000).toFixed(2)});
}
```

验证分片效果

分别进入 shard1、shard2、shard3和shard4 中的数据库 进行验证

### 分片集群安全认证

1.进入路由创建**管理员**和**普通用户**

```
// 创建管理员
use admin
db.createUser({
	user:"root",
	pwd:"123456",
	roles:[{role:"root",db:"admin"}]
})

// 创建普通用户
use lagou_resume
// 读写
db.createUser({
	user:"zlg",
	pwd:"123456",
	roles:[{role:"readWrite",db:"lagou_resume"}]
})
// 只读
db.createUser({
	user:"zlg",
	pwd:"123456",
	roles:[{role:"read",db:"lagou_resume"}]
})
```

2.关闭所有的**配置节点、分片节点 和 路由节点**

安装psmisc ：

> yum install psmisc

安装完之后可以使用killall 命令 快速关闭多个进程

> killall mongod
>
> killall mongo
>
> killall mongos
>
> ps -ef|grep mongo

3.**生成密钥文件 并修改权限**

在 shard-cluster 根目录下：

>  mkdir data/mongodb -p
>
> openssl rand -base64 756 > data/mongodb/testKeyFile.file
>
> chmod 600 data/mongodb/keyfile/testKeyFile.file

4.**配置节点**集群和**分片节点**集群开启安全认证和指定密钥文件

每个mongo的配置文件中都需要添加下列代码：

```
auth=true
keyFile=data/mongodb/testKeyFile.file
```

5.在**路由配置文件**中设置密钥文件

路由中不需要开启auth=true安全认证

> keyFile=data/mongodb/testKeyFile.file

6.启动**所有的配置节点 分片节点 和 路由节点** 使用路由进行权限验证

在 shard-cluster 根目录下：

编写一个shell 脚本 批量启动：vim startup.sh

```
./bin/mongod -f config/config-17011.conf
./bin/mongod -f config/config-17013.conf
./bin/mongod -f config/config-17015.conf
./bin/mongod -f shard/shard1/shard1-37011.conf
./bin/mongod -f shard/shard1/shard1-37013.conf
./bin/mongod -f shard/shard1/shard1-37015.conf
./bin/mongod -f shard/shard1/shard1-37017.conf
./bin/mongod -f shard/shard2/shard2-47011.conf
./bin/mongod -f shard/shard2/shard2-47013.conf
./bin/mongod -f shard/shard2/shard2-47015.conf
./bin/mongod -f shard/shard2/shard2-47017.conf
./bin/mongod -f shard/shard3/shard3-57011.conf
./bin/mongod -f shard/shard3/shard3-57013.conf
./bin/mongod -f shard/shard3/shard3-57015.conf
./bin/mongod -f shard/shard3/shard3-57017.conf
./bin/mongod -f shard/shard4/shard4-58011.conf
./bin/mongod -f shard/shard4/shard4-58013.conf
./bin/mongod -f shard/shard4/shard4-58015.conf
./bin/mongod -f shard/shard4/shard4-58017.conf
./bin/mongos -f route/route-27017.conf
```

修改执行权限：chmod +x startup.sh

启动：./startup.sh

启动成功后效果：

![image-20200910054127683](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200910054127683.png)

7.**进入路由权限验证**：./bin/mongo --port 27017

```
use lagou_resume
db.auth("zlg","123456")

use admin
db.auth("root","123456")

// 测试看是否有权限
show dbs
db.lagou_resume_datas.find()
```

8.**Spring boot 连接安全认证的分片集群**

配置文件：

```
spring.data.mongodb.host=192.168.91.105
spring.data.mongodb.port=27017
spring.data.mongodb.database=lagou_resume
spring.data.mongodb.username=zlg
spring.data.mongodb.password=123456

#spring.data.mongodb.uri=mongodb://账号:密码@IP:端口/数据库名
```





### 开发问题

#### Linux 根目录爆满（/dev/mapper/centos-root 100%）

使用 `df -h`命令查看硬盘空间使用情况

进入磁盘占用过多的目录。如 `cd /` 进入到根目录

使用 `du -h -x --max-depth=1` 查看哪个目录占用过高

删除资源占用过多的文件











