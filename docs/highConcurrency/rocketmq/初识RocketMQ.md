##  一.RocketMQ架构与实战

### 1.RocketMQ概述

RocketMQ是阿里巴巴中间件团队自研的一款**高性能、高吞吐量、低延迟、高可用、高可靠**（具备金融级稳定性）的分布式消息中间件。官网：[RocketMQ](https://github.com/apache/rocketmq)

RocketMQ主要的几个使用场景：

- **应用解耦**：系统的耦合性越高，容错性就越低。如用户创建订单后，耦合库存系统、物流系统、支付系统，只要其中一个子系统出故障或其它原因暂时不可用，都会造成下单操作异常。
- **流量削峰**：应用系统如果遇到系统请求流量的瞬间猛增，有可能会将系统压垮。消息队列可以将大量请求缓存起来，分散到很长一段时间处理，这样可以大大提到系统的稳定性和用户体验。
- **数据分发**：通过消息队列可以让数据在多个系统之间进行流通。数据的产生方不需要关心谁来使用数据，只需要将数据发送到消息队列，数据使用方直接在消息队列中直接获取数据即可。

RocketMQ角色：

- NameServer：管理Broker。一个几乎无状态节点，可集群部署，节点之间无任何信息同步。
- Broker：暂存和传输消息。Broker分为Master与Slave，一个Master可以对应多个Slave，Master与Slave 的对应关系通过指定**相同的BrokerName**，不同的BrokerId来定义，**BrokerId为0表示Master，非0表示Slave**。Master也可以部署多个。**每个Broker与NameServer集群中的所有节点建立长连接，定时注册Topic信息到所有NameServer**。 注意：当前RocketMQ版本在部署架构上支持一Master多Slave，但只有**BrokerId=1的从服务器才会参与消息的读负载**。
- Producer：消息的发送者。与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer获取Topic路由信息。并向提供Topic 服务的Master建立长连接，且**定时向Master发送心跳**。Producer完全无状态，可集群部署。
- Consumer：消息接收者。与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer获取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且**定时向Master、Slave发送心跳**。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，消费者在向Master拉取消息时，Master服务器会根据拉取偏移量与最大偏移量的距离（判断是否读老消息，产生读I/O），以及从服务器是否可读等因素建议下一次是从Master还是Slave拉取。
- Topic：区分消息的种类。一个发送者可以发送消息给一个或者多个Topic；一个消息的接收者可以订阅一个或者多个Topic消息。
- Message Queue：相当于是Topic的分区。用于并行发送和接收消息。

![image-20201213011549590](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201213011549590.png)

执行流程：

1. 启动NameServer，NameServer起来后监听端口，等待Broker、Producer、Consumer连上来，相当于一个路由控制中心。
2. Broker启动，跟**所有**的NameServer保持长连接，定时发送心跳包。心跳包中包含当前Broker信息(IP+端口等)以及存储所有Topic信息。注册成功后，NameServer集群中就有Topic跟Broker的映射关系。
3. 收发消息前，先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic。
4. Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic存在哪些Broker上，**轮询**从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发消息。
5. Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息。

### 2.RocketMQ特性及消费模式

#### 2.1 RocketMQ特性

![image-20201213231128051](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201213231128051.png)

- 订阅与发布：消息的发布是指某个生产者向某个 `topic`发送消息；消息的订阅是指某个消费者关注了某个 `topic`中带有某些tag的消息。
- **消息顺序**：消息有序指的是一类消息消费时，能按照发送的顺序来消费。
- **消息过滤**：RocketMQ的消费者可以根据Tag进行消息过滤，也支持自定义属性过滤。消息过滤目前是在Broker端实现的。
- 消息可靠性：RocketMQ支持消息的高可靠，影响消息可靠性的几种情况：
    - 硬件资源可立即恢复（依赖刷盘方式同步还是异步）：Broker非正常关闭、Broker异常Crash、OS Crash、机器掉电等情况；
    - 单点故障（同步双写技术）：机器无法开机（可能是cpu、主板、内存等关键设备损坏）、磁盘设备损坏
- 至少一次：至少一次(At least Once)指每个消息必须投递一次。Consumer先Pull消息到本地，消费完成后，才向服务器返回ack，如果没有消费一定不会ack消息，所以RocketMQ可以很好的支持此特性。
- 回溯消费：回溯消费是指Consumer已经消费成功的消息，由于业务上需求需要重新消费，要支持此功能，Broker在向Consumer投递成功消息后，消息仍然需要保留。并且重新消费一般是按照时间维度。
- 事务消息：RocketMQ事务消息（Transactional Message）是指应用**本地事务和发送消息操作**可以被定义到全局事务中，要么同时成功，要么同时失败（分布式事务）。
- **定时消息**：定时消息（延迟队列）是指消息发送到broker后，不会立即被消费，等待特定时间投递给真正的topic。broker有配置项 `messageDelayLevel`，默认值为“1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h”，18个level。定时消息会暂存在名为 `SCHEDULE_TOPIC_XXXX`的topic中，并根据delayTimeLevel存入特定的queue， `queueId = delayTimeLevel – 1`，即一个queue只存相同延迟的消息，保证具有相同发送延迟的消息能够顺序消费。broker会调度地消费 `SCHEDULE_TOPIC_XXXX`，将消息写入真实的topic。
- 消息重试：Consumer消费消息失败后，要提供一种重试机制，令消息再消费一次。Consumer消费消息失败通常可以认为有以下几种情况：由于消息本身的原因、依赖的下游应用服务不可用等。
- 消息重投：生产者在发送消息时，【**同步消息**失败会重投，**异步消息**有重试，**oneway**没有任何保证】。如下方法可以设置消息重试策略：
    - `retryTimesWhenSendFailed`：**同步发送失败重投次数，默认为2**，因此生产者会最多尝试发送`retryTimesWhenSendFailed + 1`次。不会选择上次失败的broker，**尝试向其他broker发送**，最大程度保证消息不丢失。
    - `retryTimesWhenSendAsyncFailed`：**异步发送失败重试次数**，异步重试不会选择其他broker，**仅在同一个broker上做重试**，不保证消息不丢。
    - `retryAnotherBrokerWhenNotStoreOK`：**消息刷盘（主或备）超时或slave不可用**（返回状态非SEND_OK），是否尝试发送到**其他broker**，默认false。十分重要消息可以开启。
- 流量控制：生产者流控（不会尝试消息重投），因为broker处理能力达到瓶颈；消费者流控，因为消费能力达到瓶颈。
    - 生产者流控：
        - commitLog文件被锁时间超过osPageCacheBusyTimeOutMills时，参数默认为1000ms，发生流控。
        - 如果开启transientStorePoolEnable = true，且broker为异步刷盘的主机，且transientStorePool中资源不足，拒绝当前send请求，发生流控。
        - broker每隔10ms检查send请求队列头部请求的等待时间，如果超过waitTimeMillsInSendQueue，默认200ms，拒绝当前send请求，发生流控。
        - broker通过拒绝send 请求方式实现流量控制。
    - 消费者流控：
        - 消费者本地缓存消息数超过pullThresholdForQueue时，默认1000。
        - 消费者本地缓存消息大小超过pullThresholdSizeForQueue时，默认100MB。
        - 消费者本地缓存消息跨度超过consumeConcurrentlyMaxSpan时，默认2000。
        - 消费者流控的结果是降低拉取频率。
- 死信队列：死信队列（Dead-Letter Queue）用于处理无法被正常消费的消息（Dead-Letter Message）。当一条消息初次消费失败，消息队列会自动进行消息重试；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，消息队列不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。在RocketMQ中，可以通过使用console控制台对死信队列中的消息进行重发来使得消费者实例再次进行消费。

#### 2.2 消费者模式

RocketMQ消息订阅有两种模式，一种是**Push模式**（MQPushConsumer），即MQServer主动向消费端推送；另外一种是**Pull模式**（MQPullConsumer），即消费端在需要时，主动到MQ Server拉取。**本质都是采用消费端主动拉取的方式，即consumer轮询从broker拉取消息**。

- Push模式特点：实时性高，延迟低。缺点：瞬间推送大量消息时，容易造成消息积压。
- Pull模式特点：可根据自己处理能力量力而行。缺点：如何控制Pull的频率。折中的办法就是长轮询。

Push方式里，consumer把**长轮询的动作封装了，并注册MessageListener监听器**，取到消息后，唤醒MessageListener的consumeMessage()来消费，对用户而言，感觉消息是被推送过来的。

Pull方式里，取消息的过程需要用户自己主动调用，首先通过打算消费的Topic拿到MessageQueue的集合，遍历MessageQueue集合，然后针对每个MessageQueue批量取消息，一次取完后，记录该队列下一次要取的开始offset，直到取完了，再换另一个MessageQueue。

### 3.RocketMQ相关术语

- **消息模式**（Message Model）：ocketMQ主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。每个Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。MessageQueue 用于存储消息的物理地址，每个Topic中的消息地址存储于多个 Message Queue 中。ConsumerGroup 由多个Consumer 实例构成。
- Producer：消息生产者，负责产生消息；Consumer：消息消费者，负责消费消息。
- PushConsumer：消费的一种类型，该模式下Broker收到数据后会主动推送给消费端。
- PullConsumer：消费的一种类型，应用通常主动调用Consumer的拉消息方法从Broker服务器拉消息、主动权由应用控制。 
- ProducerGroup：同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。
- ConsumerGroup：同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。**消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易**。消费者组的消费者实例必须**订阅完全相同的Topic**。RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。
- Broker：消息中转角色，负责存储消息，转发消息，一般也称为 Server。在 JMS 规范中称为Provider。
- **广播消费**：一条消息被多个 Consumer 消费，即使这些 Consumer 属于同一个 Consumer Group，消息也会被 Consumer Group 中的每个 Consumer 都消费一次。
- **集群消费**：一个 Consumer Group 中的 Consumer 实例平均分摊消费消息。
- 顺序消息：消费消息的顺序要同发送消息的顺序一致，在RocketMQ 中主要指的是局部顺序。
- 普通顺序消息：顺序消息的一种，正常情况下可以保证完全的顺序消息，但是一旦发生通信异常，Broker 重启，由于队列总数发生发化，哈希取模后定位的队列会发化，产生短暂的消息顺序不一致。
- 严格顺序消息：顺序消息的一种，无论正常异常情况都能保证顺序，但是牺牲了分布式 Failover特性，即Broker集群中只要有一台机器不可用，则整个集群都不可用，服务可用性大大降低。 
- MessageQueue：在 RocketMQ 中，所有消息队列都是持久化的，长度无限的数据结构，所谓长度无限是指队列中的每个存储单元都是定长，访问其中的存储单元使用Offset来访问，offset 为 java long 类型，64 位。
- 标签（Tag）：为消息设置的标志，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。

### 4.RocketMQ安装与配置

#### 4.1 安装Java环境

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

#### 4.2 安装RocketMQ

##### 4.2.1 准备工作

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

##### 4.2.2 修改脚本

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

#### 4.3 RocketMQ环境测试

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

#### 4.4 RocketMQ多实例部署

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

[root@linux100 ~]# mqnamesrv

[root@linux101 ~]# mqnamesrv

[root@linux101 ~]# mqbroker -n 'linux101:9876;linux100:9876'
The broker[linux101, 192.168.91.101:10911] boot success. serializeType=JSON and name server is linux101:9876;linux100:9876
```

### 5.RocketMQ实战

#### 5.1 Java端相关API使用

##### 5.1.1 消息生产者

生产者同步发送消息：

```java
/**
 * RocketMQ相关API使用
 * 同步发送消息
 */
public class MyProducer {

    public static void main(String[] args) throws UnsupportedEncodingException, InterruptedException, RemotingException, MQClientException, MQBrokerException {
        // 在实例化生产者的同时，指定了生产组名称
        DefaultMQProducer producer = new DefaultMQProducer("myproducer_grp_01");

        // 指定NameServer的地址
        producer.setNamesrvAddr("192.168.91.100:9876");

        // 对生产者进行初始化，然后就可以使用了
        producer.start();

        // 创建消息，第一个参数是主题名称，第二个参数是消息内容
        Message message = new Message(
                "tp_demo_01",
                "hello lagou 01".getBytes(RemotingHelper.DEFAULT_CHARSET)
        );
        // 发送消息
        final SendResult result = producer.send(message);
        System.out.println(result);

        // 关闭生产者
        producer.shutdown();
    }

}
```

生产者异步发送消息：

```java
/**
 * RocketMQ相关API使用
 * 异步发送消息
 */
public class MyAsyncProducer {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException {
        // 实例化生产者，并指定生产组名称
        DefaultMQProducer producer = new DefaultMQProducer("producer_grp_01");

        // 指定nameserver的地址
        producer.setNamesrvAddr("192.168.91.100:9876");

        // 初始化生产者
        producer.start();

        for (int i = 0; i < 100; i++) {

            Message message = new Message(
                    "tp_demo_02",
                    ("hello lagou " + i).getBytes("utf-8")
            );

            // 消息的异步发送
            producer.send(message, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println("发送成功:" + sendResult);
                }

                @Override
                public void onException(Throwable throwable) {
                    System.out.println("发送失败：" + throwable.getMessage());
                }
            });
        }

        // 由于是异步发送消息，上面循环结束之后，消息可能还没收到broker的响应
        // 如果不sleep一会儿，就报错
        Thread.sleep(10_000);

        // 关闭生产者
        producer.shutdown();
    }
}
```

##### 5.1.2 消息消费者

消费者拉消息：

```java
/**
 * RocketMQ相关API使用
 * 拉取消息的消费者
 */
public class MyPullConsumer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException, UnsupportedEncodingException {
        // 拉取消息的消费者实例化，同时指定消费组名称
        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("consumer_grp_01");
        // 设置nameserver的地址
        consumer.setNamesrvAddr("192.168.91.100:9876");

        // 对消费者进行初始化，然后就可以使用了
        consumer.start();

        // 获取指定主题的消息队列集合
        final Set<MessageQueue> messageQueues = consumer.fetchSubscribeMessageQueues("tp_demo_01");

        // 遍历该主题的各个消息队列，进行消费
        for (MessageQueue messageQueue : messageQueues) {
            // 第一个参数是MessageQueue对象，代表了当前主题的一个消息队列
            // 第二个参数是一个表达式，对接收的消息按照tag进行过滤
            // 支持"tag1 || tag2 || tag3"或者 "*"类型的写法；null或者"*"表示不对消息进行tag过滤
            // 第三个参数是消息的偏移量，从这里开始消费
            // 第四个参数表示每次最多拉取多少条消息
            final PullResult result = consumer.pull(messageQueue, "*", 0, 10);
            // 打印消息队列的信息
            System.out.println("message******queue******" + messageQueue);
            // 获取从指定消息队列中拉取到的消息
            final List<MessageExt> msgFoundList = result.getMsgFoundList();
            if (msgFoundList == null) continue;
            for (MessageExt messageExt : msgFoundList) {
                System.out.println(messageExt);
                System.out.println(new String(messageExt.getBody(), "utf-8"));
            }
        }

        // 关闭消费者
        consumer.shutdown();
    }
}
```

消费者推消息：

```java
/**
 * RocketMQ相关API使用
 * 推送消息的消费
 */
public class MyPushConsumer {
    public static void main(String[] args) throws MQClientException, InterruptedException {

        // 实例化推送消息消费者的对象，同时指定消费组名称
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_grp_02");

        // 指定nameserver的地址
        consumer.setNamesrvAddr("192.168.91.100:9876");

        // 订阅主题
        consumer.subscribe("tp_demo_02", "*");

        // 添加消息监听器，一旦有消息推送过来，就进行消费
        consumer.setMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {

                final MessageQueue messageQueue = context.getMessageQueue();
                System.out.println(messageQueue);

                for (MessageExt msg : msgs) {
                    try {
                        System.out.println(new String(msg.getBody(), "utf-8"));
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                    }
                }

                // 消息消费成功
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                // 消息消费失败
//                return ConsumeConcurrentlyStatus.RECONSUME_LATER;
            }
        });

        // 初始化消费者，之后开始消费消息
        consumer.start();

        // 此处只是示例，生产中除非运维关掉，否则不应停掉，长服务
//        Thread.sleep(30_000);
//        // 关闭消费者
//        consumer.shutdown();
    }
}
```

#### 5.2 SpringBoot整合RocketMQ

引入依赖：

```xml
<properties>
    <rocketmq-spring-boot-starter-version>2.0.3</rocketmq-spring-boot-starter-version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.apache.rocketmq</groupId>
        <artifactId>rocketmq-spring-boot-starter</artifactId>
        <version>${rocketmq-spring-boot-starter-version}</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.6</version>
    </dependency>
</dependencies>
```

添加配置：

```properties
spring.application.name=springboot_rocketmq

# rocketmq的nameserver地址
rocketmq.name-server=192.168.91.100:9876
# 指定生产组名称
rocketmq.producer.group=producer_grp_02
```

生产者单元测试：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {RocketSpringbootApplication.class})
class RocketSpringbootApplicationTests {
    
    @Autowired
    private RocketMQTemplate rocketMQTemplate;
    
    @Test
    public void testSendMessage() {
        // 用于向broker发送消息
        // 第一个参数是topic名称
        // 第二个参数是消息内容
        this.rocketMQTemplate.convertAndSend(
                "tp_springboot_01",
                "springboot: hello lagou"
        );
    }
    
    @Test
    public void testSendMessages() {
        for (int i = 0; i < 100; i++) {
            // 用于向broker发送消息
            // 第一个参数是topic名称
            // 第二个参数是消息内容
            this.rocketMQTemplate.convertAndSend(
                    "tp_springboot_01",
                    "springboot: hello lagou" + i
            );
        }
    }

}
```

消费者监听（推）模式：

```java
@Slf4j
@Component
@RocketMQMessageListener(topic = "tp_springboot_01", consumerGroup = "consumer_grp_03")
public class MyRocketListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        // 处理broker推送过来的消息
        log.info(message);
    }
}
```

## 二.RocketMQ高级特性

### 1.消息发送与消费:star:

#### 1.1 消息发送与消息消费

生产者向消息队列里写入消息，不同的业务场景需要生产者采用不同的写入策略。比如**同步发送、异步发送、Oneway发送、延迟发送、发送事务消息**等。 默认使用的是 `DefaultMQProducer`类，发送消息要经过五个步骤：

- 设置Producer的GroupName；
- 设置InstanceName，当一个Jvm需要启动多个Producer的时候，通过设置不同的 `InstanceName`来区分，不设置的话系统使用默认名称“DEFAULT”；
- 设置发送失败重试次数，当网络出现异常的时候，这个次数影响消息的重复投递次数；
- 设置NameServer地址；
- 组装消息并发送。

消息发生返回状态（SendResult#SendStatus）有如下四种，不同状态在不同的刷盘策略和同步策略的配置下含义是不同的：

- `FLUSH_DISK_TIMEOUT`：表示**没有在规定时间内完成刷盘**（需要Broker的刷盘策略被设置成SYNC_FLUSH才会报这个错误）。
- `FLUSH_SLAVE_TIMEOUT`：表示在主备方式下，并且Broker被设置成SYNC_MASTER方式，没有在**设定时间内完成主从同步**。
- `SLAVE_NOT_AVAILABLE`：这个状态产生的场景和FLUSH_SLAVE_TIMEOUT类似，表示在主备方式下，并且Broker被设置成SYNC_MASTER，但是**没有找到被配置成Slave的Broker**。
- `SEND_OK`：表示**发送成功**，发送成功的具体含义，比如消息是否已经被存储到磁盘？消息是否被同步到了Slave上？消息在Slave上是否被写入磁盘？需要结合所配置的**刷盘策略、主从策略**来定。这个状态还可以简单理解为，没有发生上面列出的三个问题状态就是SEND_OK。

提升写入的性能：一般一条消息发出去要经过这三步骤：客户端发送请求到服务器、服务器处理该请求、服务器向客户端返回应答。

- 对速度要求高，可靠性要求不高的场景下，如日志收集类应用，可以**采用Oneway方式发送**（将数据写入客户端的Socket缓冲区就返回）。
- 增加Producer的并发量，使用多个Producer同时发送。

消息消费有几个要点：

- 消息消费方式（Pull和Push）
- 消息消费的模式（广播模式和集群模式）
- 流量控制（可以结合sentinel来实现）
- 并发线程数设置
- 消息的过滤（Tag、Key） TagA||TagB||TagC * null

有以下几种方式可以提高Consumer的处理能力：

- 提高消息并行度：在同一个ConsumerGroup下（Clustering方式），可以通过**增加Consumer实例的数量**来提高并行度。还可以提高单个Consumer实例中的并行处理的线程数，可以在同一个Consumer内增加
    并行度来提高吞吐量（设置方法是修改 `consumeThreadMin`和 `consumeThreadMax`）。
- 以批量方式进行消费：可以**通过批量方式消费**来提高消费的吞吐量。实现方法是设置Consumer的
    `consumeMessageBatchMaxSize`这个参数，默认是1，如果设置为N，在消息多的时候每次收到的是个长度为N的消息链表。
- 检测延时情况，跳过非重要消息：Consumer在消费的过程中，如果发现由于某种原因发生严重的消息堆积，短时间无法消除堆积，这个时候可以选择**丢弃不重要的消息**，使Consumer尽快追上Producer的进度。

#### 1.2 消息发送与消费示例

消息发送：

```java
/**
 * @author zlg
 * 消息发送
 */
public class MyProducer {
    
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        
        // 该producer是线程安全的，可以多线程使用。
        // 建议使用多个Producer实例发送
        // 实例化生产者实例，同时设置生产组名称
        DefaultMQProducer producer = new DefaultMQProducer("producer_grp_04");
        
        // 设置实例名称。一个JVM中如果有多个生产者，可以通过实例名称区分
        // 默认DEFAULT
        producer.setInstanceName("producer_grp_04_01");
        
        // 设置同步发送重试的次数
        producer.setRetryTimesWhenSendFailed(2);
        
        // 设置异步发送的重试次数
        producer.setRetryTimesWhenSendAsyncFailed(2);
        // 设置nameserver的地址
        producer.setNamesrvAddr("192.168.91.100:9876");
        
        // 对生产者进行初始化
        producer.start();
        
        // 组装消息
        Message message = new Message("tp_demo_04", "hello lagou 04".getBytes());
        
        // 同步发送消息，如果消息发送失败，则按照setRetryTimesWhenSendFailed设置的次数进行重试
        // broker中可能会有重复的消息，由应用的开发者进行处理
        final SendResult result = producer.send(message);
        
        producer.send(message, new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {
                // 发送成功的处理逻辑
            }
            
            @Override
            public void onException(Throwable e) {
                // 发送失败的处理逻辑
                // 重试次数耗尽，发生的异常
            }
        });
        
        // 将消息放到Socket缓冲区，就返回，没有返回值，不会等待broker的响应
        // 速度快，会丢消息
        // 单向发送
        producer.sendOneway(message);
        
        final SendStatus sendStatus = result.getSendStatus();
    }
}
```

消息消费：

```java
/**
 * @author zlg
 * 消息消费
 */
public class MyConsumer {
    
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        
        // 消息的拉取
        DefaultMQPullConsumer pullConsumer = new DefaultMQPullConsumer();
        
        // 消费的模式：集群
        pullConsumer.setMessageModel(MessageModel.CLUSTERING);
        // 消费的模式：广播
        pullConsumer.setMessageModel(MessageModel.BROADCASTING);
        
        
        final Set<MessageQueue> messageQueues = pullConsumer.fetchSubscribeMessageQueues("tp_demo_05");
        
        for (MessageQueue messageQueue : messageQueues) {
            // 指定消息队列，指定标签过滤的表达式，消息偏移量和单次最大拉取的消息个数
            pullConsumer.pull(messageQueue, "TagA||TagB", 0L, 100);
        }
        
        // 消息的推送
        DefaultMQPushConsumer pushConsumer = new DefaultMQPushConsumer();
        
        pushConsumer.setMessageModel(MessageModel.BROADCASTING);
        pushConsumer.setMessageModel(MessageModel.CLUSTERING);
        
        // 设置消费者的线程数
        pushConsumer.setConsumeThreadMin(1);
        pushConsumer.setConsumeThreadMax(10);
        
        // subExpression表示对标签的过滤：
        // TagA||TagB|| TagC    *表示不对消息进行标签过滤
        pushConsumer.subscribe("tp_demo_05", "*");
        
        // 设置消息批处理的一个批次中消息的最大个数
        pushConsumer.setConsumeMessageBatchMaxSize(10);
        
        // 在设置完之后调用start方法初始化并运行推送消息的消费者
        pushConsumer.start();
    }
}
```

### 2.消息存储:star:

消息存储介质为关系型数据库DB和文件系统，性能上文件系统优于关系型数据库。如ActiveMQ默认采用KahaDB做消息存储。目前RocketMQ/Kafka/RabbitMQ均采用**消息刷盘至所部署虚拟机/物理机的文件系统来做持久化**（同步刷盘和异步刷盘）。

RocketMQ的消息用**顺序写**，保证了消息存储的速度。RocketMQ消息的存储是由 `ConsumeQueue`和``CommitLog`配合完成 的，消息真正的物理存储文件是`CommitLog`，ConsumeQueue是消息的逻辑队列，类似数据库的**索引文件，存储的是指向物理存储的地址**。每 个Topic下的每个Message Queue都有一个对应的ConsumeQueue文件。

![image-20201214003951687](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214003951687.png)

消息存储架构图中主要有下面三个跟消息存储相关的文件构成：

CommitLog：消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容，消息内容不是定长的。单个文件大小默认1G ，文件名长度为20位，左边补零，剩余为起始偏移量。消息主要是顺序写入日志文件，当文件满了，写入下一个文件。

```shell
[root@linux100 store]# pwd
/root/store
[root@linux100 store]# ls
abort  checkpoint  commitlog  config  consumequeue  index  lock
[root@linux100 store]# ll -h commitlog/
-rw-r--r--. 1 root root 1.0G 12月 13 21:58 00000000000000000000
```

ConsumeQueue：消息消费队列，引入的目的主要是提高消息消费的性能。如果要遍历commitlog文件根据topic检索消息是非常低效，Consumer即可根据ConsumeQueue来查找待消费的消息。其中，ConsumeQueue（逻辑消费队列）作为消费消息的索引：

- **保存了指定Topic下的队列消息在CommitLog中的起始物理偏移量offset**；
- **消息大小size**；
- **消息Tag的HashCode值**。

consumequeue文件可以看成是基于topic的commitlog索引文件，故consumequeue文件夹的组织方式如下：`topic/queue/file三层组织结构`。

```shell
# 存储路径：$HOME/store/consumequeue/{topic}/{queueId}/{fileName}
[root@linux100 store]# ls consumequeue/
%DLQ%consuer_grp_09_01    RMQ_SYS_TRACE_TOPIC  TopicTest   tp_demo_11   tp_flashsale
%RETRY%consuer_grp_09_01  SCHEDULE_TOPIC_XXXX  tp_demo_09  tp_flashpay
[root@linux100 store]# ls consumequeue/tp_demo_11/
0  1  2  3  4  5  6  7
[root@linux100 store]# ll -h consumequeue/tp_demo_11/0
-rw-r--r--. 1 root root 5.8M 12月 12 15:47 00000000000000000000
```

consumequeue文件采取定长设计，每个条目共20个字节，分别为：8字节的commitlog物理偏移量、4字节的消息长、8字节tag hashcode。单个文件由30W个条目组成，可以像数组一样随机访问每一个条目。每个ConsumeQueue文件大小约5.72M。

IndexFile：索引文件提供了一种可以**通过key或时间区间来查询消息的方法**。固定的单个IndexFile文件大小约为400M，一个IndexFile可以保存 2000W个索引，IndexFile的底层存储设计为在文件系统中实现**HashMap结构**，故rocketmq的索引文件其**底层实现为hash索引**。

```shell
# 存储位置：$HOME/store/index/${fileName}
# fileName是以创建时的时间戳命名的
[root@linux100 store]# ls index/
20201209232852588
[root@linux100 store]# ll -h index/
-rw-r--r--. 1 root root 401M 12月 13 21:58 20201209232852588
```

### 3.过滤消息

RocketMQ这么做是在于其Producer端写入消息和Consumer端订阅消息采用**分离存储的机制**来实现的，Consumer端订阅消息是需要通过 `ConsumeQueue`这个消息消费的逻辑队列拿到一个**索引**，然后再从 `CommitLog` 里面读取真正的**消息实体内容**，所以说到底也是还绕不开其存储结构。

其ConsumeQueue的存储结构如下，可以看到其中有8个字节存储的Message Tag的哈希值，基于Tag的消息过滤正式基于这个字段值的。

![image-20201214010949792](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214010949792.png)

#### 3.1 Tag过滤方式

Consumer端在订阅消息时除了指定Topic还可以指定TAG，如果一个消息有多个TAG，可以用 `||`分隔。

-  Consumer端会将这个订阅请求构建成一个 SubscriptionData，发送一个Pull消息的请求给Broker端。
- Broker端从RocketMQ的文件存储层—Store读取数据之前，会用这些数据先构建一个MessageFilter，然后传给Store。
-  Store从 ConsumeQueue读取到一条记录后，会用它记录的消息tag hash值去做过滤。
- 在**服务端只是根据hashcode进行判断**，无法精确对tag原始字符串进行过滤，在**消息消费端**拉取到消息后，还需要**对消息的原始tag字符串进行比对**，如果不同，则丢弃该消息，不进行消息消费。

生产者示例：

```java
public class MyProducer {

    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException {

        DefaultMQProducer producer = new DefaultMQProducer("producer_grp_06");
        producer.setNamesrvAddr("192.168.91.100:9876");
        producer.start();

        Message message = null;
        for (int i = 0; i < 100; i++) {
            message = new Message(
                    "tp_demo_06",
                    "tag-" + (i % 3),
                    ("hello lagou - " + i).getBytes()
            );

            producer.send(message, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println(sendResult.getSendStatus());
                }

                @Override
                public void onException(Throwable e) {
                    System.out.println(e.getMessage());
                }
            });
        }

        Thread.sleep(3_000);
        producer.shutdown();
    }
}
```

消费者示例：

```java
public class MyConsumerTagFilter {
    public static void main(String[] args) throws MQClientException {

        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_grp_06_03");

        consumer.setNamesrvAddr("192.168.91.100:9876");

//        consumer.subscribe("tp_demo_06", "*");
//        consumer.subscribe("tp_demo_06", "tag-1");
        // 设置消息过滤标签，接受tag-1和tag-0的消息，其余过滤掉
        consumer.subscribe("tp_demo_06", "tag-1||tag-0");

        consumer.setMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {

                final MessageQueue messageQueue = context.getMessageQueue();
                final String brokerName = messageQueue.getBrokerName();
                final String topic = messageQueue.getTopic();
                final int queueId = messageQueue.getQueueId();
                System.out.println(brokerName + "\t" + topic + "\t" + queueId);

                for (MessageExt msg : msgs) {
                    System.out.println(msg);
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        // 初始化并启动消费者
        consumer.start();
    }
}
```

#### 3.2 SQL92过滤方式

仅对**push的消费者**起作用。真正的 SQL expression 的构建和执行由rocketmq-filter模块负责的。RocketMQ使用了BloomFilter避免了每次都去执行。SQL92的表达式上下文为消息的属性。

![image-20201214011834692](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214011834692.png)

编辑配置文件：conf/broker.conf

```shell
# 启动SQL92
enablePropertyFilter=true

# 查看enablePropertyFilter
mqbroker -p | grep enablePropertyFilter
# 或
mqbroker -c /opt/rocket/conf/broker.conf -p | grep enablePropertyFilter
```

开启支持SQL92的特性后，然后重启broker：

```shell
mqbroker -n localhost:9876 -c /opt/rocket/conf/broker.conf
```

RocketMQ仅定义了几种基本的语法，用户可以扩展：

- 数字比较： >, >=, <, <=, BETWEEN, =；
- 字符串比较： =, <>, IN; IS NULL或者IS NOT NULL;
- 逻辑比较： AND, OR, NOT;
- Constant types are：数字如：123, 3.1415; 字符串如：'abc'，必须是单引号引起来 NULL,特殊常量 布尔型如：TRUE or FALSE;

生产者示例：

```java
public class MyProducer {

    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException {

        DefaultMQProducer producer = new DefaultMQProducer("producer_grp_06");
        producer.setNamesrvAddr("192.168.91.100:9876");

        producer.start();

        Message message = null;

        for (int i = 0; i < 100; i++) {
            message = new Message("tp_demo_06_02", ("hello lagou - " + i).getBytes());

            String value = null;
            switch (i % 3) {
                case 0:
                    value = "v0";
                    break;
                case 1:
                    value = "v1";
                    break;
                default:
                    value = "v2";
                    break;
            }
            // 给消息添加用户属性
            message.putUserProperty("mykey", value);

            producer.send(message, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println(sendResult.getSendStatus());
                }
                @Override
                public void onException(Throwable e) {
                    System.out.println(e.getMessage());
                }
            });

        }

        Thread.sleep(3_000);
        producer.shutdown();
    }
}
```

消费者示例：

```java
public class MyConsumerPropertiesFilter {
    public static void main(String[] args) throws MQClientException {

        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_grp_06_02");

        consumer.setNamesrvAddr("192.168.91.100:9876");

//        consumer.subscribe("tp_demo_06_02", MessageSelector.bySql("mykey in ('v0', 'v1')"));
//        consumer.subscribe("tp_demo_06_02", MessageSelector.bySql("mykey = 'v0'"));
        // SQL过滤，只要message中mykey属性值不为空，就都接受
        consumer.subscribe("tp_demo_06_02", MessageSelector.bySql("mykey IS NOT NULL"));

        consumer.setMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {

                final MessageQueue messageQueue = context.getMessageQueue();
                final String brokerName = messageQueue.getBrokerName();
                final String topic = messageQueue.getTopic();
                final int queueId = messageQueue.getQueueId();
                System.out.println(brokerName + "\t" + topic + "\t" + queueId);

                for (MessageExt msg : msgs) {
                    System.out.println(msg);
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        // 初始化并启动消费者
        consumer.start();
    }
}
```

#### 3.3 Filter Server方式

允许用户自定义Java函数，根据Java函数的逻辑对消息进行过滤。要使用Filter Server，首先要在启动Broker前在配置文件里加上 filterServer-Nums=3 这样的配置，Broker在启动的时候，就会在本机启动3个Filter Server进程。Filter Server类似一个RocketMQ的Consumer进程，它从本机Broker获取消息，然后根据用户上传过来的Java函数进行过滤，过滤后的消息再传给远端的Consumer。

这种方式会占用很多Broker机器的CPU资源，要根据实际情况谨慎使用。上传的java代码也要经过检查，不能有申请大内存、创建线程等这样的操作，否则容易造成Broker服务器宕机。

### 4.负载均衡:star:

RocketMQ中的负载均衡都在**Client端**完成，具体来说的话，主要可以分为Producer端发送消息时候的负载均衡和Consumer端订阅消息的负载均衡。

#### 4.1 Producer负载均衡

![image-20201214014515022](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214014515022.png)

如上图所示，5 个队列可以部署在一台机器上，也可以分别部署在 5 台不同的机器上，发送消息通过轮询队列的方式发送，每个队列接收平均的消息量。通过增加机器，可以水平扩展队列容量。 另外也可以自定义方式选择发往哪个队列。

```shell
# 创建主题
[root@linux100 ~]# mqadmin updateTopic -n localhost:9876 -t tp_demo_06 -w 6 -b localhost:10911
```

自定义方式选择发往哪个队列：

```java
public class MyProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        DefaultMQProducer producer = new DefaultMQProducer("producer_grp_06_01");
        producer.setNamesrvAddr("192.168.91.100:9876");
        producer.start();

        Message message = new Message("tp_demo_06", "hello lagou".getBytes());
		// 指定要发送的MQ
        final SendResult result = producer.send(message, new MessageQueue("tp_demo_06", "linux100", 5));
        System.out.println(result.getSendStatus());

        producer.shutdown();
    }
}
```

#### 4.2 Consumer负载均衡

![image-20201214015005626](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214015005626.png)

如图所示，如果有 5 个队列，2 个 consumer，那么第一个 Consumer 消费 3 个队列，第二consumer 消费 2 个队列。 这样即可达到平均消费的目的，可以水平扩展 Consumer 来提高消费能力。但是 Consumer 数量要小于等于队列数 量，如果 Consumer 超过队列数量，那么多余的Consumer 将不能消费消息 。

在RocketMQ中，Consumer端的两种消费模式（Push/Pull）底层都是基于**拉模式**来获取消息的。在Consumer端来做负载均衡，Broker端中多个MessageQueue分配给同一个ConsumerGroup中的哪些Consumer消费。

**Consumer从Broker处获得全局信息，然后自己做负载均衡，只处理分给自己的那部分消息**。

DefaultMQPullConsumer有两个辅助方法可以帮助实现负载均衡，一个是 `registerMessageQueueListener`函数，一个是 `MQPullConsumerScheduleService`（使用这个Class类似使用DefaultMQPushConsumer，但是它把Pull消息的主动性留给了使用者）。

```java
public class MyConsumerPull {
    public static void main(String[] args) throws InterruptedException, RemotingException, MQClientException, MQBrokerException {
        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("consumer_grp_06_01");
        consumer.setNamesrvAddr("192.168.91.100:9876");
        consumer.start();
		// 指定从哪个MQ拉取数据
        final PullResult pullResult = consumer.pull(new MessageQueue(
                        "tp_demo_06",
                        "node1",
                        4
                ),"*",0L,10);
        System.out.println(pullResult);

        pullResult.getMsgFoundList().forEach(messageExt -> {
            System.out.println(messageExt);
        });

        consumer.shutdown();
    }
}
```

DefaultMQPushConsumer的负载均衡过程不需要使用者操心，客户端程序会自动处理，每个
DefaultMQPushConsumer启动后，会马上会触发一个**doRebalance动作**；而且在同一个ConsumerGroup里加入新的DefaultMQPush-Consumer时，各个Consumer都会被触发**doRebalance动作**。

**负载均衡的分配粒度只到Message Queue，把Topic下的所有Message Queue分配到不同的Consumer中**。如下图所示，具体的负载均衡算法有几种，默认用的是 `AllocateMessageQueueAveragely`。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214020129736.png" alt="image-20201214020129736" style="zoom:50%;" />

可以手动设置负载均衡算法：

```java
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_push_grp_01");
consumer.setNamesrvAddr("192.168.91.100:9876");
// 设置负载均衡算法
consumer.setAllocateMessageQueueStrategy(new AllocateMessageQueueAveragely());
consumer.setMessageListener(new MessageListenerConcurrently() {
	@Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
        // todo 处理接收到的消息
        return null;
    }
});
consumer.start();
```

消息消费队列在同一消费组不同消费者之间的负载均衡，其核心设计理念是在**一个消息消费队列在同一时间只允许被同一消费组内的一个消费者消费，一个消息消费者能同时消费多个消息队列**。

### 5.消息重试:star:

#### 5.1 顺序消息重试

对于顺序消息，当消费者消费消息失败后，消息队列 RocketMQ 会**自动不断进行消息重试**（每次间隔时间为 1 秒），这时，应用可能会出现消息消费被阻塞的情况。

```java
public class MyConsumerOrderly {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_08_01");
        consumer.setNamesrvAddr("192.168.91.100:9876");

        // 设置消息重试次数
        consumer.setMaxReconsumeTimes(5);
        // 一次获取一条消息
        consumer.setConsumeMessageBatchMaxSize(1);
        // 订阅主题
        consumer.subscribe("tp_demo_08", "*");
        // 顺序消费
        consumer.setMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {

                for (MessageExt msg : msgs) {
                    System.out.println(msg.getMsgId() + "\t" + msg.getQueueId() + "\t" + new String(msg.getBody()));
                }
//                return ConsumeOrderlyStatus.SUCCESS;  // 确认消息
//                return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT; // 引发重试
                return null; // 引发重试
            }
        });
		
        // 并行消费
        /*consumer.setMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {

                for (MessageExt msg : msgs) {
                    System.out.println(msg.getReconsumeTimes());
                }
                // 稍后消费
                return ConsumeConcurrentlyStatus.RECONSUME_LATER;
            }
        });*/
        
        consumer.start();
    }
}
```

#### 5.2 无序消息重试

对于无序消息（普通、定时、延时、事务消息），当消费者消费消息失败时，可以通过**设置返回状态**达到消息重试的结果。无序消息的重试**只针对集群消费模式生效**；广播模式不提供失败重试特性，即消费失败后，失败消息不再重试，继续消费新的消息。

消息队列 RocketMQ 默认允许**每条消息最多重试 16 次**，一条消息无论重试多少次，这些重试消息的 Message ID 不会改变。每次重试的间隔时间如下：

![image-20201214021627934](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214021627934.png)

如果消息重试 16 次后仍然失败，消息将不再投递。如果严格按照上述重试时间间隔计算，某条消息在一直消费失败的前提下，将会在接下来的 4 小时 46 分钟之内进行 16 次重试，超过这个时间范围消息将不再重试投递。

**消费失败后，重试配置方式**。需要在消息监听器接口的实现中明确进行配置（三种方式任选一种）：

- 返回 ConsumeConcurrentlyStatus.RECONSUME_LATER; （推荐）
- 返回 Null
- 抛出异常

```java
public class MyConcurrentlyMessageListener implements MessageListenerConcurrently {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
        //处理消息
        doConsumeMessage(msgs);
        //方式1：返回 ConsumeConcurrentlyStatus.RECONSUME_LATER，消息将重试
        return ConsumeConcurrentlyStatus.RECONSUME_LATER;
        //方式2：返回 null，消息将重试
        return null;
        //方式3：直接抛出异常， 消息将重试
        throw new RuntimeException("Consumer Message exceotion");
    }
}
```

**消费失败后，不重试配置方式**。需要捕获消费逻辑中可能抛出的异常，最终返回
`ConsumeConcurrentlyStatus.CONSUME_SUCCESS`，此后这条消息将不会再重试。

```java
public class MyConcurrentlyMessageListener implements MessageListenerConcurrently {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
        try {
            doConsumeMessage(msgs);
        } catch (Throwable e) {
            //捕获消费逻辑中的所有异常，并返回ConsumeConcurrentlyStatus.CONSUME_SUCCESS
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
        //消息处理正常，直接返回 ConsumeConcurrentlyStatus.CONSUME_SUCCESS
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
}
```

**自定义消息最大重试次数**。 RocketMQ 允许 Consumer 启动的时候设置最大重试次数，重试时间间隔将按照如下策略：

- 最大重试次数小于等于 16 次，则重试时间间隔同上表描述。
- 最大重试次数大于 16 次，超过 16 次的重试时间间隔均为每次 2 小时。

```java
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_grp_04_01");
// 设置重新消费的次数
// 共16个级别，大于16的一律按照2小时重试
consumer.setMaxReconsumeTimes(20);
```

注意：消息最大重试次数的设置对**相同 Group ID 下的所有 Consumer 实例有效**。对一个Consumer实例设置了最大重试次数，即该实例同一GroupID下所有Consumer实例配置均生效。**最后启动的 Consumer 实例会覆盖之前的启动实例的配置**。

**获取消息重试次数**。消费者收到消息后，可按照如下方式获取消息的重试次数：

```java
public class MyConcurrentlyMessageListener implements MessageListenerConcurrently {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
        for (MessageExt msg : msgs) {
            // 获取消息的重试次数
            System.out.println(msg.getReconsumeTimes());
        }
        doConsumeMessage(msgs);
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
}
```

### 6.死信队列



可视化工具：rocketmq-console 地址，使用 JDK11 启动需要在项目里添加额外依赖和配置：

```xml
<!--从jdb8升级到了jdk11，java.lang.ClassNotFoundException: javax.xml.bind.XXException-->
<!-- jdk9开始，引入模块的概念，se中不再包含javaEE的包导致的。这导致解析配置文件失败 -->
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-impl</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-core</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
```



```properties
# 添加namesrvAddr
rocketmq.config.namesrvAddr=192.168.91.100:9876
```





### 7.延迟消息:star:



### 8.顺序消息:star:





```shell
# 创建主题，8写8读
[root@node1 ~]# mqadmin updateTopic -b 192.168.91.100:10911 -n localhost:9876 -r 8 -t tp_demo_11 -w 8
# 删除主题的操作
[root@node1 ~]# mqadmin deleteTopic -c DefaultCluster deleteTopic -n localhost:9876 -t tp_demo_11
# 主题描述
[root@node1 ~]# mqadmin topicStatus -n localhost:9876 -t tp_demo_11
#Broker Name                      #QID  #Min Offset           #Max Offset             #Last Updated
linux100                          0     0                     0                       
linux100                          1     0                     0                       
linux100                          2     0                     0                       
linux100                          3     0                     0                       
linux100                          4     0                     0                       
linux100                          5     0                     0                       
linux100                          6     0                     0                       
linux100                          7     0                     0                       
```



### 9.事务消息:star:



### 10.消息查询





### 11.消息限流:star:

























