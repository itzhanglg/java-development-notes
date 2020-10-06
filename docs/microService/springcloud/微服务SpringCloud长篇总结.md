
### 1.微服务概述

#### 1.1 技术维度理解

微服务化的核心就是将传统的一站式应用，根据业务**拆分**成一个一个的服务，彻底地去耦合,每一个微服务提供单个业务功能的服务，一个服务做一件事，从技术角度看就是一种小而独立的处理过程，类似**进程**概念，能够自行单独启动或销毁，拥有自己**独立的数据库**。

-   单机系统(All In One)：只有一个大的工程  -- war包
-   分布式系统：
    -   各自模块/服务，各自独立出来，分灶吃饭  -- 拆分
    -   各自微小的一个进程，让专业的人专业的模块，来做专业的事情  -- 各自独立的进程
    -   独立部署  -- 拥有自己独立的数据库

#### 1.2 微服务架构

业界大牛马丁.福勒（Martin Fowler） 这样描述微服务：论文网址: [https://martinfowler.com/articles/microservices.html](https://martinfowler.com/articles/microservices.html) ,强调的是服务的大小，它关注的是**某一个点**，是具体解决某一个问题/提供落地对应服务的一个服务应用.

微服务架构是⼀种架构模式，它提倡**将单⼀应⽤程序划分成⼀组⼩的服务**，服务之间互相协调、互相配合，为⽤户提供最终价值。**每个服务运⾏在其独⽴的进程中**，服务与服务间采⽤轻量级的通信机制互相协作（通常是基于**HTTP协议的RESTful API**）。每个服务都围绕着具体业务进⾏构建，并且能够被独⽴的部署到⽣产环境、类⽣产环境等。另外，应当尽量**避免统⼀的、集中式的服务管理机制**，对具体的⼀个服务⽽⾔，应根据业务上下⽂，选择合适的语⾔、⼯具对其进⾏构建。

#### 1.3 微服务优缺点

优点:

-   开发简单、开发效率提高，一个服务可能就是专一的只干一件事
-   微服务能够被小团队单独开发，这个小团队是2到5人的开发人员组成
-   微服务能使用不同的语言开发
-   微服务允许容易且灵活的方式集成自动部署，通过持续集成工具，如Jenkins，Hudson，bamboo
-   微服务只是业务逻辑的代码，不会和HTML，CSS或其它界面组件混合
-   每个微服务都有自己的存储能力，可以有自己的数据库。也可以有统一数据库

缺点:

-   开发人员要处理分布式系统的复杂性
-   系统部署依赖
-   服务间通信成本
-   数据一致性
-   系统集成测试...

#### 1.4 微服务技术栈有哪些

| 微服务条目                    | 落地技术                                     |
| :----------------------- | :--------------------------------------- |
| 服务开发                     | Springboot、Spring、SpringMVC              |
| 服务配置与管理                  | Netflix公司的Archaius、阿里的Diamond等           |
| **服务注册与发现**              | **Eureka**、Consul、Zookeeper等             |
| **服务调用**                 | **Rest**、RPC、gRPC                        |
| **服务熔断器**                | **Hystrix**、Envoy等                       |
| **负载均衡**                 | **Ribbon**、Nginx等                        |
| **服务接口调用**(客户端调用服务的简化工具) | **Feign**等                               |
| 消息队列                     | Kafka、RabbitMQ、ActiveMQ等                 |
| 服务配置中心管理                 | SpringCloudConfig、Chef等                  |
| **服务路由**（API网关）          | **Zuul**等                                |
| 服务监控                     | Zabbix、Nagios、Metrics、Spectator等         |
| 全链路追踪                    | Zipkin，Brave、Dapper等                     |
| 服务部署                     | Docker、OpenStack、Kubernetes等             |
| 数据流操作开发包                 | SpringCloud Stream（封装与Redis,Rabbit、Kafka等发送接收消息） |
| 事件消息总线                   | Spring Cloud Bus                         |
| ......                   |                                          |

### 2.springcloud概述

#### 2.1 springcloud网址

官网：[http://projects.spring.io/spring-cloud/](http://projects.spring.io/spring-cloud/)
参考书: [https://springcloud.cc/spring-cloud-netflix.html](https://springcloud.cc/spring-cloud-netflix.html)
API：[https://springcloud.cc/spring-cloud-dalston.html](https://springcloud.cc/spring-cloud-dalston.html)
springcloud中国社区：[http://springcloud.cn/](http://springcloud.cn/)
springcloud中文网：[https://springcloud.cc/](https://springcloud.cc/)

springcloud技术的五大神兽: Eureka, Ribbon/Feign, Hystrix, Zuul, SpringCloudConfig

#### 2.2 官网描述

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200301221943600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

SpringCloud，基于SpringBoot提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

SpringBoot并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，**最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包**。

**SpringCloud: 分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体，俗称微服务全家桶。**

#### 2.3 springcloud与springboot关系

Spring Boot 是 Spring 的一套快速配置脚手架，可以基于Spring Boot 快速开发单个微服务，Spring Cloud是一个基于Spring Boot实现的云应用开发工具；**Spring Boot专注于快速、方便集成的单个个体，Spring Cloud是关注全局的服务治理框架；**Spring Boot使用了默认大于配置的理念，很多集成方案已经帮你选择好了，能不配置就不配置，Spring Cloud很大的一部分是基于Spring Boot来实现,可以不基于Spring Boot吗？不可以。

**Spring Boot可以离开Spring Cloud独立使用开发项目，但是Spring Cloud离不开Spring Boot，属于依赖的关系。**

>   spring -> spring boot > Spring Cloud 这样的关系。

#### 2.4 springcloud与dubbo区别

社区活跃度：dubbo: [http://github.com/dubbo ](http://github.com/dubbo ) springcloud: [http://github.com/spring-cloud](http://github.com/spring-cloud)

老系统用Dubbo多，新系统用SpringCloud多；

SpringCloud抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式。

### 3.Eureka服务注册与发现

#### 3.1 eureka是什么

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现**服务注册和发现**。

Eureka 采用了 **C-S 的设计架构**。Eureka Server 作为服务注册功能的服务器，它是服务注册中心。

而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。SpringCloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。

#### 3.2 原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200301221959359.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)



**Eureka Server** 提供服务注册和发现, **Service Provider** 服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到, **Service Consumer** 服务消费方从Eureka获取注册服务列表，从而能够消费服务.

Eureka包含两个组件：Eureka Server和Eureka Client。

-   **Eureka Server** 提供服务注册。各个节点启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。
-   **EurekaClient** 是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）。

#### 3.3 eureka自我保护

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，EurekaServer就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。

在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时，该Eureka Server节点就会自动退出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话讲解：**好死不如赖活着**

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

在Spring Cloud中，可以使用`eureka.server.enable-self-preservation = false`  禁用自我保护模式。

**某时刻某一个微服务不可用了，eureka不会立刻清理，依旧会对该微服务的信息进行保存.**

#### 3.4 ACID概念与CAP图

**传统的ACID概念**:

-   A(Atomicity)原子性: 事务里的所有操作要么全部做完,要么都不做,事务成功的条件是事务里的所有操作都成功,只要有一个操作失败整个事务就失败,需要回滚。
-   C(Consistency)一致性: 数据库要一直处于一致的状态,事务的运行不会改变数据库原本的一致性约束。
-   I(Isolation)独立性: 并发的事务之间不会互相影响,如果一个事务要访问的数据正在被另外一个事务修改,只要另外一个事务未提交,它所访问的数据就不受未提交事务的影响。
-   D(Durability)持久性: 指一旦事务提交后，他所做的修改将永久的保存在数据库上，即使出现宕机也不会丢失。

**CAP图:**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200301222014698.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

CAP理论的核心:一个分布式系统不可能同时很好的满足一致性,可用性和分区容错性这三个需求,因此,根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类:

-   CA-单点集群,满足一致性,可用性的系统,通常在可扩展性上不太强大。
-   CP-满足一致性,分区容错性的系统,通常性能不是特别高。
-   AP-满足可用性,分区容错性的系统,通常可能对一致性要求低一些。

CAP理论就是说在分布式存储系统中,最多只能实现上面的两点。而由于当前的网络硬件肯定会出现延迟丢包等问题,所以**分区容错性是我们必须需要实现的**。所以我们只能在**一致性和可用性**之间进行权衡,没有NoSQL系统能同时保证这三点。

#### 3.5 eureka比zookeeper好在哪里

著名的CAP理论指出,一个分布式系统不可能同时满足C(一致性)、A(可用性)和P(分区容错性)。由于分区容错性P在是分布式系统中必须要保证的,因此我们只能在A和C之间进行权衡。因此 **Zookeeper保证的是CP，Eureka则是AP。**

**Zookeeper保证CP:**

当向注册中心查询服务列表时,我们可以容忍注册中心返回的是几分钟以前的注册信息,但不能接受服务直接down掉不可用。也就是说,服务注册功能对可用性的要求要高于一致性。但是zk会出现这样一种情况,当master节点因为网络故障与其他节点失去联系时,剩余节点会重新进行leader选举。问题在于,选举leader的时间太长, 30 ~ 120s,且选举期间整个zk集群都是不可用的,这就导致在选举期间注册服务瘫痪。在云部署的环境下,因网络问题使得zk集群失去master节点是较大概率会发生的事,虽然服务能够最终恢复,但是漫长的选举时间导致的注册长期不可用是不能容忍的。

**Eureka保证AP:**

Eureka看明白了这一点,因此在设计时就优先保证可用性。Eureka各个节点都是平等的,几个节点挂掉不会影响正常节点的工作 ,剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或时如果发现连接失败,则会自动切换至其它节点,只要有一台Eureka还在,就能保证注册服务可用(保证可用性),只不过查到的信息可能不是最新的(不保证强一致性)。除此之外, Eureka还有一种自我保护机制,如果在15分钟内超过85%的节点都没有正常的心跳,那么Eureka就认为客户端与注册中心出现了网络故障,此时会出现以下几种情况:

1.  Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
2.  Eureka仍然能够接受新服务的注册和查询请求,但是不会被同步到其它节点上(即保证当前节点依然可用)
3.  当网络稳定时,当前实例新的注册信息会被同步到其它节点中

因此, **Eureka可以很好的应对因网络故障导致部分节点失去联系的情况,而不会像zookeeper那样使整个注册服务瘫痪。**

#### 3.6 相关配置信息

Eureka服务器:

```xml
<!--eureka-server服务端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

```yaml
server:
    port: 7001
eureka:
    instance:
        hostname: localhost  #eureka服务端的实例名称
    client:
        register-with-eureka: false  #false表示不向注册中心注册自己
        fetch-registry: false  #false表示自己端就是注册中心
        service-url:  #单机  #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
            defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

```java
@SpringBootApplication
@EnableEurekaServer  //EurekaServer服务器端启动类，接受其它微服务注册进来
public class EurekaServer7001_App{
  public static void main(String[] args){
    SpringApplication.run(EurekaServer7001_App.class, args);
  }
}
```

微服务提供者:

```xml
<!-- 将微服务provider者注册进eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

```yaml
server:
  port: 8001
  
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.transsion.springcloud.entities    # 所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml                       # mapper映射文件
    
spring:
  application:
    name: microservicecloud-dept 
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01              # 数据库名称
    username: root
    password: root
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间
      
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
#      defaultZone: http://localhost:7001/eureka/
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7004/eureka/,http://eureka7003.com:7003/eureka/      
  instance:
    instance-id: microservicecloud-dept8001   #自定义服务名称信息
    prefer-ip-address: true     #访问路径可以显示IP地址     
```

```java
@SpringBootApplication
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
@EnableDiscoveryClient //服务发现
public class DeptProvider8001_App
{
	public static void main(String[] args)
	{
		SpringApplication.run(DeptProvider8001_App.class, args);
	}
}
```

微服务消费者:

```yaml
eureka:
  client:
    register-with-eureka: false
    service-url: 
#      defaultZone: http://localhost:7001/eureka/
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7004/eureka/,http://eureka7003.com:7003/eureka/  
```

```java
@SpringBootApplication
@EnableEurekaClient
public class DeptConsumer80_App
{
	public static void main(String[] args)
	{
		SpringApplication.run(DeptConsumer80_App.class, args);
	}
}
```

### 4.Ribbon负载均衡

#### 4.1 概述

官网: [https://github.com/Netflix/ribbon/wiki/Getting-Started](https://github.com/Netflix/ribbon/wiki/Getting-Started)

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端 负载均衡**的工具。

Ribbon是Netflix发布的开源项目，主要功能是**提供客户端的软件负载均衡算法**，将Netflix的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。

#### 4.2 负载均衡

LB即负载均衡(Load Balance)，在微服务或分布式集群中经常用的一种应用。负载均衡简单的说就是**将用户的请求平摊的分配到多个服务上，从而达到系统的HA。**常见的负载均衡有软件Nginx，LVS，硬件 F5等。

**集中式LB：**在**服务的消费方和提供方之间使用独立的LB设施**(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；

**进程内LB：**将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

**Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。**

#### 4.3 ribbon负载均衡

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200301222159354.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

Ribbon在工作时分成两步: 1.**先选择 EurekaServer** ,它优先选择在同一个区域内负载较少的server. 2.**再根据用户指定的策略**，在从server取到的服务注册列表中选择一个地址。其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。

**Ribbon其实就是一个软负载均衡的客户端组件，他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。**

#### 4.4 ribbon核心组件IRule

IRule: 根据特定算法中从服务列表中选取一个要访问的服务.官方源码: [https://github.com/Netflix/ribbon/tree/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer](https://github.com/Netflix/ribbon/tree/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer)

-   **RoundRobinRule**: 轮询
-   **RandomRule**: 随机
-   **AvailabilityFilteringRule**: 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,还有并发的连接数量超过阈值的服务,然后对剩余的服务列表按照轮询策略进行访问
-   **WeightedResponseTimeRule**: 根据平均响应时间计算所有服务的权重,响应时间越快服务权重越大被选中的概率越高。刚启动时如果统计信息不足,则使用RoundRobinRule策略,等统计信息足够,会切换到WeightedResponseTimeRule
-   **RetryRule**: 先按照RoundRobinRule的策略获取服务,如果获取服务失败则在指定时间内会进行重试,获取可用的服务
-   **BestAvailableRule**: 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,然后选择一个并发量最小的服务
-   **ZoneAvoidanceRule**: 默认规则,复合判断server所在区域的性能和server的可用性选择服务器

#### 4.5 相关配置信息

```xml
<!-- Ribbon相关 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaClient
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效
@RibbonClient
public class DeptConsumer80_App
{
	public static void main(String[] args)
	{
		SpringApplication.run(DeptConsumer80_App.class, args);
	}
}
```

```java
//示例
@Configuration
public class ConfigBean{
  //指定ribbon给出的访问策略
  //RandomRule extends AbstractLoadBalancerRule; AbstractLoadBalancerRule implements IRule
  @Bean
  public IRule myRule() 
  {
    //return new RoundRobinRule();
    return new RandomRule();//达到的目的，用我们重新选择的随机算法替代默认的轮询。
    //return new RetryRule();
  }
}
```

### 5.Feign负载均衡

#### 5.1 spring官网解释

网址: [http://projects.spring.io/spring-cloud/spring-cloud.html#spring-cloud-feign](http://projects.spring.io/spring-cloud/spring-cloud.html#spring-cloud-feign)

Feign是一个声明式WebService客户端。使用Feign能让编写Web Service客户端更加简单, 它的使用方法是定义一个接口，然后在上面添加注解，同时也支持JAX-RS标准的注解。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

**Feign是一个声明式的Web服务客户端，使得编写Web服务客户端变得非常容易，只需要创建一个接口，然后在上面添加注解即可。**

#### 5.2 开源项目解释

网址: [https://github.com/OpenFeign/feign](https://github.com/OpenFeign/feign)

**Feign能干什么**?Feign旨在使编写Java Http客户端变得更容易。

在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，**往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用**。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，**我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)**，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

**Feign集成了Ribbon**:

利用Ribbon维护了微服务的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，**通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用**.

**总结: Feign通过接口的方法调用Rest服务（之前是Ribbon+RestTemplate），该请求发送给Eureka服务器,通过Feign直接找到服务接口，由于在进行服务调用的时候融合了Ribbon技术，所以也支持负载均衡 (采用ribbon的默认轮询策略)作用。**

#### 5.3 相关配置信息

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

```java
//新建service包
//新增注解@FeignClient,与主启动类的@EnableFeignClients对应
@FeignClient(value = "MICROSERVICECLOUD-DEPT")
public interface DeptClientService
{
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list();

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept);
}
```

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients(basePackages= {"com.transsion.springcloud"})
@ComponentScan("com.transsion.springcloud")
public class DeptConsumer80_Feign_App
{
    public static void main(String[] args)
    {
        SpringApplication.run(DeptConsumer80_Feign_App.class, args);
    }
}
```

### 6.Hystrix断路器

#### 6.1 hystrix概述

官网:  [https://github.com/Netflix/Hystrix/wiki/How-To-Use](https://github.com/Netflix/Hystrix/wiki/How-To-Use)

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“**扇出**”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“**雪崩效应**”.

Hystrix是一个用于处理分布式系统的**延迟和容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。**

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），**向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

Hystrix可以 服务熔断, 服务降级, 服务限流, 接近实时监控等...

```xml
<!-- hystrix -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

#### 6.2 服务熔断

服务熔断: 熔断机制是应对雪崩效应的一种微服务链路保护机制。

当扇出链路的某个微服务不可用或者响应时间太长时，**会进行服务的降级，进而熔断该节点微服务的调用，快速返回"错误"的响应信息。**当检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败就会启动熔断机制。**熔断机制的注解是@HystrixCommand。**

```java
@RestController
public class DeptController
{
    @Autowired
    private DeptService service = null;

    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    //一旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法
    @HystrixCommand(fallbackMethod = "processHystrix_Get")
    public Dept get(@PathVariable("id") Long id)
    {
        Dept dept = this.service.get(id);       
        if (null == dept) {
            throw new RuntimeException("该ID：" + id + "没有没有对应的信息");
        }       
        return dept;
    }

    public Dept processHystrix_Get(@PathVariable("id") Long id)
    {
        return new Dept().setDeptno(id).setDname("该ID：" + id + "没有没有对应的信息,null--@HystrixCommand").setDb_source("no this database in MySQL");
    }
}

@SpringBootApplication
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
@EnableDiscoveryClient //服务发现
@EnableCircuitBreaker//对hystrixR熔断机制的支持
public class DeptProvider8001_Hystrix_App
{
    public static void main(String[] args)
    {
        SpringApplication.run(DeptProvider8001_Hystrix_App.class, args);
    }
}
```

#### 6.3 服务降级

**整体资源快不够了，忍痛将某些服务先关掉，待渡过难关，再开启回来. 服务降级处理是在客户端实现完成的，与服务端没有关系.**

```java
//@FeignClient(value = "MICROSERVICECLOUD-DEPT")
@FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory = DeptClientServiceFallbackFactory.class)
public interface DeptClientService
{
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list();

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept);
}

@Component // 不要忘记添加，不要忘记添加
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService>
{
    @Override
    public DeptClientService create(Throwable throwable)
    {
        return new DeptClientService() {
            @Override
            public Dept get(long id)
            {
                return new Dept().setDeptno(id).setDname("该ID：" + id + "没有没有对应的信息,Consumer客户端提供的降级信息,此刻服务Provider已经关闭")
                        .setDb_source("no this database in MySQL");
            }

            @Override
            public List<Dept> list()
            {
                return null;
            }

            @Override
            public boolean add(Dept dept)
            {
                return false;
            }
        };
    }
}
```

```yaml
# 开启服务降级
feign: 
  hystrix: 
    enabled: true
```

#### 6.4 服务熔断降级总结

【服务熔断】:一般是**某个服务故障或者异常**引起,类似现实世界中的“**保险丝**“,当某个异常条件被触发,直接**熔断整个服务**,而不是一直等到此服务超时。

【服务降级】:所谓降级,一般是从**整体负荷**考虑。就是当某个服务熔断之后,**服务器将不再被调用**,此时客户端可以自己准备**一个本地的fallback回调,返回一个default值**。这样做,虽然服务水平下降,但好歹可用,比直接挂掉要强。

#### 6.5 服务监控

除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用监控（Hystrix Dashboard），Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。Spring Cloud也提供了Hystrix Dashboard的整合，对监控内容转化成可视化界面。

监控地址:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200301222317219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

1: Delay:该参数用来控制服务器上轮询监控信息的延迟时间,默认为2000毫秒,可以通过配置该属性来降低客户端的网络和CPU消耗。2: Title:该参数对应了头部标题Hystrix Stream之后的内容,默认会使用具体监控实例的URL,可以通过配置该信息来展示更合适的标题。

监控结果:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200301222331446.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

看图: 7色1圈1线.

**1圈**: 实心圆共有两种含义。它通过**颜色的变化代表了实例的健康程度**，它的健康度从**绿色<黄色<橙色<红色递减**。
该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，**流量越大该实心圆就越大**。所以通过该实心圆的展示，就可以在大量的实例中快速的发现故障实例和高压力实例。

**1线**: 曲线用来记录**2分钟内流量的相对变化**，可以通过它来观察到流量的上升和下降趋势。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200301222345749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

```xml
<!-- hystrix和 hystrix-dashboard相关 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
<!-- actuator监控信息完善,微服务提供者都添加监控依赖配置 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableHystrixDashboard     // 开启HystrixDashboard监控
public class DeptConsumer_DashBoard_App
{
    public static void main(String[] args)
    {
        SpringApplication.run(DeptConsumer_DashBoard_App.class, args);
    }
}
```

### 7.Zuul路由网关

#### 7.1 zuul概述

官网: [https://github.com/Netflix/zuul/wiki/Getting-Started](https://github.com/Netflix/zuul/wiki/Getting-Started)

Zuul包含了对请求的**路由和过滤**两个最主要的功能：

-   路由功能: 负责**将外部请求转发到具体的微服务实例**上，是实现外部访问统一入口的基础.
-   过滤器功能: 负责**对请求的处理过程进行干预**，是实现请求校验、服务聚合等功能的基础.

**Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。**

**注意：Zuul服务最终还是会注册进Eureka, 提供 代理+路由+过滤 三大功能.**

#### 7.2 相关配置

`@EnableZuulProxy` : 开启zuul代理

```yaml
# before    http://myzuul.com:9527/microservicecloud-dept/dept/get/2
    
zuul:
  prefix: /transsion      # 统一公共前缀
#  ignored-services: microservicecloud-dept  # 配置单个微服务名称失效,原映射路径失效
  ignored-services: "*"     # 配置多个微服务名称失效
  routes: 	# 配置路由访问映射规则
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
 
# after     http://myzuul.com:9527/mydept/dept/get/1
```

### 8.SpringCloudConfig分布式配置中心

#### 8.1 springcloudconfig概述

微服务意味着要将单体应用中的业务拆分成一个个子服务,每个服务的粒度相对较小,因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行,所以**一套集中式的、动态的配置管理设施**是必不可少的。**SpringCloud提供了 ConfigServer来解决这个问题,我们每一个微服务自己带着一个application.yml,上百个配置文件的管理就比较麻烦了**...

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200301222410319.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持,**配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置.**

SpringCloud Config分为**服务端和客户端**两部分:

-   **服务端**也称为**分布式配置中心,它是一个独立的微服务应用**,用来连接配置服务器并为户端提供获取配置信息,加密/解密信息等访问接口.
-   **客户端**则是通过指定的配置中心来管理应用资源,以及与业务相关的配置内容,并在启动的时候从配置中心获取和加载配置信息,配置服务器默认采用git来存储配置信息,这样就有助于对环境配置进行版本管理,并且可以通过git客户端工具来方便的管理和访问配置内容。

**springcloudconfig能做什么**:

-   集中管理配置文件
-   不同环境不同配置,动态化的配置更新,分环境部署比如dev/test/prod/beta/release
-   运行期间动态调整配置,不再需要在每个服务部署的机器上编写配置文件,服务会向配置中心统一拉取配置自己的信息
-   当配置发生变动时,服务不需要重启即可感知到配置的变化并应用新的配置
-   将配置信息以REST接口的形式暴露

springcloudconfig与github整合配置:

SpringCloud Config默认使用Git来存储配置文件(也有其它方式,比如支持SVN和本地文件),但最推荐的还是Git,而且使用的是http/https访问的形式.

#### 8.2 相关配置信息
```xml
<!-- springCloud Config -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
</dependency>
<!-- 避免Config的Git插件报错：org/eclipse/jgit/api/TransportConfigCallback -->
<dependency>
	<groupId>org.eclipse.jgit</groupId>
	<artifactId>org.eclipse.jgit</artifactId>
	<version>4.10.0.201712302008-r</version>
</dependency>
<!-- 图形化监控 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

`@EnableConfigServer` : 启动config配置

```yaml
spring:
  application:
    name:  microservicecloud-config  #仓库名称
  cloud:
    config:
      server:
        git:
          uri: https://github.com/itzhanglg/microservicecloud-config.git #GitHub上面的git仓库名字
```

配置读取规则:

- `/{application}-{profile}.yml` : `http://config-3344.com:3344/application-dev.yml` 
- `/{application}/{profile}[/{label}]` : `http://config-3344.com:3344/application/dev/master` 
- `/{label}/{application}-{profile}.yml` : `http://config-3344.com:3344/master/application-dev.yml` 

bootstrap.yml:

applicaiton.yml是用户级的资源配置项, bootstrap.yml是系统级的, 优先级更加高.
Spring Cloud会创建一个Bootstrap Context,作为Spring应用的Application Context的父上下文。初始化的时候 **Bootstrap Context负责从外部源加载配置属性并解析配置**这两个上下文共享一个从外部获取的Environment. **Bootstrap属性有高优先级,默认情况下,它们不会被本地配置覆盖**。Bootstrap context和Application Context有着不同的约定.所以新增了一个bootstrap.yml文件,保证Bootstrap Context和Application Context配置的分离。

```yaml
spring:
  cloud:
    config:
      name: microservicecloud-config-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: test   #本次访问的配置项
      label: master   
      uri: http://config-3344.com:3344  #本微服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址
```




