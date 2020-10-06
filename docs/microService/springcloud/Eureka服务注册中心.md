#### 1.什么是Eureka

注册中心：一个中心化组件来进行服务的登记和管理，管理所有服务信息和状态。注册中心有 **Eureka、Nacos、Consul、Zookeeper**，之间的对比为下图：

![image-20200803000459588](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vaXR6bGcvbXlwaWN0dXJlcy9yYXcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDgwMzAwMDQ1OTU4OC5wbmc?x-oss-process=image/format,png)

Eureka是Netflix开发的服务发现框架，是一个RESTful风格的服务，是一个用于服务发现和注册的基础组件，是搭建Spring Cloud微服务的前提之一，它屏蔽了Server和client的交互细节，使得开发者将精力放到业务上。

服务注册与发现包括两个部分：服务端（Eureka Server）和客服端（Eureka Client）

![image-20200803225445890](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vaXR6bGcvbXlwaWN0dXJlcy9yYXcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDgwMzIyNTQ0NTg5MC5wbmc?x-oss-process=image/format,png)

- **服务端(Eureka Server)** ：一个公共服务，为Client提供服务注册和发现的功能，维护注册到自身的Client的相关信息，同时提供接口给Client获取注册表中其他服务的信息，使得动态变化的Client能够进行服务间的相互调用。
- **客户端(Eureka Client)** ：Client将自己的服务信息通过一定的方式登记到Server上，并在正常范围内维护自己信息一致性，方便其他服务发现自己，同时可以通过Server获取到自己依赖的其他服务信息，完成服务调用，还内置了负载均衡器，用来进行基本的负载均衡。

#### 2.Eureka的一些基础概念

**服务注册 Register**：当 `Eureka` 客户端向 `Eureka Server` 注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页等。

**服务续约 Renew**：`Eureka` 客户会每隔30秒(默认情况下)发送一次心跳来续约。 通过续约来告知 `Eureka Server` 该 `Eureka` 客户仍然存在，没有出现问题。 正常情况下，如果 `Eureka Server` 在90秒没有收到 `Eureka` 客户的续约，它会将实例从其注册表中删除。

**获取注册列表信息 Fetch Registries**： `Eureka` 客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与 `Eureka` 客户端的缓存信息不同, `Eureka` 客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，`Eureka` 客户端则会重新获取整个注册表信息。 `Eureka` 服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。`Eureka` 客户端和 `Eureka` 服务器可以使用JSON / XML格式进行通讯。在默认的情况下 `Eureka` 客户端使用压缩 `JSON` 格式来获取注册列表的信息。

**服务下线 Cancel**：Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：`DiscoveryManager.getInstance().shutdownComponent();`

**服务剔除 Eviction**： 在默认的情况下，当Eureka客户端连续90秒(3个续约周期)没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除。

#### 3.Eureka基础架构

**Eureka通过心跳检测、健康检查和客户端缓存等机制，提高系统的灵活性、伸缩性和可用性**。https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance

![image-20200803224619624](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vaXR6bGcvbXlwaWN0dXJlcy9yYXcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDgwMzIyNDYxOTYyNC5wbmc?x-oss-process=image/format,png)

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

![image-20200803003619475](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vaXR6bGcvbXlwaWN0dXJlcy9yYXcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDgwMzAwMzYxOTQ3NS5wbmc?x-oss-process=image/format,png)

#### 5.Eureka常用配置

服务端配置：在spring boot启动类中设置 `@EnableEurekaServer` 注解开启 Eureka 服务

```yml
eureka:  
  instance:    
  	hostname: xxxxx    # 主机名称    
	prefer-ip-address: true/false   # 注册时显示ip
  server:    
  	enableSelfPreservation: true   # 启动自我保护
	renewalPercentThreshold: 0.85  # 续约配置百分比
```

客服端配置：在spring boot启动类中设置 `@EnableDiscoveryClient` 注解开启 Eureka 服务

```yml
eureka:  
  client:    
  	register-with-eureka: true/false  # 是否向注册中心注册自己    
  	fetch-registry: # 指定此客户端是否能获取eureka注册信息    
  	service-url:    # 暴露服务中心地址      
  	  defaultZone: http://xxxxxx   # 默认配置  
  instance: instance-id: xxxxx # 指定当前客户端在注册中心的名称
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

![image-20200803010746384](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vaXR6bGcvbXlwaWN0dXJlcy9yYXcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDgwMzAxMDc0NjM4NC5wbmc?x-oss-process=image/format,png)

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
- [深入理解Eureka(源码)](https://juejin.im/post/6844903481019465742)
- [SpringColud Eureka的服务注册与发现](https://juejin.im/post/6844904191127879693)



