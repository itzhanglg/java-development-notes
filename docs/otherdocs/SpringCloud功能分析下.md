### 需求

根据如下描述，改造Spring Cloud（上）的作业，完成练习：

1)Eureka注册中心  替换为  Nacos注册中心

2)Config+Bus配置中心  替换为 Nacos配置中心

3)Feign调用 替换为 Dubbo RPC调用

4)使用Sentinel对GateWay网关的入口资源进行限流（限流参数自定义并完成测试即可）   

注意：1）所有替换组件使用单节点即可

### 思路分析

SCN微服务结构图：

![image-20200810000055231](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200810000055231.png)

SCA微服务结构图：

![image-20200809235836346](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200809235836346.png)

- Nacos可以作为注册中心和配置中心，相当于Eureka+Config+Bus；Nacos可以服务发现和健康检查、动态配置、动态DNS管理、服务元数据管理等；Nacos的数据模型为NGS/NGD，某环境下某项目的某个服务（NGS）和某环境下某项目的某个配置文件（NGD），相当于Maven的GAV坐标。
- Sentinel是⼀个面向云原生微服务的流量控制、熔断降级组件。Sentinel分为核心库（Java客户都）和控制台（Dashboard）。Sentinel主要应用削峰填谷、高流量预热、实时熔断等。
- Dubbo可以替换OpenFeign和Ribbon，使用Dubbo RPC和Dubbo LB。

### 核心配置和代码

#### GateWay流量控制

使用Sentinel对GateWay网关的入口资源进行限流：[网关流量控制](https://sentinelguard.io/zh-cn/docs/api-gateway-flow-control.html)

使用时只需注入对应的 `SentinelGatewayFilter` 实例以及 `SentinelGatewayBlockExceptionHandler` 实例即可。

引入依赖：

```xml
<!-- Sentinel支持采用 GateWay 作为流量控制和熔断控制 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
    <version>x.y.z</version>
</dependency>
```

配置类：

```java
@Configuration
public class GatewayConfiguration {

    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;

    public GatewayConfiguration(ObjectProvider<List<ViewResolver>> viewResolversProvider,
                                ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }

    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
        // Register the block exception handler for Spring Cloud Gateway.
        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }

    @Bean
    @Order(-1)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }
}
```

那么这里面的 route ID（如 `product_route`）都会被标识为 Sentinel 的资源。

可以在 `GatewayCallbackManager` 注册回调进行定制：

- `setBlockHandler`：注册函数用于实现自定义的逻辑处理被限流的请求，对应接口为 `BlockRequestHandler`。默认实现为 `DefaultBlockRequestHandler`，当被限流时会返回类似于下面的错误信息：`Blocked by Sentinel: FlowException`。

#### Nacos注册中心配置

依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

配置：

```yaml
spring:
  cloud:
    nacos:
      discovery:
        # 集群中各节点信息都配置在这里（域名-VIP-绑定映射到各个实例的地址信息）
        server-addr: 127.0.0.1:8848,127.0.0.1:8849,127.0.0.1:8850
        cluster-name: sh
        namespace: cab03401-3609-40c5-a232-4d7fda74fdb8
```

#### Nacos配置中心配置

依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

配置：

```yaml
spirng:
  cloud:
	nacos:
	  config:
		server-addr: 127.0.0.1:8848
		# 锁定server端的配置文件（读取它的配置项）
		namespace: f965f7e4-7294-40cf-825c-ef363c269d37  # 命名空间id
		group: DEFAULT_GROUP  # 默认分组就是DEFAULT_GROUP，如果使用默认分组可以不配置
		file-extension: yaml  #默认properties
		# 优先级：根据规则生成的dataId > 扩展的dataId（对于扩展的dataId，[n] n越大优先级越高）
		# 根据规则拼接出来的dataId效果：lagou-service-resume.yaml
		ext-config[0]:
		  data-id: abc.yaml
		  group: DEFAULT_GROUP
		  refresh: true #开启扩展dataId的动态刷新
		ext-config[1]:
          data-id: def.yaml
		  group: DEFAULT_GROUP
		  refresh: true #开启扩展dataId的动态刷新
```

####  Nacos 实现 Sentinel 规则持久化

Sentinel规则控制存储在对应的微服务中（内存中），每次重启相应微服务，都会导致Sentinel中配置的规则失效，需要使用Nacos来实现持久化。

依赖：

```xml
<!--sentinel 核心环境 依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
<!-- Sentinel支持采用 Nacos 作为规则配置数据源，引入该适配依赖 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
<!-- Sentinel支持采用 GateWay 作为流量控制和熔断控制 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
</dependency>
```

配置：

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8080 # sentinel dashboard/console 地址
        port: 8719   #  sentinel会在该端口启动http server，那么这样的话，控制台定义的一些限流等规则才能发送传递过来，
        #如果8719端口被占用，那么会依次+1
      # Sentinel Nacos数据源配置，Nacos中的规则会自动同步到sentinel流控规则中
      datasource:
        # 自定义的流控规则数据源名称
        flow:
          nacos:
            server-addr: ${spring.cloud.nacos.discovery.server-addr}
            data-id: ${spring.application.name}-flow-rules
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow  # 类型来自RuleType类
        # 自定义的降级规则数据源名称
        degrade:
          nacos:
            server-addr: ${spring.cloud.nacos.discovery.server-addr}
            data-id: ${spring.application.name}-degrade-rules
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: degrade  # 类型来自RuleType类
```

#### Dubbo RPC配置

Nacos+Sentinel+Dubbo 三剑合璧

依赖：

```xml
<!--spring cloud+dubbo 依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-dubbo</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-apache-dubbo-adapter</artifactId>
</dependency>
```

配置：

```yaml
dubbo:
  scan:
    # dubbo 服务扫描基准包
    base-packages: com.fishleap.service.impl
  protocol:
    # dubbo 协议
    name: dubbo
    # dubbo 协议端口（ -1 表示自增端口，从 20880 开始）
    port: -1
    host: 127.0.0.1
  registry:
    # 挂载到 Spring Cloud 注册中心
    address: spring-cloud://localhost
  cloud:
    # 订阅服务提供方的应用列表，订阅多个服务提供者使用 "," 连接
    subscribed-services: lagou-service-email
```

创建公共Dubbo-API工程（公共服务接口），去掉相应微服务中的控制层，引入API工程的jar包，服务实现类使用Dubbo的@Service注解。

### 问题

1.Sentinel集群配置数据库插入sql脚本时报MySQL：ERROR 1067 - Invalid default value for 'end_time'

本地库执行开发库的创建表脚本，报错Invalid default value for 'create_time'，本以为是sql_mode设置的问题，按照开发库设置了一遍还是报错，最后查了下才想到可能是版本的问题，本地数据库版本号5.5，开发库是5.7，而使用current_timestamp作为datetime的默认值，只有在5.6之后的版本才支持。



2.Caused by: java.lang.IllegalStateException: Failed to check the status of the service com.fishleap.service.EmailService. No provider available for the service com.fishleap.service.EmailService

服务提供者启动失败 或 远程服务没有注入到dubbo容器中（@Service注解可能用的Spirng的注解）



3.Parameter 0 of method modifyRequestBodyGatewayFilterFactory in org.springframework.cloud.gateway.config.GatewayAutoConfiguration required a bean of type 'org.springframework.http.codec.ServerCodecConfigurer' that could not be found.

网关工程中引入了 spring-boot-starter-web 模块依赖，排斥掉该依赖就可以了











