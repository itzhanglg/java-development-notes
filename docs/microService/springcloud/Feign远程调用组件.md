### 1.Feign概述

Feign是Netflix开发的一个轻量级RESTful的HTTP服务客户端，是以Java接口注解的方式调用Http请求，而不用像Java中通过封装HTTP请求报文的方式直接调用，Feign被广泛应用在Spring Cloud 的解决方案中。

- Feign不需要我们去拼接url然后调用restTemplate的api，在SpringCloud中，创建一个接口（消费者端）并在接口上添加相关注解
- SpringCloud对Feign进行了增强，使Feign支持SpringMVC注解（OpenFeign）

本质：封装了Http调用流程，面向接口化编程，类似Dubbo的服务调用。

### 2.Feign的应用

`Netflix` 的 `Open Feign` 是 `Spring` 弃用 `Ribbion` 的 `RestTemplate` 的替代方案。**在消费者端定义与服务端映射的接口**，然后你就可以**通过调用消费者端的接口方法来调用提供者端的服务**了(目标REST服务)，除了编写接口的定义，开发人员不需要编写其他调用服务的代码，是现在常用的方案。

引入依赖：

```xml
<dependency>    
    <groupId>org.springframework.cloud</groupId>    
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

根据需要选择添加Feign相关配置：

```yaml
feign:
  client:  
	config:   
	  default:   
		connectTimeout: 5000  # 指定Feign客户端连接提供者的超时时限,取决于网络环境 ************
		readTimeout: 5000   # 指定Feign客户端从请求到获取到提供者给出的响应的超时时限  取决于业务逻辑运算时间    ************
  compression:    
	request:      
	  enabled: true   # 开启对请求的压缩      
	  mime-types: text/xml,application/xml,application/json    
	  min-request-size: 2048   # 指定启用压缩的最小文件大小    
	response:      
	  enabled: true   # 开启对响应的压缩
  hystrix:	
    enabled: true  # 开启熔断功能
    command:
      default:
        circuitBreaker:
          sleepWindowInMilliseconds: 30000
          requestVolumeThreshold: 50
        execution:
          timeout:
            enabled: true
          isolation:
            strategy: SEMAPHORE
            semaphore:
              maxConcurrentRequests: 50
            thread:
              timeoutInMilliseconds: 100000  # 全局超时熔断时间    ************
            
ribbon:
  #请求连接超时时间    ************
  ConnectTimeout: 2000
  #请求处理超时时间    ************
  ReadTimeout: 5000
  #对所有操作都进⾏重试
  OkToRetryOnAllOperations: true
  ####根据如上配置，当访问到故障请求的时候，它会再尝试访问⼀次当前实例（次数由MaxAutoRetries配置），
  ####如果不行，就换一个实例进行访问，如果还不行，再换一次实例访问（更换次数由MaxAutoRetriesNextServer配置），
  ####如果依然不行，返回失败信息。
  MaxAutoRetries: 0 #对当前选中实例重试次数，不包括第⼀次调⽤
  MaxAutoRetriesNextServer: 0 #切换实例的重试次数
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule  #负载策略调整
```

启动类上添加 **@EnableFeignClients** 注解来启用 Feign 客户端

远程调用client service层：

```java
// 使用@FeignClient表示服务 这里的值是提供者的名称
// name属性用于指定调用服务提供者名称，和服务提供者yaml文件中spring.application.name保持一致
@FeignClient(name="provider-application")// 这里的值是提供者相应的路径
@RequestMapping("/provider")
public interface FeginService {
    // 这里的路径也要和提供者相同 参数也需要一样
    // 参数绑定时可以使用@PathVariable、@RequestParam、@RequestHeader等，value属性必选设置
    @GetMapping("/{id}")
    Map providerMethod(@PathVariable(value = "id") int id);
}
```

controller层：

```java
@RestController
public class FeignController {
    // 调用服务
    @Autowired
    private FeginService feginService;
    
    @RequestMapping(value = "/consumer/{id}")    
    public Map consumerTest(@PathVariable(value = "id")Integer id) {        
        return feginService.providerMethod(id);    
    }
}
```

#### Feign注解剖析

@FeignClient注解主要被@Target({ElementType.TYPE})修饰，表示该注解主要使用在接口上。它具备了如下的属性：

- **name**：指定FeignClient的名称，如果使用了Ribbon，name就作为微服务的名称，用于服务发现
- **path**：定义当前FeignClient的统一前缀
- **fallback**：定义容错的处理类，当调用远程接口失败或者超时时，会调用对应的接口的容错逻辑，fallback指定的类必须实现@Feign标记的接口
- fallbacjFactory：工厂类，用于生成fallback类实例，通过这个属性可以实现每个接口通用的容错逻辑们介绍重复的代码
- **configuration**：Feign配置类，可以自定或者配置Feign的Encoder，Decoder，LogLevel，Contract
- url：url一般用于调试，可以指定@FeignClient调用的地址
- decode404：当发生404错误时，如果该字段为true，会调用decoder进行解码，否则抛出FeignException

### 3.Feign超时设置

Feign的调用分两层，即Ribbon层的调用和Hystrix的调用，高版本的Hystrix默认是关闭的。

**对负载均衡ribbon的支持**：**Feign默认的请求处理超时时长1s，可以调整请求处理超时时长，Feign自己有超时设置，如果配置Ribbon的超时，则会以Ribbon的为准**。可以通过 `ribbon.xx` 来进行全局配置，也可以通过 `服务名.ribbon.xx` 来对指定服务进行细节配置。

![image-20200804203442447](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200804203442447.png)


若出现上面错误，需要添加下列配置：Feign的超时时长设置就是下面Ribbon的超时时长设置

```yaml
ribbon:
  #请求连接超时时间    ************
  ConnectTimeout: 3000
  #请求处理超时时间    ************
  ReadTimeout: 6000
```

**对熔断器Hystrix的支持**：开启Hystrix后，Feign中的方法都会被进行一个管理，一旦出现问题就进入对应的回退逻辑处理；当前有两个超时时间设置（Feign/hystrix），熔断的时候是根据这**两个时间的最小值**来进行的，即处理时长超过最短的那个超时时间了就熔断进入回退降级逻辑。

```yaml
feign:
  hystrix:
	enabled: true  # 开启Feign的熔断功能
```

如果开启了Hystrix，Hystrix的超时报错信息如下：

![image-20200804203348457](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200804203348457.png)

此时可以添加如下配置:

```yaml
feign:
  hystrix:
    command:
      default:
        circuitBreaker:
          sleepWindowInMilliseconds: 30000
          requestVolumeThreshold: 50
        execution:
          timeout:
            enabled: true
          isolation:
            strategy: SEMAPHORE
            semaphore:
              maxConcurrentRequests: 50
            thread:
              timeoutInMilliseconds: 16000  # 全局超时熔断时间    ************
```

自定义FallBack处理类（需要实现FeignClient接口）

```java
// 降级回退逻辑需要定义一个类，实现FeignClient接口，实现接口中的方法
@Component // 别忘了这个注解，还应该被扫描到
public class UserFallback implements UserServiceFeignClient {
    @Override
    public Integer findDefaultResumeState(Long userId) {
        return 6;
    }
}
```

在@FeignClient注解中添加 fallback 属性和值为 UserFallback.class。**注意**使用 **fallback** 属性时，类上的
`@RequestMapping` 注解的url前缀限定需要改成配置在@FeignClient的 **path** 属性中。

### 4.Feign的工作原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200804175606326.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)


主程序入口添加了@EnableFeignClients注解开启对FeignClient扫描加载处理。根据Feign Client的开发规范，定义接口并加@FeignClientd注解。

当程序启动时，回进行包扫描，扫描所有@FeignClients的注解的类，并且讲这些信息注入Spring IOC容器中，当定义的的Feign接口中的方法呗调用时，通过JDK的代理方式，来生成具体的RequestTemplate.当生成代理时，Feign会为每个接口方法创建一个RequestTemplate。当生成代理时，Feign会为每个接口方法创建一个RequestTemplate对象，改对象封装可HTTP请求需要的全部信息，如请求参数名，请求方法等信息都是在这个过程中确定的。

然后RequestTemplate生成Request,然后把Request交给Client去处理，这里指的时Client可以时JDK原生的URLConnection,Apache的HttpClient,也可以时OKhttp，最后Client被封装到LoadBalanceClient类，这个类结合Ribbon负载均衡发器服务之间的调用。

#### Feign开启GZIP压缩

Feign 支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗。通过下面的参数 即可开启请求
与响应的压缩功能：

```yaml
feign:
  compression:    
	request:      
	  enabled: true   # 开启对请求的压缩      
	  mime-types: text/xml,application/xml,application/json # 设置压缩的数据类型，默认值
	  min-request-size: 2048   # 指定启用压缩的最小文件大小    
	response:      
	  enabled: true   # 开启对响应的压缩
```

需要注意的是，在采用了压缩之后，需要使用二级制的方式进行数据传递，所有返回值就需要使用 ResponseEntity<byte[]> 接收：

```java
@FeignClient(name = "github-client")
public interface HelloFeignService {
    /*
    这个返回类型如果采用了压缩，那么就是二进制的方式，就需要使用ResponseEntity<byte[]>作为返回值
     */
    @RequestMapping(value = "/search/repositories",method = RequestMethod.GET)
    ResponseEntity<byte[]> searchRepositories(@RequestParam("q")String parameter);
}
```

#### Feign开启日志

Feign是http请求客户端，类似浏览器，请求和接受响应时可以打印出详细的一些日志信息（响应头、状态码）。默认Feign的日志没有开启，需要在注解类添加配置的类：

```java
@FeignClient(name = "github-client",configuration = FeignConfig.class)
```

注解类的代码如下：

```java
// Feign的日志级别（Feign请求过程信息）
// NONE：默认的，不显示任何日志----性能最好
// BASIC：仅记录请求方法、URL、响应状态码以及执行时间----生产问题追踪
// HEADERS：在BASIC级别的基础上，记录请求和响应的header
// FULL：记录请求和响应的header、body和元数据----适用于开发及测试环境定位问题
@Configuration
public class FeignConfig {

    /**
     * Logger.Level 的具体级别如下：
       NONE：不记录任何信息
       BASIC：仅记录请求方法、URL以及响应状态码和执行时间
       HEADERS：除了记录 BASIC级别的信息外，还会记录请求和响应的头信息
       FULL：记录所有请求与响应的明细，包括头信息、请求体、元数据
     * @return
     */
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }

}
```

配置log日志级别为debug：

```yaml
logging:
  level:
    # Feign日志只会对日志级别为debug的做出响应
	com.fishleap.client.UserServiceFeignClient: debug
```

### 参考

- [Spring Cloud OpenFeign 源码解析](https://juejin.im/post/6844904066229927950)
- [Feign的工作原理](https://juejin.im/post/6844903837543694349)
- [Feign源码分析:记初次使用Feign踩的一些坑](https://juejin.im/post/6844904018850873358)



