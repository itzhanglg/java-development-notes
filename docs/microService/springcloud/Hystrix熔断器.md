Hystrix 组件是 SpringCloud 提供的服务保护框架。又称断路器，俗称“豪猪”，具有自我保护能力。

#### 1.雪崩效应

连环雪崩效应：微服务中一个请求可能需要多个微服务接口才能实现，会形成复杂的调用链路。当扇出的链路上某个微服务的响应时间过长或不可用，导致所有的请求都处于延迟状态，若延迟到达系统最大值，可能会使系统资源耗尽，导致整个分布式系统都不可用。

- 扇入：代表该微服务被调用的次数，次数越大，该模块的复用性越好
- 扇出：代表该微服务调用其他微服务的个数，个数越多，该模块业务就越复杂

Hystrix保证服务高可用的3个技术：服务熔断、服务降级、线程隔离

- 服务熔断：当一个服务在一定时间内，接口被调用**失败次数**达到一个阈值或百分比，会开启熔断器，在服务熔断期间，服务会拒绝客户端访问开启熔断的接口。
- 服务降级：如果接口设置了降级方法，当调用接口失败、超时、拒绝访问，则会触发调用降级方法并返回结果给用户
- 线程隔离：对接口另外创建线程池。对调用频率高的接口，防止该接口把 web 容器的线程池消耗光，对该接口进行线程隔离。

通过服务熔断和接口降级，当接口调用失败、超时、被拒，能够及时返回友好提示给客户端，同时也减轻服务压力，保证服务的高可用。服务熔断一般是某个服务故障引起，而服务降级一般是从整体负荷考虑；服务降级一定会出现服务熔断，而服务熔断不一定会出现服务降级。通过线程隔离，解决雪崩效应和连环雪崩效应，保证服务的高可用。

Hystrix 比较消耗服务器资源，所以仅给服务中并发量较高的接口开启 Hystrix 保护即可。Hystrix 通过 @HystrixCommand() 给接口进行增强。@HystrixCommand() 内部可以配置熔断时间等参数，**@HystrixCommand() 中配置的参数会覆盖 application.yml 中配置的  Hystrix 参数**。

#### 2.Hystrix应用和相关配置

2.1 引入依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2.2 全局配置：

```yaml
hystrix:
  command: #用于控制HystrixCommand的行为
    default:
      execution:
        isolation:
          strategy: THREAD #控制HystrixCommand的隔离策略，THREAD->线程池隔离策略(默认)，SEMAPHORE->信号量隔离策略
          thread:
            timeoutInMilliseconds: 1000 #配置HystrixCommand执行的超时时间，执行超过该时间会进行服务降级处理，默认1s
            interruptOnTimeout: true #配置HystrixCommand执行超时的时候是否要中断
            interruptOnCancel: true #配置HystrixCommand执行被取消的时候是否要中断
          timeout:
            enabled: true #配置HystrixCommand的执行是否启用超时时间
          semaphore:
            maxConcurrentRequests: 10 #当使用信号量隔离策略时，用来控制并发量的大小，超过该并发量的请求会被拒绝。最大并发请求，默认为：10 条。当该接口已经接收 10 条请求在处理，后续请求触发熔断
      fallback:
        enabled: true #用于控制是否启用服务降级
      circuitBreaker: #用于控制HystrixCircuitBreaker的行为
        enabled: true #用于控制断路器是否跟踪健康状况以及熔断请求
        requestVolumeThreshold: 20 #窗口时间内允许接口调用报错的请求数，默认20
        errorThresholdPercentage: 50 # 窗口时间内允许接口调用报错占总请求数的百分比。默认：50%
        sleepWindowInMilliseconds: 5000 # 熔断后休眠时⻓。默认：5000ms
        forceOpen: false #强制打开断路器，拒绝所有请求
        forceClosed: false #强制关闭断路器，接收所有请求
      requestCache:
        enabled: true #用于控制是否开启请求缓存
      metrics:
        rollingStats:
          ### 默认10秒钟内，请求次数达到20个，并且失败率在50%以上就跳闸，跳闸后活动窗⼝为5s
          ### 窗口时间，默认：10000ms，即 10s。timeInMilliseconds/numBuckets
          ### 每隔 1s 就会统计这 1s 内请求：success、failure、timeout、rejection 请求的数量
          timeInMilliseconds: 10000
          ### 桶的数量。默认：10。设置 numBuckets 的值时，保证 timeInMilliseconds % numBuckets == 0 即可
          numBuckets: 10  
  collapser: #用于控制HystrixCollapser的执行行为
    default:
      maxRequestsInBatch: 100 #控制一次合并请求合并的最大请求数
      timerDelayinMilliseconds: 10 #控制多少毫秒内的请求会被合并成一个
      requestCache:
        enabled: true #控制合并请求是否开启缓存
  threadpool: #用于控制HystrixCommand执行所在线程池的行为
    default:
      coreSize: 10 #线程池的核心线程数，默认10
      maximumSize: 10 #线程池的最大线程数，超过该线程数的请求会被拒绝
      maxQueueSize: -1 #用于设置线程池的最大队列大小，-1采用SynchronousQueue，其他正数采用LinkedBlockingQueue
      queueSizeRejectionThreshold: 5 #用于设置线程池队列的拒绝阀值，由于LinkedBlockingQueue不能动态改版大小，使用时需要用该参数来控制线程数，默认5
      
# springboot中暴露健康检查等断点接⼝观察跳闸状态
management:
  endpoints:
	web:
	  exposure:
		include: "*"
  # 暴露健康接⼝的细节
  endpoint:
	health:
	  show-details: always  
### 访问健康检查接⼝：http://localhost:8090/actuator/health     
```

实例配置：实例配置只需要将全局配置中的default换成与之对应的key即可

```yaml
hystrix:
  command:
    HystrixComandKey: #将default换成HystrixComrnandKey
      execution:
        isolation:
          strategy: THREAD
  collapser:
    HystrixCollapserKey: #将default换成HystrixCollapserKey
      maxRequestsInBatch: 100
  threadpool:
    HystrixThreadPoolKey: #将default换成HystrixThreadPoolKey
      coreSize: 10
```

配置文件中相关key的说明:

- HystrixComandKey对应@HystrixCommand中的commandKey属性；
- HystrixCollapserKey对应@HystrixCollapser注解中的collapserKey属性；
- HystrixThreadPoolKey对应@HystrixCommand中的threadPoolKey属性。

2.3 启动类上添加注解 **@EnableCircuitBreaker** 来开启Hystrix的断路器功能。`@EnableCircuitBreaker  `，`@EnableDiscoveryClient`，`@SpringBootApplication` 这三个注解可以使用**@SpringCloudApplication**来替代。

2.4 给接口添加Hystrix修饰，增强接口的高可用性。使用 `@HystrixCommand()` 注解。**注意：失败调用的方法方法参数和返回值必须和原方法相同。服务降级只作用在消费端**。

```java
@RestController
public class MemberController {

    // 1. 设置回调方法；设置忽略的异常，如果是报这个异常，Hystrix 不会进行捕获并执行降级方法，接口会直接抛出异常
    @HystrixCommand(fallbackMethod = "inserMemberFallback", ignoreExceptions = 
                    {ArithmeticException.class})
    @GetMapping("/member/insert")
    public String insertMember() {
        int i = 10/0;
        return "添加会员信息成功！";
    }

    // 2.设置 commandProperties 参数，设置窗口时间内允许的错误请求阈值，错误请求百分比，熔断器持续时间配置；
    // 设置 threadPoolProperties 参数，设置线程隔离的线程池线程数，最大等待请求数
    @HystrixCommand(
        fallbackMethod = "deleteMemberFallback",
        commandProperties = {
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = 
                             "5"),
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value 
                             ="30000"),
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value 
                             ="60")
            }
    )
    @GetMapping("/member/delete/{id}")
    public String deleteMember(@PathVariable("id") String id) {
        if("1".equals(id)) {
            int i = 10/0;
        }
        return "删除会员信息成功";
    }
    
	@HystrixCommand(
		// 线程池标识，要保持唯⼀，不唯⼀的话就共⽤了
    	threadPoolKey = "findResumeOpenStateTimeoutFallback",
        // 线程池细节属性配置
    	threadPoolProperties = {
    		@HystrixProperty(name="coreSize",value = "1"), // 线程数
    		@HystrixProperty(name="maxQueueSize",value="20") // 等待队列⻓度
    	},
    	// commandProperties熔断的⼀些细节属性配置
    	commandProperties = {
    		// 每⼀个属性都是⼀个HystrixProperty
    		@HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",
							value="2000"),
    		// hystrix⾼级配置，定制⼯作过程细节
            // 8秒钟内，请求次数达到2个，并且失败率在50%以上就跳闸，跳闸后活动窗⼝设置为3s
    		// 统计时间窗⼝定义
    		@HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds",value = 
                             "8000"),
            // 统计时间窗⼝内的最⼩请求数
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = 
                             "2"),
            // 统计时间窗⼝内的错误数量百分⽐阈值
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = 
                             "50"),
            // ⾃我修复时的活动窗⼝⻓度
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = 
                             "3000")
    	},
    	fallbackMethod = "myFallBack" // 回退⽅法
    )
    @GetMapping("/checkStateTimeoutFallback/{userId}")
	public Integer findResumeOpenStateTimeoutFallback(@PathVariable Long userId) {
		// 使⽤ribbon不需要我们⾃⼰获取服务实例然后选择⼀个那么去访问了（⾃⼰的负载均衡）
		String url = "http://service-resume/resume/openstate/" + userId;
		Integer forObject = restTemplate.getForObject(url, Integer.class);
		return forObject;
	}
    
    // 注意：该⽅法形参和返回值与原始⽅法保持⼀致
    public Integer myFallBack(Long userId) {
    	return -123333; // 兜底数据
    }

    public String inserMemberFallback() {
        return "调用 insertMember() 失败，执行降级方法！";
    }

    public String deleteMemberFallback(String id) {
        return "调用 deleteMember() 失败，执行降级方法！";
    }
}
```

#### 3.HystrixCommand注解详解

@HystrixCommand中的常用参数：

- fallbackMethod：指定服务降级处理方法；
- ignoreExceptions：忽略某些异常，不发生服务降级；
- commandKey：命令名称，用于区分不同的命令；
- groupKey：分组名称，Hystrix会根据不同的分组来统计命令的告警及仪表盘信息；
- threadPoolKey：线程池名称，用于划分线程池。

测试案例：

```java
// 设置命令、分组及线程池名称
@HystrixCommand(fallbackMethod = "getDefaultUser",
    commandKey = "getUserCommand",
    groupKey = "getUserGroup",
    threadPoolKey = "getUserThreadPool")
public CommonResult getUserCommand(@PathVariable Long id) {
    return restTemplate.getForObject(userServiceUrl + "/user/{1}", CommonResult.class, id);
}

// 使用ignoreExceptions忽略某些异常降级
@HystrixCommand(fallbackMethod = "getDefaultUser2", ignoreExceptions = {NullPointerException.class})
public CommonResult getUserException(Long id) {
    if (id == 1) {
        throw new IndexOutOfBoundsException();
    } else if (id == 2) {
        throw new NullPointerException();
    }
    return restTemplate.getForObject(userServiceUrl + "/user/{1}", CommonResult.class, id);
}

public CommonResult getDefaultUser(@PathVariable Long id) {
    User defaultUser = new User(-1L, "defaultUser", "123456");
    return new CommonResult<>(defaultUser);
}

public CommonResult getDefaultUser2(@PathVariable Long id, Throwable e) {
    LOGGER.error("getDefaultUser2 id:{},throwable class:{}", id, e.getClass());
    User defaultUser = new User(-2L, "defaultUser2", "123456");
    return new CommonResult<>(defaultUser);
}
```

#### 4.Hystrix的请求缓存

当系统并发量越来越大时，我们需要使用缓存来优化系统，达到减轻并发请求线程数，提供响应速度的效果。

相关注解：

- @CacheResult：开启缓存，默认所有参数作为缓存的key，cacheKeyMethod可以通过返回String类型的方法指定key；
- @CacheKey：指定缓存的key，可以指定参数或指定参数中的属性值为缓存key，cacheKeyMethod还可以通过返回String类型的方法指定；
- @CacheRemove：移除缓存，需要指定commandKey。

测试案例：

```java
@CacheResult(cacheKeyMethod = "getCacheKey")
@HystrixCommand(fallbackMethod = "getDefaultUser", commandKey = "getUserCache")
    public CommonResult getUserCache(Long id) {
    LOGGER.info("getUserCache id:{}", id);
    return restTemplate.getForObject(userServiceUrl + "/user/{1}", CommonResult.class, id);
}

/**
 * 为缓存生成key的方法
 */
public String getCacheKey(Long id) {
    return String.valueOf(id);
}

// 远程微服务被调用方法
@GetMapping("/testCache/{id}")
public CommonResult testCache(@PathVariable Long id) {
    userService.getUserCache(id);
    userService.getUserCache(id);
    userService.getUserCache(id);
    return new CommonResult("操作成功", 200);
}
```

上面远程调用testCache方法，里面查询了3次id值，但在控制台里只打印了一次日志，即有两次走的是缓存。

在缓存使用过程中，我们需要在每次使用缓存的请求前后对**HystrixRequestContext**进行初始化和关闭，否则会出现如下异常：

```java
java.lang.IllegalStateException: Request caching is not available. Maybe you need to initialize the HystrixRequestContext?
	at com.netflix.hystrix.HystrixRequestCache.get(HystrixRequestCache.java:104) ~[hystrix-core-1.5.18.jar:1.5.18]
	at com.netflix.hystrix.AbstractCommand$7.call(AbstractCommand.java:478) ~[hystrix-core-1.5.18.jar:1.5.18]
	at com.netflix.hystrix.AbstractCommand$7.call(AbstractCommand.java:454) ~[hystrix-core-1.5.18.jar:1.5.18]
```

可以使用过滤器，在每个请求前后初始化和关闭HystrixRequestContext：

```java
@Component
@WebFilter(urlPatterns = "/*",asyncSupported = true)
public class HystrixRequestContextFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HystrixRequestContext context = HystrixRequestContext.initializeContext();
        try {
            filterChain.doFilter(servletRequest, servletResponse);
        } finally {
            context.close();
        }
    }
}
```

#### 参考

- [Spring Cloud Hystrix：服务容错保护](https://juejin.im/post/6844903945026928654)
- [Hystrix Dashboard：断路器执行监控](https://juejin.im/post/6844903951179972622)
- [Hystrix 超时配置的N种玩法](https://juejin.im/post/6844903855545663502)
- [带你走进 SpringCloud2.0（五）：Hystrix](https://juejin.im/post/6844904176032432141)
- [Hystrix Dashboard：断路器执行监控](https://juejin.im/post/6844903951179972622)



