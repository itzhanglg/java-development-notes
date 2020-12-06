## 一.初识Kafka

Kafka是由Scala语言开发的一个**多分区、多副本基于ZooKeeper协调的分布式消息系统**。目前Kafka定位为一个**分布式流式处理平台，它以高吞吐、可持久化、可水平扩展、支持流数据处理**等多种特性而被广泛使用。Kafka可以扮演以下三种角色：

- **消息系统**：Kafka 和传统的消息系统都具备系统解耦、冗余存储、流量削峰、缓冲、异步通信、扩展性、可恢复性等功能。同时Kafka还提供消息顺序性保障及回溯消费的功能。
- **存储系统**：Kafka把消息持久化到磁盘，相比其它基于内存存储的系统而言，降低了数据丢失的风险。只需要把对应的数据保留策略设置为“永久”或启用主题的日志压缩功能即可将Kafka作为长期的数据存储系统来使用。
- **流式处理平台**：Kafka不仅为每个流行的流式处理框架通过可靠的数据来源，还提供一个完整的流式处理类库，比如窗口、连接、变换和聚合等各类操作。

### 1.基本概念

#### 1.1 Kafka的体系结构

一个典型的Kafka体系结构包括若干个Producer、Broker、Consumer及一个Zookeeper集群。如下图：

![image-20201126002206662](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201126002206662.png)

- **Producer**（生产者）：负责创建消息，将消息投递给Broker中；
- **Broker**（服务代理节点）：可以看作为一个独立的Kafka服务节点或Kafka服务实例。一个或多个Broker组成了一个Kafka集群；
- **Consumer**（消费者）：连接Kafka并订阅消息，进行相应的业务逻辑处理；
- **Zookeeper**（协调服务）：负责Kafka集群元数据的管理、控制器的选举等操作。

#### 1.2 主题、分区、副本

Kafka中的消息以**主题（Topic）为单位**进行归类，生产者负责将消息发送到特定的主题（发送到Kafka集群中的每一条消息都要指定一个主题），而消费者负责订阅主题并进行消费。

主题是一个逻辑上的概念，可以分为**多个分区**（Partition），一个分区只属于一个主题，也成为主题分区（Topic-Partition）。同一主题下的不同分区包含的消息是不同的，分区在存储层面可以看作一个**可追加的日志（Log）文件，消息在被顺序追加到每个分区日志文件的尾部时，都会分配一个特定的偏移量**（offset）。

offset是消息在分区中的唯一标识，Kafka通过它来保证消息在分区内的顺序性，不过offset并不跨越分区，即Kafka保证的是**分区有序而不是主题有序**。

Kafka中的分区可以分布在不同的服务器（broker）上，也就是说，一个主题可以横跨多个broker，以此来提供比单个broker更强大的性能，通过**增加分区的数量可以实现水平扩展**。每一条消息被发送到broker之前，会根据**分区规则**选择存储到哪个具体的分区。如果分区规则设定得合理，所有的消息都可以均匀地分配到不同的分区中。

Kafka 为分区引入了**多副本**（Replica）机制，通过**增加副本数量可以提升容灾能力**。同一分区的不同副本中保存的是相同的消息（在同一时刻，副本之间并非完全一样），**副本之间是“一主多从”的关系**，其中leader副本负责处理读写请求，**生产者和消费者只与leader副本进行交互**，而follower副本只负责与leader副本的消息同步。**副本处于不同的broker中**，当leader副本出现故障时，从follower副本中重新选举新的leader副本对外提供服务。Kafka通过多副本机制实现了故障的自动转移，当Kafka集群中某个broker失效时仍然能保证服务可用。Kafka 消费端也具备一定的容灾能力。Consumer 使用拉（Pull）模式从服务端拉取消息，并且保存消费的具体位置。

#### 1.3 副本：AR、ISR、OSR

分区中的**所有副本统称为AR**（Assigned Replicas）。所有与leader副本**保持一定程度同步的副本（包括leader副本在内）组成ISR**（In-Sync Replicas），ISR集合是AR集合中的一个子集。消息会先发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步，同步期间内follower副本相对于leader副本而言会有一定程度的滞后（可忍受的滞后范围）。与leader副本同步**滞后过多的副本（不包括leader副本）组成OSR**（Out-of-Sync Replicas），由此可见，**AR=ISR+OSR**。**在正常情况下，所有的 follower 副本都应该与 leader 副本保持一定程度的同步，即 AR=ISR，OSR集合为空**。

leader副本负责维护和跟踪ISR集合中所有follower副本的滞后状态，当follower副本落后太多或失效时，leader副本会把它从ISR集合中剔除。如果OSR集合中有follower副本“追上”了leader副本，那么leader副本会把它从OSR集合转移至ISR集合。默认情况下，当leader副本发生故障时，只有在ISR集合中的副本才有资格被选举为新的leader。

#### 1.4 ISR：HW、LEO

ISR与HW（High Watermark，高水位）和LEO（Log End Offset）也有紧密的关系。HW标识了**一个特定的消息偏移量（offset），消费者只能拉取到这个offset之前的消息**。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201126070640939.png" alt="image-20201126070640939" style="zoom:70%;" />

如上图，它代表一个日志文件，这个日志文件中有 9 条消息，**第一条消息的 offset（LogStartOffset）为0，最后一条消息的offset为8**，offset为9的消息用虚线框表示，代表下一条待写入的消息。日志文件的HW为6，表示消费者只能拉取到offset在0至5之间的消息，而offset为6的消息对消费者而言是不可见的。

LEO（Log End Offset）标识**当前日志文件中下一条待写入消息的offset**，上图中offset为9的位置即为当前日志文件的LEO，LEO的大小相当于当前日志分区中最后一条消息的offset值加1。**分区ISR集合中的每个副本都会维护自身的LEO，而ISR集合中最小的LEO即为分区的HW，对消费者而言只能消费HW之前的消息**。

Kafka 的复制机制既不是完全的同步复制，也不是单纯的异步复制。同步需要所有副本将消息同步完才会确认已成功提交，影响性能；异步只要leader副本将消息写入就认为已成功提交，会造成数据丢失。Kafka使用的这种ISR的方式则有效地权衡了数据可靠性和性能之间的关系。

### 2.安装与配置

#### 2.1 Linux上安装rz和sz命令

安装lrz主要用于文件上传或下载，按个人需要安装即可。

使用yum安装：

```shell
yum -y install lrzsz
# 使用上传文件，执行命令rz，会跳出文件选择窗口，选择好文件，点击确认即可
rz
# 下载文件，执行命令sz
sz
```

或者使用wget安装：

```shell
# 下载
wget http://www.ohse.de/uwe/releases/lrzsz-0.12.20.tar.gz
# 解压并进入根目录
tar -zxvf lrzsz-0.12.20.tar.gz && cd lrzsz-0.12.20
# 配置、编译和安装
./configure && make && make install
#上面安装过程默认把lsz和lrz安装到了/usr/local/bin/目录下，现在我们并不能直接使用，
#下面创建软链接，并命名为rz/sz：
cd /usr/bin
ln -s /usr/local/bin/lrz rz
ln -s /usr/local/bin/lsz sz
# 安装完后可以直接拖到window本地文件到xshell上
```

#### 2.2 安装Java环境

安装：

```shell
# 上传jdk-8u261-linux-x64.rpm到服务器并安装
rpm -ivh jdk-8u261-linux-x64.rpm
# 配置环境变量
vim /etc/profile
# profile文件末尾配置
export JAVA_HOME=/usr/java/jdk1.8.0_261-amd64
export PATH=$JAVA_HOME/bin:$PATH
# 生效
source /etc/profile
```

验证：

```shell
[root@atzlg8 jdk1.8.0_261-amd64]# java -version
java version "1.8.0_261"
Java(TM) SE Runtime Environment (build 1.8.0_261-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.261-b12, mixed mode)
```

#### 2.3 Zookeeper安装与配置

ZooKeeper是安装Kafka集群的必要组件，Kafka通过ZooKeeper来实施对元数据信息的管理，包括集群、broker、主题、分区等内容。

> ZooKeeper是一个开源的分布式协调服务，是Google Chubby的一个开源实现。分布式应用程序可以基于ZooKeeper实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、配置维护等功能。在ZooKeeper中共有3个角色：leader、follower和observer，同一时刻 ZooKeeper集群中只会有一个leader，其他的都是follower和observer。observer不参与投票，默认情况下 ZooKeeper 中只有 leader 和 follower 两个角色。

1.上传zookeeper-3.4.14.tar.gz到服务器

2.解压：tar -zxvf zookeeper-3.4.14.tar.gz -C /opt/servers

3.切换到conf目录：cd zookeeper-3.4.14/conf

4.复制并重命名配置文件：cp zoo_sample.cfg zoo.cfg

5.编辑配置文件：vim zoo.cfg

```shell
# 修改Zookeeper保存数据的目录，dataDir
dataDir=/var/fishleap/zookeeper/data
```

6.配置zookeeper的环境变量

```shell
# ZOOKEEPER_PREFIX指向Zookeeper的解压目录；
export ZOOKEEPER_PREFIX=/opt/servers/zookeeper-3.4.14
# 将Zookeeper的bin目录添加到PATH中：
export PATH=$PATH:$ZOOKEEPER_PREFIX/bin
# 设置环境变量ZOO_LOG_DIR，指定Zookeeper保存日志的位置；
export ZOO_LOG_DIR=/var/fishleap/zookeeper/log
```

7.验证

```shell
[root@atzlg8 zookeeper-3.4.14]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /opt/servers/zookeeper-3.4.14/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@atzlg8 zookeeper-3.4.14]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/servers/zookeeper-3.4.14/bin/../conf/zoo.cfg
Mode: standalone

[root@atzlg8 zookeeper-3.4.14]# cd /var/fishleap/zookeeper/
[root@atzlg8 zookeeper]# ls
data  log
```

#### 2.4 kafka安装与配置

1.上传kafka_2.12-1.0.2.tgz到服务器并解压：tar -zxf kafka_2.12-1.0.2.tgz -C /opt/servers

2.配置环境变量并全局生效

```shell
vim /etc/profile
# 添加内容
export KAFKA_HOME=/opt/servers/kafka_2.12-1.0.2
export PATH=$PATH:$KAFKA_HOME/bin
# 生效
source /etc/profile
```

3.配置/opt/kafka_2.12-1.0.2/config中的server.properties文件

```shell
# Kafka连接Zookeeper的地址，此处使用本地启动的Zookeeper实例，连接地址是localhost:2181，
# 后面的 myKafka 是Kafka在Zookeeper中的根节点路径
zookeeper.connect=localhost:2181/mykafka
# 配置Kafka存储持久化数据的目录
log.dirs=/var/fishleap/kafka/kafka-logs
# 创建持久化目录
mkdir -p /var/fishleap/kafka/kafka-logs
```

4.若此前Zookeeper没有启动，下面先启动Zookeeper

```shell
[root@atzlg8 bin]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /opt/servers/zookeeper-3.4.14/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
# 查看zookeeper的状态
[root@atzlg8 bin]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/servers/zookeeper-3.4.14/bin/../conf/zoo.cfg
Mode: standalone
```

5.启动kafka：进入Kafka安装的根目录，执行如下命令

```shell
[root@atzlg8 kafka_2.12-1.0.2]# kafka-server-start.sh config/server.properties
# 启动成功，可以看到控制台输出的最后一行的started状态
[2020-11-24 21:44:26,073] INFO ......
[2020-11-24 21:44:26,073] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

6.启动zk客户端查看zookeeper上mykafka节点

```shell
[root@atzlg8 ~]# zkCli.sh
WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] ls /
[mykafka, zookeeper]
[zk: localhost:2181(CONNECTED) 1] ls /mykafka
[cluster, controller, controller_epoch, brokers, admin, isr_change_notification, consumers, log_dir_event_notification, latest_producer_id_block, config]
[zk: localhost:2181(CONNECTED) 2] quit
Quitting...
```

7.停止kafka：此时Kafka是前台模式启动，要停止，使用Ctrl+C 或 `kafka-server-stop.sh` 。

```shell
[2020-11-24 21:52:32,244] INFO [KafkaServer id=0] shut down completed (kafka.server.KafkaServer)
```

若要后台启动kafka，可以使用下列命令：

```shell
[root@atzlg8 kafka_2.12-1.0.2]# kafka-server-start.sh -daemon config/server.properties
# 或者
[root@atzlg8 kafka_2.12-1.0.2]# kafka-server-start.sh config/server.properties &

# 查看Kafka的后台进程
ps aux | grep kafka
# 通过jps命令查看Kafka服务进程是否已经启动，jps命令只是用来确认Kafka服务的进程已经正常启动
jps -l
# 查看进程
[root@atzlg8 ~]# jps
15625 Jps
14589 QuorumPeerMain
15406 Kafka

# 停止后台运行的Kafka
[root@atzlg8 kafka_2.12-1.0.2]# kafka-server-stop.sh
```

### 3.生产与消费

#### 3.1 用于测试类工作的生产与消费脚本

**kafka-topics.sh 用于管理主题**

```shell
# 列出现有的主题
[root@atzlg8 ~]# kafka-topics.sh --list --zookeeper localhost:2181/mykafka
# 创建主题，该主题包含一个分区，该分区为Leader分区，它没有Follower分区副本
[root@atzlg8 ~]# kafka-topics.sh --zookeeper localhost:2181/mykafka --create --topic topic_1 --partitions 1 --replication-factor 1
# 查看分区信息
[root@atzlg8 ~]# kafka-topics.sh --zookeeper localhost:2181/mykafka --list
# 查看指定主题的详细信息
[root@atzlg8 ~]# kafka-topics.sh --zookeeper localhost:2181/mykafka --describe --topic topic_1
# 删除指定主题
[root@atzlg8 ~]# kafka-topics.sh --zookeeper localhost:2181/mykafka --delete --topic topic_1
```

其中--zookeeper指定了Kafka所连接的ZooKeeper服务地址，--topic指定了所要创建主题的名称，--replication-factor 指定了副本因子，--partitions 指定了分区个数，--create是创建主题的动作指令；还可以通过--describe展示主题的更多具体信息。

**kafka-console-producer.sh用于生产消息**

```shell
# 开启生产者
[root@atzlg8 ~]# kafka-console-producer.sh --broker-list localhost:9020 --topic topic_1
>
```

其中--broker-list指定了连接的Kafka集群地址，--topic指定了发送消息时的主题。

**kafka-console-consumer.sh用于消费消息**

```shell
# 开启消费者
[root@atzlg8 ~]# kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic_1
# 开启消费者方式二，从头消费，不按照偏移量消费
[root@atzlg8 ~]# kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic_1 --from-beginning
```

其中--bootstrap-server指定了连接的Kafka集群地址，--topic指定了消费者订阅的主题。

#### 3.2 Java端消息的发送与接收

**生产者主要的对象有KafkaProducer、ProducerRecord，其中 KafkaProducer 是用于发送消息的类， ProducerRecord 类用于封装Kafka的消息**。生产者生产消息后，需要broker端的确认，可以同步确认，也可以异步确认。同步确认效率低，异步确认效率高，但是需要设置回调对象。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201127230548833.png" alt="image-20201127230548833" style="zoom:80%;" />

KafkaProducer 的创建需要指定相关参数，可以从 `org.apache.kafka.clients.producer.ProducerConfig` 配置类中找到。其中常用的参数含义如下：

- **bootstrap.servers**：**配置生产者如何与broker建立连接**。如果是kafka集群，不需要配置全部broke地址，当生产者连接上指定的broker之后，再通过该连接发现集群中的其它节点。
- **key.serializer**：要**发送信息的key数据的序列化类**。
- **value.serializer**：要**发送消息的alue数据的序列化类**。
- **acks**：默认值："1"。
    - acks="0"：**生产者只要把消息放到缓冲区，就认为消息已经发送完成**。该情形不能保证broker是否真的收到了消息，retries配置也不会生效。发送的消息的返回的消息偏移量永远是-1。
    - acks="1"：**消息只要写到主分区，而不等待副本分区确认，就认为消息已经发送完成**。在该情形下，如果主分区收到消息确认之后就宕机了，而副本分区还没来得及同步该消息，则该消息丢失。
    - acks="all"：**首领分区会等待所有的ISR副本分区确认记录**。该情形保证了只要有一个ISR副本分区存活，消息就不会丢失。
- **retries**：**重试次数**。当消息发送出现错误的时候，系统会重发消息。设置 `MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION=1` 可以保证在重试中消息的有序性，否则在重试此失败消息的时候，其他的消息可能发送成功了。

Kafka不支持消息的推送，我们可以自己实现。Kafka采用的是消息的拉取（poll方法）。消费者主要的对象有KafkaConsumer用于消费消息的类。ConsumerConfig类中包含了所有的可以配置的参数。相关参数和含义：

- `bootstrap.servers`：与kafka建立初始连接的broker地址列表；
- `key.deserializer`：key的反序列化器；
- `value.deserializer`：value的反序列化器；
- `group.id`：消费组id，标识该消费者所属的消费组；
- `auto.offset.reset`：**当kafka中没有初始偏移量或当前偏移量在服务器中不存在，该如何处理**。
    - earliest：自动重置偏移量到最早的偏移量；
    - latest：自动重置偏移量为最新的偏移量；
    - none：如果消费组原来的（previous）偏移量不存在，则向消费者抛异常；
    - anything：向消费者抛异常。

Java端引入Kafka依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <!-- 高版本兼容低版本,使用和broker一致的版本 -->
    <version>1.0.2</version>
</dependency>
```

要往Kafka中写入消息，首先要创建一个生产者客户端实例并设置一些配置参数，然后构建消息的ProducerRecord对象，其中必须包含所要发往的主题及消息的消息体，进而再通过生产者客户端实例将消息发出，最后可以通过 close（）方法来关闭生产者客户端实例并回收相应的资源。

```java
/**
 * @author zlg
 * 同步等待消息的确认
 */
public class MyProducer1 {
    
    public static void main(String[] args) {
    
        // 指定相关参数
        final HashMap<String, Object> configs = new HashMap<>(16);
        // 设置连接Kafka的初始连接用到的服务器地址
        configs.put("bootstrap.servers", "atzlg8:9092");
        // 设置key的序列化
        configs.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
        // 设置value的序列化器
        configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // broker确认模式
        configs.put("acks", "1");
     
        // 生产者用来发送消息
        try (final KafkaProducer<Integer, String> kafkaProducer = new KafkaProducer<Integer, String>(configs);) {
            
            // 用来封装消息:参数含义(主题名称、分区编号、key、value值)
            final ProducerRecord<Integer, String> producerRecord = new ProducerRecord<Integer, String>("topic_1", 0, 0,"hello world");
            // 发送消息，同步等待消息的确认
            kafkaProducer.send(producerRecord).get(3_000, TimeUnit.MILLISECONDS);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
        
    }
}
```

```java
/**
 * @author zlg
 * @create 使用回调异步等待消息的确认
 */
public class MyProducer2 {
    
    public static void main(String[] args) {
    
        // 指定相关参数
        final HashMap<String, Object> configs = new HashMap<>(16);
        // 设置连接Kafka的初始连接用到的服务器地址
        configs.put("bootstrap.servers", "atzlg8:9092");
        // 设置key的序列化
        configs.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
        // 设置value的序列化器
        configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
     
        // 生产者用来发送消息
        try (final KafkaProducer<Integer, String> kafkaProducer = new KafkaProducer<Integer, String>(configs);) {
            
            // 用来封装消息:参数含义(主题名称、分区编号、key、value值)
//            final ProducerRecord<Integer, String> producerRecord = new ProducerRecord<Integer, String>("topic_1", 0, 1,"hello world 2");
    
            // 批量发送消息
            for (int i = 100; i < 200; i++) {
                final ProducerRecord<Integer, String> producerRecord = new ProducerRecord<Integer, String>(
                        "topic_1", 0, i,"hello world " + i);
    
                // 发送消息，使用回调异步等待消息的确认
                kafkaProducer.send(producerRecord, (final RecordMetadata metadata, final Exception exception) -> {
        
                    if (exception == null) {
                        System.out.println("主题：" + metadata.topic() + "\n"
                                + "分区：" + metadata.partition() + "\n"
                                + "偏移量：" + metadata.offset() + "\n"
                                + "序列化的key字节：" + metadata.serializedKeySize() + "\n"
                                + "序列化的value字节：" + metadata.serializedValueSize() + "\n"
                                + "时间戳：" + metadata.timestamp());
                    } else {
                        System.out.println("有异常：" + exception.getMessage());
                    }
        
                });
            }
            
        } catch (Exception e) {
            e.printStackTrace();
        }
        
    }
}
```

消费者端：创建一个消费者客户端实例并配置相应的参数，然后订阅主题并消费即可。

```java
/**
 * @author zlg
 * @create 消费者
 */
public class MyConsumer1 {
    
    private static final Pattern PATTERN = Pattern.compile("topic_[0-9]");
    
    public static void main(String[] args) {
        final HashMap<String, Object> configs = new HashMap<>(16);
        // 指定bootstrap.servers属性作为初始化连接Kafka的服务器
        configs.put("bootstrap.servers", "atzlg8:9092");
        // key的反序列化器
        configs.put("key.deserializer", "org.apache.kafka.common.serialization.IntegerDeserializer");
        // value的反序列化器
        configs.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        // 消费组id
        configs.put("group.id", "consumer.demo");
        // 如果找不到当前消费者的有效偏移量,则自动重置到最开始
        // latest标识直接重置到消息偏移量的最后一个
        configs.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    
        // 创建消费者
        final KafkaConsumer<Integer, String> consumer = new KafkaConsumer<>(configs);
    
        // 消费者订阅主题或分区
        consumer.subscribe(PATTERN, new ConsumerRebalanceListener() {
            @Override
            public void onPartitionsRevoked(final Collection<TopicPartition> partitions) {
                partitions.forEach(tp -> {
                    System.out.println("剥夺的分区：" + tp.partition());
                });
            }
    
            @Override
            public void onPartitionsAssigned(final Collection<TopicPartition> partitions) {
                partitions.forEach(tp -> {
                    System.out.println(tp.partition());
                });
            }
        });
        
        // 批量从主题的分区拉取消息，若主题中没有可以消费的消息，每过3秒重新拉取一次
        final ConsumerRecords<Integer, String> consumerRecords = consumer.poll(3_000);
        // 获取topic_1主题的消息
        final Iterable<ConsumerRecord<Integer, String>> topic1Iterable = consumerRecords.records("topic_1");
        // 遍历topic_1主题的消息
        topic1Iterable.forEach(record -> {
            System.out.println("========================================");
            System.out.println("消息头字段：" + Arrays.toString(record.headers().toArray()));
            System.out.println("消息的key：" + record.key());
            System.out.println("消息的偏移量：" + record.offset());
            System.out.println("消息的分区号：" + record.partition());
            System.out.println("消息的序列化key字节数：" + record.serializedKeySize());
            System.out.println("消息的序列化value字节数：" + record.serializedValueSize());
            System.out.println("消息的时间戳：" + record.timestamp());
            System.out.println("消息的时间戳类型：" + record.timestampType());
            System.out.println("消息的主题：" + record.topic());
            System.out.println("消息的值：" + record.value());
        });
        
        // 关闭消费者
        consumer.close();
        
    }
}
```

#### 3.3 SpringBoot整合Kafka

引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

application.properties

```properties
spring.application.name=springboot-kafka
server.port=8080

# 用于建立初始连接的broker地址
spring.kafka.bootstrap-servers=atzlg8:9092

# producer用到的key和value的序列化类
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.IntegerSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
# 默认的批处理记录数,每个批次最多放多少条记录
spring.kafka.producer.batch-size=16384
# 32MB的总发送缓存,总的可用发送缓冲区大小
spring.kafka.producer.buffer-memory=33554432

# consumer用到的key和value的反序列化类
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.IntegerDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
# consumer的消费组id
spring.kafka.consumer.group-id=spring-consumer02
# 如果该消费者的偏移量不存在，则自动设置为最早的偏移量
spring.kafka.consumer.auto-offset-reset=earliest
# 是否自动提交消费者偏移量
spring.kafka.consumer.enable-auto-commit=true
# 每隔100ms向broker提交一次偏移量
spring.kafka.consumer.auto-commit-interval=100
```

生产者：

```java
/**
 * @author zlg
 * @create 生产者
 */
@RestController
public class ProducerController {
    
    @Autowired
    private KafkaTemplate<Integer, String> kafkaTemplate;
    
    // localhost:8080/send/sync/hello%20world
    @RequestMapping("/send/sync/{message}")
    public String sendMessageSync(@PathVariable String message) {
    	// 若没有topic_2主题，则应用启动时KafkaAdmin会自动帮忙创建topic_2主题
        final ListenableFuture<SendResult<Integer, String>> future = kafkaTemplate.send("topic_2", 0, 0, message);
        try {
            // 同步发送消息
            final SendResult<Integer, String> sendResult = future.get();
            final RecordMetadata metadata = sendResult.getRecordMetadata();
            System.out.println(metadata.topic() + "\t" + metadata.partition() + "\t" + metadata.offset());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    
        return "success";
    }
    
    // localhost:8080/send/async/hello%20world%20abc
    @RequestMapping("/send/async/{message}")
    public String sendMessageAsync(@PathVariable String message) {
        final ListenableFuture<SendResult<Integer, String>> future = kafkaTemplate.send("topic_2", 0, 1, message);
        
        // 设置回调函数，异步等待broker端的返回结果
        future.addCallback(new ListenableFutureCallback<SendResult<Integer, String>>() {
            @Override
            public void onFailure(final Throwable throwable) {
                System.out.println("发送消息失败：" + throwable.getMessage());
            }
    
            @Override
            public void onSuccess(final SendResult<Integer, String> sendResult) {
                final RecordMetadata metadata = sendResult.getRecordMetadata();
                System.out.println("发送消息成功：" + metadata.topic() + "\t" + metadata.partition() + "\t" + metadata.offset());
            }
        });
        
        return "success";
    }
}
```

消费者监听：

```java
/**
 * @author zlg
 * @create 监听类
 */
@Component
public class MyConsumer {
    
    /**
     * ConsumerRecord Kafka消息
     * acknowledgment 是否手动确认
     * @Payload 验证支持
     * @Header KafkaHeaders定义的特定标头值
     * @Headers 给Map的参数访问所有标头
     * MessageHeaders 获取所有header信息
     * MessageHeaderAccessor 方便地访问所有方法参数
     */
    @KafkaListener(topics = "topic_2")
    public void onMessage(ConsumerRecord<Integer, String> consumerRecord) {
        final Optional<ConsumerRecord<Integer, String>> optional = Optional.ofNullable(consumerRecord);
        optional.ifPresent(record -> {
            System.out.println(
                    record.topic() + "\t"
                    + record.partition() + "\t"
                    + record.offset() + "\t"
                    + record.key() + "\t"
                    + record.value());
        });
    }

}
```

启动类：

```java
@SpringBootApplication
public class SpringbootKafkaApplication {

    public static void main(String[] args) {
        SpringApplication.run(Demo02SpringbootKafkaApplication.class, args);
    }

}
```

应用启动后，在浏览器上访问：http://localhost:8080/send/sync/hello%20world  、 http://localhost:8080/send/async/hello%20world%20async  得到如下结果：

```java
// http://localhost:8080/send/sync/hello%20world  同步获取消息
topic_2	0	0
topic_2	0	0	0	hello world
// http://localhost:8080/send/async/hello%20world%20async  异步获取消息
topic_2	0	2	1	hello world async
发送消息成功：topic_2	0	2
```

另外KafkaConfig配置类中也可以配置主题、KafkaAdmin、KafkaTemplate等，创建新的主题，覆盖原有配置等内容。

![image-20201130005725151](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201130005725151.png)

配置示例：

```java
/**
 * @author zlg
 * @create Kafka配置类
 */
@Configuration
public class KafkaConfig {
    // 创建新的主题
    @Bean
    public NewTopic topic1() {
        return new NewTopic("ntp-01", 5, (short) 1);
    }
    @Bean
    public NewTopic topic2() {
        return new NewTopic("ntp-02", 3, (short) 1);
    }
    
    // 配置KafkaAdmin：增删改查主题
    @Bean
    public KafkaAdmin kafkaAdmin() {
        // 覆盖原有配置文件中的配置
        final Map<String, Object> config = new HashMap<>(16);
        config.put("bootstrap.servers", "atzlg8:9092");
        return new KafkaAdmin(config);
    }
    
    // 配置KafkaTemplate：发送/接受消息
    @Autowired
    @Bean
    public KafkaTemplate<Integer, String> kafkaTemplate(ProducerFactory<Integer,String> producerFactory) {
        // 覆盖ProducerFactory原有设置，也就是application配置文件中的设置
        Map<String, Object> configOverrides = new HashMap<>(16);
        configOverrides.put(ProducerConfig.BATCH_SIZE_CONFIG, 160);
        return new KafkaTemplate<>(producerFactory, configOverrides);
    }
}
```

### 4.服务端参数配置

下面挑选一些重要的Kafka服务端参数（broker configs）来说明以下，这些参数都配置在`$KAFKA_HOME/config/server.properties` 文件中。

**zookeeper.connect**

**该参数指明broker要连接的ZooKeeper集群的服务地址**（包含端口号）。如果ZooKeeper集群中有多个节点，则可以用逗号将每个节点隔开，类似于 localhost1：2181，localhost2：2181，localhost3：2181这种格式。最佳的实践方式是在Zookeeper根路径下再加一个chroot路径（自定义，如localhost：2181/kafka），这样既可以明确指明该chroot路径下的节点是为Kafka所用的，也可以实现多个Kafka集群复用一套ZooKeeper集群。如果不指定chroot，那么默认使用ZooKeeper的根路径。

```shell
zookeeper.connect=node1:2181,node2:2181,node3:2181/myKafka
```

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201130223256798.png" alt="image-20201130223256798" />

**listeners**

**该参数指明broker监听客户端连接的地址列表**，即为客户端要连接broker的入口地址列表，配置格式为 `protocol：//hostname：port`，其中protocol代表协议类型，hostname代表主机名，port代表服务端口，此参数的默认值为 null。比如此参数配置为 PLAINTEXT：//198.162.0.2：9092，如果有多个地址，则中间以逗号隔开。advertised.listeners 主要用于 IaaS（Infrastructure as a Service）环境，可以设置advertised.listeners参数绑定公网IP供外部客户端使用，而配置listeners参数来绑定私网IP地址供broker间通信使用。

- listener.security.protocol.map：监听器名称和安全协议的映射配置。每个监听器的名称只能在map中出现一次。
- listeners：用于配置broker监听的URI以及监听器名称列表，使用逗号隔开多个URI及监听器名称；如果监听器名称代表的不是安全协议，必须配置listener.security.protocol.map；每个监听器必须使用不同的网络端口。
- inter.broker.listener.name：用于配置broker之间通信使用的监听器名称，该名称必须在advertised.listeners列表中。
- advertised.listeners：需要将该地址发布到zookeeper供客户端使用，如果客户端使用的地址与listeners配置不同。可以在zookeeper的 `get /myKafka/brokers/ids/<broker.id>`  中找到。advertised.listeners的地址必须是listeners中配置的或配置的一部分。

![image-20201130224301458](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201130224301458.png)

**broker.id**

**该参数用来指定Kafka集群中broker的唯一标识**，默认值为-1。如果没有设置，那么Kafka会自动生成一个。这个参数还和meta.properties文件及服务端参数broker.id.generation.enable和reserved.broker.max.id有关。

![image-20201130225038791](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201130225038791.png)

**log.dir和log.dirs**

Kafka 把所有的消息都保存在磁盘上，而这两个参数用来**配置 Kafka 日志文件存放的根目录**。一般情况下，log.dir 用来配置单个根目录，而 log.dirs 用来配置多个根目录（以逗号分隔）。log.dir和log.dirs其实都可以用来配置单个或多个根目录。log.dirs 的优先级比 log.dir 高，但是如果没有配置log.dirs，则会以 log.dir 配置为准。默认情况下只配置了 log.dir 参数，其默认值为/tmp/kafka-logs。

如果指定了多个路径，那么broker 会根据“最少使用”原则，把同一个分区的日志片段保存到同一个路径下。broker 会往拥有最少数目分区的路径新增分区，而不是往拥有最小磁盘空间的路径新增分区。

![image-20201130225306417](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201130225306417.png)

**message.max.bytes**

该参数用来**指定broker所能接收消息的最大值**，默认值为1000012（B），约等于976.6KB。如果 Producer 发送的消息大于这个参数所设置的值，那么（Producer）就会报出RecordTooLargeException的异常。如果需要修改这个参数，那么还要考虑max.request.size（客户端参数）、max.message.bytes（topic端参数）等参数的影响。

## 二.生产者

生产者就是负责向Kafka发送消息的应用程序。对于**KafkaProducer**而言，它是**线程安全**的，我们可以在多线程的环境中复用它，而对于消费者客户端**KafkaConsumer**而言，它是**非线程安全**的，因为它具备了状态。

### 1.客户端开发

#### 1.1 开发步骤

一个正常的生产逻辑需要具备以下几个步骤：**配置生产者客户端参数及创建相应的生产者实例、构建待发送的消息、发送消息、关闭生产者实例**。将上午中的生产者代码优化后如下：

```java
public class MyProducer {
    
    private static final String BROKERLIST = "atzlg8:9092";
    private static final String TOPIC = "topic_1";
    
    private static Properties initConfig() {
        // 指定相关参数
        final Properties configs = new Properties();
        // 设置连接Kafka的初始连接用到的服务器地址
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BROKERLIST);
        // 设置key的序列化
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, IntegerSerializer.class.getName());
        // 设置value的序列化器
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        // broker确认模式
        configs.put(ProducerConfig.ACKS_CONFIG, "1");
        return configs;
    }
    
    public static void main(String[] args) {
    
        Properties configs = initConfig();
     
        // 生产者用来发送消息
        try (final KafkaProducer<Integer, String> producer = new KafkaProducer<>(configs);) {
            
            // 用来封装消息:参数含义(主题名称、分区编号、key、value值)
            final ProducerRecord<Integer, String> record = new ProducerRecord<>(TOPIC, 0, 0,"hello world");
            // 发送消息，同步等待消息的确认
//            final long time = Duration.ofMillis(3_000).toMillis();
            producer.send(record).get(3_000, TimeUnit.MILLISECONDS);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
        
    }
}
```

#### 1.2 创建生产者及必要的参数配置

在Kafka生产者客户端KafkaProducer中有3个参数是必填的：

- `bootstrap.servers`：指定生产者客户端连接Kafka集群所需的broker地址清单，可以设置一个或多个地址，中间以逗号隔开，此参数的默认值为“”。不需要配置所有地址信息，生产者会从给定的broker里查找到其他broker的信息。建议至少设置两个以上的broker 地址信息，当其中任意一个宕机时，生产者仍然可以连接到 Kafka集群上。
- `key.serializer` 和 `value.serializer`：broker 端接收的消息必须以字节数组（byte[]）的形式存在。生产者在发往broker之前需要将消息中对应的key（key.serializer序列化器）和value（value.serializer序列化器）做相应的序列化操作来转换成字节数组。这里必须填写序列化器的全限定名。
- **client.id**：设定KafkaProducer对应的客户端id，默认值为“”。如果客户端不设置，则KafkaProducer会自动生成一个非空字符串，即字符串“producer-”与数字的拼接。

我们可以直接使用客户端中的`org.apache.kafka.clients.producer.ProducerConfig`类来做一定程度上的预防措施，每个静态参数在 ProducerConfig类中都有对应的名称。key.serializer和value.serializer参数对应类的全限定名比较长，可以通过`StringSerializer.class.getName()`做下改进。

如在创建 KafkaProducer 实例时并没有设定key.serializer 和value.serializer 这两个配置参数，那么就需要在构造方法中添加对应的序列化器：

```java
KafkaProducer<Integer, String> producer = new KafkaProducer<>(configs, new IntegerSerializer(), new StringSerializer());
```

#### 1.3 构建消息对象及消息的发送

与业务相关的消息体只是构建的消息对象 ProducerRecord 其中的一个 value 属性。

```java
public class ProducerRecord<K, V> {
    private final String topic;  // 主题(必填)
    private final Integer partition;  // 分区号
    private final Headers headers;  // 消息头部
    private final K key;  // 键
    private final V value;  // 值(必填)
    private final Long timestamp;  // 消息的时间戳
    
    public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value) {}
    public ProducerRecord(String topic, Integer partition, K key, V value, Iterable<Header> headers) {}
    public ProducerRecord(String topic, Integer partition, K key, V value) {}
    public ProducerRecord(String topic, K key, V value) {}
    public ProducerRecord(String topic, V value) {}
    // ......
}    
```

其中`topic`和`partition`字段分别代表消息要发往的主题和分区号。`headers`字段是消息的头部（设定一些与应用相关的信息）。`key`是用来指定消息的键，可以用来计算分区号进而可以让消息发往特定的分区。消息以主题为单位进行归类，而这个key可以让消息再进行二次归类，同一个key的消息会被划分到同一个分区中，有key的消息还可以支持日志压缩的功能。`value`是指消息体，一般不为空，如果为空则表示特定的消息—墓碑消息。`timestamp`是指消息的时间戳，它有CreateTime（消息创建的时间）和LogAppendTime（消息追加到日志文件的时间）两种类型。

创建生产者实例和构建消息之后，就可以开始发送消息了。发送消息主要有三种模式：**发后即忘**（fire-and-forget）、**同步**（sync）及**异步**（async）。

KafkaProducer的send()方法有两个重载方法：

```java
public Future<RecordMetadata> send(ProducerRecord<K, V> record) {}
public Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback) {}
```

要**实现同步的发送方式，可以利用返回的Future对象实现**，send（）方法本身就是异步的，send（）方法返回的Future对象可以使调用方稍后获得发送的结果。

```java
// 发后即忘：只管往Kafka中发送消息而并不关心消息是否正确到达。
Future<RecordMetadata> future = producer.send(record);

// 同步：利用返回的Future对象实现，RecordMetadata对象里包含了消息的一些元数据信息
RecordMetadata recordMetadata = producer.send(record).get();
// Future 表示一个任务的生命周期，可以判断任务是否已经完成或取消，以及获取任务的结果和取消任务等
RecordMetadata recordMetadata = producer.send(record).get(3_000, TimeUnit.MILLISECONDS);
```

**异步发送一般在send（）方法里指定一个Callback的回调函数，Kafka在返回响应时调用该函数来实现异步的发送确认**。对于同一个分区而言，如果消息record1于record2之前先发送，那么KafkaProducer就可以保证对应的callback1在callback2之前调用，即**回调函数的调用也可以保证分区有序**。

```java
// 异步
kafkaProducer.send(record, (final RecordMetadata metadata, final Exception exception) -> {
    if (exception == null) {
        System.out.println("主题：" + metadata.topic() + "\n"
                         + "分区：" + metadata.partition() + "\n"
                         + "偏移量：" + metadata.offset() + "\n"
                         + "序列化的key字节：" + metadata.serializedKeySize() + "\n"
                         + "序列化的value字节：" + metadata.serializedValueSize() + "\n"
                         + "时间戳：" + metadata.timestamp());
    } else {
        System.out.println("有异常：" + exception.getMessage());
    }
});
```

**close（）方法会阻塞，等待之前所有的发送请求完成后再关闭 KafkaProducer**。如果调用了带超时时间timeout的close（）方法，那么只会在等待timeout时间内来完成所有尚未完成的请求处理，然后强行退出。在实际应用中，一般使用的都是无参的close（）方法。

#### 1.4 序列化

![image-20201205122942405](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201205122942405.png)

**生产者需要用序列化器（Serializer）把对象转换成字节数组才能通过网络发送给Kafka。而消费者需要用反序列化器（Deserializer）把从 Kafka 中收到的字节数组转换成相应的对象**。ByteArray、ByteBuffer、Bytes、Double、Integer、Long、String这几种类都实现了`org.apache.kafka.common.serialization.Serializer` 序列化接口。生产者使用的序列化器和消费者使用的反序列化器是需要一一对应的。

```java
public interface Serializer<T> extends Closeable {
    void configure(Map<String, ?> configs, boolean isKey);
    byte[] serialize(String topic, T data);
    void close();
}
```

`configure（）`方法用来配置当前类（创建KafkaProducer实例的时候调用的，主要用来确定编码类型），`serialize（）`方法用来执行序列化操作。而`close（）`方法用来关闭当前的序列化器，一般情况下 close（）是一个空方法，如果实现了此方法，则必须确保此方法的幂等性，因为这个方法很可能会被KafkaProducer调用多次。

如果 Kafka 客户端提供的几种序列化器都无法满足应用需求，则可以选择使用如 Avro、JSON、Thrift、ProtoBuf和Protostuff等通用的序列化工具来实现，或者使用自定义类型的序列化器来实现。

自定义序列化器:

```java
public class UserSerializer implements Serializer<User> {
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        // do nothing
        // 用于接收对序列化器的配置参数，并对当前序列化器进行配置和初始化的
    }
    
    @Override
    public byte[] serialize(String topic, User data) {
        try {
            // 如果数据是null，则返回null
            if (data == null) {
                return null;
            }
            final Integer userId = data.getUserId();
            final String username = data.getUsername();
    
            if (userId != null) {
                if (username != null) {
                    final byte[] bytes = username.getBytes("UTF-8");
                    int length = bytes.length;
                    // 第一个4个字节用于存储userId的值
                    // 第二个4个字节用于存储username字节数组的长度int值
                    // 第三个长度，用于存放username序列化之后的字节数组
                    ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + length);
                    // 设置userId
                    buffer.putInt(userId);
                    // 设置username字节数组长度
                    buffer.putInt(length);
                    // 设置username字节数组
                    buffer.put(bytes);
                    // 以字节数组形式返回user对象的值
                    return buffer.array();
                }
            }
        } catch (Exception e) {
            throw new SerializationException("数据序列化失败");
        }
        return null;
    }
    
    @Override
    public void close() {
        // do nothing
        // 用于关闭资源等操作。需要幂等，即多次调用，效果是一样的。
    }
}
```

生产者中需要配置自定义的序列化器：

```java
configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
// 设置自定义的序列化器
configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, UserSerializer.class);

ProducerRecord<String, User> record = new ProducerRecord<String, User>(
    "tp_user_01",   // topic
    user.getUsername(),   // key
    user                  // value
);
```

实体类：

```java
public class User {
    private Integer userId;
    private String username;
    //......
}    
```

#### 1.5 分区器

![image-20201205123139662](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201205123139662.png)

**消息在通过send（）方法发往broker的过程中，有可能需要经过拦截器（Interceptor）、序列化器（Serializer）和分区器（Partitioner）的一系列作用之后才能被真正地发往 broker**。消息经过序列化之后就需要确定它发往的分区，如果消息ProducerRecord中指定了partition字段，那么就不需要分区器的作用，因为partition代表的就是所要发往的分区号，否则就需要依赖分区器，根据key这个字段来计算partition的值。**分区器的作用就是为消息分配分区**。

Kafka中提供的默认分区器是`org.apache.kafka.clients.producer.internals.DefaultPartitioner`，它实现了`org.apache.kafka.clients.producer.Partitioner`接口。

```java
// 父接口
public interface Configurable {
    // 用来获取配置信息及初始化数据
    void configure(Map<String, ?> configs);
}

public interface Partitioner extends Configurable, Closeable {
    // 计算分区号，返回值为int类型
    // 参数含义：主题、键、序列化后的键、值、序列化后的值，以及集群的元数据信息
    int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);
	// 关闭分区器的时候用来回收一些资源
    void close();
}

/**
 * The default partitioning strategy:
 * <ul>
 * <li>If a partition is specified in the record, use it
 * <li>If no partition is specified but a key is present choose a partition based on a hash of the key
 * <li>If no partition or key is present choose a partition in a round-robin fashion
 */
public class DefaultPartitioner implements Partitioner {
	private final ConcurrentMap<String, AtomicInteger> topicCounterMap = new ConcurrentHashMap<>();
    
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        //若keyBytes为null，就根据counter递增值进行轮询得到分区编号
        if (keyBytes == null) {
            int nextValue = nextValue(topic);
            List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
            // 若存在可用分区，则在可用分区里进行轮询
            if (availablePartitions.size() > 0) {
                int part = Utils.toPositive(nextValue) % availablePartitions.size();
                return availablePartitions.get(part).partition();
            } else {
                // 若没有可用分区，就轮询所有分区（不可用分区）
                return Utils.toPositive(nextValue) % numPartitions;
            }
        } else {
            // 若keyBytes有值，则根据keyBytes的hash值计算分区编号
            return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
        }
    }

    //获取递增值counter
    private int nextValue(String topic) {
        AtomicInteger counter = topicCounterMap.get(topic);
        //第一次
        if (null == counter) {
            counter = new AtomicInteger(ThreadLocalRandom.current().nextInt());
            AtomicInteger currentCounter = topicCounterMap.putIfAbsent(topic, counter);
            if (currentCounter != null) {
                counter = currentCounter;
            }
        }
        //第一次之后递增
        return counter.getAndIncrement();
    }
}
```

在默认分区器 DefaultPartitioner 的实现中，close（）是空方法，而在 partition（）方法中定义了主要的分区分配逻辑。如果 key 不为 null，那么默认的分区器会对 key 进行哈希（采用MurmurHash2算法，具备高运算性能及低碰撞率），最终根据得到的哈希值来计算分区号，拥有相同key的消息会被写入同一个分区。如果key为null，那么消息将会以轮询的方式发往主题内的各个可用分区。

注意：如果 key 不为 null，那么计算得到的分区号会是**所有分区**中的任意一个；如果 key为null，那么计算得到的分区号仅为**可用分区**中的任意一个。

生产者发送消息时根据record计算分区编号：

```java
public class KafkaProducer<K, V> implements Producer<K, V> {
 
    //发送消息
    @Override
    public Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback) {
        // intercept the record, which can be potentially modified; this method does not throw exceptions
        ProducerRecord<K, V> interceptedRecord = this.interceptors.onSend(record);
        return doSend(interceptedRecord, callback);
    }
    
    // 真真发送消息的方法
    private Future<RecordMetadata> doSend(ProducerRecord<K, V> record, Callback callback) {
        int partition = partition(record, serializedKey, serializedValue, cluster);
        //......
    }
    
    /**
     * 根据发送的消息计算分区编号，若record指定了partition就返回当前值，
     * 否则就调用DefaultPartitioner的partition方法来计算分区编号。
     */
    private int partition(ProducerRecord<K, V> record, byte[] serializedKey, byte[] serializedValue, Cluster cluster) {
        Integer partition = record.partition();
        return partition != null ?
                partition :
                partitioner.partition(
                        record.topic(), record.key(), serializedKey, record.value(), serializedValue, cluster);
    }
    
}    
```

除了使用 Kafka 提供的默认分区器进行分区分配，还可以使用自定义的分区器，只需同DefaultPartitioner一样实现Partitioner接口即可。

```java
import org.apache.kafka.common.utils.Utils;

/**
 * @author zlg
 * @create 自定义分区器
 */
public class MyPartitioner implements Partitioner {
    private final AtomicInteger counter = new AtomicInteger(0);
    
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        final List<PartitionInfo> partitionInfos = cluster.partitionsForTopic(topic);
        final int numPartitions = partitionInfos.size();
        if (null == keyBytes) {
            return counter.getAndIncrement() % numPartitions;
        } else {
            return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
        }
    }
    
    @Override
    public void close() { }
    
    @Override
    public void configure(final Map<String, ?> map) { }
}
```

自定义分区器后，生产者中需要配置参数`partitioner.class`来显示指定自定义的分区器。

```java
configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, MyPartitioner.class.getName());
```

#### 1.6 生产者拦截器

![image-20201205134129790](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201205134129790.png)

 Kafka一共有两种拦截器：**生产者拦截器和消费者拦截器**。这里主要叙述生产者拦截器。生产者拦截器既可以用来在消息发送前做一些准备工作，比如按照某个规则过滤不符合要求的消息、修改消息的内容等，也可以用来在发送回调逻辑前做一些定制化的需求，比如统计类工作。

生产者拦截器的使用也很方便，主要是自定义实现`org.apache.kafka.clients.producer.ProducerInterceptor`接口，它和Partitioner接口一样，有一个同样的父接口Configurable。

```java
public interface ProducerInterceptor<K, V> extends Configurable {
    //KafkaProducer在将消息序列化和计算分区之前调用，对消息进行相应的定制化操作
    ProducerRecord<K, V> onSend(ProducerRecord<K, V> record);
	//KafkaProducer在消息被应答（Acknowledgement）之前或消息发送失败时调用，优先Callback之前执行
    void onAcknowledgement(RecordMetadata metadata, Exception exception);
	//在关闭拦截器时执行一些资源的清理工作
    void close();
}
```

下面看下生产者拦截器的具体用法，通过onSend（）方法来为每条消息添加一个前缀“prefix-”，并且通过onAcknowledgement（）方法来计算发送消息的成功率。

```java
/**
 * @author zlg
 * @create 生产者拦截器
 */
public class ProducerInterceptorPrefix implements ProducerInterceptor<String, String> {
    private volatile long sendSuccess = 0;
    private volatile long sendFailure = 0;
    
    @Override
    public ProducerRecord<String, String> onSend(final ProducerRecord<String, String> record) {
        String modifiedValue = "prefix-" + record.value();
        return new ProducerRecord<>(record.topic(), record.partition(), record.timestamp(),record.key(), modifiedValue, record.headers());
    }
    
    @Override
    public void onAcknowledgement(final RecordMetadata recordMetadata, final Exception exception) {
        if (null == exception) {
            sendSuccess++;
        } else {
            sendFailure++;
        }
    }
    
    @Override
    public void close() {
        double successRatio = (double) sendSuccess / (sendFailure + sendFailure);
        System.out.println("[INFO]发送成功率 = " + String.format("%f", successRatio * 100) + "%");
    }
    
    @Override
    public void configure(final Map<String, ?> map) {}
}
```

自定义拦截器类后，需要在 KafkaProducer 的配置参数`interceptor.classes`中指定这个拦截器，此参数的默认值为“”。

```java
configs.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, ProducerInterceptorPrefix.class.getName());
```

**KafkaProducer中还可以指定多个拦截器以形成拦截链**。拦截链会按照 `interceptor.classes` 参数配置的拦截器的顺序来一一执行（配置的时候，各个拦截器之间使用逗号隔开），**应答时也是按照配置顺序来执行**（没有使用栈结构）。若继续添加一个拦截器类ProducerInterceptorPrefix2，用来为每条消息添加另一个前缀“prefix2-"，则修改配置为：

```java
configs.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, ProducerInterceptorPrefix.class.getName() + "," + ProducerInterceptorPrefix2.class.getName);
```

则最终消费到的内容为“prefix2-prefix-hello world”的消息。**在拦截链中，如果某个拦截器执行失败，那么下一个拦截器会接着从上一个执行成功的拦截器继续执行**。

### 2.原理分析

#### 2.1 整体架构

消息在真正发往Kafka之前，可能需要经历拦截器（Interceptor）、序列化器（Serializer）和分区器（Partitioner）等一系列的作用，之后会发生什么呢？生产者客户端整体架构图如下：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201128211455514.png" alt="image-20201128211455514" style="zoom:80%;" />

整个生产者客户端由两个线程协调运行，这两个线程分别为**主线程和Sender线程**（发送线程）。在主线程中由KafkaProducer创建消息，然后通过可能的拦截器、序列化器和分区器的作用之后缓存到**消息累加器**（RecordAccumulator，也称为消息收集器）中。Sender 线程负责从RecordAccumulator中获取消息并将其发送到Kafka中。

##### 2.1.1 主线程

主线程中发送过来的消息都会被追加到RecordAccumulator的某个双端队列（Deque）中，在RecordAccumulator 的内部为**每个分区都维护了一个双端队列**，队列中的内容就是ProducerBatch，即 `Deque＜ProducerBatch＞`。消息写入缓存时，追加到双端队列的尾部；Sender读取消息时，从双端队列的头部读取。**ProducerBatch中可以包含一至多个 ProducerRecord，是一个消息批次，可以减少网络请求的次数以提升整体的吞吐量**。若要向很多分区发送消息，可以将 `buffer.memory` 参数适当调大以增加整体的吞吐量。

##### 2.1.2 RecordAccumulator

RecordAccumulator 主要用来缓存消息以便 Sender 线程可以批量发送，进而减少网络传输的资源消耗以提升性能。RecordAccumulator **缓存的大小**可以通过生产者客户端参数 `buffer.memory` 配置，默认值为 33554432B，即**32MB**。如果生产者发送消息的速度超过发送到服务器的速度，则会导致生产者空间不足，这个时候KafkaProducer的send（）方法调用要么被阻塞，要么抛出异常，这个取决于参数 `max.block.ms` 的配置，此参数的默认值为60000，即60秒。

消息在网络上都是以字节（Byte）的形式传输的，在发送之前需要创建一块内存区域来保存对应的消息。在Kafka生产者客户端中，通过java.io.ByteBuffer实现消息内存的创建和释放。**在RecordAccumulator的内部还有一个BufferPool，它主要用来实现ByteBuffer的复用，以实现缓存的高效利用**。BufferPool只针对**特定大小的ByteBuffer**进行管理，而其他大小的ByteBuffer不会缓存进BufferPool中，这个特定的大小由batch.size参数来指定，默认值为16384B，即**16KB**。可以适当地调大 `batch.size` 参数以便多缓存一些消息。

ProducerBatch的大小和 `batch.size` 参数也有着密切的关系。当**一条消息（ProducerRecord）流入RecordAccumulator时，会先寻找与消息分区所对应的双端队列（如果没有则新建），再从这个双端队列的尾部获取一个 ProducerBatch（如果没有则新建），查看 ProducerBatch 中是否还可以写入这个 ProducerRecord，如果可以则写入，如果不可以则需要创建一个新的ProducerBatch。在新建ProducerBatch时评估这条消息的大小是否超过batch.size参数的大小，如果不超过，那么就以 batch.size 参数的大小来创建 ProducerBatch，这样在使用完这段内存区域之后，可以通过BufferPool 的管理来进行复用；如果超过，那么就以评估的大小来创建ProducerBatch，这段内存区域不会被复用**。

##### 2.1.3 Sender 

Sender 从 RecordAccumulator 中获取缓存的消息之后，会进一步将原本 `<Partition，Deque＜ProducerBatch>>`的保存形式转变成 `<Node，List<ProducerBatch>>`的形式，其中Node表示Kafka集群的broker节点。对于 **KafkaProducer**的应用逻辑而言，我们只**关注向哪个分区中发送哪些消息**，而对于**网络连接**来说，生产者客户端**关心与哪一个具体的broker节点建立的连接**，并不关心消息属于哪一个分区。

Sender还会进一步将 `<Node，List<ProducerBatch>>` 封装成 `<Node, Request>` 的形式，这样就可以将Request请求（Kafka的各种协议请求，对消息发送而言就是具体的ProducerRequest）发往各个Node了。

请求在从Sender线程发往Kafka之前还会保存到**InFlightRequests**中，InFlightRequests保存对象的具体形式为`Map<NodeId，Deque<Request>>`，它的主要作用是**缓存已经发出去但还没有收到响应的请求**（NodeId 是一个 String 类型，表示节点的 id 编号）。可以配置参数 `max.in.flight.requests.per.connection` 限制每个连接最多缓存的请求数，**默认值是5**，即每个连接最多只能缓存 5 个未响应的请求，超过该数值之后就不能再向这个连接发送更多的请求。

#### 2.2 元数据的更新

InFlightRequests还可以获得leastLoadedNode（负载最小的Node），负载最小是通过每个Node在InFlightRequests中还未确认的请求决定的，**未确认的请求越多则认为负载越大**。如下图，Node1的负载最小，即Node1为当前的leastLoadedNode。leastLoadedNode的概念可以用于多个应用场合，比如元数据请求、消费者组播协议的交互。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201128223642441.png" alt="image-20201128223642441" style="zoom:67%;" />

元数据是指Kafka集群的元数据，这些元数据具体记录了集群中有哪些主题，这些主题有哪些分区，每个分区的leader副本分配在哪个节点上，follower副本分配在哪些节点上，哪些副本在AR、ISR等集合中，集群中有哪些节点，控制器节点又是哪一个等信息。

**当客户端中没有需要的元数据信息，比如没有指定的主题信息，或者超过 `metadata.max.age.ms` 时间没有更新元数据都会引起元数据的更新操作**。客户端参数 `metadata.max.age.ms` 的默认值为300000，即**5分钟**。元数据的**更新操作是在客户端内部**进行的，对客户端的外部使用者不可见。**当需要更新元数据时，会先挑选出leastLoadedNode，然后向这个Node发送MetadataRequest请求来获取具体的元数据信息**。这个更新操作是由Sender线程发起的，在创建完MetadataRequest之后同样会存入InFlightRequests，之后的步骤就和发送消息时的类似。**元数据虽然由Sender线程负责更新，但是主线程也需要读取这些信息，这里的数据同步通过synchronized和final关键字来保障**。

### 3.重要的生产者参数

下面再选择一些重要的参数进行讲解：`linger.ms`、 `batch.size`、 `max.in.flight.requests.per.connection` 等参数。

`acks`：**用来指定分区中必须要有多少个副本收到这条消息，生产者才会认为这条消息是成功写入的**。它涉及消息的可靠性和吞吐量之间的权衡。acks参数有3种类型的值（都是**字符串类型**）。

- acks=1。默认值即为1。生产者发送消息之后，只要分区的leader副本成功写入消息，那么它就会收到来自服务端的成功响应。acks设置为1，是消息可靠性和吞吐量之间的折中方案。
- acks=0。生产者发送消息之后不需要等待任何服务端的响应。在其他配置环境相同的情况下，acks 设置为 0 可以达到最大的吞吐量。
- acks=-1或acks=all。生产者在消息发送之后，需要等待ISR中的所有副本都成功写入消息之后才能够收到来自服务端的成功响应。在其他配置环境相同的情况下，acks 设置为-1（all）可以达到最强的可靠性。要获得更高的消息可靠性需要配合 `min.insync.replicas` 等参数的联动。

注意：acks参数配置的值是一个字符串类型，而不是整数类型。

`max.request.size`：**用来限制生产者客户端能发送的消息的最大值**，默认值为 1048576B，即 1MB。

`retries`：**用来配置生产者重试的次数**，默认值为0，即在发生异常的时候不进行任何重试动作。当发生网络抖动、leader副本选举临时性异常时，生产者可以通过配置retries大于0的值来通过内部恢复。但也并不是所有异常都可以通过重试来解决，如消息过大，超过 `max.request.size` 参数配置的值。

`retry.backoff.ms`：重试还和另一个参数retry.backoff.ms有关，这个参数的默认值为100，它用来**设定两次重试之间的时间间隔**，避免无效的频繁重试。

注意：在需要保证消息顺序的场合建议把参数 `max.in.flight.requests.per.connection` 配置为1，而不是把acks配置为0，不过这样也会影响整体的吞吐。

`compression.type`：用来**指定消息的压缩方式**，默认值为“none”，即默认情况下，消息不会被压缩。消息压缩是一种使用时间换空间的优化方式，如果对时延有一定的要求，则不推荐对消息进行压缩。

`connections.max.idle.ms`：用来**指定在多久之后关闭限制的连接**，默认值是540000（ms），即9分钟。

`linger.ms`：用来**指定生产者发送 ProducerBatch 之前等待更多消息（ProducerRecord）加入ProducerBatch 的时间**，默认值为 0。生产者客户端会在 ProducerBatch 被填满或等待时间超过linger.ms 值时发送出去。

`receive.buffer.bytes`：用来**设置Socket接收消息缓冲区（SO_RECBUF）的大小**，默认值为32768（B），即32KB。如果设置为-1，则使用操作系统的默认值。

`send.buffer.bytes`：用来**设置Socket发送消息缓冲区（SO_SNDBUF）的大小**，默认值为131072（B），即128KB。与receive.buffer.bytes参数一样，如果设置为-1，则使用操作系统的默认值。

`request.timeout.ms`：用来**配置Producer等待请求响应的最长时间**，默认值为30000（ms）。请求超时之后可以选择进行重试。注意这个参数需要比broker端参数`replica.lag.time.max.ms`的值要大，这样可以减少因客户端重试而引起的消息重复的概率。

下面是其它部分生产者客户端参数列表：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201129001929563.png" alt="image-20201129001929563" style="zoom:60%;" />

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201129002627038.png" alt="image-20201129002627038" style="zoom:72%;" />

## 三.消费者

与生产者对应的是消费者，应用程序可以通过KafkaConsumer来订阅主题，并从订阅的主题中拉取消息。

### 1.消费者与消费组

消费者（Consumer）负责订阅Kafka中的主题（Topic），并且从订阅的主题上拉取消息。每个消费者都有一个对应的消费组（Consumer Group）。当消息发布到主题后，只会被投递给订阅它的**每个**消费组中的一个消费者。

**一个消费者可以消费多个分区，但是每一个分区只能被一个消费组中的一个消费者所消费，如果消费者的个数大于分区个数的情况，就会有消费者分配不到任何分区**。分区如何分配是基于默认的**分区分配策略**进行分析的，可以通过消费者客户端参数 `partition.assignment.strategy` 来设置消费者与订阅主题之间的分区分配策略。

对于消息中间件而言，一般有两种消息投递模式：**点对点（P2P，Point-to-Point）模式和发布/订阅（Pub/Sub）模式**。点对点模式是基于队列的，消息生产者发送消息到队列，消息消费者从队列中接收消息。发布订阅模式定义了如何向一个内容节点发布和订阅消息，这个内容节点称为主题（Topic），主题可以认为是消息传递的中介，消息发布者将消息发布到某个主题，而消息订阅者从主题中订阅消息。Kafka 同时支持两种消息投递模式，而这正是得益于消费者与消费组模型的契合：

- 如果所有的消费者都隶属于**同一个消费组**，那么所有的消息都会被**均衡**地投递给每一个消费者，即**每条消息只会被一个消费者处理**，这就相当于点对点模式的应用。
- 如果所有的消费者都隶属于**不同的消费组**，那么所有的消息都会被**广播**给所有的消费者，即**每条消息会被所有的消费者处理**，这就相当于发布/订阅模式的应用。

消费组是一个逻辑上的概念，它**将旗下的消费者归为一类，每一个消费者只隶属于一个消费组**。每一个消费组都会有一个固定的名称，消费者在进行消费前需要指定其所属消费组的名称，这个可以通过消费者客户端参数 `group.id` 来配置，默认值为空字符串。

消费者并非逻辑上的概念，它是实际的应用实例，它可以是**一个线程，也可以是一个进程**。同一个消费组内的消费者既可以部署在同一台机器上，也可以部署在不同的机器上。

### 2.客户端开发

消费逻辑需要具备以下几个步骤：配置消费者客户端参数及创建相应的消费者实例、订阅主题、拉取消息并消费、提交消费位移、关闭消费者实例。

```java
/**
 * @author zlg
 * @create 消费者客户端开发
 */
public class MyConsumer {
    
    private static final Logger logger = LoggerFactory.getLogger(MyConsumer2.class);
    private static final String BROKERLIST = "atzlg8:9092";
    private static final String TOPIC = "topic_1";
    private static final String GROUPID = "consumer.demo";
    private static final String CLIENTID = "consumer.client.id.demo";
    private static final AtomicBoolean ISRUNNING = new AtomicBoolean(true);
    
    private static Properties initConfig() {
        Properties configs = new Properties();
        // 指定bootstrap.servers属性作为初始化连接Kafka的服务器
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BROKERLIST);
        // key的反序列化器
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class.getName());
        // value的反序列化器
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // 消费组id
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, GROUPID);
        // 消费者客户端id
        configs.put(ConsumerConfig.CLIENT_ID_CONFIG, CLIENTID);
        // 如果找不到当前消费者的有效偏移量,则自动重置到最开始
        // latest标识直接重置到消息偏移量的最后一个
        configs.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        return configs;
    }
    
    public static void main(String[] args) {
        final Properties props = initConfig();
    
        // 创建消费者
        final KafkaConsumer<Integer, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList(TOPIC));
        
        try {
            // 使用while循环模拟推消息模式，只要主题中分区有消息就消费
            while (ISRUNNING.get()) {
                // 批量从主题的分区拉取消息，若主题中没有可以消费的消息，每过3秒重新拉取一次
                final ConsumerRecords<Integer, String> records = consumer.poll(3_000);
                for (ConsumerRecord<Integer, String> record : records) {
                    System.out.println("========================================");
                    System.out.println("消息头字段：" + Arrays.toString(record.headers().toArray()));
                    System.out.println("消息的key：" + record.key());
                    System.out.println("消息的偏移量：" + record.offset());
                    System.out.println("消息的分区号：" + record.partition());
                    System.out.println("消息的序列化key字节数：" + record.serializedKeySize());
                    System.out.println("消息的序列化value字节数：" + record.serializedValueSize());
                    System.out.println("消息的时间戳：" + record.timestamp());
                    System.out.println("消息的时间戳类型：" + record.timestampType());
                    System.out.println("消息的主题：" + record.topic());
                    System.out.println("消息的值：" + record.value());
                }
                
            }
        } catch (Exception exception) {
            logger.error("occur exception", exception);
        } finally {
            consumer.close();
        }
        
    }
}
```

#### 2.1 参数配置及创建消费者

在Kafka消费者客户端KafkaConsumer中有4个参数是必填的：

- `bootstrap.servers`：**指定连接Kafka集群所需的broker地址清单**。可以设置一个或多个地址，中间用逗号隔开，此参数的默认值为“”。和KafkaProducer一样，不需要设置Kafka集群中全部的broker地址，消费者会从现有的配置中查找到全部的Kafka集群成员。
- `group.id`：**消费者隶属的消费组的名称**，默认值为“”。一般设置成具有一定的业务意义的名称。
- `key.deserializer` 和 `value.deserializer`：与生产者客户端 KafkaProducer中的key.serializer和value.serializer参数对应。消费者从broker端获取的消息格式都是字节数组（byte[]）类型，所以需要执行相应的反序列化操作才能还原成原有的对象格式。注意必须是**全限定名称**。
- `client.id`：设定**KafkaConsumer对应的客户端id**，默认值也为“”。如不设置，KafkaConsumer会自动生成**一个“consumer-”与数字**拼接的字符串。

我们可以直接使用客户端中的 `org.apache.kafka.clients.consumer.ConsumerConfig` 来获取配置参数，可以使用 `类名.class.getName()` 方式来获取反序列器的全限定名称。

#### 2.2 订阅主题与分区

一个消费者可以订阅一个或多个主题。subscribe的几个重载方法如下：

```java
// 以集合的形式订阅多个主题，listener用来设置相应的再均衡监听器的
public void subscribe(Collection<String> topics,ConsumerRebalanceListener listener) {}
public void subscribe(Collection<String> topics) {}
// 以正则表达式的形式订阅特定模式的主题
public void subscribe(Pattern pattern, ConsumerRebalanceListener listener) {}
public void subscribe(Pattern pattern) {}
```

对于消费者使用集合的方式（subscribe（Collection））来订阅主题，订阅了什么主题就消费什么主题中的消息。**如果前后两次订阅了不同的主题，那么消费者以最后一次的为准**。

如果消费者采用的是正则表达式的方式（subscribe（Pattern））订阅，如果有人又创建了新的主题，并且主题的名字与正则表达式相匹配，那么这个消费者就可以消费到新添加的主题中的消息。

```java
consumer.subscribe(Pattern.compile("topic-.*"));
```

消费者可以直接订阅某些主题的特定分区，使用 `assign()` 方法来实现。

```java
// partitions指定需要订阅的分区集合
public void assign(Collection<TopicPartition> partitions) {}

// 在Kafka的客户端中，用来表示分区。可以和我们通常所说的主题—分区的概念映射起来
public final class TopicPartition implements Serializable {
    private int hash = 0;
    // 自身的分区编号
    private final int partition;
    // 分区所属的主题
    private final String topic;

    public TopicPartition(String topic, int partition) {
        this.partition = partition;
        this.topic = topic;
    }
    // ......
}    
```

使用 `assign()` 方法

```java
consumer.assign(Arrays.asList(new TopicPartition("topic_1", 0)));
```

当不知道主题中有多少个分区时，KafkaConsumer 中的 `partitionsFor（）`方法可以用来查询指定主题的元数据信息。

```java
// 查询指定主题的元数据信息
public List<PartitionInfo> partitionsFor(String topic) { }

// 主题的分区元数据信息
public class PartitionInfo {
    private final String topic;  // 主题名称
    private final int partition;  // 分区编号
    private final Node leader;  // 分区的leader副本所在的位置
    private final Node[] replicas;  // 分区的AR集合
    private final Node[] inSyncReplicas;  // 分区的ISR集合
    private final Node[] offlineReplicas;  // 分区的OSR集合
	// ......
}    
```

我们还可以使用 KafkaConsumer 中的 `unsubscribe（）` 方法来取消所有方式生成的主题订阅。如果将`subscribe（Collection）`或 `assign（Collection）` 中的集合参数设置为空集合，那么作用等同于`unsubscribe（）`方法。

```java
consumer.unsubscribe();
consumer.subscribe(new ArrayList<String>());
consumer.assign(new ArrayList<TopicPartition>());
```

如果没有订阅任何主题或分区，那么再继续执行消费程序的时候会报出IllegalStateException异常。

**集合**订阅的方式subscribe（Collection）、**正则表达式**订阅的方式subscribe（Pattern）和**指定分区**的订阅方式assign（Collection）分表代表了三种不同的订阅状态：**AUTO_TOPICS**、**AUTO_PATTERN**和**USER_ASSIGNED**（如果没有订阅，那么订阅状态为**NONE**）。然而**这三种状态是互斥的，在一个消费者中只能使用其中的一种，否则会报出IllegalStateException异常**。

- 通过 subscribe（）方法订阅主题具有消费者**自动再均衡**的功能，在多个消费者的情况下可以根据**分区分配策略来自动分配各个消费者与分区的关系**。当消费组内的消费者增加或减少时，分区分配关系会自动调整，以实现消费负载均衡及故障自动转移。
- 通过assign（）方法订阅分区时，是不具备消费者自动均衡的功能的。

从两个方法的参数可以看出，两种类型的subscribe（）都有 `ConsumerRebalanceListener` 类型参数的方法，而assign（）方法却没有。

#### 2.3 反序列化

Kafka所提供的反序列化器可用于ByteBuffer、ByteArray、Bytes、Double、Float、Integer、Long、Short 及String类型，这些序列化器也都实现了 `Deserializer` 接口，与KafkaProducer中提及的Serializer接口一样，Deserializer接口也有三个方法。

```java
public interface Deserializer<T> extends Closeable {
    // 配置当前类
    void configure(Map<String, ?> configs, boolean isKey);
	// 用来执行反序列化。如果data为null，那么处理的时候直接返回null而不是抛出一个异常
    T deserialize(String topic, byte[] data);
	// 关闭当前序列化器
    void close();
}
```

在实际应用中，在Kafka提供的序列化器和反序列化器满足不了应用需求的前提下，推荐使用Avro、JSON、Thrift、ProtoBuf或Protostuff等通用的序列化工具来包装，以求尽可能实现得更加通用且前后兼容。**使用通用的序列化工具也需要实现 Serializer 和 Deserializer 接口，因为Kafka客户端的序列化和反序列化入口必须是这两个类型**。下面看如何使用通用的序列化工具实现自定义的序列化器和反序列化器的封装。

Protostuff的Maven依赖：

```xml
<dependency>
    <groupId>io.protostuff</groupId>
    <artifactId>protostuff-core</artifactId>
    <version>1.5.4</version>
</dependency>
<dependency>
    <groupId>io.protostuff</groupId>
    <artifactId>protostuff-runtime</artifactId>
    <version>1.5.4</version>
</dependency>
```

序列化器的serialize（）方法和deserialize（）方法：

```java
import io.protostuff.*;
/**
 * @author zlg
 * @create 自定义序列化器和反序列化器中的相关方法
 */
// 序列化
public byte[] serialize(String topic, Object data) {
    if (data == null) {
        return null;
    }
    Schema schema = (Schema) RuntimeSchema.getSchema(data.getClass());
    LinkedBuffer buffer = LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE);
    byte[] protostuff = null;
    try {
        protostuff = ProtostuffIOUtil.toByteArray(data, schema, buffer);
    } catch (Exception exception) {
        throw new IllegalStateException(exception.getMessage(), exception);
    } finally {
        buffer.clear();
    }
    return protostuff;
}

// 反序列化
public Object deserialize(String topic, byte[] data) {
    if (null == data) {
        return null;
    }
    final Schema<Object> schema = RuntimeSchema.getSchema(Object.class);
    final Object o = new Object();
    ProtostuffIOUtil.mergeFrom(data, o, schema);
    return o;
}
```

上午中根据生产者中自定的序列化器，接下来在消费者中自定义反序列化器如下：

```java
/**
 * @author zlg
 * @create 自定义反序列化器
 */
public class UserDeserializer implements Deserializer<User> {
    
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {}
    
    @Override
    public User deserialize(String topic, byte[] data) {
        ByteBuffer buffer = ByteBuffer.allocate(data.length);
        
        buffer.put(data);
        buffer.flip();
        
        final int userId = buffer.getInt();
        final int usernameLength = buffer.getInt();
        
        String username = new String(data, 8, usernameLength);
        return new User(userId, username);
    }
    
    @Override
    public void close() {}
}
```

消费者中进行反序列化器配置：

```java
import java.util.function.Consumer;
/**
 * @author zlg
 * @create 消费者
 */
public class UserConsumer {
    
    public static void main(String[] args) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "atzlg8:9092");
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // 设置自定义的反序列化器
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, UserDeserializer.class);
        
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, "user_consumer");
        configs.put(ConsumerConfig.CLIENT_ID_CONFIG, "consumer_id");
        configs.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        
        KafkaConsumer<String, User> consumer = new KafkaConsumer<String, User>(configs);
        
        // 订阅主题
        consumer.subscribe(Collections.singleton("tp_user_01"));

        final ConsumerRecords<String, User> records = consumer.poll(Long.MAX_VALUE);
        
        records.forEach(new Consumer<ConsumerRecord<String, User>>() {
            @Override
            public void accept(ConsumerRecord<String, User> record) {
                System.out.println(record.value());
            }
        });
        
        // 关闭消费者
        consumer.close();
    }
}
```

#### 2.4 消息消费

消息的消费一般有两种模式：推模式和拉模式。推模式是服务端主动将消息推送给消费者，而拉模式是消费者主动向服务端发起请求来拉取消息。**Kafka中的消费是基于拉模式的**。

```java
// 返回的是所订阅的主题（分区）上的一组消息，若没有可供消费的消息，则返回的结果为空或空的消息集合
// timeout用来控制poll（）方法的阻塞时间，在消费者的缓冲区里没有可用数据时会发生阻塞
// timeout的设置取决于应用程序对响应速度的要求
// public ConsumerRecords<K, V> poll(long timeout) {}
ConsumerRecords<Integer, String> records = consumer.poll(3_000);

// Duration：JDK8中新增的一个与时间有关的类型，Kafka2.0.0版本之后中改用的方法，上面方法已废弃
public ConsumerRecords<K, V> poll(final Duration timeout) {}
```

消费者消费到的每条消息的类型为 `ConsumerRecord`（注意与ConsumerRecords的区别），这个和生产者发送的消息类型ProducerRecord相对应。

```java
public class ConsumerRecord<K, V> {
    public static final long NO_TIMESTAMP = -1L;
    public static final int NULL_SIZE = -1;
    public static final int NULL_CHECKSUM = -1;
    private final String topic;  //主题名称
    private final int partition;  //分区编号
    private final long offset;  //消息所在分区的偏移量
    private final long timestamp;  //时间戳
    //有两种类型：CreateTime 和LogAppendTime，分别代表消息创建的时间戳和消息追加到日志的时间戳
    private final TimestampType timestampType;  
    private final int serializedKeySize;  //key经过序列化之后的大小，若key为null，则值为-1
    private final int serializedValueSize;  //value经过序列化之后的大小，若value为空，一样
    private final Headers headers;  //消息的头部内容
    private final K key;  //消息的键
    private final V value;  //消息的值
    private volatile Long checksum;  //CRC32的校验值
	// ......
}    
```









### 











