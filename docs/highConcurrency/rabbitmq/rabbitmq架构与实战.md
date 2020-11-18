## 一.RabbitMQ介绍、概念、基本架构

### 1.RabbitMQ介绍

RabbitMQ（兔子MQ），非常热门的一款开源消息中间件，互联网和传统行业都有所应用（最早为了解决电信行业系统之间的可靠通信设计）。

特点：高可用、易扩展，遵循AMQP协议，采用Erlang开发，RabbitMQ也支持MQTT等其它协议。具有强大的插件功能，官网：[https://www.rabbitmq.com/community-plugins.html](https://www.rabbitmq.com/community-plugins.html)

### 2.RabbitMQ整体逻辑架构

Producer发送消息给RabbitMQ的Exchange交换机，Consumer从RabbitMQ中的Queue里主动拉或监听推消息。RabbitMQ的一个Broker实例，有多个vhost虚拟主机，一个vhost包含多个Exchange和Queue。

![image-20201117231911398](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201117231911398.png)

### 3.RabbitMQ Exchange类型

RabbitMQ常用的交换器类型有：fanout、direct、topic、headers四种。

- **fanout**（广播）：把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中；
- **direct**（直接）：把消息路由到那些BindingKey和RoutingKey完全匹配的队列中；
- **topic**（主题）：将消息路由到BindingKey和RoutingKey相匹配的队列中；相比direct支持“*”（一个单词）和“#”（多个单词）模糊匹配；
- **headers**（headers键值对）：根据消息的headers属性中的键值对和绑定时指定的键值对是否完全匹配来路由消息，不依赖路由键的匹配规则。

### 4.RabbitMQ数据存储

RabbitMQ消息有持久化消息和非持久化消息，这两种消息都会写入磁盘。

- 持久化消息在到达队列时就会落盘，同时会在内存中备份一份，当内存不足是，会从内存中进行清楚；
- 非持久化消息一般只存在内存中，当内存压力大时数据会刷盘处理，以节省内存空间。

RabbitMQ存储层包含**队列索引**和**消息存储**两个部分。

![image-20201118214345604](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201118214345604.png)

下面为队列索引和消息的存储路径：

```sh
[root@atzlg8 ~]# cd /var/lib/rabbitmq/mnesia/rabbit@atzlg8/msg_stores/vhosts/628WB79CIFDYO9LJI6DKMI09L/
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# ls
msg_store_persistent  msg_store_transient  queues  recovery.dets
```

#### 4.1 队列索引(rabbit_queue_index)

索引维护队列的落盘消息的信息，如存储地点、是否已被给消费者接收、是否已被消费者ack等。每个队列都有相对应的索引。

```sh
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# ls queues/NJTLNCM3SHLLBXINIFLBRNC7/
0.idx  journal.jif
```

索引使用顺序的**段文件**来存储，后缀为.idx，文件名从0开始累加，每个段文件中包含固定的
`segment_entry_count` 条记录，默认值是**16384**。每个index从磁盘中读取消息的时候，**至少要在内存**
**中维护一个段文件**，所以设置 `queue_index_embed_msgs_below` 值得时候要格外谨慎，一点点增大也
可能会引起内存爆炸式增长。

#### 4.2 消息存储(rabbit_msg_store)

消息以**键值对**的形式存储到文件中，一个虚拟主机上的**所有队列使用同一块存储**，每个节点只有一个。存储分为持久化存储（msg_store_persistent）和短暂存储（msg_store_transient）。持久化存储的内容在broker重启后不会丢失，短暂存储的内容在broker重启后丢失。

```sh
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# ls
msg_store_persistent  msg_store_transient  queues  recovery.dets
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# ls msg_store_persistent/
0.rdq
```

store使用文件来存储，后缀为.rdq，经过**store处理的所有消息都会以追加的方式写入**到该文件中，当该文件的大小超过指定的限制（file_size_limit）后，将会关闭该文件并创建一个新的文件以供新的消息写入。文件名从0开始进行累加。在**进行消息的存储时**，RabbitMQ会在**ETS**（Erlang Term Storage）**表中记录消息在文件中的位置映射和文件的相关信息**。

消息（包括消息头、消息体、属性）可以直接存储在index中，也可以存储在store中。最佳的方式是较小的消息存在index中，而较大的消息存在store中。这个消息大小的界定可以通过`queue_index_embed_msgs_below` 来配置，默认值为**4096B**。

rabbitmq.conf中配置消息：

```properties
## Size in bytes below which to embed messages in the queue index.
## Related doc guide: https://rabbitmq.com/persistence-conf.html
##
queue_index_embed_msgs_below = 4096
## You can also set this size in memory units
##
# queue_index_embed_msgs_below = 4kb
```

如果消息小于这个值，就在索引中存储，如果消息大于这个值就在store中存储：

- 大于这个值的消息存储于msg_store_persistent目录中的`<num>.rdq`文件中；
- 小于这个值的消息存储于`<num>.idx`索引文件中。

**读取消息**：先根据**消息的ID**（msg_id）找到对应存储的文件，如果文件存在并且未被锁住，则直接打开文件，从指定位置读取消息内容。如果**文件不存在或者被锁住了，则发送请求由store进行处理**。

**删除消息**：只是**从ETS表删除指定消息的相关信息，同时更新消息对应的存储文件和相关信息**。并不立即对文件中的消息进行删除，仅仅是**标记为垃圾数据**而已。当一个文件中都是垃圾数据时可以将这个文件删除。当检测到前后两个文件中的有效数据可以合并成一个文件，并且所有的垃圾数据的大小和所有文件（至少有3个文件存在的情况下）的数据大小的比值超过设置的阈值`garbage_fraction`（默认值0.5）时，才会触发垃圾回收，将这两个文件合并，执行合并的两个文件一定是**逻辑上相邻的两个文件**。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201118215350930.png" alt="image-20201118215350930" style="zoom:50%;" />

合并逻辑：

1. 锁定这两个文件；
2. 先整理前面的文件的有效数据，再整理后面的文件的有效数据；
3. 将后面文件的有效数据写入到前面的文件中；
4. 更新消息在ETS表中的记录；
5. 删除后面文件。

#### 4.3 队列结构

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201118215827024.png" alt="image-20201118215827024" style="zoom:50%;" />

通常队列由`rabbit_amqqueue_process`和`rabbit_backing_queue`这两部分组成。

- rabbit_amqqueue_process**负责协议相关的消息处理**，即接收生产者发布的消息、向消费者交付消息、处理消息的确认（包括生产端的confirm和消费端的ack）等；
- rabbit_backing_queue是**消息存储的具体形式和引擎**，并向rabbit_amqqueue_process提供相关的接口以供调用。

若消息投递的目的队列是空的，无需排队等待时，并且有消费者订阅了这个队列，那么该消息会直接发送给消费
者，不会经过队列这一步。当消息无法直接投递给消费者时，需要暂时将消息存入队列，以便重新投递。**消息存入队列后，不是固定不变的，它会随着系统的负载在队列中不断流动，消息的状态会不断发送变化**。

`rabbit_variable_queue.erl` 源码中定义了RabbitMQ队列的4种状态：

- alpha：消息索引和消息内容都存内存，最耗内存，很少消耗CPU；
- beta：消息索引存内存，消息内容存磁盘；
- gama：消息索引内存和磁盘都有，消息内容存磁盘；
- delta：消息索引和内容都存磁盘，基本不消耗内存，消耗更多CPU和I/O操作。

在运行时，RabbitMQ会根据**消息传递的速度**定期计算一个**当前内存中能够保存的最大消息数量**
（target_ram_count），如果**alpha状态的消息数量**大于此值，则会引起消息的状态转换，多余的消息可能会转换到beta、gama或者delta状态。区分这4种状态的主要作用是满足不同的内存和CPU需求。

对于**普通没有设置优先级和镜像的队列**来说，`rabbit_backing_queue`的默认实现是`rabbit_variable_queue`，其内部通过5个子队列Q1、Q2、delta、Q3、Q4来体现消息的各个状态。

![image-20201118224156364](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201118224156364.png)

消费者获取消息也会引起消息的状态转换。当消费者获取消息时：

1. 首先会从Q4中获取消息，如果获取成功则返回。
2. 如果Q4为空，则尝试从Q3中获取消息，系统首先会判断Q3是否为空，如果为空则返回队列为空，即此时队列中无消息。
3. 如果Q3不为空，则取出Q3中的消息；进而再判断此时Q3和Delta中的长度，如果都为空，则可以认为 Q2、Delta、 Q3、Q4 全部为空，此时将Q1中的消息直接转移至Q4，下次直接从Q4 中获取消息。
4. 如果Q3为空，Delta不为空，则将Delta的消息转移至Q3中，下次可以直接从Q3中获取消息。在将消息从Delta转移到Q3的过程中，是按照索引分段读取的，首先读取某一段，然后判断读取的消息的个数与Delta中消息的个数是否相等，如果相等，则可以判定此时Delta中己无消息，则直接将Q2和刚读取到的消息一并放入到Q3中，如果不相等，仅将此次读取到的消息转移到Q3。

为什么Q3为空则可以认定整个队列为空？

- 如果Q3为空，Delta不为空，那么在Q3取出最后一条消息的时候，Delta 上的消息就会被转移到Q3这样与 Q3 为空矛盾；
- 如果Delta 为空且Q2不为空，则在Q3取出最后一条消息时会将Q2的消息并入到Q3中，这样也与Q3为空矛盾；
- 在Q3取出最后一条消息之后，如果Q2、Delta、Q3都为空，且Q1不为空时，则Q1的消息会被转移到Q4，这与Q4为空矛盾。

通常在负载正常时，如果消费速度大于生产速度，对于不需要保证可靠不丢失的消息来说，极有可能只会处于alpha状态。

对于持久化消息，它一定会进入gamma状态，在开启publisher confirm机制时，只有到了gamma 状态时才会确认该消息己被接收，若消息消费速度足够快、内存也充足，这些消息也不会继续走到下一个状态。

**为什么消息的堆积导致性能下降**？在系统负载较高时，消息若不能很快被消费掉，这些消息就会进入到很深的队列中去，这样会增加处理每个消息的平均开销。因为要花更多的时间和资源处理“堆积”的消息，如此用来处理新流入的消息的能力就会降低，使得**后流入的消息又被积压到很深的队列中，继续增大处理每个消息的平均开销**，继而情况变得越来越恶化，使得系统的处理能力大大降低。应对这一问题一般有3种措施：

- 增加prefetch_count的值，即一次发送多条消息给消费者，加快消息被消费的速度。
- 采用multiple ack，降低处理 ack 带来的开销。
- 流量控制。

## 二.RabbitMQ安装和配置

RabbitMQ的安装需要首先安装Erlang，因为它是基于Erlang的VM运行的。RabbitMQ需要的依赖：socat和logrotate，logrotate操作系统中已经存在了，只需要安装socat就可以了。RabbitMQ与Erlang的兼容关系详见：https://www.rabbitmq.com/which-erlang.html 。

1.安装依赖：yum install socat -y

2.安装Erlang

```shell
# 在线下载erlang-23.0.2-1.el7.x86_64.rpm
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v23.0.2/erlang-23.0.2-1.el7.x86_64.rpm
# 使用rpm进行安装
rpm -ivh erlang-23.0.2-1.el7.x86_64.rpm
```

3.安装RabbitMQ

```shell
# 在线下载rabbitmq-server-3.8.4-1.el7.noarch.rpm
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.5/rabbitmq-server-3.8.5-1.el7.noarch.rpm
# 使用rpm进行安装
rpm -ivh rabbitmq-server-3.8.5-1.el7.noarch.rpm
```

4.启动RabbitMQ的管理插件：rabbitmq-plugins enable rabbitmq_management

```shell
[root@atzlg8 applets]# rabbitmq-plugins enable rabbitmq_management
Enabling plugins on node rabbit@atzlg8:
rabbitmq_management
The following plugins have been enabled:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch

set 3 plugins.
Offline change; changes will take effect at broker restart.
```

5.开启RabbitMQ

```shell
systemctl start rabbitmq-server
# 或者
rabbitmq-server
# 后台启动
rabbitmq-server -detached
```

6.RabbitMQ后台启动后，添加用户、添加权限、设置标签

```shell
# 添加用户
rabbitmqctl add_user root 123456
# 给用户添加权限：给root用户在虚拟主机"/"上的配置、写、读的权限
rabbitmqctl set_permissions root -p / ".*" ".*" ".*"
# 给用户设置标签：management、policymaker、monitoring、administrator
rabbitmqctl set_user_tags root administrator
```

用户标签和权限：

![image-20201118204345302](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201118204345302.png)

7.打开浏览器，访问：http://192.168.91.112:15672 ，即 http://IP:15672

8.使用刚才创建的用户登陆。

9.补充：重新设置rabbitmq的脚本(resetrabbit)

```shell
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
rabbitmqctl add_user root 123456
rabbitmqctl set_user_tags root administrator
rabbitmqctl set_permissions root -p / ".*" ".*" ".*"
```

## 三.RabbitMQ常用命令

```shell
# 前台启动Erlang VM和RabbitMQ
rabbitmq-server
# 后台启动
rabbitmq-server -detached
# 停止RabbitMQ和Erlang VM
rabbitmqctl stop

# 查看所有队列
rabbitmqctl list_queues --formatter pretty_table
# 查看所有交换器
rabbitmqctl list_exchanges --formatter pretty_table
# 查看绑定键信息
rabbitmqctl list_bindings --formatter pretty_table
# 查看节点状态
rabbitmqctl status

# 查看所有可用的插件
rabbitmq-plugins list
# 查看启动的插件
rabbitmq-plugins list --enabled
# 启用插件
rabbitmq-plugins enable <plugin-name>
# 停用插件
rabbitmq-plugins disable <plugin-name>

# 列出所有用户：
rabbitmqctl list_users
# 添加用户
rabbitmqctl add_user username password
# 删除用户：
rabbitmqctl delete_user username
# 修改用户密码：
rabbitmqctl change_password username newpassword

# 列出用户权限：
rabbitmqctl list_user_permissions username
# 设置用户权限：
rabbitmqctl set_permissions -p vhostpath username ".*" ".*" ".*"
# 清除用户权限：
rabbitmqctl clear_permissions -p vhostpath username

# 列出所以虚拟主机:
rabbitmqctl list_vhosts
# 列出虚拟主机上的所有权限:
rabbitmqctl list_permissions -p vhostpath
# 创建虚拟主机:
rabbitmqctl add_vhost vhostpath
# 删除虚拟主机:
rabbitmqctl delete_vhost vhost vhostpath

# 在Erlang VM运行的情况下启动RabbitMQ应用
rabbitmqctl start_app
rabbitmqctl stop_app
# 移除所有数据，要在 rabbitmqctl stop_app 之后使用:
rabbitmqctl reset
```

## 四.RabbitMQ工作流程

### 1.工作流程解析

**生产者发送消息的流程**：

1. 生产者连接RabbitMQ Broker，配置ConnectionFactory，建立TCP连接Connection，开启信道Channel；
2. 生产者声明一个队列Queue，并设置相关属性：queueName、durable、exclusive、autoDelete、arguments；
3. 生产者声明一个交换器Exchange，并设置相关属性：exchangeName、durable、autoDelete、arguments；
4. 生产者将Queue绑定（binding）到到指定的Exchange上，并设置routingKey（路由Key）；
5. 生产者发送（basicPublish）消息给交换器，相应的交换器根据接收到的 routingKey 查找相匹配的队列。如果找到，则将消息存入相应的队列中；如果没有找到，则根据生产者配置的属性选择丢弃还是回退给生产者；
6. 关闭信道Channel和连接Connection。

**消费者接收消息的过程**：

1. 消费者连接RabbitMQ Broker，配置ConnectionFactory，建立TCP连接Connection，开启信道Channel；
2. 消费者向RabbitMQ Broker 请求消费（basicConsume）相应队列Queue中的消息，可能会设置相应的回调函数（DeliverCallback、CancelCallback）， 以及做一些准备工作；
3. 等待RabbitMQ Broker 回应并投递相应队列中的消息， 消费者接收消息。消费者确认( ack) 接收到的消息；
4. RabbitMQ 从队列中删除相应己经被确认的消息；
5. 关闭信道Channel和连接Connection；

### 2.Hello World示例

生产者直接发送消息给RabbitMQ，另一端消费。未定义和指定Exchange的情况下，使用的是AMQP default这个内置的Exchange。

生产者：

```java
/**
 * @author zlg
 * Rabbitmq是一个消息broker：接收消息，传递给下游应用
 * Producing就是指发送消息，发送消息的程序是Producer
 * Queue指的是RabbitMQ内部的一个组件，消息存储于queue中
 * Consuming就是接收消息。一个等待消费消息的应用程序称为Consumer
 */
public class HelloProducer {

    public static void main(String[] args) {

        // 创建连接工厂并初始化
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("atzlg8");  // 设置服务器主机名或IP地址
        factory.setVirtualHost("/");  // 设置Erlang的虚拟主机名称
        factory.setUsername("root");
        factory.setPassword("123456");
        factory.setPort(5672);  // 设置客户端与服务器的通信端口，默认值为5672

        // try-wtih-resource释放资源
        // 建立TCP连接,获取通道
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 声明消息队列：消息队列名称、是否持久化、是否排他性、是否自动删除、队列属性信息
            channel.queueDeclare("queue.biz", false, false, true, null);
            // 声明交换器：交换器名称、类型、是否持久化、是否自动删除、属性map集合
            channel.exchangeDeclare("ex.biz", BuiltinExchangeType.DIRECT, false, false, null);
            // 将队列和交换器进行绑定，并指定路由键
            channel.queueBind("queue.biz", "ex.biz", "hello.world");
            // 发送消息：交换器名称、路由键、消息的属性MessageProperties对象、消息的字节数组
            channel.basicPublish("ex.biz", "hello.world", null, "hello world demo".getBytes());

        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
```

消费者主动拉模式：

```java
public class HelloGetConsumer {

    public static void main(String[] args) throws Exception {

        ConnectionFactory factory = new ConnectionFactory();
        // 制定协议、用户名、密码、host、port、虚拟主机
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        // 指定从哪个队列消费消息：队列名称、死否自动确认消息
        GetResponse getResponse = channel.basicGet("queue.biz", true);
        System.out.println(new String(getResponse.getBody()));

        // 释放资源
        channel.close();
        connection.close();

    }
}
```

消费者监听回调函数模式：

```java
public class HelloPushConsumer {

    public static void main(String[] args) throws Exception {

        ConnectionFactory factory = new ConnectionFactory();
        // 制定协议、用户名、密码、host、port、虚拟主机
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        // 确保MQ中有该队列，若没有就创建
//        channel.queueDeclare("queue.biz", false, false, true, null);

        /**
         * 监听消息，一旦有消息推送过来，就调用第一个lambda表达式
         * 参数列表：队列名称、是否自动确认消息、deliverCallback、cancelCallback
         * autoAck – true 只要服务器发送了消息就表示消息已经被消费者确认; false服务端等待客户端显式地发送确认消息
         * deliverCallback – 服务端推送过来的消息回调函数
         * cancelCallback – 客户端忽略该消息的回调方法
         * Returns:服务端生成的consumerTag
         */
        channel.basicConsume("queue.biz", true, (consumerTag, message) -> {
            System.out.println(new String(message.getBody()));
        }, consumerTag -> {});

//        channel.close();
//        connection.close();

    }
}
```

### 3.Connection和Channel关系

生产者和消费者，需要与RabbitMQ Broker 建立TCP连接Connection 。一旦TCP 连接建立起来，客户端紧接着创建一个AMQP 信道（Channel），每个信道都会被指派一个唯一的ID。信道是建立在Connection 之上的虚拟连接， RabbitMQ 处理的每条AMQP 指令都是通过信道完成的。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201118234457699.png" alt="image-20201118234457699" style="zoom:50%;" />

RabbitMQ 采用类似NIO的做法，复用TCP 连接，减少性能开销，便于管理。

- 当每个信道的流量不是很大时，复用单一的Connection 可以在产生性能瓶颈的情况下有效地节省 TCP 连接资源。
- 当信道本身的流量很大时，一个Connection 就会产生性能瓶颈，流量被限制。需要建立多个Connection ，分摊信道。具体的调优看业务需要。

信道在AMQP 中是一个很重要的概念，大多数操作都是在信道这个层面进行的。

```java
channel.exchangeDeclare
channel.queueDeclare
channel.queueBind
channel.basicPublish
channel.basicConsume
channel.basicGet    
// ...    
```

RabbitMQ 相关的API与AMQP紧密相连，比如channel.basicPublish 对应AMQP 的Basic.Publish命令。

## 五.RabbitMQ工作模式

官网地址：[https://www.rabbitmq.com/getstarted.htm](https://www.rabbitmq.com/getstarted.htm)

消息的推拉：实现RabbitMQ的消费者有两种模式，推模式（Push）和拉模式（Pull）。 实现推模式推荐的方式
是**继承 DefaultConsumer 基类**，也可以使用Spring AMQP的 **SimpleMessageListenerContainer** 。 推模式是最常用的，但是有些情况下推模式并不适用的，比如说： 由于某些限制，消费者在某个条件成立时才能消费消息，需要批量拉取消息进行处理，实现拉模式 RabbitMQ的Channel提供了 **basicGet 方法**用于拉取消息。

### 1.Work Queue

生产者发消息，启动**多个消费者实例来消费消息**，每个消费者仅消费部分信息，可达到**负载均衡**的效果。（多个消费者监听同一个消息队列）

生产者：

```java
/**
 * @author zlg
 * 生产者生产的消息会轮询依次发送到多个消费者
 */
public class Producer {

    public static void main(String[] args) throws Exception{

        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        /**
         * 声明消息队列：消息队列名称、是否持久化、是否排他性、是否自动删除、队列属性信息
         * 声明交换器：交换器名称、类型、是否持久化、是否自动删除、属性map集合
         * 将队列和交换器进行绑定，并指定路由键
         * 发送消息：交换器名称、路由键、消息的属性MessageProperties对象、消息的字节数组
         */
        channel.queueDeclare("queue.wq", true, false, false, null);
        channel.exchangeDeclare("ex.wq", BuiltinExchangeType.DIRECT, true, false, null);
        channel.queueBind("queue.wq", "ex.wq", "key.wq");

        for (int i = 0; i < 15; i++) {
            channel.basicPublish("ex.wq", "key.wq", null, ("工作队列" + i).getBytes("utf-8"));
        }
        
        channel.close();
        connection.close();

    }
}
```

消费者：

```java
public class Consumer {

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare("queue.wq", true, false, false, null);

        channel.basicConsume("queue.wq", (consumerTag, message) -> {
            System.out.println(new String(message.getBody(), "utf-8"));
        }, consumerTag -> {
            System.out.println("cancel："+consumerTag);
        });

    }
}
```

模拟时，需要在IDEA的configurations里进行如下配置，并行启动多个消费者进程。

![image-20201118235906776](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201118235906776.png)

### 2.发布订阅模式

**使用fanout类型交换器，routingKey忽略。每个消费者定义生成一个队列并绑定到同一个Exchange，每个消费者都可以消费到完整的消息**。消息广播给所有订阅该消息的消费者。

生产者将消息发送给交换器。交换器非常简单，从生产者接收消息，将消息推送给消息队列。交换器必须清楚地知道要怎么处理接收到的消息。应该是追加到一个指定的队列，还是追加到多个队列，还是丢弃。规则就是**交换器类型**。

可以使用下列命令查看所有交换器，列出RabbitMQ的交换器，包括了 amq.* 的和默认的（未命名）的交换器。

```shell
rabbitmqctl list_exchanges
```

当没有指定交换器时，依然可以向队列发送消息，这是因为使用了默认的交换器。

创建临时队列：

- 首先，无论何时连接RabbitMQ的时候，都需要一个新的，空的队列。可以使用随机的名字创建队列，也可以让服务器帮我们生成随机的消息队列名字。
- 其次，一旦我们断开到消费者的连接，该队列应该自动删除。

```java
// queueName一般的格式类似： amq.gen-JzTY20BRgKO-HjmUJj0wLg
String queueName = channel.queueDeclare().getQueue();
```

上述声明了一个非持久化的、排他的、自动删除的队列，并且名字是服务器随机生成的。

发布订阅模式代码示例代码：

生产者：

```java
/**
 * @author zlg
 * 所有消息广播到所有消费者（复制）
 */
public class Producer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        // 声明fanout类型的交换器
        channel.exchangeDeclare("ex.fanout", BuiltinExchangeType.FANOUT, true, false ,null);
    
        for (int i = 0; i < 20; i++) {
            // fanout类型交换器不需要指定路由键
            channel.basicPublish("ex.fanout", "",
                    null, ("hello world fanout" + i).getBytes("utf-8"));
        }
        
        channel.close();
        connection.close();
        
    }
}
```

消费者One：

```java
public class OneConsumer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        // 声明临时队列，队列名称由RabbitMQ自动生成
        String queueName = channel.queueDeclare().getQueue();
        System.out.println("临时队列名称：" + queueName);
        channel.exchangeDeclare("ex.fanout", BuiltinExchangeType.FANOUT, true, false ,null);
        
        // fanout类型的交换器绑定时不需要routingkey
        channel.queueBind(queueName, "ex.fanout", "");
        
        channel.basicConsume(queueName, (consumerTag, message) -> {
            System.out.println("One    "+new String(message.getBody(), "utf-8"));
        }, consumerTag -> {});
    
    }
}
```

消费者Two：

```java
public class TwoConsumer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        // 声明临时队列，队列名称由RabbitMQ自动生成
        String queueName = channel.queueDeclare().getQueue();
        System.out.println("临时队列名称：" + queueName);
        channel.exchangeDeclare("ex.fanout", BuiltinExchangeType.FANOUT, true, false ,null);
        
        // fanout类型的交换器绑定时不需要routingkey
        channel.queueBind(queueName, "ex.fanout", "");
        
        channel.basicConsume(queueName, (consumerTag, message) -> {
            System.out.println("Two    "+new String(message.getBody(), "utf-8"));
        }, consumerTag -> {});
    
    }
}
```

### 3.路由模式

使用 direct 类型的Exchange，发N条消费并使用不同的 routingKey ，消费者定义队列并将队列、 routingKey 、Exchange绑定。此时使用 direct 模式Exchagne必须要 routingKey 完全匹配的情况下消息才会转发到对应的队列中被消费。

生产者：

```java
/**
 * @author zlg
 * 消息根据routingkey路由到指定的队列和消费者
 */
public class Producer {
    
    public static String[] LOG_LEVEL = {"FATAL", "ERROR", "WARN"};
    public static Random random = new Random();
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        // 声明交换器
        channel.exchangeDeclare("ex.routing", "direct", false, false , null);
        
        // 发送消息
        for (int i = 0; i < 100; i++) {
            String level = LOG_LEVEL[random.nextInt(3)];
            System.out.println(level);
            channel.basicPublish("ex.routing", level,
                    null, ("这是【 " + level + "】的消息").getBytes("utf-8"));
        }
        
        channel.close();
        connection.close();
        
    }
}
```

接受Warn日志级别的消费者：

```java
public class WarnConsumer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        channel.queueDeclare("queue.warn", false, false, false, null);
        channel.exchangeDeclare("ex.routing", "direct", false, false , null);
        channel.queueBind("queue.warn", "ex.routing", "WARN");
        
        channel.basicConsume("queue.warn", (consumerTag, message) -> {
            System.out.println("ErrorConsumer收到消息：" + new String(message.getBody(), "utf-8"));
        }, consumerTag -> {});
        
    }
}
```

接受Error日志级别的消费者：

```java
public class ErrorConsumer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        channel.queueDeclare("queue.error", false, false, false, null);
        channel.exchangeDeclare("ex.routing", "direct", false, false , null);
        channel.queueBind("queue.error", "ex.routing", "ERROR");
        
        channel.basicConsume("queue.error", (consumerTag, message) -> {
            System.out.println("ErrorConsumer收到消息：" + new String(message.getBody(), "utf-8"));
        }, consumerTag -> {});
        
    }
}
```

### 4.direct交换器（路由模式）

使用 direct 类型的交换器，只要消息的 routingKey 和队列的 bindingKey 对应，消息就可以推送给该队列。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201119003532757.png" alt="image-20201119003532757" style="zoom:50%;" />

上图中的交换器 X 是 direct 类型的交换器，绑定的两个队列中，一个队列的 bindingKey 是orange ，另一个队列的 bindingKey 是 black 和 green 。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201119003615769.png" alt="image-20201119003615769" style="zoom:50%;" />

使用 direct 类型的交换器 X ，建立了两个绑定：队列Q1根据 bindingKey 的值black 绑定到交换器 X ，队列Q2根据 bindingKey 的值 black 绑定到交换器 X ；交换器 X 会将消息发送给队列Q1和队列Q2。交换器的行为跟 fanout 的行为类似，也是广播。

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201119003816180.png" alt="image-20201119003816180" style="zoom:50%;" />

即我们可以将日志级别作为 routingKey。

生产者：

```java
public class Producer {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("node1");
        factory.setVirtualHost("/");
        factory.setUsername("root");
        factory.setPassword("123456");
        factory.setPort(5672);
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        String servrity = null;
		// 声明direct类型的交换器logs
        channel.exchangeDeclare("direct_logs", BuiltinExchangeType.DIRECT);
        
        for (int i = 0; i < 100; i++) {
            switch (i % 3) {
                case 0:
                    servrity = "info";
                    break;
                case 1:
                    servrity = "warn";
                    break;
                case 2:
                    servrity = "error";
                    break;
                default:
                    System.err.println("log错误，程序退出");
                    System.exit(-1);
            }
            String logStr = "这是 【" + servrity + "】 的消息";
            
            channel.basicPublish("direct_logs", servrity, null,
                    logStr.getBytes("UTF-8"));
        }
    }
}
```

消费者：

```java
public class ErrorConsumer {
    public static void main(String[] args) throws IOException, TimeoutException
    {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("node1");
        factory.setVirtualHost("/");
        factory.setUsername("root");
        factory.setPassword("123456");
        factory.setPort(5672);
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        channel.exchangeDeclare("direct_logs", BuiltinExchangeType.DIRECT);
        String queueName = channel.queueDeclare().getQueue();
		// 将logs交换器和queueName队列通过bindingKey：error绑定
        channel.queueBind(queueName, "direct_logs", "error");
        
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };
        channel.basicConsume(queueName, deliverCallback, consumerTag -> {});
    }
}
```

### 5.主题模式（路由模式）

使用 topic 类型的交换器，队列绑定到交换器、 bindingKey 时使用通配符，交换器将消息路由转发到具体队列时会根据消息 routingKey 模糊匹配，比较灵活。

- **routingKey**：必须是点分单词。单词可以随便写，生产中一般使用消息的特征。如：“stock.usd.nyse”，“nyse.vmw”，“quick.orange.rabbit”等。该点分单词字符串最长255字节。
- **bindingKey**：也必须是这种形式。只要队列的 bindingKey 的值与消息的 routingKey 匹配，队列就可以收到该消息。可以使用`*` (star)匹配一个单词或`#` 匹配0到多个单词。

两种 bindingKey 特殊情况：

- 如果在 topic 类型的交换器中 bindingKey 使用 `#` ，则就是 fanout 类型交换器的行为。
- 如果在 topic 类型的交换器中 bindingKey 中不使用 `*`和 `#` ，则就是 direct 类型交换器的行为。

**具体案例**

生产者（区域.业务.日志级别）：

```java
public class Producer {
    
    public static final String[] LOG_LEVEL = {"FATAL", "ERROR", "WARN"};
    public static final String[] LOG_AREA = {"shanghai", "beijing", "shenzhen"};
    public static final String[] LOG_BIZ = {"edu-online", "biz-online", "emp-online"};
    public static final Random RANDOM = new Random();
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
    
        // 声明交换器，这里不需要关系消费者和绑定关系
        channel.exchangeDeclare("ex.topic", "topic", true ,false , null);
        
        for (int i = 0; i < 100; i++) {
            
            String level = LOG_LEVEL[RANDOM.nextInt(LOG_LEVEL.length)];
            String area = LOG_AREA[RANDOM.nextInt(LOG_AREA.length)];
            String biz = LOG_BIZ[RANDOM.nextInt(LOG_BIZ.length)];
    
            String routingkey = area + "." + biz + "." + level;
            String message = "LOG : 这是level：【" + level + "】的，来自地区 【" + area + "】业务【" +
                    biz + "】的日志，LOG_SEQ = " + i;
            
            channel.basicPublish("ex.topic", routingkey,  null, message.getBytes("utf-8"));
        }
        
    }
}
```

只关注Error级别日志的消费者：

```java
public class ErrorConsumer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
    
        // 声明一个临时队列，名称由服务器生成
        String queue = channel.queueDeclare().getQueue();
        channel.exchangeDeclare("ex.topic", "topic", true ,false , null);
        channel.queueBind(queue, "ex.topic", "#.ERROR");
        
        // 监听消息
        channel.basicConsume(queue, (consumerTag, message) -> {
            System.out.println(new String(message.getBody(), "utf-8"));
        }, consumerTag -> {});
    }
}
```

关注ShangHai地区Warn级别日志的消费者：

```java
public class ShangHaiWarnConsumer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
    
        // 声明一个临时队列，名称由服务器生成
        String queue = channel.queueDeclare().getQueue();
        channel.exchangeDeclare("ex.topic", "topic", true ,false , null);
        channel.queueBind(queue, "ex.topic", "shanghai.*.WARN");
        
        // 监听消息
        channel.basicConsume(queue, (consumerTag, message) -> {
            System.out.println(new String(message.getBody(), "utf-8"));
        }, consumerTag -> {});
    }
}
```

## 六.Spring整合RabbitMQ

spring-amqp是对AMQP的一些概念的一些抽象，spring-rabbit是对RabbitMQ操作的封装实现。主要有几个核心类 RabbitAdmin 、 RabbitTemplate 、 SimpleMessageListenerContainer 等。

- **RabbitAdmin** 类完成对Exchange，Queue，Binding的操作，在容器中管理了 RabbitAdmin 类的时候，可以对Exchange，Queue，Binding进行自动声明。
- **RabbitTemplate** 类是发送和接收消息的工具类。
- **SimpleMessageListenerContainer** 是消费消息的容器。

目前一般选择使用注解方式来使用。

### 1.基于配置文件的整合

#### 1.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>2.2.7.RELEASE</version>
</dependency>
```

#### 1.2 配置文件

producer：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit.xsd"
>

    <!-- 创建连接工厂 -->
    <rabbit:connection-factory id="connectionFactory" username="root" password="123456"
                               host="atzlg8" port="5672" virtual-host="/" />

    <!-- 自动查找类型是Queue、Exchange、Binding的bean，并为用户向RabbitMQ声明 -->
    <rabbit:admin id="rabbitAdmin" connection-factory="connectionFactory" />

    <!-- 创建一个rabbit的template对象(org.springframework.amqp.rabbit.core.RabbitTemplate)，
          以便于访问broker- -->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory" />

    <!-- 为消费者创建一个队列，如果broker中存在，则使用同名存在的队列，否则创建一个新的 -->
    <rabbit:queue id="q1" name="queue.q1" durable="false" exclusive="false" auto-delete="false" />

    <!-- 声明一个交换器 -->
    <rabbit:direct-exchange id="directExchange" name="ex.direct" durable="false" auto-delete="false" >
        <rabbit:bindings>
            <!--exchange：其他绑定到该交换器的交换器名称-->
            <!--queue：绑定到该交换器的queue的bean名称-->
            <!--key：显式声明的路由key-->
            <rabbit:binding queue="q1" key="routing.q1" />
        </rabbit:bindings>
    </rabbit:direct-exchange>

</beans>
```

consumer：

```xml
<!-- 创建连接工厂 -->
    <rabbit:connection-factory id="connectionFactory" username="root" password="123456"
                               host="atzlg8" port="5672" virtual-host="/" />

    <!-- 自动查找类型是Queue、Exchange、Binding的bean，并为用户向RabbitMQ声明 -->
    <rabbit:admin id="rabbitAdmin" connection-factory="connectionFactory" />

    <!-- 创建一个rabbit的template对象(org.springframework.amqp.rabbit.core.RabbitTemplate)，
          以便于访问broker- -->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory" />

<!-- 为消费者创建一个队列，如果broker中存在，则使用同名存在的队列，否则创建一个新的 -->
	<rabbit:queue id="q1" name="queue.q1" durable="false" exclusive="false" auto-delete="false" />


<!-- 消费者需要配置的：push推消息模式,pull模式不需要配置下列监听 -->
    <rabbit:listener-container connection-factory="connectionFactory">
        <rabbit:listener ref="myListenerMessage" queues="q1" />
    </rabbit:listener-container>
    <!-- 自定义队列消息监听类 -->
    <bean id="myListenerMessage" class="com.fishleap.listener.MyListenerMessage" />
```

#### 1.3 应用类

producer：

```java
/**
 * @author zlg
 */
public class ProducerApp {
    
    public static void main(String[] args) throws Exception {
        AbstractApplicationContext context = new ClassPathXmlApplicationContext("spring-rabbit.xml");
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);
    
        MessagePropertiesBuilder propertiesBuilder = MessagePropertiesBuilder.newInstance();
        propertiesBuilder.setContentEncoding("utf-8");
        propertiesBuilder.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
    
        for (int i = 0; i < 20; i++) {
            Message message = MessageBuilder.withBody(("hello world "+i).getBytes("utf-8"))
                    .andProperties(propertiesBuilder.build()).build();
    
    
            rabbitTemplate.send("ex.direct", "routing.q1", message);
        }
    
        context.close();
    }
    
}
```

consumer：

```java
// pull消息模式
public class ConsumerPullApp {
    
    public static void main(String[] args) throws UnsupportedEncodingException {
        AbstractApplicationContext context = new ClassPathXmlApplicationContext("spring-rabbit.xml");
        RabbitTemplate template = context.getBean(RabbitTemplate.class);
    
        Message message = template.receive("queue.q1");
        System.out.println(new String(message.getBody(), message.getMessageProperties().getContentEncoding()));
        
        context.close();
    }
}

// push推消息模式
public class ConsumerPushApp {
    
    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("spring-rabbit.xml");
    }
}
```

push推消息模式所需要的监听类：

```java
// 也可以实现MessageListener类
public class MyMessageListener implements ChannelAwareMessageListener {
    
    @Override
    public void onMessage(final Message message, final Channel channel) throws Exception {
        System.out.println(new String(message.getBody(), message.getMessageProperties().getContentEncoding()));
    }
    
}
```

### 2.基于注解的整合

#### 2.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>2.2.7.RELEASE</version>
</dependency>
```

#### 2.2 配置类

producer:

```java
@Configuration
public class RabbitConfig {
    
    // ConnectionFactory
    @Bean
    public ConnectionFactory connectionFactory() {
        return new CachingConnectionFactory(URI.create("amqp://root:123456@atzlg8:5672/%2f"));
    }
    
    // RabbitTemplate
    @Bean
    @Autowired
    public RabbitTemplate rabbitTemplate(ConnectionFactory factory) {
        return new RabbitTemplate(factory);
    }
    
    // RabbitAdmin
    @Bean
    @Autowired
    public RabbitAdmin rabbitAdmin(ConnectionFactory factory) {
        return new RabbitAdmin(factory);
    }
    
    // Queue
    @Bean
    public Queue queue() {
        return QueueBuilder.nonDurable("queue.anno").build();
    }
    
    // Exchange
    @Bean
    public Exchange exchange() {
//        ExchangeBuilder.fanoutExchange("ex.anno.fanout").build();
        return new FanoutExchange("ex.anno.fanout", false ,false);
        
    }
    
    // Binding
    @Bean
    @Autowired
    public Binding binding(Queue queue, Exchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("key.fanout").noargs();
    }
    
}
```

consumer拉模式:

```java
@Configuration
public class RabbitConfig {
    
    // ConnectionFactory
    @Bean
    public ConnectionFactory connectionFactory() {
        return new CachingConnectionFactory(URI.create("amqp://root:123456@atzlg8:5672/%2f"));
    }
    
    // RabbitTemplate
    @Bean
    @Autowired
    public RabbitTemplate rabbitTemplate(ConnectionFactory factory) {
        return new RabbitTemplate(factory);
    }
    
    // RabbitAdmin
    @Bean
    @Autowired
    public RabbitAdmin rabbitAdmin(ConnectionFactory factory) {
        return new RabbitAdmin(factory);
    }
    
    // Queue
    @Bean
    public Queue queue() {
        return QueueBuilder.nonDurable("queue.anno").build();
    }
}
```

consumer推模式:

```java
@Configuration
@ComponentScan("com.fishleap.listener")
@EnableRabbit  // xml中也可以使用<rabbit:annotation-driven />  启用@RabbitListener注解
public class RabbitConfig {
    
    // ConnectionFactory
    @Bean
    public ConnectionFactory connectionFactory() {
        return new CachingConnectionFactory(URI.create("amqp://root:123456@atzlg8:5672/%2f"));
    }
    
    // RabbitTemplate
    @Bean
    @Autowired
    public RabbitTemplate rabbitTemplate(ConnectionFactory factory) {
        return new RabbitTemplate(factory);
    }
    
    // RabbitAdmin
    @Bean
    @Autowired
    public RabbitAdmin rabbitAdmin(ConnectionFactory factory) {
        return new RabbitAdmin(factory);
    }
    
    // Queue
    @Bean
    public Queue queue() {
        return QueueBuilder.nonDurable("queue.anno").build();
    }
    
    @Bean("rabbitListenerContainerFactory")
    @Autowired
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setAcknowledgeMode(AcknowledgeMode.AUTO);
        factory.setConcurrentConsumers(10);
        factory.setMaxConcurrentConsumers(15);
        // 按照批次消费消息
        factory.setBatchSize(10);
        return factory;
    }
    
}
```

push推模式所需要的监听类:

```java
@Component
public class MyMessageListener {
    
    /**
     * com.rabbitmq.client.Channel channel对象
     * org.springframework.amqp.core.Message message对象 可以直接操作原生的AMQP消息
     * org.springframework.messaging.Message to use the messaging abstraction counterpart
     * @Payload 注解方法参数，改参数的值就是消息体
     * @Header 注解方法参数，访问指定的消息头字段的值
     * @Headers 该注解的方法参数获取该消息的消息头的所有字段，参数类型对应于map集合。
     * MessageHeaders 参数类型，访问所有消息头字段
     * MessageHeaderAccessor or AmqpMessageHeaderAccessor 访问所有消息头字段
     */
    @RabbitListener(queues = "queue.anno")
    public void onMessage(@Payload String messageStr) {
        System.out.println(messageStr);
    }
    
//    @RabbitListener(queues = "queue.anno")
//    public void onMessage(Message message) throws UnsupportedEncodingException {
//        System.out.println(new String(message.getBody(), message.getMessageProperties().getContentEncoding()));
//    }
    
}
```

#### 2.3 应用类

producer:

```java
public class ProducerApp {
    
    public static void main(String[] args) throws UnsupportedEncodingException {
        AbstractApplicationContext context = new AnnotationConfigApplicationContext(RabbitConfig.class);
        RabbitTemplate template = context.getBean(RabbitTemplate.class);
        
        MessageProperties properties = MessagePropertiesBuilder
                .newInstance()
                .setContentEncoding("utf-8")
                .setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN)
                .setHeader("mykey","myvalue")
                .build();
                
        Message message = MessageBuilder
                .withBody("你好，哈哈哈！".getBytes("utf-8"))
                .andProperties(properties)
                .build();
        
        template.send("ex.anno.fanout", "key.fanout", message);
        
        context.close();
    }

}
```

consumer拉模式:

```java
public class ConsumerPullApp {
    
    public static void main(String[] args) throws UnsupportedEncodingException {
        AbstractApplicationContext context = new AnnotationConfigApplicationContext(RabbitCsumConfig.class);
        RabbitTemplate template = context.getBean(RabbitTemplate.class);
    
        Message message = template.receive("queue.anno");
        System.out.println(new String(message.getBody(), message.getMessageProperties().getContentEncoding()));
        
        context.close();
    
    }
}
```

consumer推模式:

```java
public class ConsumerPushApp {
    public static void main(String[] args) {
        new AnnotationConfigApplicationContext(RabbitPushConfig.class);
    }
}
```

## 七.SpringBoot整合RabbitMQ

### 1.引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 2.配置文件

application.properties配置文件:

```properties
spring.application.name=rabbitmq-springboot
spring.rabbitmq.host=192.168.91.112
spring.rabbitmq.port=5672
spring.rabbitmq.username=root
spring.rabbitmq.password=123456
spring.rabbitmq.virtual-host=/
```

### 3.配置类

producer:

```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue queue() {
        return new Queue("queue.boot", false, false, false, null);
    }
    
    // 交换器名称，交换器类型（），是否是持久化的，是否自动删除，交换器属性Map集合
    // return new CustomExchange("custom.biz.ex", ExchangeTypes.DIRECT, false, false, null);
    @Bean
    public Exchange exchange() {
        return new TopicExchange("ex.boot", false, false, null);
    }
    
    @Bean
    public Binding binding() {
        // 绑定的目的地，绑定的类型：到交换器还是到队列，交换器名称，路由key，绑定的属性
        return new Binding("queue.boot", Binding.DestinationType.QUEUE,
                "ex.boot", "key.boot", null);
    }
    
}
```

consumer:

```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue queue() {
        return new Queue("queue.boot", false, false, false, null);
    }
    
}
```

消费者端推模式监听类:

```java
@Component
public class MyMessageListener {

    /**
     * com.rabbitmq.client.Channel channel对象
     * org.springframework.amqp.core.Message message对象 可以直接操作原生的AMQP消息
     * org.springframework.messaging.Message to use the messaging abstraction counterpart
     * @Payload 注解方法参数，改参数的值就是消息体
     * @Header 注解方法参数，访问指定的消息头字段的值
     * @Headers 该注解的方法参数获取该消息的消息头的所有字段，参数类型对应于map集合。
     * MessageHeaders 参数类型，访问所有消息头字段
     * MessageHeaderAccessor or AmqpMessageHeaderAccessor 访问所有消息头字段
     */
    // 可以点击RabbitListener注解进入查看有哪些参数可以使用
    @RabbitListener(queues = "queue.boot")
    public void onMessage(@Payload String message, @Header(name = "mykey")String value) {
        System.out.println(message);
        System.out.println("mykey = " + value);
    }
    
}
```

### 4.应用类

发送消息:

```java
@RestController
public class MessageController {
    
    @Autowired
    private AmqpTemplate rabbitTemplate;
    
    @GetMapping("/rabbitmq/{message}")
    public String sendMessage(@PathVariable String message) throws UnsupportedEncodingException {
    
        MessageProperties properties = MessagePropertiesBuilder.newInstance()
                .setContentEncoding("utf-8")
                .setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN)
                .setHeader("mykey", "myvalue")
                .build();
        
        Message msg = MessageBuilder.withBody(message.getBytes("utf-8"))
                .andProperties(properties)
                .build();
        
        rabbitTemplate.send("ex.boot", "key.boot", msg);
        
        return "OK";
    }
    
}
```

启动类：

```java
@SpringBootApplication
public class RabbitmqSpringbootApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(RabbitmqSpringbootApplication.class, args);
    }
    
}
```

启动完成后使用 http://localhost:8080/rabbitmq/aaabbb 进行测试，页面返回OK。控制台打印：

```
aaabbb
mykey = myvalue
```

测试时为了简便可以不用编写两个应用：生产者和消费者。可以编写一个应用即当生产者也当消费者，自己生产自己消费。









