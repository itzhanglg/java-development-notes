## 一.消息可靠性

在支付平台中，我们转账发送消息给消费者的过程中，如何保证转账不出现问题呢？也就是**如何保证消息的可靠性**，消息可靠了，数据才能正确。支付平台必须保证数据的正确性，**保证数据并发安全性，保证数据最终一致性**。

支付平台保证数据数据一致性有下面两种方式：

- **分布式锁**：在操作某条数据时先锁定，可以用redis或zookeeper等常用框架来实现。如修改账单时，先锁定该账单，其它并发操作只能等待上一个操作的锁释放后再依次执行。优点：**保证数据强一致性**；缺点：高并发场景下可能有性能问题。
- **消息队列**：当消费者收到消息并消费完成后，需要发送ack给消息队列，若中间件超过指定时间还没收到ack消息，则定时去重发消息。优点：高并发、异步；缺点：有一定延时、数据弱一致性。

如下图我们可以从以下几个方面来保证消息的可靠性：

![image-20201120215105474](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201120215105474.png)

### 1.异常捕获机制

先执行行业务操作，业务操作成功后执行行消息发送，消息发送过程通过try catch 方式捕获异常，在异常处理的代码块中执行**回滚业务操作**或者执行**重发操作**等。这是一种最大努力确保的方式，并无法保证100%绝对可靠，因为这里没有异常并不代表消息就一定投递成功。

```java
boolean result = doBiz();
if (result) {
    try {
        sendMsg();
    } catch (Exception e) {
        // retrySend();
        // delaySend();
        rollbackBiz();
    }
}
```

另外，我们也可以通过 `spring.rabbitmq.template.retry.enabled=true` 配置开启发送端重试。

### 2.发送端确认机制

发送方确认（publisher confirm）机制：生产者将信道设置成confirm(确认)模式，一旦信道进入confirm 模式，所有在**该信道上面发布的消息都会被指派一个唯一的ID(从1 开始)**，一旦消息被投递到所有匹配的队列之后（如果消息和队列是持久化的，那么确认消息会在消息持久化后发出），RabbitMQ 就会发送**一个确认(Basic.Ack)给生产者(包含消息的唯一ID)**，这样生产者就知道消息已经正确送达了。

![image-20201120221350690](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201120221350690.png)

**Broker回复生产者的确认消息可以通过 `channel.basicAck()` 方法来实现，方法中deliveryTag参数包含了确认消息的序号，multiple参数设置为true表示这个序号之前的所有消息都已经得到处理了**。生产者可以连续的向RabbitMQ中投递消息，通过回调方式处理ACK响应，若Broker内部错误导致消息丢失等异常情况发生，就会响应一条nack(Basic.Nack)命令。

#### 2.1 单消息同步等待

```java
/**
 * @author zlg
 * 单消息确认同步等待
 */
public class ProducerOnePCSync {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
    
        // 向RabbitMQ服务器发送AMQP命令,将当前通道标记为发送方确认通道
        final AMQP.Confirm.SelectOk selectOk = channel.confirmSelect();
        
        channel.queueDeclare("queue.pc", true, false, false, null);
        channel.exchangeDeclare("ex.pc", "direct", true, false, null);
        channel.queueBind("queue.pc", "ex.pc", "key.pc");
        
        // 发送消息
        String message = "hello world";
        channel.basicPublish("ex.pc", "key.pc", null, message.getBytes());
        
        try {
            // 同步的方式等待RabbitMQ确认消息
            channel.waitForConfirmsOrDie(5_000);
            System.out.println("消息被确认：message = " + message);
        } catch (IOException e) {
            e.printStackTrace();
            System.err.println("消息被拒绝！ message = " + message);
        } catch (InterruptedException e) {
            e.printStackTrace();
            System.err.println("在不是Publisher Confirms的通道上使用该方法");
        } catch (TimeoutException e) {
            e.printStackTrace();
            System.err.println("等待消息确认超时！ message = " + message);
        }
        
    }
}
```

waitForConfirm方法有个重载的，可以自定义timeout超时时间，超时后会抛TimeoutException。waitForConfirmsOrDie方法也一样，有多个重载方法，Broker端在返回nack(Basic.Nack)之后该方法会抛出java.io.IOException。在处理异常时需要根据异常类型来区别进行处理。

#### 2.2 批量消息同步等待

也可以通过批处理（**批量发送消息后仅调用一次waitForConfirms方法**）来改善上面的性能。但是如果发生了超时或者nack（失败）后那就需要批量量重发消息或者通知上游业务批量回滚，批量重发消息肯定会造成部分消息重复。

批消息确认还是需要同步等待，例如下面代码每10条消息等待一次：

```java
/**
 * @author zlg
 * 批消息还是需要同步等待,每10条消息等待一次
 */
public class ProducerBatchPCSync {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
    
        // 向RabbitMQ服务器发送AMQP命令,将当前通道标记为发送方确认通道
        final AMQP.Confirm.SelectOk selectOk = channel.confirmSelect();
        
        channel.queueDeclare("queue.pc", true, false, false, null);
        channel.exchangeDeclare("ex.pc", "direct", true, false, null);
        channel.queueBind("queue.pc", "ex.pc", "key.pc");
        
        // 发送消息
        String message = "hello world - ";
        // 批处理大小
        int batchSize = 10;
        // 对需要等待确认消息的计数
        int outstandingConfirms = 0;
        for (int i = 0; i < 103; i++) {
            channel.basicPublish("ex.pc", "key.pc", null, (message + i).getBytes());
            outstandingConfirms++;
            if (outstandingConfirms == batchSize) {
                // 此时已经有一个批次的消息需要同步等待broker的确认消息
                channel.waitForConfirmsOrDie(5_000);
                System.out.println("批消息已经被确认!");
                outstandingConfirms = 0;
            }
        }
        if (outstandingConfirms > 0) {
            channel.waitForConfirmsOrDie(5_000);
            System.out.println("剩余消息已经被确认!");
        }
        
        channel.close();
        connection.close();
        
    }
}
```

#### 2.3 消息异步处理

我们也可以通过异步调用方式来处理Broker的响应。addConfirmListener 方法可以添加ConfirmListener 这个回调接口，这个回调接口包含handleAck（处理响应的Basic.Ack）和handleNack（处理响应的Basic.Nack）这两个方法。

消息确认异步处理：

```java
/**
 * @author zlg
 * 消息确认异步处理
 */
public class ProducerPCAsync {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();
        
        // 向RabbitMQ服务器发送AMQP命令，将当前通道标记为发送方确认通道
        final AMQP.Confirm.SelectOk selectOk = channel.confirmSelect();
        
        channel.queueDeclare("queue.pc", true, false, false, null);
        channel.exchangeDeclare("ex.pc", "direct", true, false, null);
        channel.queueBind("queue.pc", "ex.pc", "key.pc");

//        ConfirmCallback clearOutstandingConfirms = new ConfirmCallback() {
//            @Override
//            public void handle(long deliveryTag, boolean multiple) throws IOException {
//                if (multiple) {
//                    System.out.println("编号小于等于 " + deliveryTag + " 的消息都已经被确认了");
//                } else {
//                    System.out.println("编号为：" + deliveryTag + " 的消息被确认");
//                }
//            }
//        };
        
        ConcurrentNavigableMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();
        
        ConfirmCallback clearOutstandingConfirms = (deliveryTag, multiple) -> {
            if (multiple) {
                System.out.println("编号小于等于 " + deliveryTag + " 的消息都已经被确认了");
                // 获取map集合的子集
                final ConcurrentNavigableMap<Long, String> headMap
                        = outstandingConfirms.headMap(deliveryTag, true);
                // 清空outstandingConfirms中已经被确认的消息信息
                headMap.clear();
                
            } else {
                // 移除已经被确认的消息
                outstandingConfirms.remove(deliveryTag);
                System.out.println("编号为：" + deliveryTag + " 的消息被确认");
            }
        };
        
        // 设置channel的监听器，处理确认的消息和不确认的消息
        channel.addConfirmListener(clearOutstandingConfirms, (deliveryTag, multiple) -> {
            if (multiple) {
                // 将没有确认的消息记录到一个集合中
                System.out.println("消息编号小于等于：" +  deliveryTag + " 的消息不确认");
                ConcurrentNavigableMap<Long, String> headMap =
outstandingConfirms.headMap(sequenceNumber, true);
            } else {
                System.out.println("编号为：" + deliveryTag + " 的消息不确认");
                outstandingConfirms.remove(sequenceNumber);
            }
        });
        
        String message = "hello-";
        for (int i = 0; i < 1000; i++) {
            // 获取下一条即将发送的消息的消息ID
            final long nextPublishSeqNo = channel.getNextPublishSeqNo();
            channel.basicPublish("ex.pc", "key.pc", null, (message + i).getBytes());
            System.out.println("编号为：" + nextPublishSeqNo + " 的消息已经发送成功，尚未确认");
            outstandingConfirms.put(nextPublishSeqNo, (message + i));
        }
        
        // 等待消息被确认
        Thread.sleep(10000);
        
        channel.close();
        connection.close();
    }
}
```

#### 2.4 SpringBoot案例

##### 2.4.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

##### 2.4.2 配置文件

```properties
spring.application.name=publisherconfirm
spring.rabbitmq.host=atzlg8
spring.rabbitmq.virtual-host=/
spring.rabbitmq.username=root
spring.rabbitmq.password=123456
spring.rabbitmq.port=5672

spring.rabbitmq.publisher-confirm-type=correlated
spring.rabbitmq.publisher-returns=true
```

##### 2.4.3 配置类

```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue queue() {
        return new Queue("queue.biz", false, false, false, null);
    }
    
    @Bean
    public Exchange exchange() {
        return new DirectExchange("ex.biz", false, false, null);
    }
    
    @Bean
    public Binding binding() {
        return BindingBuilder.bind(queue()).to(exchange()).with("biz").noargs();
    }
}
```

##### 2.4.4 应用类

控制层：

```java
@RestController
public class BizController {
    
    private RabbitTemplate rabbitTemplate;
    
    @Autowired
    public void setRabbitTemplate(RabbitTemplate rabbitTemplate) {
        
        this.rabbitTemplate = rabbitTemplate;
        this.rabbitTemplate.setConfirmCallback((correlationData, flag, cause) -> {
            if (flag) {
                try {
                    System.out.println("消息确认：" + correlationData.getId() + " "
                    + new String(correlationData.getReturnedMessage().getBody(), "utf-8"));
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            } else {
                System.out.println(cause);
            }
        });
    }
    
    @RequestMapping("/biz")
    public String doBiz() throws UnsupportedEncodingException {
        MessageProperties props = new MessageProperties();
        props.setCorrelationId("1234");
        props.setConsumerTag("msg1");
        props.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        props.setContentEncoding("utf-8");
        // props.setDeliveryMode(MessageDeliveryMode.NON_PERSISTENT); // 1
        // props.setDeliveryMode(MessageDeliveryMode.PERSISTENT); // 2
        
        CorrelationData cd = new CorrelationData();
        cd.setId("msg1");
        cd.setReturnedMessage(new Message("这是msg1的响应".getBytes("utf-8"), null));
        
        Message message = new Message("这是等待确认的消息".getBytes("utf-8"), props);
        rabbitTemplate.convertAndSend("ex.biz", "biz", message, cd);
        return "ok";
    }
    
    @RequestMapping("/bizfalse")
    public String doBizFalse() throws UnsupportedEncodingException {
        MessageProperties props = new MessageProperties();
        props.setCorrelationId("1234");
        props.setConsumerTag("msg1");
        props.setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN);
        props.setContentEncoding("utf-8");
        
        Message message = new Message("这是等待确认的消息".getBytes("utf-8"), props);
        rabbitTemplate.convertAndSend("ex.bizFalse", "biz", message);
        return "ok";
    }
}
```

主启动类：

```java
@SpringBootApplication
public class RabbitmqDemo {
    public static void main(String[] args) {
    	SpringApplication.run(RabbitmqDemo.class, args);
    }
}
```

结果：

![image-20201120231241331](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201120231241331.png)

### 3.RabbitMQ事务机制

从事务开启到事务提交都没有发生异常就代表消息发送成功了，但是事务机制在性能你方面的开销比较大，一般不推荐使用这种方式。

```java
try {
    // 将channel设置为事务模式
    channel.txSelect();
    // 发送消息到交换器
    channel.basicPublish("ex.default", "key.default", null, "hello".getBytes());
    // 提交事务:只有broke成功接受到消息才能提交成功
    channel.txCommit();
} catch (Exception e) {
    // 事务回滚
    channel.txRollback();
}
```

### 4.持久化存储机制

当RabbitMQ遇到重启、断电、停机等异常时，数据就会丢失。RabbitMQ可以从Exchange、Queue、Message三个方法来保障消息的持久性：

- **Exchange的持久化**。通过定义时设置durable 参数为ture来保证Exchange相关的元数据不丢失。
- **Queue的持久化**。也是通过定义时设置durable 参数为ture来保证Queue相关的元数据不丢失。
- **消息的持久化**。通过将消息的投递模式 (BasicProperties 中的 deliveryMode 属性)设置为 **2**，即可实现消息的持久化，保证消息自身不丢失。

```java
/**
 * @author zlg
 * 持久化Exchange、Queue、Message
 */
public class ProducerPersistent {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        // duration:持久化消息队列
        channel.queueDeclare("queue.persistent", true, false ,false, null);
        // duration:持久化交换器
        channel.exchangeDeclare("ex.persistent", "direct", true, false, null);
        
        channel.queueBind("queue.persistent", "ex.persistent", "key.persistent");
        
        // 持久化消息
        final AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                .deliveryMode(2)  // 表示持久化消息
                .build();
    
        // 发送消息
        channel.basicPublish("ex.persistent", "key.persistent",
                properties,  // 设置消息的属性,此时消息是持久化消息
                "hello world".getBytes());
        
        channel.close();
        connection.close();
    }
}
```

RabbitMQ中的持久化消息都需要写入磁盘（当系统内存不不足时，非持久化的消息也会被刷盘处理），这些处理理动作都是在“持久层”中完成的。持久层是一个逻辑上的概念，实际包含两个部分：

- **队列索引**(rabbit_queue_index)：负责维护Queue中消息的信息，包括消息的存储位置、是否已交给消费者、是否已被消费及Ack确认等，每个Queue都有与之对应的rabbit_queue_index。
- **消息存储**(rabbit_msg_store)：以键值对的形式存储消息，它被所有队列共享，在每个节点中有且只有一个。

```shell
# $RABBITMQ_HOME/var/lib/mnesia/rabbit@$HOSTNAME/msg_stores/vhosts/$VHostId
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# pwd /var/lib/rabbitmq/mnesia/rabbit@atzlg8/msg_stores/vhosts/628WB79CIFDYO9LJI6DKMI09L

# 实际存储消息位置：包含 queues、msg_store_persistent、msg_store_transient 这3个目录
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# ls
msg_store_persistent  msg_store_transient  queues  recovery.dets

# queues目录中保存着rabbit_queue_index相关的数据
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# ls queues/NJTLNCM3SHLLBXINIFLBRNC7/
0.idx  journal.jif

# msg_store_persistent保存着持久化消息数据，msg_store_transient保存着⾮持久化相关的数据
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# ls msg_store_persistent/
0.rdq
[root@atzlg8 628WB79CIFDYO9LJI6DKMI09L]# ls msg_store_transient/
0.rdq
```

RabbitMQ通过配置`queue_index_embed_msgs_below`可以**根据消息大小决定存储位置**，默认`queue_index_embed_msgs_below`是**4096**字节(包含消息体、属性及headers)，小于该值的消息存在rabbit_queue_index中。

rabbitmq.conf中配置：

```properties
## Size in bytes below which to embed messages in the queue index.
## Related doc guide: https://rabbitmq.com/persistence-conf.html
##
queue_index_embed_msgs_below = 4096
## You can also set this size in memory units
##
# queue_index_embed_msgs_below = 4kb
```

### 5.消费端确认机制

#### 5.1 概念

如何保证消息可以被消费者成功消费呢？**RabbitMQ在消费端会有Ack机制**，即消费端消费消息后需要发送Ack确认报文给Broker端，告知自己是否已消费完成，否则可能会一直重发消息直到消息过期（AUTO模式）。

消费端确认机制有以下几种处理方式：

- **采用NONE模式**：消费的过程中自行捕获异常，引发异常后直接记录日志并落到异常恢复表，再通过后台定时任务扫描异常恢复表尝试做重试动作。如果业务不自行处理则有丢失数据的风险；
- **采用AUTO（自动Ack）模式**：不主动捕获异常，当消费过程中出现异常时会将消息放回Queue中，然后消息会被重新分配到其他消费者节点（如果没有则还是选择当前节点）重新被消费，默认会一直重发消息并直到消费完成返回Ack或者一直到过期；
- **采用MANUAL（手动Ack）模式**：消费者手动控制流程并调用channel相关的方法返回Ack。

生产者：

```java
public class Producer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();
    
        channel.queueDeclare("queue.ca", false, false, false , null);
        channel.exchangeDeclare("ex.ca", "direct", false ,false ,null);
        channel.queueBind("queue.ca", "ex.ca", "key.ca");
    
        for (int i = 0; i < 5; i++) {
            channel.basicPublish("ex.ca", "key.ca", null, ("hello - " + i).getBytes());
        }
        
//        channel.close();
//        connection.close();
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
    
        channel.queueDeclare("queue.ca", false, false, false , null);
        
        // pull消息模式
//        final GetResponse getResponse = channel.basicGet("queue.ca", false);
//        channel.basicReject(getResponse.getEnvelope().getDeliveryTag(), true);
    
        // push消息模式
        // autoAck:false表示手动确认消息,第三个参数:consumerTag
        channel.basicConsume("queue.ca", false, "myConsumer", new DefaultConsumer(channel){
    
            @Override
            public void handleDelivery(final String consumerTag, final Envelope envelope,final AMQP.BasicProperties properties, final byte[] body) throws IOException {
                
                System.out.println(new String(body));
                // 确认消息
//                channel.basicAck(envelope.getDeliveryTag(), false);
                // 消息的唯一标识、是否是批量确认、是否需要将消息重新入列（用于拒收多条消息）
//                channel.basicNack(envelope.getDeliveryTag(), false, true);
                // 用于拒收一条消息
                channel.basicReject(envelope.getDeliveryTag(), true);
                
            }
        });
        
//        channel.close();
//        connection.close();
        
    }
}
```

#### 5.2 SpringBoot案例

##### 5.2.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

##### 5.2.2 配置文件

```properties
spring.application.name=consumer_ack
spring.rabbitmq.host=atzlg8
spring.rabbitmq.virtual-host=/
spring.rabbitmq.username=root
spring.rabbitmq.password=123456
spring.rabbitmq.port=5672

# 最大重试次数
spring.rabbitmq.listener.simple.retry.max-attempts=5
# 是否开启消费者重试（false时关闭消费者重试，
# 意思不是“不重试”，而是一直收到消息直到ack确认或者一直到超时）
spring.rabbitmq.listener.simple.retry.enabled=true
#重试间隔时间（单位毫秒）
spring.rabbitmq.listener.simple.retry.initial-interval=5000
# 重试超过最大次数后是否拒绝
spring.rabbitmq.listener.simple.default-requeue-rejected=false
#ack模式
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

##### 5.2.3 配置类

```java
@Configuration
public class RabbitConfig {
    @Bean
    public Queue queue() {
    	return new Queue("q.biz", false, false, false, null);
    }
    @Bean
    public Exchange exchange() {
    	return new DirectExchange("ex.biz", false, false, null);
    }
    @Bean
    public Binding binding() {
    return BindingBuilder.bind(queue()).to(exchange()).with("biz").noargs();
    }
}
```

##### 5.2.4 监听类

上面是消费端通过配置文件application.properties指定ackMode的。下面是消费端通过监听注解配置指定的ackMode，需要注意的是channel.basicAck这几个手动Ack确认的方法。选择application.properties配置文件和RabbitListener注解（ackMode属性）配置都可以的。

```java
/**
 * @author zlg
 */
//@Component
public class MessageListener {
    
    private Random random = new Random();
    
    /**
     * NONE模式，则只要收到消息后就立即确认（消息出列，标记已消费），有丢失数据的风险
     * AUTO模式，看情况确认，如果此时消费者抛出异常则消息会返回到队列中
     * MANUAL模式，需要显式的调用当前channel的basicAck方法
     * @param channel
     * @param deliveryTag
     * @param message
     */
// @RabbitListener(queues = "q.biz", ackMode = "AUTO")
    @RabbitListener(queues = "q.biz", ackMode = "MANUAL")
// @RabbitListener(queues = "q.biz", ackMode = "NONE")
    public void handleMessageTopic(Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag, @Payload String message) {
        System.out.println("RabbitListener消费消息，消息内容：" + message);
        try {
            if (random.nextInt(10) % 3 != 0) {
                // 手动nack，告诉broker消费者处理失败，最后一个参数表示是否需要将消息重新入列
                // channel.basicNack(deliveryTag, false, true);
                // 手动拒绝消息。第二个参数表示是否重新入列
                channel.basicReject(deliveryTag, true);
            } else {
                // 手动ack，deliveryTag表示消息的唯一标志，multiple表示是否是批量确认
                channel.basicAck(deliveryTag, false);
                System.err.println("已确认消息：" + message); 
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

##### 5.2.5 应用类

```java
@RestController
public class BizController {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    private Random random = new Random();
    
    @RequestMapping("/biz")
    public String getBizMessage() {
        String message = rabbitTemplate.execute(new ChannelCallback<String>() {
            @Override
            public String doInRabbit(Channel channel) throws Exception {
                final GetResponse getResponse = channel.basicGet("q.biz", false);
                if (getResponse == null) return "你已消费完所有的消息";
                String message = new String(getResponse.getBody(), "utf-8");
                
                if (random.nextInt(10) % 3 == 0) {
                    channel.basicAck(getResponse.getEnvelope().getDeliveryTag(), false);
                    return "已确认的消息：" + message;
                } else {
                    // 拒收一条消息
                    channel.basicReject(getResponse.getEnvelope().getDeliveryTag(), true);
                    // 可以拒收多条消息
//                        channel.basicNack(getResponse.getEnvelope().getDeliveryTag(), false, true);
                    return "拒绝的消息：" + message;
                }
            }
        });
        return message;
    }
}
```

主入口类：

```java
@SpringBootApplication
public class RabbitmqDemo {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public static void main(String[] args) {
        SpringApplication.run(RabbitmqDemo.class, args);
    }
    
    @Bean
    public ApplicationRunner runner() {
        return args -> {
            Thread.sleep(5000);
            for (int i = 0; i < 10; i++) {
                MessageProperties props = new MessageProperties();
                props.setDeliveryTag(i);
                
                Message message = new Message(("消息：" + i).getBytes("utf-8"), props);
                // this.rabbitTemplate.convertAndSend("ex.biz", "biz", "消息：" + i);
                this.rabbitTemplate.convertAndSend("ex.biz", "biz", message);
            }
        };
    }
}
```

### 6.消费端限流

当消息投递速度远快于消费速度时，随着时间积累就会出现“消息积压”。消息中间件本身是具备一定的缓冲能力的，但这个能力是有容量限制的，如果长期运行并没有任何处理，最终会导致Broker崩溃，而分布式系统的故障往往会发生上下游传递，造成雪崩效应。

下面从几个角度介绍Qos与限流：

1.**RabbitMQ 可以对内存和磁盘使用量设置阈值，当达到阈值后，生产者将被阻塞(block)，直到对应项指标恢复正常**。全局上可以防止超大流量、消息积压等导致的Broker被压垮。当内存受限或磁盘可用空间受限的时候，服务器都会暂时阻止连接，服务器将暂停从发布消息的已连接客户端的套接字读取数据。连接心跳监视也将被禁用。所有网络连接将在rabbitmqctl和管理插件中显示为“已阻止”，兼容的客户端被阻止时将收到通知。

在/etc/rabbitmq/rabbitmq.conf中配置磁盘可用空间大小：

![image-20201122094432989](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201122094432989.png)

2.RabbitMQ 还**默认提供了一种基于credit flow 的流控机制，面向每一个连接进行流控**。当单个队列达到最大流速时，或者多个队列达到总流速时，都会触发流控。触发单个链接的流控可能是因为connection、channel、queue的某一个过程处于flow状态，这些状态都可以从监控平台看到。

![image-20201122095106885](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201122095106885.png)

![image-20201122095217400](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201122095217400.png)

3.RabbitMQ中有一种**QoS保证机制，可以限制Channel上接收到的未被Ack的消息数量，如果超过这个数量限制RabbitMQ将不会再往消费端推送消息**。这是一种流控手段，可以防止大量消息瞬时从Broker送达消费端造成消费端巨大压力（甚至压垮消费端）。注意的是**QoS机制仅对于消费端推模式有效，对拉模式无效。而且不支持NONE Ack模式**。

执行`channel.basicConsume` 方法之前通过 `channel.basicQoS` 方法可以设置该数量。消息的发送是异步的，确认也是异步的。在消费者消费慢的时候，可以设置Qos的prefetchCount，它表示broker在向消费者发送消息的时候，一旦发送了prefetchCount个消息而没有一个消息确认的时候，就停止发送。消费者确认多少，broker就发送多少，消费者等待处理的个数永远限制在prefetchCount个。

如果对于每个消息都发送确认，增加了网络流量，此时可以批量确认消息。如果设置了multiple为true，消费者在确认的时候，比如说id是8的消息确认了，则在8之前的所有消息都确认了。

小结：

- 从上游来讲我们应该在**生产端应用程序中也可以加入限流、应急开关等控制手段**，避免超过Broker端的极限承载能力或者压垮下游消费者。
- 从下游来讲我们期望下游消费端能尽快消费完消息，而且还要防止瞬时大量消息压垮消费端（推模式），我们期望**消费端处理速度是最快、最稳定而且还相对均匀**（比较理想化）。

提升**下游应用的吞吐量和缩短消费过程的耗时**，优化主要以下几种方式：

- 优化应用程序的性能，缩短响应时间（需要时间）
- 增加消费者节点实例（成本增加，而且底层数据库操作这些也可能是瓶颈）
- 调整并发消费的线程数（线程数并非越大越好，需要大量压测调优至合理值）

```java
public class Consumer {
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
    	// 设置channel并发请求最大数
        //factory.setRequestedChannelMax(10);
        // 设置自定义线程工厂
        //final ThreadFactory threadFactory = Executors.defaultThreadFactory();
        //factory.setThreadFactory(threadFactory);
        
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();
    
        channel.queueDeclare("queue.qos", false, false, false, null);
    
        // 使用basic做限流，仅对消息推送模式生效。
        // 表示Qos是10个消息，最多有10个消息等待确认
        channel.basicQos(10);
        // 表示最多10个消息等待确认。如果global设置为true，则表示只要是使用当前的channel的Consumer，该设置都生效
        // false表示仅限于当前Consumer
        //channel.basicQos(10, false);
        // 第一个参数表示未确认消息的大小，Rabbit没有实现，不用管。
        //channel.basicQos(1000, 10, true);
    
        channel.basicConsume("queue.qos", false, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                // some code going on
                // 可以批量确认消息，减少每个消息都发送确认带来的网络流量负载。
                channel.basicAck(envelope.getDeliveryTag(), true);
            }
        });
        
        channel.close();
        connection.close();
    }
}
```

RabbitMQ配置文件中注入监听容器工厂（推模式）：

```java
@Bean
public RabbitListenerContainerFactory
    rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
    // SimpleRabbitListenerContainerFactory发现消息中有content_type有text就会默认将其
    // 转换为String类型的，没有content_type都按byte[]类型
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);
    // 设置并发线程数
    factory.setConcurrentConsumers(10);
    // 设置最大并发线程数
    factory.setMaxConcurrentConsumers(20);
    return factory;
}
```

### 7.消息可靠性保证

一般消息中间件的**消息传输保障**分为以下三个层级。RabbitMQ 支持其中的“最多一次”和“最少一次”。

- At most once：最多一次。消息可能会丢失，但绝**不会重复**传输；
- At least once：最少一次。消息绝不会丢失，但**可能会重复**传输；
- Exactly once：恰好一次。每条消息肯定会被**传输一次且仅传输一次**。

“**最少一次**”投递实现需要考虑以下这个几个方面的内容：

- 消息生产者需要开启事务机制或者**publisher confirm 机制**，以确保消息可以可靠地传输到RabbitMQ 中。
- 消息生产者需要配合使用 **mandatory 参数**或者备份交换器来确保消息能够从交换器路由到队列中，进而能够保存下来而不会被丢弃。
- 消息和队列都需要进行**持久化处理**，以确保RabbitMQ 服务器在遇到异常情况时不会造成消息丢失。
- 消费者在消费消息的同时需要将autoAck 设置为false，然后通过**手动确认的方式**去确认已经正确消费的消息，以避免在消费端引起不必要的消息丢失。

“**最多一次**”的方式就无须考虑以上那些方面，生产者随意发送，消费者随意消费，不过这样很难确保消息不会丢失。

“**恰好一次**”是RabbitMQ 目前无法保障的。有下面两种情况：

- 消费者在消费完一条消息之后向RabbitMQ 发送确认Basic.Ack 命令，此时由于网络断开或者其他原因造成RabbitMQ 并没有收到这个确认命令，那么RabbitMQ 不会将此条消息标记删除。在重新建立连接之后，消费者还是会消费到这一条消息，这就造成了重复消费。
- 生产者在使用publisher confirm机制的时候，发送完一条消息等待RabbitMQ返回确认通知，此时网络断开，生产者捕获到异常情况，为了确保消息可靠性选择重新发送，这样RabbitMQ 中就有两条同样的消息，在消费的时候消费者就会重复消费。

### 8.消息幂等性

当我们追求高性能就无法保证消息的顺序，而追求可靠性那么就可能产生重复消息。一般解决重复消息的办法是，在**消费端让我们消费消息的操作具备幂等性**。

一个幂等操作的特点是，其**任意多次执行所产生的影响均与一次执行的影响相同**。一个幂等的方法，使用同样的参数，对它进行多次调用和一次调用，对系统产生的影响是一样的，**不用担心重复执行会对系统造成任何改变**。

对于幂等性的一些常见做法：

- **借助数据库唯一索引，重复请求时会直接报索引冲突，事务直接回滚**。现实中，数据库唯一索引的方式通常做为兜底保证；
- **前置检查机制**。例如在更改余额时，先去校验相关表中是否存在当前交易记录，若不存在则执行正常的更新操作。为了防止并发问题，我们通常需要借助“**排他锁**”来完成。在支付宝有一条铁律叫：**一锁、二判、三操**
    **作**。也可以使用**乐观锁**（扩展一个版本号字段做判断条件）或**CAS机制**。
- **唯一Id机制**，比较通用的方式。对于每条消息我们都可以生成唯一Id，消费前判断Tair表中是否存在（MsgId做Tair排他锁的key），消费成功后将状态写入Tair中，这样就可以防止重复消费了。

对于接口请求类的幂等性保证要相对更复杂，我们通常要求上游请求时传递一个类GUID的请求号（或TOKEN），如果我们发现**已经存在**了并且上一次请求处理结果是成功状态的则不继续往下执行，直接返回“重复请求”的提示和上次的处理结果。如果请求ID都**不存在**或者上次处理结果是失败/异常的，那就继续处理流程，并最终记录最终的处理结果。这个请求序号由上游自己生成，上游通常需要根据请求参数、时间间隔等因子来生成请求ID。同样也需要利用这个请求ID做分布式锁的KEY实现排他。

## 二.可靠性分析

在使用任何消息中间件的过程中，难免会出现消息丢失等异常情况，这个时候就需要有一个良好的机制来跟踪记录消息的过程（轨迹溯源），帮助我们排查问题。有下面两种方式来**跟踪消息的投递、消费过程**。

### 1.Firehose 功能

Firehose 可以记录每一次发送或者消费消息的记录，方便RabbitMQ 的使用者进行调试、排错等。Firehose 的原理是将生产者投递给RabbitMQ 的消息，或者RabbitMQ 投递给消费者的消息**按照指定的格式发送到默认的交换器（amq.rabbitmq.trace）上**。

这个默认的交换器是一个topic 类型的交换器。发送到这个交换器上的消息的路由键为 `publish.{exchangename} `和 `deliver.{queuename}` 。其中 exchangename 和 queuename 为交换器和队列的名称，分别对应生产者投递到交换器的消息和消费者从队列中获取的消息。

```shell
# 开启Firehose命令，其中[-p vhost]是可选参数，用来指定虚拟主机vhost。
rabbitmqctl trace_on [-p vhost]
# 对应的关闭命令
rabbitmqctl trace_off [-p vhost]
```

Firehose 默认情况下处于**关闭状态**，并且Firehose 的状态是**非持久化**的，会在RabbitMQ服务重启的时候还原成默认的状态。Firehose 开启之后多少会影响RabbitMQ 整体服务性能，因为它会引起额外的消息生成、路由和存储。

下面看下案例：

```shell
# 开启
[root@atzlg8 ~]# rabbitmqctl trace_on
Starting tracing for vhost "/" ...
Trace enabled for vhost /
```

生产者，生产100条消息：

```java
public class MyProducer {
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare("queue.tc.demo", false, false, false, null);
        channel.exchangeDeclare("ex.tc.demo", "direct", false);
        channel.queueBind("queue.tc.demo", "ex.tc.demo", "key.tc");

        for (int i = 0; i < 100; i++) {
            channel.basicPublish("ex.tc.demo", "key.tc", null, ("hello" + i).getBytes());
        }

        channel.close();
        connection.close();
    }
}
```

消费者，消费25条消息：

```java
public class MyConsumer {
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare("queue.tc.demo", false, false, false, null);

        for (int i = 0; i < 25; i++) {
            final GetResponse getResponse = channel.basicGet("queue.tc.demo", true);
            System.out.println(new String(getResponse.getBody()));
        }

        channel.close();
        connection.close();
    }
}
```

创建两个队列：

![image-20201122112748990](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201122112748990.png)

将amq.rabbitmq.trace默认交换器和创建的两个队列进行绑定：

![image-20201122113323484](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201122113323484.png)

启动生产者和消费者后示意图：

![image-20201122114312363](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201122114312363.png)

### 2.rabbitmq_tracing 插件

rabbitmq_tracing 插件相当于Firehose 的GUI 版本，它同样能跟踪RabbitMQ 中消息的流入流出情况。rabbitmq_tracing 插件同样会对流入流出的消息进行封装，然后将封装后的消息日志存入相应的trace 文件中。

```shell
# 查看所有可用的插件
rabbitmq-plugins list
# 查看启动的插件
rabbitmq-plugins list --enabled
# 启动rabbitmq_ tracing 插件
rabbitmq-plugins enable rabbitmq_tracing
# 关闭
rabbitmq-plugins disable rabbitmq_tracing
```

开启使用后，使用面板添加两个跟踪条目，添加后如下图：

![image-20201122115131922](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201122115131922.png)

添加栏中，Name表示rabbitmq_tracing的一个条目的名称，Format可以选择Text或JSON，连接的用户名写root，密码写123456。

- Pattern：发布的消息：`publish.<exname>`；
- Pattern：消费的消息：`deliver.<queuename>`。

## 三.TTL机制

### 1.TTL使用场景和介绍

在京东下单，订单创建成功，等待支付，一般会给30分钟的时间，开始倒计时。如果在这段时间内用户没有支付，则默认订单取消。如何实现该需求呢？

1.定期轮询（数据库等）：用户下单成功，将订单信息放入数据库，同时将支付状态放入数据库，用户付款更改数据库状态。定期轮询数据库支付状态，如果超过30分钟就将该订单取消。

- 优点：设计实现简单；
- 缺点：需要对数据库进行大量的IO操作，效率低下。

2.`Timer`：

```java
public class TimerDemo {
    
    public static void main(String[] args) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("HH:mm:ss");
        Timer timer = new Timer();
        TimerTask timerTask = new TimerTask() {
            @Override
            public void run() {
                System.out.println("用户没有付款，交易取消：" +
                        simpleDateFormat.format(new Date(System.currentTimeMillis())));
                timer.cancel();
            }
        };
        System.out.println("等待用户付款：" +
                simpleDateFormat.format(new Date(System.currentTimeMillis())));
        // 10秒后执行timerTask
        timer.schedule(timerTask, 10 * 1000);
    }

}
```

缺点：Timers没有持久化机制、不灵活 (只可以设置开始时间和重复间隔，对等待支付貌似够用)、不能利用线程池，一个timer一个线程、没有真正的管理计划。

3.`ScheduledExecutorService`：

```java
public class ScheduledExecutorServiceDemo {
    
    public static void main(String[] args) {
        SimpleDateFormat format = new SimpleDateFormat("HH:mm:ss");
        // 线程工厂
        ThreadFactory factory = Executors.defaultThreadFactory();
        // 使用线程池
        ScheduledExecutorService service = new ScheduledThreadPoolExecutor(10, factory);
        System.out.println("开始等待用户付款10秒：" + format.format(new Date()));
        service.schedule(new Runnable() {
            @Override
            public void run() {
                System.out.println("用户未付款，交易取消：" +
                        format.format(new Date()));
            }// 等待10s 单位秒
        }, 10, TimeUnit.SECONDS);
        
    }
}
```

可以多线程执行，一定程度上避免任务间互相影响，单个任务异常不影响其它任务。在高并发的情况下，不建议使用定时任务去做，因为太浪费服务器性能。

3.还可以使用其它也写框架来做，如Quartz、Redis Zset、JCronTab、SchedulerX等

4.RabbitMQ中TTL（Time to Live）过期时间。RabbitMQ 可以对**消息**和**队列**两个维度来**设置TTL**。任何消息中间件的容量和堆积能力都是有限的，如果有一些消息总是不被消费掉，那么需要有一种过期的机制来做兜底。目前有两种方法可以设置消息的TTL。

- 通过**Queue属性**设置，队列中所有消息都有相同的过期时间。
- 对**消息自身**进行单独设置，每条消息的TTL 可以不同。

**如果两种方法一起使用，则消息的TTL 以两者之间较小数值为准**。通常来讲，消息在队列中的生存时间一旦超过设置的TTL 值时，就会变成“死信”(Dead Message)，消费者默认就无法再收到该消息。当然，“死信”也是可以被取出来消费的。

```java
public class Producer {
    public static void main(String[] args) throws NoSuchAlgorithmException, KeyManagementException, URISyntaxException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");
        
        try (final Connection connection = factory.newConnection();
             final Channel channel = connection.createChannel()) {
            
            Map<String, Object> arguments = new HashMap<>();
//            消息队列中消息过期时间，10s
            arguments.put("x-message-ttl", 10 * 1000);
//            设置队列的空闲存活时间（如该队列没有消费者，一直没有使用，队列可以存活60s）
            arguments.put("x-expires", 60 * 1000);
            channel.queueDeclare("queue.ttl.waiting", true, false, false, arguments);
            
            channel.exchangeDeclare("ex.ttl.waiting", "direct", true, false, null);
            channel.queueBind("queue.ttl.waiting", "ex.ttl.waiting", "key.ttl.waiting");
            
            final AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                    .contentEncoding("utf-8")
                    .expiration("10000")
                    .build();
            channel.basicPublish("ex.ttl.waiting","key.ttl.waiting",
                    properties,
                    "等待的订单号".getBytes("utf-8"));
            
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        
    }
}
```

还可以通过命令行方式设置**全局TTL**，执行如下命令：

```shell
rabbitmqctl set_policy TTL ".*" '{"message-ttl":30000}' --apply-to queues
```

小结：

默认规则：如果不设置TTL，则表示此消息不会过期；如果TTL设置为0，则表示除非此时可以直接将消息投递到消费者，否则该消息会被立即丢弃。

注意理解 message-ttl 、 x-expires 这两个参数的区别，有不同的含义。但是这两个参数属性都遵循上面的默认规则。一般TTL相关的参数单位都是毫秒（ms）。

### 2.SpringBoot案例

#### 2.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 2.2 application.properties

```properties
spring.application.name=ttl
spring.rabbitmq.host=atzlg8
spring.rabbitmq.virtual-host=/
spring.rabbitmq.username=root
spring.rabbitmq.password=123456
spring.rabbitmq.port=5672
```

#### 2.3 RabbitConfig类

```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue queueTTLWaiting() {
        Map<String, Object> props = new HashMap<>();
        // 对于该队列中的消息，设置都等待10s
        props.put("x-message-ttl", 10000);
        Queue queue = new Queue("q.pay.ttl-waiting", false, false,
                false, props);
        return queue;
    }
    @Bean
    public Queue queueWaiting() {
        return new Queue("q.pay.waiting", false, false, false);
    }
    @Bean
    public Exchange exchangeTTLWaiting() {
        return new DirectExchange("ex.pay.ttl-waiting", false, false);
    }
    // 该交换器使用的时候，需要给每个消息设置有效期
    @Bean
    public Exchange exchangeWaiting() {
        return new DirectExchange("ex.pay.waiting", false, false);
    }
    
    @Bean
    public Binding bindingTTLWaiting() {
        return BindingBuilder.bind(queueTTLWaiting()).to(exchangeTTLWaiting()).with("pay.ttl-waiting").noargs();
    }
    @Bean
    public Binding bindingWaiting() {
        return BindingBuilder.bind(queueWaiting()).to(exchangeWaiting()).with("pay.waiting").noargs();
    }
}
```

#### 2.5 应用类

```java
@RestController
public class PayController {
    @Autowired
    private AmqpTemplate rabbitTemplate;
    
    // 给队列中消息都设置相同的有效期
    @RequestMapping("/pay/queuettl")
    public String sendMessage() {
        rabbitTemplate.convertAndSend("ex.pay.ttl-waiting", "pay.ttl-waiting", "发送了TTL-WAITING-MESSAGE");
        return "queue-ttl-ok";
    }
    
    @RequestMapping("/pay/msgttl")
    public String sendTTLMessage() throws UnsupportedEncodingException {
        MessageProperties properties = new MessageProperties();
        // 给当前这条消息设置有效期
        properties.setExpiration("5000");
        Message message = new Message("发送了WAITING-MESSAGE".getBytes("utf-8"), properties);
        rabbitTemplate.convertAndSend("ex.pay.waiting", "pay.waiting", message);
        return "msg-ttl-ok";
    }
}
```

主启动类：

```java
@SpringBootApplication
public class RabbitmqDemo {
    public static void main(String[] args) {
    	SpringApplication.run(RabbitmqDemo.class, args);
    }
}
```

## 四.死信队列

### 1.DLX介绍

例如：用户下单，调用订单服务，然后订单服务调用派单系统通知外卖人员送单，这时候订单系统与派单系统采用 MQ异步通讯。**在定义业务队列时可以考虑指定一个 死信交换机，并绑定一个死信队列。当消息变成死信时，该消**
**息就会被发送到该死信队列上，这样方便我们查看消息失败的原因**。

DLX，全称为Dead-Letter-Exchange，死信交换器。**消息在一个队列中变成死信（Dead Letter）之后，被重新发送到一个特殊的交换器（DLX）中，同时，绑定DLX的队列就称为“死信队列”**。

下列几种情况会导致消息变为死信：

- 消息被拒绝（Basic.Reject/Basic.Nack），并且不再投递设置requeue参数为false；
- 消息过期(rabbitmq Time-To-Live -> messageProperties.setExpiration())；
- 队列达到最大长度。

DLX可以处理异常情况下，消息不能够被消费者正确消费(消费者调用了Basic.Nack 或者Basic.Reject)而被置入死信队列中的情况，后续分析程序可以通过消费这个死信队列中的内容来分析当时所遇到的异常情况，进而可以改善
和优化系统。

原生API案例：

```java
public class Producer {
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://root:123456@atzlg8:5672/%2f");

        try (final Connection connection = factory.newConnection();
             final Channel channel = connection.createChannel()) {
            
//            声明死信交换器 DLX
            channel.exchangeDeclare("ex.dlx", "direct", true);
//            声明队列做死信队列
            channel.queueDeclare("queue.dlx", true, false, false, null);
//            绑定死信交换器和死信队列
            channel.queueBind("queue.dlx", "ex.dlx", "key.dlx");
    
//            正常业务的交换器
            channel.exchangeDeclare("ex.biz", "direct", true);
            Map<String, Object> arguments = new HashMap<>();
//            指定消息队列中的消息过期时间
            arguments.put("x-message-ttl", 10000);
//            设置该队列所关联的死信交换器（当队列消息TTL到期后依然没有消费，则加入死信队列）
            arguments.put("x-dead-letter-exchange", "ex.dlx");
//            设置该队列所关联的死信交换器的routingKey，如果没有特殊指定，使用原队列的routingKey
            arguments.put("x-dead-letter-routing-key", "key.dlx");
            channel.queueDeclare("queue.biz", true, false, false, arguments);
//            绑定业务的交换器和消息队列
            channel.queueBind("queue.biz", "ex.biz", "key.biz");

            channel.basicPublish("ex.biz", "key.biz", 
                    MessageProperties.PERSISTENT_TEXT_PLAIN, "dlx test".getBytes());

        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```

### 2.SpringBoot案例

#### 2.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 2.2 application.properties

```properties
spring.application.name=dlx
spring.rabbitmq.host=atzlg8
spring.rabbitmq.virtual-host=/
spring.rabbitmq.username=root
spring.rabbitmq.password=123456
spring.rabbitmq.port=5672
```

#### 2.3 RabbitConfig类

```java
@Configuration
public class RabbitConfig {
    
    // 正常业务队列
    @Bean
    public Queue queue() {
        Map<String, Object> props = new HashMap<>();
        // 消息的生存时间 10s
        props.put("x-message-ttl", 10000);
        // 设置该队列所关联的死信交换器（当队列消息TTL到期后依然没有消费，则加入死信队列）
        props.put("x-dead-letter-exchange", "ex.go.dlx");
        // 设置该队列所关联的死信交换器的routingKey，如果没有特殊指定，使用原队列的routingKey
        props.put("x-dead-letter-routing-key", "go.dlx");
        return new Queue("q.go", true, false, false, props);
    }
    // 死信队列(一般队列形式)
    @Bean
    public Queue queueDlx() {
        return new Queue("q.go.dlx", true, false, false);
    }
    
    // 正常业务交换器
    @Bean
    public Exchange exchange() {
        return new DirectExchange("ex.go", true, false, null);
    }
    // 死信交换器(一般交换器形式)
    @Bean
    public Exchange exchangeDlx() {
        return new DirectExchange("ex.go.dlx", true, false, null);
    }
    
    // 正常交换器绑定正常队列
    @Bean
    public Binding binding() {
        return BindingBuilder.bind(queue()).to(exchange()).with("go").noargs();
    }
    // 死信交换器绑定死信队列
    @Bean
    public Binding bindingDlx() {
        return BindingBuilder.bind(queueDlx()).to(exchangeDlx()).with("go.dlx").noargs();
    }
    
}
```

#### 2.4 应用类

```java
@RestController
public class GoController {
    @Autowired
    private AmqpTemplate rabbitTemplate;
    
    // 正常发送消息
    @RequestMapping("/go")
    public String distributeGo() {
        rabbitTemplate.convertAndSend("ex.go", "go", "送单到石景山x小区，请在10秒内接受任务");
        return "任务已经下发，等待送单。。。";
    }
    
    // 从死信队列中消费消息
    @RequestMapping("/notgo")
    public String getAccumulatedTask() {
        return (String)rabbitTemplate.receiveAndConvert("q.go.dlx");
    }
    
}
```

主启动类：

```java
@SpringBootApplication
public class RabbitmqDemo {
    public static void main(String[] args) {
    	SpringApplication.run(RabbitmqDemo.class, args);
    }
}
```

## 五.延迟队列

### 1.延迟队列概念

**延迟消息是指的消息发送出去后并不想立即就被消费，而是需要等（指定的）一段时间后才触发消费**。

业务场景：在支付宝上面买电影票，锁定了一个座位后系统默认会帮你保留15分钟时间，如果15分钟后还没付款那么不好意思系统会自动把座位释放掉。怎么实现类似的功能呢？

- **定时任务**：每分钟扫一次，发现有占座超过15分钟还没付款的就释放掉。但是这样做很低效，很多时候做的都是些无用功；
- **分布式锁、分布式缓存的被动过期时间**：15分钟过期后锁也释放了，缓存key也不存在了；
- **延迟队列**：锁座成功后会发送1条延迟消息，这条消息15分钟后才会被消费，消费的过程就是检查这个座位是否已经是“已付款”状态。

系统投递一条延迟消息，而这条消息比较特殊不会立马被消费，而是延迟到指定时间后再触发消费动作。在AMQP协议和RabbitMQ中都没有相关的规定和实现。不过，我们可以借助“死信队列”来变相实现。

### 2.延时交换器插件

我们也可以使用`rabbitmq_delayed_message_exchange`插件实现。这里和TTL方式有个很大的不同就是**TTL存放消息在死信队列(delayqueue)里，二基于插件存放消息在延时交换机里(x-delayed-message exchange)**。

![image-20201122155337083](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201122155337083.png)

1. 生产者将消息(msg)和路由键(routekey)发送指定的延时交换机(exchange)上；
2. 延时交换机(exchange)存储消息等待消息到期根据路由键(routekey)找到绑定自己的队列(queue)并把消息给它；
3. 队列(queue)再把消息发送给监听它的消费者(customer）。

插件安装步骤：

1. 下载插件：wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.8.0/rabbitmq_delayed_message_exchange-3.8.0.ez
2. 安装插件：将插件拷贝到rabbitmq-server的安装路径 `/usr/lib/rabbitmq/lib/rabbitmq_server-3.8.4/plugins`
3. 启动插件：`rabbitmq-plugins enable rabbitmq_delayed_message_exchange`
4. 重启rabbitmq-server：`systemctl restart rabbitmq-server`

### 3.SpringBoot案例

#### 3.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 3.2 application.properties

```properties
spring.application.name=delayedqueue
spring.rabbitmq.host=atzlg8
spring.rabbitmq.virtual-host=/
spring.rabbitmq.username=root
spring.rabbitmq.password=123456
spring.rabbitmq.port=5672
```

#### 3.3 RabbitConfig

```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue queue() {
        return new Queue("q.delayed", false, false, false, null);
    }
    // 自定义交换器:延迟交换器
    @Bean
    public Exchange exchange() {
        Map<String, Object> props = new HashMap<>();
        props.put("x-delayed-type", ExchangeTypes.FANOUT);
        return new CustomExchange("ex.delayed", "x-delayed-message",
                true, false, props);
    }
    @Bean
    public Binding binding() {
        return BindingBuilder.bind(queue()).to(exchange()).with("key.delayed").noargs();
    }
    
}
```

#### 3.4 监听类

使用推消息模式接收延迟队列的广播：

```java
@Component
public class MeetingListener {
    
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
    @RabbitListener(queues = "q.delayed")
    public void broadcastMeetingAlarm(Message message, Channel channel) throws IOException {
        
        System.err.println("提醒：5秒后：" + new String(message.getBody(),"utf-8"));
        channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
    }
}
```

#### 3.5 应用类

开发RestController，用于向延迟队列发送消息，并指定延迟的时长。

```java
@RestController
public class PublishController {
    
    @Autowired
    private AmqpTemplate rabbitTemplate;
    
    @RequestMapping("/prepare/{seconds}")
    public String toMeeting(@PathVariable Integer seconds) throws UnsupportedEncodingException {
        /**
         * RabbitMQ只会检查队列头部的消息是否过期，如果过期就放到死信队列
         * 假如第一个过期时间很长，10s，第二个消息3s，则系统先看第一个消息，等到第一个消息过期，放到DLX
         * 此时才会检查第二个消息，但实际上此时第二个消息早已经过期了，但是并没有先于第一个消息放到DLX。
         * 插件rabbitmq_delayed_message_exchange帮我们搞定这个。
         */
//        MessageProperties properties = new MessageProperties();
//        properties.setHeader("x-delay", (seconds - 10) * 1000);
//        Message message = new Message((seconds + "秒后召开销售部门会议。").getBytes("utf-8"), properties);
//        rabbitTemplate.convertAndSend("ex.delayed", "key.delayed", message);
    
        final Message message = new Message((seconds + "秒后召开销售部门会议。").getBytes("utf-8"), null);
        // 如果不设置message的properties，也可以使用下述方法设置x-delay属性的值
        rabbitTemplate.convertAndSend("ex.delayed", "key.delayed", message, msg -> {
            // 使用定制的属性x-delay设置过期时间，也就是提前5s提醒
            // 当消息转换完，设置消息头字段
            msg.getMessageProperties().setHeader("x-delay", (seconds - 5) * 1000);
            return msg;
        });
        
        return "已经定好闹钟了，到时提前告诉大家";
    }
}
```

主启动类：

```java
@SpringBootApplication
public class RabbitmqDemo {
    public static void main(String[] args) {
    	SpringApplication.run(RabbitmqDemo.class, args);
    }
}
```

结果：按照时长倒序发送请求，结果时间先到的先消费。例如依次发送 http://localhost:8080/prepare/45、再35，再25，其结果依次为：

```
提醒：5秒后：25秒后召开销售部门会议。
提醒：5秒后：35秒后召开销售部门会议。
提醒：5秒后：45秒后召开销售部门会议。
```

## 六.延迟支付案例

### 1.需求

基于RabbitMQ的TTL以及死信队列，使用SpringBoot实现延迟付款，手动补偿操作。

1、用户下单后展示等待付款页面

2、在页面上点击付款的按钮，如果不超时，则跳转到付款成功页面

3、如果超时，则跳转到用户历史账单中查看因付款超时而取消的订单。

### 2.代码实现

#### 2.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 2.2 application.properties

```properties
spring.application.name=delay-pay
spring.rabbitmq.host=atzlg8
spring.rabbitmq.port=5672
spring.rabbitmq.username=root
spring.rabbitmq.password=123456
spring.rabbitmq.virtual-host=/

# publisher confirm
spring.rabbitmq.publisher-confirm-type=correlated
spring.rabbitmq.publisher-returns=true

# Consumer ACK
# 是否开启消费者重试（false时关闭消费者重试，
# 意思不是“不重试”，而是一直收到消息直到ack确认或者一直到超时）
spring.rabbitmq.listener.simple.retry.enabled=true
# 最大重试次数
spring.rabbitmq.listener.simple.retry.max-attempts=5
#重试间隔时间（单位毫秒）
spring.rabbitmq.listener.simple.retry.initial-interval=5000
# 重试超过最大次数后是否拒绝
spring.rabbitmq.listener.simple.default-requeue-rejected=false
#ack模式
spring.rabbitmq.listener.simple.acknowledge-mode=manual
#设置并发线程数
spring.rabbitmq.listener.simple.concurrency=10
#设者最大并发线程数
spring.rabbitmq.listener.simple.max-concurrency=20
```

#### 2.3 配置类

```java
/**
 * @author zlg
 */
@Configuration
public class RabbitConfig {
    
    public static final String DEAD_LETTER_EXCHANGE = "ex.waiting.dlx";
    public static final String DEAD_LETTER_ROUTING_KEY = "key.waiting.dlx";
    public static final String DEAD_LETTER_QUEUE = "queue.waiting.dlx";
    public static final String PAY_EXCHANGE = "ex.waiting";
    public static final String PAY_ROUTING_KEY = "key.waiting";
    public static final String PAY_QUEUE = "queue.waiting";
    
    // 声明业务队列
    @Bean
    public Queue queue() {
        Map<String,Object> argus =  new HashMap<>();
        // 消息的所有生存时间
        argus.put("x-message-ttl", 10000);
        // 关联死信交换器
        argus.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE);
        // 关联死信交换器的routingkey
        argus.put("x-dead-letter-routing-key", DEAD_LETTER_ROUTING_KEY);
        return new Queue(PAY_QUEUE, true, false, false, argus);
    }
    // 死信队列
    @Bean
    public Queue queueDlx() {
        return new Queue(DEAD_LETTER_QUEUE, true, false, false, null);
    }
    
    // 声明业务交换器
    @Bean
    public Exchange exchanger() {
        return new DirectExchange(PAY_EXCHANGE, true, false, null);
    }
    // 死信交换器
    @Bean
    public Exchange exchangeDlx() {
        return new DirectExchange(DEAD_LETTER_EXCHANGE, true, false, null);
    }
    
    // 业务队列绑定到业务交换器
    @Bean
    public Binding binding() {
        return BindingBuilder.bind(queue()).to(exchanger()).with(PAY_ROUTING_KEY).noargs();
    }
    // 死信队列绑定到死信交换器
    @Bean
    public Binding bindingDlx() {
        return BindingBuilder.bind(queueDlx()).to(exchangeDlx()).with(DEAD_LETTER_ROUTING_KEY).noargs();
    }
    
    // rabbitmq监听容器工厂(推模式)
    /*@Bean
    public RabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
        // SimpleRabbitListenerContainerFactory发现消息中有content_type有text就会默认将其
        // 转换为String类型的，没有content_type都按byte[]类型
        final SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        // 消费端手动确认,也可以在配置文件中进行配置
        factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        // 设置并发线程数
        factory.setConcurrentConsumers(10);
        // 设者最大并发线程数
        factory.setMaxConcurrentConsumers(20);
        return factory;
    }*/
    
}
```

#### 2.4 监听类

```java
/**
 * @author zlg
 */
@Component
public class MyMessageListener {
    
    private static final Logger logger = LoggerFactory.getLogger(MyMessageListener.class);
    @Autowired
    private OrderService orderService;
    
    @RabbitListener(queues = RabbitConfig.DEAD_LETTER_QUEUE, ackMode = "MANUAL")
    public void handleDlxMessage(Message message, Channel channel) {
    
        // 从队列中取出订单号
        String orderId = new String(message.getBody(), StandardCharsets.UTF_8);
        logger.info("消费者接收到的订单号 【{}】", orderId);
        // 取消订单
        String flag = "cancle";
        orderService.updateOrderStatus(orderId, flag);
        logger.info("该订单【{}】已取消", orderId);

        try {
            // 手动ack，deliveryTag表示消息的唯一标志，multiple表示是否是批量确认
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
//            System.err.println("已确认消息：" + message);
            logger.info("已确认消息, {}", orderId);
        } catch (IOException e) {
            e.printStackTrace();
        }
    
    }
}
```

#### 2.5 应用类

```java
/**
 * @author zlg
 */
@RestController
public class PayController {
    
    private static final Logger logger = LoggerFactory.getLogger(PayController.class);
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private OrderService orderService;
    
    // 创建订单接口
    @RequestMapping("/order/create")
    public String order() {
        String orderId = UUID.randomUUID().toString();
        orderService.createOrder(orderId);
        // 发送延迟等待消息
        rabbitTemplate.convertAndSend(RabbitConfig.PAY_EXCHANGE, RabbitConfig.PAY_ROUTING_KEY, orderId);
        return "该订单创建成功，待支付，订单号为：" + orderId;
    }
    
    // 支付接口
    @RequestMapping("/order/pay/{orderId}")
    public String pay(@PathVariable String orderId) {
        final OrderInfo orderInfo = orderService.findOrderByOrderId(orderId);
        if (orderInfo.getOrderStatus().equals(OrderStatusEnum.HAS_CANCEL.getFlag())) {
            logger.info("该订单【{}】已经取消", orderId);
            final List orderList = list();
            return "该订单已经取消，历史订单表记录为：\n" + Arrays.toString(orderList.toArray());
        }
        String message = rabbitTemplate.execute(new ChannelCallback<String>() {
            @Override
            public String doInRabbit(Channel channel) throws Exception {
                // 主动从业务队列中拉取消息
                final GetResponse getResponse = channel.basicGet(RabbitConfig.PAY_QUEUE, false);
                if (getResponse == null) {
                    logger.info("该消息已经消费");
                    return "你已消费完所有的消息";
                }
                String message = new String(getResponse.getBody(), StandardCharsets.UTF_8);
                channel.basicAck(getResponse.getEnvelope().getDeliveryTag(), false);
                logger.info("该消息【{}】已确认", message);
                return message;
            }
        });
        if (!message.equals(orderId)) {
            return message;
        }
        String flag = "pay";
        orderService.updateOrderStatus(orderId, flag);
        return "该订单成功支付，订单号为：" + message;
    }
    
    // 订单列表
    @RequestMapping("/order/list")
    public List list() {
        return orderService.findAll();
    }

}
```

#### 2.6 业务类

```java
/**
 * @author zlg
 */
@Service
public class OrderServiceImpl implements OrderService {
    
//    @Autowired
//    private PlatformTransactionManager transactionManager;
    /**
     * 模拟数据库
     */
    private static volatile List<OrderInfo> orderList = new ArrayList<>();
    
    // 根据orderId修改订单状态
    @Override
    public void updateOrderStatus(final String orderId, String flag) {
//        final TransactionStatus status  = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            // 业务代码
            orderList.stream().filter(orderInfo -> {
                if (orderId.equals(orderInfo.getOrderId())) {
                    if ("cancle".equals(flag)) {
                        orderInfo.setOrderStatus(OrderStatusEnum.HAS_CANCEL.getFlag());
                    }else if ("pay".equals(flag)) {
                        orderInfo.setOrderStatus(OrderStatusEnum.HAS_PAY.getFlag());
                    }
                }
                return true;
            }).collect(Collectors.toList());
//            transactionManager.commit(status);
        } catch (Exception e) {
            e.printStackTrace();
//            transactionManager.rollback(status);
        }
    }
    
    // 创建订单
    @Override
    public void createOrder(final String orderId) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
        final OrderInfo orderInfo = new OrderInfo();
        orderInfo.setSeq(formatter.format(LocalDateTime.now()) + orderId);
        orderInfo.setOrderId(orderId);
        orderInfo.setOrderStatus(OrderStatusEnum.WAIT_PAY.getFlag());
        orderInfo.setOrderName("Jack");
        orderInfo.setCreateDate(LocalDateTime.now().toString());
        orderInfo.setOrderAmt("600");
        orderList.add(orderInfo);
    }
    
    // 根据订单号查询订单
    @Override
    public OrderInfo findOrderByOrderId(final String orderId) {
        for (final OrderInfo orderInfo : orderList) {
            if (orderId.equals(orderInfo.getOrderId())) {
                return orderInfo;
            }
        }
        return null;
    }
    
    @Override
    public List findAll() {
        return orderList;
    }
}
```

#### 2.7 实体类

```java
/**
 * @author zlg
 */
public class OrderInfo implements Serializable {
    
    private String seq;
    private String orderId;
    private String orderStatus;
    private String orderName;
    private String createDate;
    private String orderAmt;
    // ......
}    
```

枚举类：

```java
/**
 * @author zlg
 */
public enum OrderStatusEnum {
    
    WAIT_PAY("0", "未支付"),HAS_PAY("1", "已支付"),HAS_CANCEL("2", "已取消");
    
    private String flag;
    private String statusInfo;
    
    OrderStatusEnum(final String flag, final String statusInfo) {
        this.flag = flag;
        this.statusInfo = statusInfo;
    }
    
    public String getFlag() {
        return flag;
    }
    
    public void setFlag(final String flag) {
        this.flag = flag;
    }
    
    public String getStatusInfo() {
        return statusInfo;
    }
    
    public void setStatusInfo(final String statusInfo) {
        this.statusInfo = statusInfo;
    }
    
}
```

主启动类：

```java
@SpringBootApplication
public class RabbitmqDelaypayApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(RabbitmqDelaypayApplication.class, args);
    }
    
}
```

结果演示：

![image-20201123005309071](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005309071.png)

创建订单接口：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005550920.png" alt="image-20201123005550920" style="zoom:50%;" />

TTL时间内支付订单接口：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005612543.png" alt="image-20201123005612543" style="zoom:50%;" />

TTL时间内重复消费时结果：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005645985.png" alt="image-20201123005645985" style="zoom:50%;" />

TTL过期后结果：

![image-20201123005851804](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005851804.png)



