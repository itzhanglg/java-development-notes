### 习题

基于RocketMQ设计秒杀。

要求：

1. 秒杀商品LagouPhone，数量100个。

2. 秒杀商品不能超卖。

3. 抢购链接隐藏

4. Nginx+Redis+RocketMQ+Tomcat+MySQL

提示：

![image-20201214031553629](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214031553629.png)

### 实现思路

秒杀对应到架构设计，其实就是高可用、一致性和高性能的要求。

- 高性能：动静分离、热点优化、系统优化；
- 一致性：下单减库存、付款减库存、预扣库存（常见）【保证卖出去（业务手段）、避免超卖（技术手段）】
    - 高并发读：不同层次尽可能过滤掉无效请求；
    - 高并发写：主要是在存储层作选项和相关性能优化。
- 高可用：流量削峰（答题、排队、过滤）、PlanB。

RocketMQ架构体系：

![image-20201214040657470](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214040657470.png)

### 搭建过程

#### 1.安装Java环境

安装：

```shell
# 上传jdk-11.0.5_linux-x64_bin.rpm到服务器并安装
rpm -ivh jdk-11.0.5_linux-x64_bin.rpm
# 配置环境变量
vim /etc/profile
# profile文件末尾配置
export JAVA_HOME=/usr/java/jdk-11.0.5
export PATH=$JAVA_HOME/bin:$PATH
# 生效
source /etc/profile
```

验证：

```shell
[root@linux100 jdk-11.0.5]# java -version
java version "11.0.5" 2019-10-15 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.5+10-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.5+10-LTS, mixed mode)
```

#### 2.安装RocketMQ

##### 2.1 准备工作

1.下载RocketMQ安装包：rocketmq-all-4.5.1-bin-release.zip

> wget https://archive.apache.org/dist/rocketmq/4.5.1/rocketmq-all-4.5.1-bin-release.zip

2.解压

> unzip rocketmq-all-4.5.1-bin-release.zip -d /opt/servers/
>
> 重命名：mv rocketmq-all-4.5.1-bin-release/ rocket

3.配置环境变量

```shell
# 编辑
vim /etc/profile
# 配置
export ROCKET_HOME=/opt/servers/rocket
export PATH=$PATH:$ROCKET_HOME/bin
# 生效
source /etc/profile
```

##### 2.2 修改脚本

修改脚本：`:%d` 删除脚本内容。注：java1.8无需要改任何配置即可启动。

vim bin/runserver.sh

```shell
vim bin/runserver.sh
# 删除
UseCMSCompactAtFullCollection
UseParNewGC
UseConcMarkSweepGC
# 修改内存：
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=64mm -XX:MaxMetaspaceSize=160mm"
-Xloggc 修改为 -Xlog:gc
```

改动后脚本：

```shell
#!/bin/sh
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements. See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#===========================================================================================
# Java Environment Setting
#===========================================================================================
error_exit ()
{
    echo "ERROR: $1 !!"
	exit 1
}
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"
export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
export BASE_DIR=$(dirname $0)/..
export CLASSPATH=.:${BASE_DIR}/conf:${JAVA_HOME}/jre/lib/ext:${BASE_DIR}/lib/*
#===========================================================================================
# JVM Configuration
#===========================================================================================
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=64m -XX:MaxMetaspaceSize=160m"
JAVA_OPT="${JAVA_OPT} -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8"
JAVA_OPT="${JAVA_OPT} -verbose:gc -Xlog:gc:/dev/shm/rmq_srv_gc.log -XX:+PrintGCDetails"
JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
JAVA_OPT="${JAVA_OPT} -XX:-UseLargePages"
# JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${JAVA_HOME}/jre/lib/ext:${BASE_DIR}/lib"
#JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
JAVA_OPT="${JAVA_OPT} ${JAVA_OPT_EXT}"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"

$JAVA ${JAVA_OPT} $@
```

vim bin/runbroker.sh

```shell
vim bin/runbroker.sh
# 删除：
PrintGCDateStamps
PrintGCApplicationStoppedTime
PrintAdaptiveSizePolicy
UseGCLogFileRotation
NumberOfGCLogFiles=5
GCLogFileSize=30m
```

改动后脚本：

```shell
#!/bin/sh

# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements. See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#===========================================================================================
# Java Environment Setting
#===========================================================================================
error_exit ()
{
	echo "ERROR: $1 !!"
	exit 1
}

[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"

export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
export BASE_DIR=$(dirname $0)/..
export CLASSPATH=.${JAVA_HOME}/jre/lib/ext:${BASE_DIR}/lib/*:${BASE_DIR}/conf:${CLASSPATH}
#===========================================================================================
# JVM Configuration
#===========================================================================================
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0"
JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:/dev/shm/mq_gc_%p.log -XX:+PrintGCDetails"
JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
JAVA_OPT="${JAVA_OPT} -XX:+AlwaysPreTouch"
JAVA_OPT="${JAVA_OPT} -XX:MaxDirectMemorySize=15g"
JAVA_OPT="${JAVA_OPT} -XX:-UseLargePages -XX:-UseBiasedLocking"
#JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
JAVA_OPT="${JAVA_OPT} ${JAVA_OPT_EXT}"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"

numactl --interleave=all pwd > /dev/null 2>&1
if [ $? -eq 0 ]
then
	if [ -z "$RMQ_NUMA_NODE" ] ; then
		numactl --interleave=all $JAVA ${JAVA_OPT} $@
	else
		numactl --cpunodebind=$RMQ_NUMA_NODE --membind=$RMQ_NUMA_NODE $JAVA ${JAVA_OPT} $@
	fi
else
	$JAVA ${JAVA_OPT} --add-exports=java.base/jdk.internal.ref=ALL-UNNAMED $@
fi
```

vim bin/tools.sh

```shell
vim bin/tools.sh
# 删除下面语句即可
# JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib:${JAVA_HOME}/jre/lib/ext"
```

改动后脚本：

```shell
#!/bin/sh

# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements. See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#===========================================================================================
# Java Environment Setting
#===========================================================================================
error_exit ()
{
	echo "ERROR: $1 !!"
	exit 1
}

[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"

export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
export BASE_DIR=$(dirname $0)/..
# export CLASSPATH=.:${BASE_DIR}/conf:${CLASSPATH}
export CLASSPATH=.${JAVA_HOME}/jre/lib/ext:${BASE_DIR}/lib/*:${BASE_DIR}/conf:${CLASSPATH}
#===========================================================================================
# JVM Configuration
#===========================================================================================
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn256m -XX:PermSize=128m -XX:MaxPermSize=128m"
# JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib:${JAVA_HOME}/jre/lib/ext"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"

$JAVA ${JAVA_OPT} $@
```

#### 3.RocketMQ环境测试

1.启动rocketmq

```shell
# 1.先启动NameServer
[root@atzlg8 rocket]# mqnamesrv
# 2.再启动Broker
[root@atzlg8 rocket]# mqbroker -n localhost:9876

# 动态查看namesrv和broker的启动日志
[root@atzlg8 rocket]# tail -f ~/logs/rocketmqlogs/namesrv.log
[root@atzlg8 rocket]# tail -f ~/logs/rocketmqlogs/broker.log
```

2.发送消息

```shell
# 设置环境变量并启动生产者
export NAMESRV_ADDR=localhost:9876 && tools.sh org.apache.rocketmq.example.quickstart.Producer

# 1.设置环境变量
export NAMESRV_ADDR=localhost:9876
# 2.使用安装包的Demo发送消息
tools.sh org.apache.rocketmq.example.quickstart.Producer
```

3.接受消息

```shell
# 设置环境变量并启动消费者
export NAMESRV_ADDR=localhost:9876 && tools.sh org.apache.rocketmq.example.quickstart.Consumer

# 1.设置环境变量
export NAMESRV_ADDR=localhost:9876
# 2.接收消息
tools.sh org.apache.rocketmq.example.quickstart.Consumer
```

4.关闭rocketmq

```shell
# 1.先关闭broker
[root@linux100 rocketmqlogs]# mqshutdown broker
# 2.再关闭namesrv
[root@linux100 rocketmqlogs]# mqshutdown namesrv
```

#### 4.RocketMQ多实例部署

```shell
# 将jdk传送到另一台服务器上/opt/applets目录下
[root@linux100 software]# scp jdk-11.0.5_linux-x64_bin.rpm 192.168.91.101:/opt/applets

# 将rocket安装目录复制传送到另一台服务器上/opt/servers目录下
[root@linux100 servers]# scp -r rocket/ 192.168.91.101:/opt/servers

# 安装jdk
[root@linux101 applets]# rpm -ivh jdk-11.0.5_linux-x64_bin.rpm 

[root@linux101 applets]# vim /etc/profile
# 配置环境变量
export JAVA_HOME=/usr/java/jdk-11.0.5
export PATH=$JAVA_HOME/bin:$PATH

export ROCKET_HOME=/opt/servers/rocket
export PATH=$PATH:$ROCKET_HOME/bin
[root@linux101 applets]# source /etc/profile

# 启动linux100的nameserver
[root@linux100 ~]# mqnamesrv
# 启动linux101的nameserver
[root@linux101 ~]# mqnamesrv
# 启动broker：
[root@linux100 ~]# mqbroker -n 'linux101:9876;linux100:9876' -c /opt/rocketmq-all-4.5.1-bin-release/conf/m-s-sync/broker-a.properties
[root@linux101 ~]# mqbroker -n 'linux101:9876;linux100:9876' -c /opt/rocketmq-all-4.5.1-bin-release/conf/m-s-sync/broker-a-s.properties
```

#### 5.创建数据库

![image-20201214032128417](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214032128417.png)

```sql
-- 创建数据库
create database db_flashsale;

use db_flashsale;

create table `tb_order` (
	`order_id` varchar(255) not null,
	`user_id` varchar(255) default null,
	`order_status` char(255) default null,
	`content` varchar(255) default null,
	primary key (`order_id`)
);
```

#### 6.安装Redis

```shell
# 下载依赖和安装包
yum install wget -y
yum install gcc -y
wget https://download.redis.io/releases/redis-5.0.8.tar.gz
# 解压
tar -zxf redis-5.0.8.tar.gz
# 编译和安装
cd redis-5.0.8
make && make install
# 将redis.conf文件复制到/usr/local/bin/目录下
cd /usr/local/bin/
cp ~/redis-5.0.8/redis.conf .
# 启动redis服务端
redis-server redis.conf
```

### 效果演示

网页：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214035952358.png" alt="image-20201214035952358" style="zoom:60%;" />

下单：

![image-20201214035443223](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214035443223.png)

消费（未支付）：

![image-20201214035541430](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214035541430.png)







