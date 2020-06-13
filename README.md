<p align="center">
<a href="http://itzlg.gitee.io/java-development-notes" target="_blank">
    <svg class="svgIcon" aria-hidden="true">
        <use xlink:href="#icon-huabanfuben"></use>
    </svg>
</a>
</p>

推荐使用  http://itzlg.gitee.io/java-development-notes 在线阅读，在线阅读内容本仓库同步一致。这种方式阅读的优势在于：阅读体验会更好。 


### 目录
  - [Java基础知识](#Java基础知识)
  - [Java高级知识](#Java高级知识)
  - [数据库](#数据库)
  - [常用框架](#常用框架)
  - [微服务](#微服务)
  - [操作系统](#操作系统)
  - [认证授权](#认证授权)
  - [分布式](#分布式)
  - [必备工具](#必备工具)
  - [前端](#前端)
  - [数据结构与算法](#数据结构与算法)
  - [资源](#资源)



##### Java基础知识
  - [Java概述,变量与运算符,流程控制](docs/javaBase/grammar.md) &nbsp;
  - [一维数组,二维数组及Arrays工具类使用](docs/javaBase/array.md) &nbsp;
  - [类与对象,类的成员及OOP特征](docs/javaBase/object01.md) &nbsp;
  - [this,super,static,final,package,import关键字及其它重要知识点](docs/javaBase/object02.md) &nbsp;
  - [异常体系概述,try,catch,finally,throw及throws关键字](docs/javaBase/exception.md) &nbsp;
  - [String,StringBuffer与StringBuilder详解](docs/javaBase/commonClass01.md) &nbsp;
  - [JDK8之前与JDK8日期时间API详解](docs/javaBase/commonClass02.md) &nbsp;
  - [Comparable与Comparator接口,System,Math,BigDecimal类详解](docs/javaBase/commonClass03.md)
  - [使用反射获取类的Class,Constructor,Method,Filed对象及泛型相关API信息](docs/javaSenior/reflection.md)
  - [静态代理与动态代理介绍及相关案例](docs/javaSenior/dynamicProxy.md)
  - [枚举概述及使用](docs/javaBase/enum.md)

##### Java高级知识
  [Java底层知识点学习目录](docs/javaSenior/study.md)

  - 容器
    - [Collection,List,Set,Map集合及Collections工具类使用](docs/javaSenior/collection/basis.md)
    - [ArrayList/LinkedList/HashMap源码学习](docs/javaSenior/collection/source.md)
  - 并发
    - 基础
      - [并发知识点长篇总结](docs/javaSenior/concurrence/conBasic01.md) &nbsp;&nbsp;
        [线程的实现方式,生命周期,重要API,通信](docs/javaSenior/concurrence/conBasic02.md) &nbsp;
    - 原理
      - [并发的三大特性,Java内存模型,死锁](docs/javaSenior/concurrence/conPrinciple01.md)
      - [Atomic相关类与CAS,Volatile,Synchronized详解](docs/javaSenior/concurrence/conPrinciple02.md)
      - [深入解析ThreadLocal](docs/javaSenior/concurrence/threadLocalAndAQS01.md) &nbsp;
        [AQS解析](docs/javaSenior/concurrence/threadLocalAndAQS02.md)
    - 线程池
      - [线程池原理](docs/javaSenior/concurrence/threadPoolStudy.md) &nbsp;
        [线程池学习](docs/javaSenior/concurrence/threadPool.md)
    - 工具类
      - [深入解析Lock]()
  - JVM
    - [Java内存区域](docs/javaSenior/JVM/memoryArea.md)
  - 其它         
    - [IO流](docs/javaSenior/ioStream.md) &nbsp;
      [JDK8](docs/javaSenior/JDK8.md) &nbsp;
      [网络](docs/javaSenior/network.md) &nbsp;
  - 编程规范
    - [Java编程规范学习](docs/javaSenior/codingStyle/codingStyle.md) &nbsp;
      []() &nbsp;
      []() &nbsp;

##### 数据库
  - Mysql
    - [SqlServer基础](docs/database/mysql/sqlserveBase.md) &nbsp;
      [Mysql相关日期处理](docs/database/mysql/mysqlDateHandle.md) &nbsp;
      [Mysql行列转换](docs/database/mysql/mysqlUnpivot.md) &nbsp;
    - [Mysql索引及高质量Sql建议](docs/database/mysql/sqlAdvise.md) 
  - [Redis](#redis)



##### 常用框架
  - [Servlet,Cookie,Session,JSP,EL表达式,JSTL标签库,AJAX,Filter,Listener基础概念](docs/javaEE/jsp.md)
  - Mybatis
    - [自定义持久层框架简化版](docs/framework/mybatis/mybatis00.md) &nbsp;
    - [Mybatis基本应用](docs/framework/mybatis/mybatis01.md) &nbsp;
    - [Mybatis缓存和插件介绍](docs/framework/mybatis/mybatis02.md) &nbsp;
    - [Mybatis架构,执行流程和设计模式](docs/framework/mybatis/mybatis03.md) &nbsp;
    - [Mybatis源码分析](docs/framework/mybatis/mybatis04.md) &nbsp;
  - Spring
    - [Spring核心思想IOC,AOP概述及自定义解决思路](docs/framework/spring/spring00.md)
    - [Spring中IOC术语]()
    - [Spring中AOP术语,声明式事务]()
    - [Spring涉及到的设计模式]()
    - [Spring源码分析]()
  - [SpringMVC](#springmvc)
    [Hibernate](#hibernate)

##### 微服务
  - SpringBoot
    - [SpringBoot基础](docs/microService/springboot/springboot.md)
    - [SpringBoot整合Thymeleaf](docs/microService/springboot/springboot_thymeleaf.md)
  - [SpringCloud](#springcloud)



##### 分布式
  - 分布式架构
    - [集群架构场景化解决方案:一致性hash算法,集群时钟同步,分布式ID,分布式调度及Session共享问题]()
    - [分布式架构理论:一致性,CAP定理,BASE定理,一致性协议及一致性算法](docs/javaEE/distribution.md)
    - [分布式架构网络通信:BIO,NIO,AIO和Netty及自定义RPC]()  
  - 消息队列
    - [RabbitMQ](#) &nbsp;
      [RocketMQ](#) &nbsp;
      [Kafka](#) &nbsp;
      [ActiveMQ](#activemq)
  - HTTP请求
    - [RestTemplate](#RestTemplate)
    - [HttpClient](#httpclient)
  - 分布式搜索引擎
    - [Elasticsearch入门](docs/javaEE/elasticsearch/elasticsearch.md)

##### 认证授权
  - [Cookie/Session/Token基础知识](#)
  - [JWT基础知识](#)
  - [SpringSecurity](#)
  - [Shiro](#shiro)
  - [SSO单点登录](#)

##### 必备工具
  - [SVN](#svn)
  - GIT
    - [Git入门使用](docs/tools/git/gitBasic.md) &nbsp;
      [GitHub简单使用](docs/tools/git/github.md)  
    - [通俗易懂|用好Git和SVN,轻松驾驭版本管理](docs/tools/git/gitAndSvn.md) &nbsp;
  - [Nginx](#nginx)
  - Docker
    - [Docker入门使用](docs/tools/docker/dockerBasic.md)



##### 前端
  - [HTML/CSS基础](docs/frontEnd/htmlCssBasic.md)
  - [JavaScript入门](#javascript)
  - jQuery
    - [jQuery基础](docs/frontEnd/jqueryBasic.md) &nbsp;
      [jQuery中Ajax](docs/frontEnd/jqueryAjax.md)
  - [Vue](#vue)
  - [Bootstrap](#bootstrap)
  - [Element](#element)
  - [Echarts](#echarts)

##### 操作系统
  - Linux
    - [Linux概述及常用命令](docs/operatingSystem/linuxBasic.md)
  - operating system  
    - [写给大忙人看的操作系统](docs/operatingSystem/os.md)  

##### 数据结构与算法
  - [数据结构](#数据结构)
  - [算法](#算法)

##### 资源
  - [推荐资源]()
    - [Github上重要的几个搜索技巧](docs/GithubSkill.md)
    - [超实用网址,GitHub项目和常见面试题](docs/resource.md)


### 待办
- [x] springboot(---正在进行中---)
- [ ] JavaEE
- [ ] mysql


### 描述
<span style="font-size:20px;">**java-development-notes介绍**</span>

本文档倾向于提供 java 开发相关基础理念知识，用来记录自己学习 java 开发过程中的相关笔记。

<span style="font-size:20px;">**关于转载**</span>

如果你需要转载本仓库的一些文章到自己的博客的话，记得注明原文地址就可以了。

<span style="font-size:20px;">**如何对该开源文档进行贡献**</span>

1. 笔记内容难免会有笔误，可以帮我找错别字。
2. 很多知识点可能没有涉及到，可以对其他知识点进行补充。
3. 现有的知识点难免存在不完善或者错误，可以对已有知识点进行修改/补充。
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
