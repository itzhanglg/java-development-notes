### 一.作业框架概述

![image-20200802204634214](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200802204634214.png)

Nginx动静分离：

- 静态资源：html、js、css、img等
- 动态资源：
    - Eureka集群：服务注册与发现，解耦了服务提供者和服务消费者
    - SpringCloudConfig+Bus：分布式配置中心、消息总线（基于MQ的，⽀持RabbitMq/Kafka），实现配置信息的自动更新
    - 邮件微服务：发送邮件
    - 验证码微服务：生成验证码和验证码校验
    - 用户微服务：注册、登陆、是否已经注册、token验证
    - GateWay网关：统一完成路由、IP防暴刷、统一认证等

Eureka可视化界面效果如下：

![image-20200802213302321](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200802213302321.png)

### 二.核心代码

#### 1.nginx配置

```
# 以static前缀开头的请求直接访问本地磁盘
location /static {
	alias   E:\develop\skillStudy\javaTrain\operation\phase-3-module-4\code\login;
	index  login.html;
}
# 以api前缀开头的请求都转发到后端
location /api {
	# http://127.0.0.1:9002/api/code/create/18986695894@163.com
	# 若为 http://localhost:9002/api 时，后面各个微服务的接口上无需再写 /api
	proxy_pass http://localhost:9002;	# 各个微服务接口前都需要加上 /api
}
```

#### 2.邮件接口

```java
/**
     * 发送验证码邮件
     * @param email 邮箱
     * @param code 验证码
     * @return 发送结果
     */
@GetMapping("/{email}/{code}")
public Boolean sendEmail(@PathVariable("email")String email, 
                         @PathVariable("code")String code) {
    return emailService.sendMail(email, "注册验证码", code);
}

public Boolean sendMail(String to, String subject, String verifyCode);
```

验证码微服务远程调用邮件微服务：

```java
// @FeignClient 表明当前类是一个Feign客户端
// value指定该客户端要请求的服务名称（登记到注册中心上的服务提供者的服务名称）
@FeignClient(value = "lagou-service-email",path = "/api/email",fallback = 
             EmailFallBack.class)
public interface EmailServiceFeignClient {
    // Feign要做的事情就是，拼装url发起请求
    // 我们调用该方法就是调用本地接口方法，那么实际上做的是远程请求
    @GetMapping("/{email}/{code}")
    public Boolean sendEmail(@PathVariable("email")String 
                             email,@PathVariable("code")String code);
}
```

服务降级EmailFallBack：

```java
// 降级回退逻辑需要定义一个类，实现FeignClient接口，实现接口中的方法
@Component
public class EmailFallBack implements EmailServiceFeignClient {
    @Override
    public Boolean sendEmail(String email, String code) {
        // 当邮件微服务不可用或超时时,服务降价默认返回false,发送邮件失败
        return false;
    }
}
```

#### 3.GateWay配置全局过滤

##### 3.1 IP接口防爆刷过滤

```java
/**
 * @author zlg
 * IP接口防爆刷过滤
 * 定义全局过滤器，会对所有路由生效
 */
@Slf4j
@Component  // 让容器扫描到，等同于注册了
@RefreshScope   // 修改config仓库的配置后发送请求手动刷新，可以不同配置；bus会自动刷新配置
public class IPInterfaceCountFilter implements GlobalFilter, Ordered, Runnable {

    // 存储同一ip在限制时间内的请求次数
    private static Map<String,Integer> ipCountMap = new ConcurrentHashMap<>();

    //单个在多少分钟内请求注册接口
    @Value("${gateway.limitMinutes}")
    private String limitMinutes;    // 1

    //不能超过的次数
    @Value("${gateway.limitCount}")
    private String limitCount;      // 1

    //创建线程定时任务,每隔1分钟执行一次任务,清空ip接口存储列表
    public IPInterfaceCountFilter () {
        Executors.newSingleThreadScheduledExecutor()
                .scheduleWithFixedDelay(this, 6, 1, TimeUnit.MINUTES);
    }

    /**
     * 过滤器核心方法
     * @param exchange 封装了request和response对象的上下文
     * @param chain 网关过滤器链（包含全局过滤器和单路由过滤器）
     * @return
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        System.out.println(limitMinutes+"====>>>"+limitCount);
        ServerHttpRequest request = exchange.getRequest();
//        String path = request.getPath().pathWithinApplication().value();
//        HttpMethod method = request.getMethod();
        // 获取路由的目标URI
//        URI targetUri = exchange.getAttribute(ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR);
//        /api/user/register/1321340821@qq.com/11/885444----POST--------null
//        System.out.println(path+"----"+method+"--------"+targetUri);

        // 从request对象中获取客户端ip
//        String clientIp = exchange.getRequest().getRemoteAddress().getHostName();
        // 获取真实请求的 IP 地址
        String clientIp = IPUtils.getIpAddress(request);
        // IPv6转成IPv4格式
        if (clientIp.equals("0:0:0:0:0:0:0:1")) {
            clientIp = "127.0.0.1";
        }
        System.out.println("ip==========>>>>>>"+clientIp);
        String path = exchange.getRequest().getPath().value();
        System.out.println("Path==========>>>>>"+exchange.getRequest().getPath().value());
        // 判断是否是注册接口
        if (path.indexOf("/register") >= 0) {
            if (ipCountMap.containsKey(clientIp)) {
                System.out.println("IPCountExist======>>>>" + ipCountMap.get(clientIp));
                if (ipCountMap.get(clientIp) >= Integer.parseInt(limitCount)) {
                    // 拒绝访问，返回
                    exchange.getResponse().setStatusCode(HttpStatus.SEE_OTHER); // 状态码 303
                    log.debug("=====>IP:" + clientIp + "当前时间段内注册次数过多!");
                    String data = "您频繁进行注册，请求已被拒绝!";
                    DataBuffer wrap = exchange.getResponse().bufferFactory().wrap(data.getBytes());
                    return exchange.getResponse().writeWith(Mono.just(wrap));
                } else {
                    ipCountMap.put(clientIp, ipCountMap.get(clientIp) + 1);
                }
            } else {
                ipCountMap.put(clientIp, 1);
                System.out.println("IPCount======>>>>" + ipCountMap.get(clientIp));
            }
        }
        // 合法请求，放行，执行后续的过滤器
        System.out.println("===============================================");
        return chain.filter(exchange);
    }


    /**
     * 返回值表示当前过滤器的顺序(优先级)，数值越小，优先级越高
     * @return 0
     */
    @Override
    public int getOrder() {
        return 0;
    }

    // 每隔1分钟清空一次ip接口请求次数的集合
    @Override
    public void run() {
        ipCountMap.clear();
    }
}
```

##### 3.2 用户进行token验证过滤

```java
/**
 * @author zlg
 * 用户进行token的验证
 * 用户微服务和验证码微服务的请求不不过滤（网关调用下游用户微服务的token验证接口）
 */
@Component
public class UserTokenFilter implements GlobalFilter, Ordered {

    @Autowired
    private UserServiceFeignClient userServiceFeignClient;

    /**
     * 过滤器核心方法
     * @param exchange 封装了request和response对象的上下文
     * @param chain 网关过滤器链（包含全局过滤器和单路由过滤器）
     * @return
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 用户微服务和验证码微服务的请求不不过滤
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getURI().getPath();
        if (path.contains("/api/user") || path.contains("/api/code")) {
            return chain.filter(exchange);
        }
        // 网关调用下游用户微服务的token验证接口
        List<HttpCookie> tokens = request.getCookies().get("token");
        if (CollectionUtils.isEmpty(tokens) || tokens.isEmpty()) {
            // 拒绝访问，返回
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED); // 状态码 401
            exchange.getResponse().getHeaders().set("Content-Type","application/json;charset=UTF-8");
            String data = "您的token无效!";
            DataBuffer wrap = exchange.getResponse().bufferFactory().wrap(data.getBytes(Charsets.UTF_8));
            return exchange.getResponse().writeWith(Mono.just(wrap));
        }
        // 查询是否有该token
        for (HttpCookie cookie : tokens) {
            String email = userServiceFeignClient.getEmail(cookie.getValue());
            if (StringUtils.isNotBlank(email)) {
                return chain.filter(exchange);
            }
        }

        // 拒绝访问，返回
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED); // 状态码 401
        exchange.getResponse().getHeaders().set("Content-Type","application/json;charset=UTF-8");
        String data = "您的token无效!";
        DataBuffer wrap = exchange.getResponse().bufferFactory().wrap(data.getBytes(Charsets.UTF_8));
        return exchange.getResponse().writeWith(Mono.just(wrap));
    }


    /**
     * 返回值表示当前过滤器的顺序(优先级)，数值越小，优先级越高
     * @return 1
     */
    @Override
    public int getOrder() {
        return 1;
    }
}
```

### 三.各个功能层的主要配置信息

#### 1.Eureka集群配置

##### 1.1 添加eureka服务依赖

```xml
<!--Eureka server依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

##### 1.2 application.yml文件配置

```yml
#eureka server服务端口
server:
  port: 8761
  
spring:
  application:
    name: lagou-cloud-eureka-server # 应用名称，应用名称会在Eureka中作为服务名称

    # eureka 客户端配置（和Server交互），Eureka Server 其实也是一个Client
eureka:
  instance:
    hostname: LagouCloudEurekaServerA  # 当前eureka实例的主机名
  client:
    service-url:
      # 配置客户端所交互的Eureka Server的地址
      #（Eureka Server集群中每一个Server其实相对于其它Server来说都是Client）
      # 集群模式下，defaultZone应该指向其它Eureka Server，如果有更多其它Server实例，逗号拼接即可
      defaultZone: http://LagouCloudEurekaServerB:8762/eureka
    register-with-eureka: true  # 集群模式下可以改成true
    fetch-registry: true # 集群模式下可以改成true
  dashboard:
    enabled: true
```

##### 1.3 启动类上添加 @EnableEurekaServer 注解

#### 2.SpringCloudConfig+Bus配置

##### 2.1创建远程仓库，上传各个微服务配置信息

如： [lagou-cloud-gateway-dev.yml](https://gitee.com/itzlg/config-repo/blob/master/lagou-cloud-gateway-dev.yml)

```yml
devDatasource:
  url: jdbc:mysql://localhost:3306/javalearn?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=GMT%2B8
  username: root
  password: root
gateway:
  limitMinutes: 1
  limitCount: 1
```

##### 2.2 添加相关依赖

```xml
<!--eureka client 客户端依赖引入-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!--config配置中心服务端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

##### 2.3 application.yml文件配置

```yml
server:
  port: 9006
  
#注册到Eureka服务中心
eureka:
  client:
    service-url:
      # 注册到集群，就把多个Eurekaserver地址使用逗号连接起来即可；注册到单实例（非集群模式），那就写一个就ok
      defaultZone: http://LagouCloudEurekaServerA:8761/eureka,http://LagouCloudEurekaServerB:8762/eureka
  instance:
    prefer-ip-address: true  #服务实例中显示ip，而不是显示主机名（兼容老的eureka版本）
    # 实例名称： 192.168.1.103:lagou-service-resume:8080，我们可以自定义它
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}:@project.version@

spring:
  application:
    name: lagou-cloud-configserver
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/itzlg/config-repo.git #配置git服务地址
          username: XXXXXX #配置git用户名
          password: XXXXXX #配置git密码
          search-paths:
            - serverConfig
      # 读取分支
      label: master
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
    
#针对的被调用方微服务名称,不加就是全局生效
#lagou-service-resume:
#  ribbon:
#    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #负载策略调整

# springboot中暴露健康检查等断点接口
management:
  endpoints:
    web:
      exposure:
        include: "*"
  # 暴露健康接口的细节
  endpoint:
    health:
      show-details: always
```

##### 2.4 启动类上添加注解，启动配置中心前还需要启动rabbitmq服务端

```
@EnableDiscoveryClient	// 开启服务注册
@EnableConfigServer     // 开启配置中心功能
```

rabbitmq效果：

![image-20200802215054044](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200802215054044.png)

#### 3.各个业务微服务配置

##### 31 添加相关依赖

```xml
<!--服务注册-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!--远程调用-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--配置中心-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
<!--消息总线-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

<!--持久化框架-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!--数据库-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

##### 3.2 bootstrap.yml文件配置

```yml
server:
  port: 8080

#注册到Eureka服务中心
eureka:
  client:
    service-url:
      # 注册到集群，就把多个Eurekaserver地址使用逗号连接起来即可；注册到单实例（非集群模式），那就写一个就ok
      defaultZone: http://LagouCloudEurekaServerA:8761/eureka,http://LagouCloudEurekaServerB:8762/eureka
  instance:
    prefer-ip-address: true  #服务实例中显示ip，而不是显示主机名（兼容老的eureka版本）
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}:@project.version@

spring:
  application:
    name: lagou-service-user
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: ${devDatasource.url}
    username: ${devDatasource.username}
    password: ${devDatasource.password}
  jpa:
    database: MySQL
    show-sql: true
    hibernate:
      naming:
        #避免将驼峰命名转换为下划线命名
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl  
  # config客户端配置,和ConfigServer通信，并告知ConfigServer希望获取的配置信息在哪个文件中       
  cloud:
    config:
      name: lagou-service-user  #配置文件名称
      profile: dev  #后缀名称
      label: master #分支名称
      uri: http://localhost:9006    #ConfigServer配置中心地址
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest  
  
#针对的被调用方微服务名称,不加就是全局生效
#lagou-service-user:  
ribbon:
  #请求连接超时时间
  ConnectTimeout: 5000
  #请求处理超时时间
  #Feign超时时长设置
  ReadTimeout: 5000
  #对所有操作都进行重试
  OkToRetryOnAllOperations: true
  ####根据如上配置，当访问到故障请求的时候，它会再尝试访问一次当前实例（次数由MaxAutoRetries配置），
  ####如果不行，就换一个实例进行访问，如果还不行，再换一次实例访问（更换次数由MaxAutoRetriesNextServer配置），
  ####如果依然不行，返回失败信息。
  MaxAutoRetries: 0 #对当前选中实例重试次数，不包括第一次调用
  MaxAutoRetriesNextServer: 0 #切换实例的重试次数
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #负载策略调整
  
# springboot中暴露健康检查等断点接口
management:
  endpoints:
    web:
      exposure:
        include: "*"
  # 暴露健康接口的细节
  endpoint:
    health:
      show-details: always 
      
logging:
  level:
    # Feign日志只会对日志级别为debug的做出响应
    com.fishleap.client.EmailServiceFeignClient: debug 
    
# 开启Feign的熔断功能
feign:
  hystrix:
    enabled: false
  client:
    config:
      default:
        connect-timeout: 20000
        read-timeout: 20000    
```

##### 3.3 启动类添加注解

```
@EnableDiscoveryClient
@EnableFeignClients
```

#### 4.GateWay网关配置

##### 4.1 添加相关依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-commons</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!--GateWay 网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--引入webflux-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<!--引入openfeign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--引入config和bus-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!--日志依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
<!--测试依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<!--lombok工具-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.4</version>
    <scope>provided</scope>
</dependency>

<!--引入Jaxb，开始-->
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-core</artifactId>
    <version>2.2.11</version>
</dependency>
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
</dependency>
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-impl</artifactId>
    <version>2.2.11</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
    <version>2.2.10-b140310.1920</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
<!--引入Jaxb，结束-->

<!-- Actuator可以帮助你监控和管理Spring Boot应用-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!--热部署-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>

<!--链路追踪-->
<!--<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>-->
```

##### 4.2 bootstrap.yml文件配置

```yml
server:
  port: 9002
  
eureka:
  client:
    serviceUrl: # eureka server的路径
      defaultZone: http://lagoucloudeurekaservera:8761/eureka/,http://lagoucloudeurekaserverb:8762/eureka   #把 eureka 集群中的所有 url 都填写了进来，也可以只写一台，因为各个 eureka server 可以同步注册表
  instance:
    #使用ip注册，否则会使用主机名注册了（此处考虑到对老版本的兼容，新版本经过实验都是ip）
    prefer-ip-address: true
    #自定义实例显示格式，加上版本号，便于多版本管理，注意是ip-address，早期版本是ipAddress
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}:@project.version@
    
spring:
  application:
  	name: lagou-cloud-gateway
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: ${devDatasource.url}
    username: ${devDatasource.username}
    password: ${devDatasource.password}
  jpa:
    database: MySQL
    show-sql: true
    hibernate:
      naming:
        #避免将驼峰命名转换为下划线命名
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
  # config客户端配置,和ConfigServer通信，并告知ConfigServer希望获取的配置信息在哪个文件中
  cloud:
    gateway:
      routes: # 路由可以有多个
        - id: service-user-router
          uri: http://127.0.0.1:8080
#          uri: lb://lagou-service-user
          predicates:
            - Path=/api/user/**
        - id: service-code-router
          uri: http://127.0.0.1:8081
#          uri: lb://lagou-service-code
          predicates:
            - Path=/api/code/**
        - id: service-email-router
          uri: http://127.0.0.1:8082
#          uri: lb://lagou-service-email
          predicates:
            - Path=/api/email/**

ribbon:
  #请求连接超时时间
  ConnectTimeout: 5000
  #请求处理超时时间
  #Feign超时时长设置
  ReadTimeout: 5000
  #对所有操作都进行重试
  OkToRetryOnAllOperations: true
  ####根据如上配置，当访问到故障请求的时候，它会再尝试访问一次当前实例（次数由MaxAutoRetries配置），
  ####如果不行，就换一个实例进行访问，如果还不行，再换一次实例访问（更换次数由MaxAutoRetriesNextServer配置），
  ####如果依然不行，返回失败信息。
  MaxAutoRetries: 0 #对当前选中实例重试次数，不包括第一次调用
  MaxAutoRetriesNextServer: 0 #切换实例的重试次数
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #负载策略调整

# springboot中暴露健康检查等断点接口
management:
  endpoints:
    web:
      exposure:
        include: "*"
  # 暴露健康接口的细节
  endpoint:
    health:
      show-details: always
```

##### 4.3 启动类添加注解

```
@EnableDiscoveryClient	// 开启服务注册
@EnableFeignClients		// 开启Feign客服端
```













### 四.问题

问题一：使用Feign远程方法调用超时问题

异常信息：

```
Exception in thread "Thread-41" feign.RetryableException: Read timed out executing GET http://lagou-service-email/api/email/384515764%40qq.com/525139
	at feign.FeignException.errorExecuting(FeignException.java:84)
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:113)
	at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:78)
	at feign.ReflectiveFeign$FeignInvocationHandler.invoke(ReflectiveFeign.java:103)
	at com.sun.proxy.$Proxy125.sendEmail(Unknown Source)
	at com.fishleap.controller.CodeController.lambda$createCode$0(CodeController.java:56)
	at java.base/java.lang.Thread.run(Thread.java:834)
Caused by: java.net.SocketTimeoutException: Read timed out
	at java.base/java.net.SocketInputStream.socketRead0(Native Method)
```

按照下面配置后还是超时异常：

```yml
ribbon:
  #请求连接超时时间
  ConnectTimeout: 5000
  #请求处理超时时间
  #Feign超时时长设置
  ReadTimeout: 5000
  #对所有操作都进行重试
  OkToRetryOnAllOperations: true
  ####根据如上配置，当访问到故障请求的时候，它会再尝试访问一次当前实例（次数由MaxAutoRetries配置），
  ####如果不行，就换一个实例进行访问，如果还不行，再换一次实例访问（更换次数由MaxAutoRetriesNextServer配置），
  ####如果依然不行，返回失败信息。
  MaxAutoRetries: 0 #对当前选中实例重试次数，不包括第一次调用
  MaxAutoRetriesNextServer: 0 #切换实例的重试次数
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #负载策略调整
 
feign:
  client:
    config:
      default:
        connect-timeout: 20000
        read-timeout: 20000  
```

问题二：当服务器和客户端都在同一台电脑上出现时，Gateway全局过滤获取IP值，`0:0:0:0:0:0:0:1` 和 `127.0.0.1` IPv6和IPv4依次轮询请求

```
// IPv6转成IPv4格式
if (clientIp.equals("0:0:0:0:0:0:0:1")) {
clientIp = "127.0.0.1";
}
```



