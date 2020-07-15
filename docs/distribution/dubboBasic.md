### 一.Dubbo概述及配置项说明

#### 1.什么是Dubbo

Apache Dubbo 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。官网：[DUBBO](http://dubbo.apache.org/zh-cn/index.html)

![image-20200713043837437](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200713043837437.png)

**节点说明**：

- Provider：暴露服务的服务提供方
- Consumer：调用远程服务的服务消费方
- Registry：服务注册与发现的注册中心
- Monitor： 统计服务的调用次数和调用时间的监控中心
- Container：服务运行容器

**调用过程及工作原理**：

**1.** 服务容器负责启动，加载，运行服务提供者，通过 main 函数初始化 Spring 上下文，根据服务提供者配置的XML文件将服务按照指定的协议发布，完成服务化的初始化工作。。

**2** 服务提供者在启动时，根据配置的服务注册中心地址连接服务注册中心，将服务提供者信息发布到注册中心，向注册中心注册自己提供的服务。

**3.** 服务消费者在启动时，消费者根据服务消费者XML配置文件的服务引用信息，连接到注册中心，向注册中心订阅自己所需的服务。

**4.** 服务注册中心根据服务订阅的关系，返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送最新的服务地址信息给消费者。

**5.** 服务消费者调用远程服务时，根据路由策略，从本地缓存的服务提供者地址列表中选择选一台提供者进行，然后根据协议类型建立链路，跨进程调用服务提供者，如果调用失败，再选另一台调用。

**6.** 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

**特性一览**：

![image-20200713044515290](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200713044515290.png)

#### 2.配置项说明

[schema 配置参考手册](http://dubbo.apache.org/zh-cn/docs/user/references/xml/introduction.html)

##### ymal或properties文件常用配置项

- dubbo.application.name: 当前提供者的名称
- dubbo.registry.address=zookeeper://127.0.0.1:2181：服务注册地址
- dubbo.protocol.name: 对外提供的时候使用的协议
- dubbo.protocol.port: 该服务对外暴露的端口是什么，在消费者使用时，则会使用这个端口
    并且使用指定的协议与提供者建立连接

##### `dubbo:application` 

对应 org.apache.dubbo.config.ApplicationConfig, 代表当前应用的信息

1. name: 当前应用程序的名称，在dubbo-admin中我们也可以看到，这个代表这个应用名称。我们在真正时是时也会根据这个参数来进行聚合应用请求。
2. owner: 当前应用程序的负责人，可以通过这个负责人找到其相关的应用列表，用于快速定位到责任人。
3. qosEnable : 是否启动QoS 默认true
4. qosPort : 启动QoS绑定的端口 默认22222
5. qosAcceptForeignIp: 是否允许远程访问 默认是false

##### `dubbo:registry` 

org.apache.dubbo.config.RegistryConfig, 代表该模块所使用的注册中心。一个模块中的服务可以将其注册到多个注册中心上，也可以注册到一个上。后面再service和reference也会引入这个注册中心。

1. id : 当当前服务中provider或者consumer中存在多个注册中心时，则使用需要增加该配置。在一些公司，会通过业务线的不同选择不同的注册中心，所以一般都会配置该值。
2. address : 当前注册中心的访问地址。
3. protocol : 当前注册中心所使用的协议是什么。也可以直接在 address 中写入，比如使用zookeeper，就可以写成 zookeeper://xx.xx.xx.xx:2181
4. timeout : 当与注册中心不再同一个机房时，大多会把该参数延长。

##### `dubbo:protocol` 

org.apache.dubbo.config.ProtocolConfig, 指定服务在进行数据传输所使用的协议。

1. id : 在大公司，可能因为各个部门技术栈不同，所以可能会选择使用不同的协议进行交互。这里
  在多个协议使用时，需要指定。
2. name : 指定协议名称。默认使用 dubbo 。

##### `dubbo:service` 

org.apache.dubbo.config.ServiceConfig, 用于指定当前需要对外暴露的服务信息，后面也会具体讲
解。和 dubbo:reference 大致相同。
1. interface : 指定当前需要进行对外暴露的接口是什么。
2. ref : 具体实现对象的引用，一般我们在生产级别都是使用Spring去进行Bean托管的，所以这里面一般也指的是Spring中的BeanId。
3. version : 对外暴露的版本号。不同的版本号，消费者在消费的时候只会根据固定的版本号进行消费。

##### `dubbo:reference` 

org.apache.dubbo.config.ReferenceConfig, 消费者的配置。

1. id : 指定该Bean在注册到Spring中的id。
2. interface: 服务接口名
3. version : 指定当前服务版本，与服务提供者的版本一致。
4. registry : 指定所具体使用的注册中心地址。这里面也就是使用上面在 dubbo:registry 中所声明的id。

##### `dubbo:method` 

org.apache.dubbo.config.MethodConfig, 用于在制定的 dubbo:service 或者 dubbo:reference 中的更具体一个层级，指定具体方法级别在进行RPC操作时候的配置，可以理解为对这上面层级中的配置针对于具体方法的特殊处理。

1. name : 指定方法名称，用于对这个方法名称的RPC调用进行特殊配置。
2. async: 是否异步 默认false

##### `dubbo:service`和`dubbo:reference`详解

这两个在dubbo中是我们最为常用的部分，其中有一些我们必然会接触到的属性。并且这里会讲到一些设置上的使用方案。

1. mock: 用于在方法调用出现错误时，当做服务降级来统一对外返回结果，后面我们也会对这个方
法做更多的介绍。
2. timeout: 用于指定当前方法或者接口中所有方法的超时时间。我们一般都会根据提供者的时长来具体规定。比如我们在进行第三方服务依赖时可能会对接口的时长做放宽，防止第三方服务不稳定导致服务受损。
3. check: 用于在启动时，检查生产者是否有该服务。我们一般都会将这个值设置为false，不让其进行检查。因为如果出现模块之间循环引用的话，那么则可能会出现相互依赖，都进行check的话，
那么这两个服务永远也启动不起来。
4. retries: 用于指定当前服务在执行时出现错误或者超时时的重试机制。
5. 注意提供者是否有幂等，否则可能出现数据一致性问题
6. 注意提供者是否有类似缓存机制，如出现大面积错误时，可能因为不停重试导致雪崩
7. executes: 用于在提供者做配置，来确保最大的并行度。
8. 可能导致集群功能无法充分利用或者堵塞
9. 但是也可以启动部分对应用的保护功能
10. 可以不做配置，结合后面的熔断限流使用

#### 3.Dubbo管理控制台

官网：[dubbo-admin](https://github.com/apache/dubbo-admin/tree/master)

主要包含：服务管理 、 路由规则、动态配置、服务降级、访问控制、权重调整、负载均衡等管理功能如我们在开发时，需要知道Zookeeper注册中心都注册了哪些服务，有哪些消费者来消费这些服务。我们可以通过部署一个管理中心来实现。其实管理中心就是一个web应用，原来是war(2.6版本以前)包需要部署到tomcat即可。现在是jar包可以直接通过java命令运行。

安装步骤：

```java
1.从git 上下载项目 https://github.com/apache/dubbo-admin
2.修改项目下的dubbo.properties文件
注意dubbo.registry.address对应的值需要对应当前使用的Zookeeper的ip地址和端口号
• dubbo.registry.address=zookeeper://zk所在机器ip:zk端口
• dubbo.admin.root.password=root
• dubbo.admin.guest.password=guest
3.切换到项目所在的路径 使用mvn 打包
mvn clean package -Dmaven.test.skip=true
4.java 命令运行
java -jar 对应的jar包
```

使用控制台：

```
1.访问http://IP:端口
2.输入用户名root,密码root
3.点击菜单查看服务提供者和服务消费者信息 
```

### 二.SPI机制

如何优雅的根据一个接口来获取该接口的所有实现类呢？

JDK SPI 正是为了优雅解决这个问题而生，**SPI 全称为 (Service Provider Interface)，即服务提供商接口**，是JDK内置的一种服务提供发现机制。目前有不少框架用它来做服务的扩展发现，**简单来说，它就是一种动态替换服务实现者的机制**。

所以，Dubbo如此被广泛接纳的其中的 **一个重要原因就是基于SPI实现的强大灵活的扩展机制**，开发者可自定义插件嵌入Dubbo，实现灵活的业务需求。

#### Java SPI

JDK为SPI的实现提供了工具类，即java.util.ServiceLoader，ServiceLoader中定义的SPI规范没有什么特别之处，**只需要有一个提供者配置文件（provider-configuration file），该文件需要在resource目录`META-INF/services`下，文件名就是服务接口的全限定名**。

1. **文件内容是提供者Class的全限定名列表**，显然提供者Class都应该实现服务接口；
2. **文件必须使用UTF-8编码**；

缺点：

虽然ServiceLoader也算是使用的延迟加载，**但是只能通过遍历获取，也就是遍历的时候，接口的实现类会全部加载并实例化一遍**。如果你并不想用某些实现类，它也被加载并实例化了，这就造成了浪费。

获取某个实现类的方式不够灵活，**只能通过Iterator形式获取**，不能根据某个参数来获取对应的实现类。

#### Dubbo SPI

Dubbo对JDK SPI进行了扩展，对服务提供者配置文件中的内容进行了改造，**由原来的提供者类的全限定名列表改成了KV形式的列表，这也导致了Dubbo中无法直接使用JDK ServiceLoader**，所以，与之对应的，在Dubbo中有ExtensionLoader。

**ExtensionLoader是扩展点载入器，用于载入Dubbo中的各种可配置组件**，比如：负载均衡策略（LoadBalance）、拦截器（Filter）、集群方式（Cluster）等。

总之，Dubbo为了应对各种场景，**它的所有内部组件都是通过这种SPI的方式来管理的**，这也是为什么Dubbo需要将服务提供者配置文件设计成KV键值对形式，**这个K就是我们在Dubbo配置文件或注解中用到的K，Dubbo直接通过服务接口（上面提到的ProxyFactory、LoadBalance、Protocol、Filter等）和配置的K从ExtensionLoader拿到服务提供的实现类**。

SPI 全称为 Service Provider Interface，是一种服务发现机制。SPI 的本质是将**接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类。这样可以在运行时，动态为接口替换实现类**。正因此特性，我们可以很容易的通过 SPI 机制为我们的程序提供拓展功能。SPI 机制在第三方框架中也有所应用，比如 Dubbo 就是通过 SPI 机制加载所有的组件。不过，Dubbo 并未使用 Java 原生的 SPI 机制，而是对其进行了增强，使其能够更好的满足需求。在 Dubbo 中，SPI 是一个非常重要的模块。基于 SPI，我们可以很容易的对 Dubbo 进行拓展。如果大家想要学习 Dubbo 的源码，SPI 机制务必弄懂。

#### 扩展功能介绍

Dubbo对SPI的扩展是 **通过ExtensionLoader来实现的**。

**Dubbo通过SPI注解定义了可扩展的接口**，如LoadBalance、Filter、Transporter等。**每个类型的扩展对应一个ExtensionLoader。SPI的value参数决定了默认的扩展实现**。

查看ExtensionLoader的源码，可以看到Dubbo对JDK SPI 做了三个方面的扩展：

- **方便获取扩展实现**：JDK SPI仅仅通过接口类名获取所有实现，**而ExtensionLoader则通过接口类名和key值获取一个实现**；
- **IOC依赖注入功能：Adaptive实现**，就是生成一个代理类，**这样就可以根据实际调用时的一些参数动态决定要调用的类了**。
    - 举例来说：接口A，实现者A1、A2。接口B，实现者B1、B2。
      现在实现者A1含有setB()方法，会自动注入一个接口B的实现者，此时注入B1还是B2呢？都不是，**而是注入一个动态生成的接口B的实现者 B$Adpative，该实现者能够根据参数的不同，自动引用B1或者B2来完成相应的功能**；
- **采用装饰器模式进行功能增强，自动包装实现，这种实现的类一般是自动激活的**，常用于包装类，比如：Protocol的两个实现类：ProtocolFilterWrapper、ProtocolListenerWrapper。
    - 还是第2个的例子，**接口A的另一个实现者AWrapper1**。大体内容如下：

        ```java
        private A a;
        AWrapper1（A a）{
           this.a=a;
        }
        ```

        因此，**当在获取某一个接口A的实现者A1的时候，已经自动被AWrapper1包装了**。

#### Dubbo SPI中的Adaptive功能

Dubbo中的Adaptive功能，主要解决的问题是如何动态的选择具体的扩展点。通过getAdaptiveExtension 统一对指定接口对应的所有扩展点进行封装，通过URL的方式对扩展点来进行动态选择。 (dubbo中所有的注册信息都是通过URL的形式进行处理的。)这里同样采用相同的方式进行实现。

（1）创建接口
api中的 HelloService 扩展如下方法, 与原先类似，在sayHello中增加 Adaptive 注解，并且在参数中提供URL参数.注意这里的URL参数的类为 `org.apache.dubbo.common.URL` ，其中@SP可以指定一个字符串参数，用于指明该SPI的默认实现。
（2）创建实现类
与上面Service实现类代码相似，只需增加URL形参即可。
（3）编写DubboAdaptiveMain
最后在获取的时候方式有所改变，需要传入URL参数，并且在参数中指定具体的实现类参数。

如：

```java
public class DubboAdaptiveMain {
  public static void main(String[] args) {
    URL url = URL.valueOf("test://localhost/hello?hello.service=dog");
    final HelloService adaptiveExtension =
ExtensionLoader.getExtensionLoader(HelloService.class).getAdaptiveExtension();
    adaptiveExtension.sayHello(url);
 }
}
```

注意：

- 因为在这里只是临时测试，所以为了保证URL规范，前面的信息均为测试值即可，关键的点在于hello.service 参数，这个参数的值指定的就是具体的实现方式。关于为什么叫hello.service 是因为这个接口的名称，其中后面的大写部分被dubbo自动转码为 . 分割。
- 通过 getAdaptiveExtension 来提供一个统一的类来对所有的扩展点提供支持(底层对所有的扩展点进行封装)。
- 调用时通过参数中增加 URL 对象来实现动态的扩展点使用。
- 如果URL没有提供该参数，则该方法会使用默认在 SPI 注解中声明的实现。

#### Dubbo调用时拦截操作

Dubbo也存在拦截（过滤）机制，可以通过该机制在执行目标程序前后执行我们指定的代码。

Dubbo的Filter机制，是专门为服务提供方和服务消费方调用过程进行拦截设计的，每次远程方法执行，该拦截都会被执行。这样就为开发者提供了非常方便的扩展性，比如为dubbo接口实现ip白名单功能、监控功能 、日志记录等。

步骤如下：

- 实现 ` org.apache.dubbo.rpc.Filter`接口
- 使用 org.apache.dubbo.common.extension.Activate 接口进行对类进行注册 通过group 可以指定生产端 消费端 如: `@Activate(group = {CommonConstants.CONSUMER)`
- 计算方法运行时间的代码实现
- 在 META-INF.dubbo 中新建 org.apache.dubbo.rpc.Filter 文件，并将当前类的全名写入: `timerFilter=包名.过滤器的名字`

注意：一般类似于这样的功能都是单独开发依赖的，所以再使用方的项目中只需要引入依赖，在调用接口时，该方法便会自动拦截。



### 推荐链接

- [dubbo实现原理简单介绍](https://www.cnblogs.com/steven520213/p/7606598.html)
- [dubbo的底层原理](https://blog.csdn.net/qq_33101675/article/details/78701305)
- [Dubbo原理及源码解析](https://segmentfault.com/blog/huangyuan_dubbo)
- [Dubbo篇之实现原理及架构详解](https://crazyfzw.github.io/2018/06/10/dubbo-architecture/)

