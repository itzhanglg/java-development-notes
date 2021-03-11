### 需求

基于RabbitMQ的TTL以及死信队列，使用SpringBoot实现延迟付款，手动补偿操作。

1、用户下单后展示等待付款页面

2、在页面上点击付款的按钮，如果不超时，则跳转到付款成功页面

3、如果超时，则跳转到用户历史账单中查看因付款超时而取消的订单。

### 代码实现

#### 1.引入依赖

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

#### 2.application.properties

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

#### 3.配置类

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

#### 4.监听类

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

#### 5.应用类

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

#### 6.业务类

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

#### 7.实体类

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

演示：

![image-20201123005309071](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005309071.png)

创建订单：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005550920.png" alt="image-20201123005550920" style="zoom:50%;" />

TTL时间内支付订单：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005612543.png" alt="image-20201123005612543" style="zoom:50%;" />

TTL时间内重复消费时：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005645985.png" alt="image-20201123005645985" style="zoom:50%;" />

TTL过期后：

![image-20201123005851804](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201123005851804.png)



