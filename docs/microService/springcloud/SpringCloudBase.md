### 一.微服务

#### 1.什么是微服务？微服务之间如何独立通讯？

什么是微服务？微服务概念起源:  [Microservices](https://martinfowler.com/articles/microservices.html)

微服务是用一组小服务的方式来构建一个应用，服务独立运行在不同的进程中，服务之间通过轻量的通讯机制（如RESTful接口）来交互，并且服务可以通过自动化部署方式独立部署。正因为微服务架构中，服务之间是相互独立的，所以不同的服务可以使用不同的语言来开发，或者根据业务的需求使用不同类型的数据库。

- 微服务架构是一个分布式系统, 按照业务进行划分成为不同的服务单元, 解决单体系统性能等不足。
- 微服务是一种架构风格，一个大型软件应用由多个服务单元组成。系统中的服务单元可以单独部署，各个服务单元之间是松耦合的。

微服务之间如何独立通讯？

**同步**：

**REST HTTP协议**：REST 请求在微服务中是最为常用的一种通讯方式, 它依赖于 HTTP\HTTPS 协议。RESTFUL 的特点是：

- 每一个 URI 代表 1 种资源 
- 客户端使用 GET、POST、PUT、DELETE 4 个表示操作方式的动词对服务端资源进行操作: GET 用来获取资源, POST 用来新建资源(也可以用于更新资源), PUT 用来更新资源, DELETE 用来删除资源 
- 通过操作资源的表现形式来操作资源，资源的表现形式是 XML 或者 HTML 
- 客户端与服务端之间的交互在请求之间是无状态的,从客户端到服务端的每个请求都必须包含理解请求所必需的信息

**RPC TCP协议**：RPC(Remote Procedure Call)远程过程调用，简单的理解是一个节点请求另一个节点提供的服务。它的工作流程是这样的：

1.执行客户端调用语句，传送参数 2. 调用本地系统发送网络消息 3. 消息传送到远程主机 4. 服务器得到消息并取得参数 5. 根据调用请求以及参数执行远程过程（服务） 6. 执行过程完毕，将结果返回服务器句柄 7. 服务器句柄返回结果，调用远程主机的系统网络服务发送结果 8. 消息传回本地主机 9. 客户端句柄由本地主机的网络服务接收消息 10. 客户端接收到调用语句返回的结果数据

**异步**：消息中间件

常见的消息中间件有 Kafka、RabbitMQ、RocketMQ、ActiveMQ , 常见的协议有AMQP、MQTTP、STOMP、XMPP。

#### 2.微服务技术栈有哪些？

- 微服务开发：快速开发服务。[Spring](https://spring.io/)、SpringMVC、[SpringBoot](https://spring.io/projects/spring-boot)
- 微服务调用协议：REST、RPC（Remote Procedure Call）、RMI（Remote Method Invocation）
- 微服务**注册发现**：发现服务，注册服务，集中管理服务。[Eureka](https://github.com/Netflix/eureka)、[Zookeeper](https://github.com/apache/zookeeper)、[Nacos](https://nacos.io/zh-cn/)、[consul](https://github.com/hashicorp/consul)
- 微服务**配置管理**：统一管理一个或多个服务的配置信息, 集中管理。[Disconf](https://github.com/knightliao/disconf)、[SpringCloudConfig](https://github.com/spring-cloud/spring-cloud-config)、[Apollo](https://github.com/ctripcorp/apollo)
- 服务**负载均衡**：[Ribbon](https://github.com/Netflix/ribbon)（客户端）、[Nginx](https://github.com/nginx/nginx)（服务端）
- **服务熔断**：当请求到达一定阈值时不让请求继续。[Hystrix](https://github.com/Netflix/Hystrix)、[Sentinel](https://github.com/alibaba/Sentinel)
- 服务**接口调用**：多个服务之间的通讯。[Feign(HTTP)](https://github.com/OpenFeign/feign)（HTTP）、[Apache HttpClient](https://hc.apache.org/httpcomponents-client-ga/)、Spring RestTemplate
- **API网关**：外部请求通过 API 网关进行拦截处理, 再转发到真正的服务。[Spring Cloud GateWay](https://spring.io/projects/spring-cloud-gateway)、[Zuul](https://github.com/Netflix/zuul)
- 服务**链路追踪**：明确服务之间的调用关系。[Zipkin](https://github.com/openzipkin/zipkin)、[Brave](https://github.com/openzipkin/brave)
- **服务监控**：以可视化或非可视化的形式展示出各个服务的运行情况 (CPU、内存、访问量等)。[Zabbix](https://github.com/jjmartres/Zabbix)、[Nagios](https://www.nagios.org/)、[Metrics](https://metrics.dropwizard.io/)
- 权限认证：根据系统设置的安全规则或者安全策略, 用户可以访问而且只能访问自己被授权的资源。[Spring Security](https://spring.io/projects/spring-security)、[Apache Shiro](http://shiro.apache.org/)
- 批处理：批量处理同类型数据或事物。[Spring Batch](https://spring.io/projects/spring-batch)
- 定时任务：定时调度任务。[Quartz](http://www.quartz-scheduler.org/)、[Elastic-job](https://gitee.com/elasticjob/elastic-job)
- 消息队列：解耦业务, 异步化处理数据。[Kafka](http://kafka.apache.org/)、[RabbitMQ](https://www.rabbitmq.com/)、[RocketMQ](http://rocketmq.apache.org/)、[ActiveMQ](http://activemq.apache.org/)
- 日志采集（elk）：收集各服务日志提供日志分析、用户画像等。[Elasticsearch](https://github.com/elastic/elasticsearch)、[Logstash](https://github.com/elastic/logstash)、[Kibana](https://github.com/elastic/kibana)
- 数据存储：存储数据。关系型：[MySql](https://www.mysql.com/)、[Oracle](https://www.oracle.com/index.html)、[MsSQL](https://docs.microsoft.com/zh-cn/sql/?view=sql-server-ver15)、[PostgreSql](https://www.postgresql.org/)；非关系型：[Mongodb](https://www.mongodb.com/)、[Elasticsearch](https://github.com/elastic/elasticsearch)
- 缓存：存储数据。[redis](https://redis.io/)
- 分库分表：数据库分库分表方案。[ShardingSphere](http://shardingsphere.apache.org/)、[Mycat](http://www.mycat.io/)
- 服务部署：将项目快速部署、上线、持续集成。[Docker](http://www.docker.com/)、[Jenkins](https://jenkins.io/zh/)、[Kubernetes(K8s)](https://kubernetes.io/)、[Mesos](http://mesos.apache.org/)

#### 3.微服务的优缺点？

微服务优点：

- 服务简单，只关注一个业务功能
- 每个微服务可由不同团队开发
- 微服务是松散耦合的
- 可用不同的编程语言与工具开发

微服务缺点：

- 运维开销，DevOps要求
- 分布式链路追踪难
- 分布式系统的复杂性，当服务数量增加，管理越加复杂

#### 4.Spring Cloud 和 Dubbo有哪些区别？

Dubbo是阿⾥巴巴公司开源的⼀个⾼性能优秀的服务框架，基于RPC调⽤，对于⽬前使⽤率较⾼的
Spring Cloud Netflix来说，它是基于HTTP的，所以效率上没有Dubbo⾼，但问题在于Dubbo体系的组件不全，不能够提供⼀站式解决⽅案，⽐如服务注册与发现需要借助于Zookeeper等实现，⽽Spring Cloud Netflix则是真正的提供了⼀站式服务化解决⽅案，且有Spring⼤家族背景。

参考：[Java微服务框架全方位对比（Dubbo 和 Spring Cloud？）](https://cloud.tencent.com/developer/article/1564212)

[Dubbo](http://dubbo.apache.org/zh-cn/)

![image-20200804004535915](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200804004535915.png)

- 优点：

1.支持各种通信协议，而且消费方和服务方使用长链接方式交互，通信速度上略胜 ；

2.采用rpc方式，性能上比Spring Cloud的rpc更好；

3.dubbo的网络消耗小于springcloud

- 缺点：

1.如果我们使用配置中心、分布式跟踪这些内容都需要自己去集成;

2.开发难度较大，原因是dubbo的jar包依赖问题很多大型工程无法解决；

[Spring Cloud](https://spring.io/cloud)

![image-20200804004740640](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200804004740640.png)

- 优点：

1、产出于Spring大家族，Spring在企业级开发框架中来头很大，可以保证后续的更新、完善。

2、spring cloud社区活跃，教程丰富，遇到问题很容易找到解决方案；

3、spring cloud功能比dubbo更加完善；

5、spring cloud采用rest访问方式，rest的技术无关性使用效果更棒；

6、spring cloud轻轻松松几行代码就完成了熔断、均衡负责、服务中心的各种平台功能；

7、从公司招聘工程师方面，spring cloud更有优势，因为其技术更新更炫；

8、提供了微服务的一整套解决方案：服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等；作为一个微服务治理的大家伙，考虑的很全面，几乎服务治理的方方面面都考虑到了，方便开发开箱即用；

- 缺点：

1.如果对于系统的响应时间有严格要求，长链接更合适。

2.接口协议约定比较自由且松散，需要有强有力的行政措施来限制接口无序升级

#### 5.Spring Cloud 与 Spring Boot 的关系？

Spring Boot 是 Spring 的一套快速配置脚手架，可以基于Spring Boot 快速开发单个微服务，Spring Cloud是一个基于Spring Boot实现的云应用开发工具；Spring Boot专注于快速、方便集成的单个微服务个体，Spring Cloud关注全局的服务治理框架；Spring Boot使用了默认大于配置的理念，很多集成方案已经帮你选择好了，能不配置就不配置，Spring Cloud很大的一部分是基于Spring Boot来实现，可以不基于Spring Boot吗？不可以。

总结：SpringBoot在Spring Clound中起到了承上启下的作用。Spring Boot可以离开Spring Cloud独立使用开发项目，但是Spring Cloud离不开Spring Boot，属于依赖的关系。

### 二.Eureka服务注册中心

#### 1.什么是Eureka

注册中心：一个中心化组件来进行服务的登记和管理，管理所有服务信息和状态。注册中心有 **Eureka、Nacos、Consul、Zookeeper**，之间的对比为下图：

![image-20200803000459588](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200803000459588.png)

Eureka是Netflix开发的服务发现框架，是一个RESTful风格的服务，是一个用于服务发现和注册的基础组件，是搭建Spring Cloud微服务的前提之一，它屏蔽了Server和client的交互细节，使得开发者将精力放到业务上。

服务注册与发现包括两个部分：服务端（Eureka Server）和客服端（Eureka Client）

![image-20200803225445890](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200803225445890.png)

- **服务端(Eureka Server)**：一个公共服务，为Client提供服务注册和发现的功能，维护注册到自身的Client的相关信息，同时提供接口给Client获取注册表中其他服务的信息，使得动态变化的Client能够进行服务间的相互调用。
- **客户端(Eureka Client)**：Client将自己的服务信息通过一定的方式登记到Server上，并在正常范围内维护自己信息一致性，方便其他服务发现自己，同时可以通过Server获取到自己依赖的其他服务信息，完成服务调用，还内置了负载均衡器，用来进行基本的负载均衡。

#### 2.Eureka的一些基础概念

**服务注册 Register**：当 `Eureka` 客户端向 `Eureka Server` 注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页等。

**服务续约 Renew**：`Eureka` 客户会每隔30秒(默认情况下)发送一次心跳来续约。 通过续约来告知 `Eureka Server` 该 `Eureka` 客户仍然存在，没有出现问题。 正常情况下，如果 `Eureka Server` 在90秒没有收到 `Eureka` 客户的续约，它会将实例从其注册表中删除。

**获取注册列表信息 Fetch Registries**： `Eureka` 客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与 `Eureka` 客户端的缓存信息不同, `Eureka` 客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，`Eureka` 客户端则会重新获取整个注册表信息。 `Eureka` 服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。`Eureka` 客户端和 `Eureka` 服务器可以使用JSON / XML格式进行通讯。在默认的情况下 `Eureka` 客户端使用压缩 `JSON` 格式来获取注册列表的信息。

**服务下线 Cancel**：Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：`DiscoveryManager.getInstance().shutdownComponent();`

**服务剔除 Eviction**： 在默认的情况下，当Eureka客户端连续90秒(3个续约周期)没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除。

#### 3.Eureka基础架构

**Eureka通过心跳检测、健康检查和客户端缓存等机制，提高系统的灵活性、伸缩性和可用性**。https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance

![image-20200803224619624](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200803224619624.png)

Eureka包含两个组件：Eureka Server 和 Eureka Client，Eureka Server提供服务发现的能力，各个微服务启动时会通过 Eureka Client 向 Eureka Server 进行注册自己的信息，Eureka Server 会存储该服务的信息。

- 服务提供者向Eureka Server中注册服务，Eureka Server接受到注册事件会在集群和分区中进行数据同步，服务消费者可以从注册中心获取服务注册信息，进行服务调用
- 每个Eureka Server同时也是Eureka Client，多个Eureka Server之间通过复制方式完成服务注册列表的同步
- 微服务启动后会周期性的向Eureka Server发送心跳（默认周期为30秒）以续约自己的信息
- Eureka Server在一定时间内（默认为90秒）没有接受到某个微服务的心跳，将会注销掉该微服务节点
- Eureka Client会缓存Eureka Server中的信息，当所有的Eureka Server节点都宕机，服务消费者依然可以使用缓存中的信息找到服务提供者

#### 4.Eureka元数据

Eureka元数据有两种：**标准元数据和自定义元数据**。

- 标准元数据：主机名、IP地址、端口号等信息，这些信息都会发布到服务注册列表中，用于服务之间调用
- 自定义元数据：可以在application.yml中配置 eureka.instance.metadata-map配置，符合key/value的存储格式。这些数据可以在远程客服端中访问

如：

```yml
eureka:
  instance:
	metadata-map:
	  # ⾃定义元数据(key/value⾃定义)
	  cluster: cl1
	  region: rn1
```

在应用程序中可以使用DiscoveryClient获取指定微服务的所有元数据信息：

```java
// 从EurekaServer获取指定微服务实例
List<ServiceInstance> serviceInstanceList =discoveryClient.getInstances("service-user");
```

通过debug可以看到自定义的元数据：

![image-20200803003619475](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200803003619475.png)

#### 5.Eureka常用配置

服务端配置：在spring boot启动类中设置 `@EnableEurekaServer` 注解开启 Eureka 服务

```yml
eureka:  
  instance:    
  	hostname: xxxxx    # 主机名称    
	prefer-ip-address: true/false   # 注册时显示ip
  server:    
  	enableSelfPreservation: true   # 启动自我保护
	renewalPercentThreshold: 0.85  # 续约配置百分比
```

客服端配置：在spring boot启动类中设置 `@EnableDiscoveryClient` 注解开启 Eureka 服务

```yml
eureka:  
  client:    
  	register-with-eureka: true/false  # 是否向注册中心注册自己    
  	fetch-registry: # 指定此客户端是否能获取eureka注册信息    
  	service-url:    # 暴露服务中心地址      
  	  defaultZone: http://xxxxxx   # 默认配置  
  instance: instance-id: xxxxx # 指定当前客户端在注册中心的名称
```

#### 6.Eureka Client客户端

**服务提供者向Eureka服务端注册服务，并完成服务续约等⼯作，服务消费者获取服务列表**。

**服务注册**（服务提供者）：

- 添加eureka-client依赖，配置Eureka服务注册中心地址
- 服务启动时会向注册中心发起注册请求，携带服务元数据信息
- Eureka注册中心会把服务节点的相关信息保存到Map中

**服务续约**（服务提供者）：

服务每隔30秒会向注册中心续约（心跳检测）一次，若没有续约，租约在90秒后到期，然后服务失效。

配置文件中配置：一般不需要调整

```yml
# 向Eureka服务中⼼集群注册服务
eureka:
  instance:
	# 租约续约间隔时间，默认30秒
	lease-renewal-interval-in-seconds: 30
	# 租约到期，服务时效时间，默认值90秒,服务超过90秒没有发⽣⼼跳，EurekaServer会将服务从列表移除
	lease-expiration-duration-in-seconds: 90
```

**获取服务列表**（服务消费者）：

服务消费者启动后会从Eureka Server服务列表中获取只读备份服务列表清单，缓存到本地。每隔30秒会重新获取并更新数据，时间间隔可以通过配置`eureka.client.registry-fetch-interval-seconds`修改。

```yml
#向Eureka服务中⼼集群注册服务
eureka:
  client:
  	# 每隔多久拉取⼀次服务列表
	registry-fetch-interval-seconds: 30
```

#### 7.Eureka Server服务端

**服务下线**：当服务正常关闭时会发送服务下线的rest请求给Eureka服务端，注册中心接口请求后将该服务置为下线状态。

**失效剔除**：Eureka Server会定时（每隔60秒 `eureka.server.eviction-interval-timer-in-ms`）进行检查，若服务节点在一点个时间（默认90秒 `eureka.instance.lease-expiration-duration-in-
seconds`）内没有收到心跳，会注销该实例。

为什么会有**自我保护**：默认情况下，如果Eureka Server在⼀定时间内（默认90秒）没有接收到某个微服务实例的⼼跳，Eureka Server将会移除该实例。但是当⽹络分区故障发⽣时，微服务与Eureka Server之间⽆法正常通信，⽽微服务本身是正常运⾏的，此时不应该移除这个微服务，所以引⼊了⾃我保护机制。

自我保护开启：如果在15分钟内超过85%的客户端节点都没有正常的⼼跳，那么Eureka就认为客户端与注册中⼼出现了⽹络故障，Eureka Server⾃动进⼊⾃我保护机制。

服务中⼼⻚⾯会显示如下提示信息：

![image-20200803010746384](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200803010746384.png)

自我保护模式时：

- 不会剔除任何服务实例（可能是服务提供者和EurekaServer之间⽹络问题），保证了⼤多数服务依然可⽤
- Eureka Server仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上，保证当前节点依然可⽤，当⽹络稳定时，当前Eureka Server新的注册信息会被同步到其它节点中

在Eureka Server⼯程中通过`eureka.server.enable-self-preservation`配置可⽤关停⾃我保护，默认值是打开：

```yml
eureka:
  server:
	enable-self-preservation: false  # 关闭⾃我保护模式（缺省为打开）
```

#### 总结和参考

服务提供者启动时会向注册中心发起注册请求，携带服务元数据信息，并每隔30秒会向注册中心发起一次心跳来续约。若没有续约，则租约90秒到期后服务失效。服务消费者从注册中心获取只读备份列表清单并缓存到本地，每隔30秒重新获取并更新数据。

Eureka 客户端正常关闭时，服务端会将服务置为下线状态（不会自动完成）；服务端每隔60秒定时检查，若发现实例在90秒内没有收到心跳，会注销该实例；在15分钟内超过85%的客服端节点都没有正常的心跳，服务端会进入自我保护模式，不会剔除任何服务实例，仍然可以接受新的服务注册和查询请求。

Eureka（AP）通过心跳检测、健康检查和客户端缓存等机制，提高系统的灵活性、伸缩性和可用性。

参考链接：

- [深入浅出 Spring Cloud 之 Eureka](https://juejin.im/post/6844904001444511758)
- [深入理解Eureka](https://juejin.im/post/6844903481019465742)
- [SpringColud Eureka的服务注册与发现](https://juejin.im/post/6844904191127879693)

### 二.Ribbon负载均衡

服务端负载均衡

客服端负载均衡



负载均衡策略



### 三.Hystrix

雪崩效应：微服务的扇入和扇出

雪崩效应的解决方案：服务熔断、服务降级、服务限流





### 四.Feign

一个轻量级RESTful的HTTP服务客户端，发起请求，远程调用

对负载均衡ribbon的支持：Feign默认的请求处理超时时⻓1s，有时候我们的业务确实执⾏的需要⼀定时间，可以调整请求处理超时时⻓，Feign⾃⼰有超时设置，如果配置Ribbon的超时，则会以Ribbon的为准

对熔断器Hystrix的支持：当前有两个超时时间设置（Feign/hystrix），熔断的时候是根据这**两个时间的最⼩值**来进⾏的，即处理时⻓超过最短的那个超时时间了就熔断进⼊回退降级逻辑

对请求压缩和响应压缩的支持

对日志级别配置的支持



### 五.GateWay

Spring Cloud GateWay天⽣就是异步⾮阻塞的，基于Reactor模型。

⼀个请求—>⽹关根据⼀定的条件匹配—匹配成功之后可以将请求转发到指定的服务地址。

- 路由（route）：ID、URL、predicates、filter
- 断言（predicates）：
- 过滤器（filter）：可以在请求之前或者之后执⾏业务逻辑，从过滤器类型的⻆度，Spring Cloud GateWay的过滤器分为GateWayFilter和GlobalFilter两种



### 六.SpringCloudConfig

Spring Cloud Config是⼀个分布式配置管理⽅案，包含了 Server端和 Client端两个部分。

- Server 端：提供配置⽂件的存储、以接⼝的形式将配置⽂件的内容提供出去
- Client 端：通过接⼝获取配置数据并初始化⾃⼰的应⽤

消息总线Bus，即我们经常会使⽤MQ消息代理构建⼀个共⽤的Topic，通过这个Topic连接各个微服
务实例，MQ⼴播的消息会被所有在注册中⼼的微服务实例监听和消费。换⾔之就是通过⼀个主题连接各个微服务，打通脉络。





### 参考链接

- [Spring Cloud官网](https://spring.io/projects/spring-cloud)
- [Spring Cloud中文社区](https://www.springcloud.cc/)
- [Spring Cloud中国社区](http://docs.springcloud.cn/user-guide/eureka/)
- [冒着挂科的风险也要给你们看的Spring Cloud入门总结](https://juejin.im/post/6844904007975043079)
- [Spring Cloud学习](https://blog.csdn.net/u012702547/article/details/78547925)



