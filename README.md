<p align="center">
<a href="http://itzlg.gitee.io/java-development-notes" target="_blank">
    <svg class="svgIcon" aria-hidden="true">
        <use xlink:href="#icon-huabanfuben"></use>
    </svg>
</a>
</p>

推荐使用  http://itzlg.gitee.io/java-development-notes 在线阅读，在线阅读内容本仓库同步一致。这种方式阅读的优势在于：阅读体验会更好。 


### 目录
  - [一.Java基础知识](#Java基础知识)
  - [二.Java高级知识](#Java高级知识)
  - [三.数据存储](#数据存储)
  - [四.常用框架](#常用框架)
  - [五.Web服务器](#Web服务器)
  - [六.分布式](#分布式)
  - [七.微服务](#微服务)
  - [八.高并发](#高并发)
  - [九.认证授权](#认证授权)
  - [十.必备工具](#必备工具)
  - [十一.前端](#前端)
  - [十二.操作系统和网络](#操作系统和网络)
  - [十三.数据结构与算法](#数据结构与算法)
  - [推荐资源](#推荐资源)



#### Java基础知识
  1. [Java概述,变量与运算符,流程控制](docs/javaBase/grammar.md)
  2. [一维数组,二维数组及Arrays工具类使用](docs/javaBase/array.md)
  3. [类与对象,类的成员及OOP特征](docs/javaBase/object01.md)
  4. [this,super,static,final,package,import关键字及其它重要知识点](docs/javaBase/object02.md)
  5. [异常体系概述,try,catch,finally,throw及throws关键字](docs/javaBase/exception.md)
  6. [String,StringBuffer与StringBuilder详解](docs/javaBase/commonClass01.md)
  7. [JDK8之前与JDK8日期时间API详解](docs/javaBase/commonClass02.md)
  8. [Comparable与Comparator接口,System,Math,BigDecimal类详解](docs/javaBase/commonClass03.md)
  9. [使用反射获取类的Class,Constructor,Method,Filed对象及泛型相关API信息](docs/javaSenior/reflection.md)
  10. [静态代理与动态代理介绍及相关案例](docs/javaSenior/dynamicProxy.md)
  11. [枚举概述及使用](docs/javaBase/enum.md)
  12. [自定义注解](docs/javaBase/自定义注解.md)


#### Java高级知识
  [Java底层知识点学习目录](docs/javaSenior/study.md)

**容器**
  1. [Collection,List,Set,Map集合及Collections工具类使用](docs/javaSenior/collection/basis.md)
  2. [ArrayList/LinkedList/HashMap源码学习](docs/javaSenior/collection/source.md)

**并发**
  1. [并发知识点长篇总结](docs/javaSenior/concurrence/conBasic01.md) &nbsp;&nbsp;&nbsp;
     [线程的实现方式,生命周期,重要API,通信](docs/javaSenior/concurrence/conBasic02.md)
  2. [并发的三大特性,Java内存模型,死锁](docs/javaSenior/concurrence/conPrinciple01.md)
  3. [Atomic相关类与CAS,Volatile,Synchronized详解](docs/javaSenior/concurrence/conPrinciple02.md)
  4. [深入解析ThreadLocal](docs/javaSenior/concurrence/threadLocalAndAQS01.md) &nbsp;&nbsp;&nbsp;
    [AQS解析](docs/javaSenior/concurrence/threadLocalAndAQS02.md)
  5. [线程池总结](docs/javaSenior/concurrence/threadPoolStudy.md) &nbsp;&nbsp;&nbsp;
    [线程池学习](docs/javaSenior/concurrence/threadPool.md)
  6. [深入解析Lock]()

**JVM**
  1. [JVM学习-01：JVM之体系结构和发展历程](docs/javaSenior/JVM/JVMLearn01.md)
  2. [JVM学习-02：JVM之类加载过程，类加载器及双亲委派机制](docs/javaSenior/JVM/JVMLearn02.md) 
  3. [JVM学习-03：JVM之运行时数据区、PC寄存器](docs/javaSenior/JVM/JVMLearn03.md) &nbsp;&nbsp;&nbsp;
     [虚拟机栈](docs/javaSenior/JVM/JVMLearn04.md) &nbsp;&nbsp;&nbsp;
     [本地方法与本地方法栈](docs/javaSenior/JVM/JVMLearn05.md) &nbsp;&nbsp;&nbsp;
     [虚拟机堆](docs/javaSenior/JVM/JVMLearn06.md)
  4. [Java内存区域](docs/javaSenior/JVM/memoryArea.md)

**其它**       
  1. [IO流](docs/javaSenior/ioStream.md)
  2. [JDK8](docs/javaSenior/JDK8.md)
  3. [网络](docs/javaSenior/network.md)

**编程规范**
  1. [Java编程规范学习](docs/javaSenior/codingStyle/codingStyle.md)



#### 操作系统和网络
**操作系统**
  1. [写给大忙人看的操作系统](docs/operatingSystem/os.md)
  2. [Shell编程基础入门](docs/operatingSystem/shell.md)

**计算机网络**
  1. [计算机网络基础知识总结](docs/operatingSystem/network.md)



#### 数据存储

**MySQL**
  1. [SQLSERVER基础](docs/database/mysql/sqlserveBase.md) &nbsp;&nbsp;&nbsp;
    [MySQL相关日期处理](docs/database/mysql/mysqlDateHandle.md) &nbsp;&nbsp;&nbsp;
    [MySQL行列转换](docs/database/mysql/mysqlUnpivot.md)
  2. [MySQL索引类型、索引原理、索引分析和优化、查询优化](docs/database/mysql/MySQL索引原理.md)
  3. [MySQL架构体系、事务和锁](docs/database/mysql/MySQL架构和事务日志.md)
  4. [MySQL架构设计、主从模式、双主模式、分库分表](docs/database/mysql/MySQL集群架构.md)
  5. [ShardingSphere中间件](docs/database/mysql/ShardingSphere中间件.md)
  6. [Mycat中间件](docs/database/mysql/Mycat中间件.md)
  7. [运维和第三方工具](docs/database/mysql/运维和第三方工具.md)
  8. [MySQL优化方案](docs/database/mysql/MySQL优化方案.md) &nbsp;&nbsp;&nbsp;
    [MySQL索引及高质量Sql建议](docs/database/mysql/sqlAdvise.md)

**MongoDB**

**FastDFS**

**OSS**

**HDFS**

**HBase**

**Oracle**
  1. [创建和管理表、其它数据库对象](docs/database/oracle/ddl.md)
  2. [DML语句相关语法、分析函数](docs/database/oracle/dml.md)
  3. [plsql基本语句、存储过程、触发器](docs/database/oracle/plsql.md)



#### 常用框架
  [Servlet,Cookie,Session,JSP,EL表达式,JSTL标签库,AJAX,Filter,Listener基础概念](docs/javaEE/jsp.md)

**Mybatis**
  1. [自定义持久层框架简化版](docs/framework/mybatis/mybatis00.md)
  2. [Mybatis基本应用](docs/framework/mybatis/mybatis01.md) &nbsp;&nbsp;&nbsp;
     [Mybatis缓存和插件介绍](docs/framework/mybatis/mybatis02.md)
  3. [Mybatis架构,执行流程和设计模式](docs/framework/mybatis/mybatis03.md) &nbsp;&nbsp;&nbsp;
     [Mybatis源码分析](docs/framework/mybatis/mybatis04.md)

**Spring**
  1. [Spring核心思想IOC,AOP概述及自定义解决思路](docs/framework/spring/spring核心思想概述.md)
  2. [Spring IOC应用](docs/framework/spring/springIOC应用.md) &nbsp;&nbsp;&nbsp;
     [Spring IOC容器源码分析](docs/framework/spring/springIOC源码分析.md)
  3. [Spring AOP应用](docs/framework/spring/springAOP应用.md) &nbsp;&nbsp;&nbsp;
     [Spring AOP源码解析](docs/framework/spring/springAOP源码分析.md)

**SpringMVC**
  1. [SpringMVC基本应用](docs/framework/springmvc/springMVC应用.md)
  2. [SpringMVC源码分析](docs/framework/springmvc/springMVC源码分析.md)
  3. [SSM整合策略](docs/framework/springmvc/SSM整合.md)

**SpringDataJPA**
  1. [SpringDataJPA基本应用](docs/framework/springdatajpa/springDataJPA基本应用.md)
  2. [SpringDataJPA执行过程源码分析](docs/framework/springdatajpa/springDataJPA执行过程源码分析)

**SpringBoot**
  1. [SpringBoot基础](docs/microService/springboot/springboot.md)
  2. [SpringBoot源码分析](docs/microService/springboot/springBoot源码分析.md)
  3. [SpringBoot数据访问](docs/microService/springboot/springBoot数据访问.md)
  4. [SpringBoot视图技术](docs/microService/springboot/springboot_thymeleaf.md)
  5. [SpringBoot缓存管理](docs/microService/springboot/springBoot缓存管理.md)  

**Netty**
  1. [Netty](#netty)


#### Web服务器
**Tomcat**

**Nginx**


#### 微服务
**SpringCloud**
  1. [微服务概念](docs/microService/springcloud/微服务概念.md)
  2. [Eureka服务注册中心](docs/microService/springcloud/Eureka服务注册中心.md)
  3. [Ribbon负载均衡](docs/microService/springcloud/Ribbon负载均衡.md)
  4. [Hystrix熔断器](docs/microService/springcloud/Hystrix熔断器.md)
  5. [Feign远程调用组件](docs/microService/springcloud/Feign远程调用组件.md)
  6. [GateWay网关](docs/microService/springcloud/GateWay网关.md)
  7. [Spring Cloud Config分布式配置中心](docs/microService/springcloud/SpringCloudConfig分布式配置中心.md)
  8. [Spring Cloud Stream消息驱动组件](docs/microService/springcloud/SpringCloudStream消息驱动组件.md)
  9. [Sleuth + Zipkin微服务之分布式链路追踪技术](docs/microService/springcloud/Sleuth+Zipkin分布式链路追踪技术.md)
  10. [Spring Cloud OAuth2 + JWT微服务统一认证方案](docs/microService/springcloud/OAuth2+JWT统一认证方案.md)
  11. [SCA Nacos服务注册和配置中心](docs/microService/springcloud/Nacos服务注册和配置中心.md)
  12. [SCA Sentinel分布式系统的流量防卫兵](docs/microService/springcloud/Sentinel流量防卫兵.md)
  13. [微服务SpringCloud长篇总结](docs/microService/springcloud/微服务SpringCloud长篇总结.md)


#### 高并发
**Redis**
  1. [缓存原理和设计](docs/highConcurrency/redis/缓存原理和设计.md)
  2. [数据类型与底层数据结构](docs/highConcurrency/redis/数据类型与底层数据结构.md)
  3. [通讯协议及事件处理机制](docs/highConcurrency/redis/通讯协议及事件处理机制.md)
  4. [Redis持久化](docs/highConcurrency/redis/Redis持久化.md)
  5. [发布与订阅、事务、Lua脚本、慢查询日志、监视器](docs/highConcurrency/redis/Redis扩展功能.md)
  6. [主从复制、哨兵模式、集群与分区](docs/highConcurrency/redis/高可用方案.md)
  7. [架构设计、缓存问题、缓存与数据库一致性、分布式锁、session分离、阿里Redis使用手册](docs/highConcurrency/redis/企业实战.md)

**RabbitMQ**
  1. [消息中间件概述](docs/highConcurrency/rabbitmq/消息中间件概述.md)
  2. [RabbitMQ概述、常用操作命令、工作流程与工作模式、SpringBoot整合RabbitMQ](docs/highConcurrency/rabbitmq/rabbitmq架构与实战.md)
  3. [消息可靠性及分析、TTL机制、死信队列、延迟队列](docs/highConcurrency/rabbitmq/rabbitmq高级特性.md)
  4. [RabbitMQ集群与运维](docs/highConcurrency/rabbitmq/rabbitmq集群与运维.md)

**Kafka**
  1. [Kafka安装与配置、生产与消费、生产者和消费者客户端开发及原理](docs/highConcurrency/kafka/初始Kafka.md)
  2. [Kafka主题与分区管理、日志存储](docs/highConcurrency/kafka/Kafka高级特性.md)

**RocketMQ**

**Elasticsearch**
  1. [Elasticsearch入门](docs/javaEE/elasticsearch/elasticsearch.md)


#### 分布式
**分布式架构**
  - [集群架构场景化解决方案:一致性hash算法,集群时钟同步,分布式ID,分布式调度及Session共享问题]()
  - [分布式架构理论:一致性,CAP定理,BASE定理,一致性协议(2PC,3PC)及一致性算法(Paxos,Raft)](docs/distribution/distributionTheory.md)
  - [分布式架构网络通信:BIO,NIO,AIO和Netty及自定义RPC](docs/distribution/network.md)  

**Zookeeper**
  - [Zookeeper基本应用](docs/distribution/zookeeperBasic.md)
  - [Zookeeper深入进阶](docs/distribution/zookeeperSenior.md)
  - [Zookeeper源码分析](docs/distribution/zookeeperSource.md)

**Dubbo**
  - [Dubbo基本应用](docs/distribution/dubboBasic.md)
  - [Dubbo源码分析](docs/distribution/dubboSenior.md)

#### 认证授权
  - [Cookie/Session/Token基础知识](#)
  - [JWT基础知识](#)
  - [SpringSecurity](#)
  - [Shiro](#shiro)
  - [SSO单点登录](#)


#### 必备工具
**Linux**
  1. [Linux概述及常用命令](docs/operatingSystem/linuxBasic.md)

**GIT**
  1. [Git入门使用](docs/tools/git/gitBasic.md) &nbsp;&nbsp;&nbsp;
     [GitHub简单使用](docs/tools/git/github.md)  
  2. [通俗易懂|用好Git和SVN,轻松驾驭版本管理](docs/tools/git/gitAndSvn.md)

**Docker**
  1. [Docker入门使用](docs/tools/docker/dockerBasic.md) 
  2. [Docker推荐文章](docs/tools/docker/dockerResources.md)


#### 前端
  - [HTML/CSS基础](docs/frontEnd/htmlCssBasic.md)
  - [JavaScript入门](#javascript)
  - jQuery
    - [jQuery基础](docs/frontEnd/jqueryBasic.md) &nbsp;
      [jQuery中Ajax](docs/frontEnd/jqueryAjax.md)
  - [Vue](#vue)
  - [Bootstrap](#bootstrap)
  - [Element](#element)
  - [Echarts](#echarts)

#### 数据结构与算法
  - [数据结构](#数据结构)
  - [算法](#算法)

#### 推荐资源
  - [Github上重要的几个搜索技巧](docs/GithubSkill.md)
  - [超实用网址,GitHub项目和常见面试题](docs/resource.md)
  - [architect-awesome开源资源](docs/resource2.md)
  - [推荐资源网址详细总结](docs/resourcelist.md)


### 待办
- [x] springboot(---正在进行中---)
- [ ] mysql


### 描述
<span style="font-size:20px;">**java-development-notes介绍**</span>

本文档倾向于提供 java 开发相关基础理念知识，用来记录自己学习 java 开发过程中的相关笔记。

<span style="font-size:20px;">**关于转载**</span>

如果你需要转载本仓库的一些文章到自己的博客的话，记得注明原文地址就可以了。
<br/>
<br/>
<br/>


<span id="busuanzi_container_site_pv" style="display: inline;">
    👁️本页总访问次数:<span id="busuanzi_value_site_pv"></span> 
</span>
<span id="busuanzi_container_site_uv" style="display: inline;"> 
    | 🧑总访客数: <span id="busuanzi_value_site_uv"></span>
</span>

<!-- <span style="font-size:20px;">**为什么要做这个开源文档？**</span>

初始想法源于自己一段比较迷茫的经历。想抽时间整理自己的一个 java 知识体系。主要目的是为了加强自己的基本功, 同时也希望能帮助正在学习 java 的小伙伴。 -->
