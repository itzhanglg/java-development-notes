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
  - [基础语法](docs/javaBase/grammar.md) &nbsp;
    [数组](docs/javaBase/array.md) &nbsp;
    [面向对象](docs/javaBase/object.md) &nbsp;
    [异常处理](docs/javaBase/exception.md) &nbsp;
  - [字符串,日期,比较器,BigDecimal等常用类学习](docs/javaBase/commonClass.md)

##### Java高级知识
  - 容器
    - [容器知识点总结](docs/javaSenior/collection/basis.md)
    - [ArrayList/LinkedList/HashMap源码学习](docs/javaSenior/collection/source.md)
  - 并发
    - [并发知识点长篇总结](docs/javaSenior/concurrence/thread.md) &nbsp;
    - 并发编程学习
      - [并发编程基础篇](docs/javaSenior/concurrence/concurrenceStudy01.md) &nbsp;
        [并发编程原理篇](docs/javaSenior/concurrence/concurrenceStudy02.md)
    - 线程池学习
      - [线程池原理]() &nbsp;
        [线程池学习](docs/javaSenior/concurrence/threadPool.md)
  - JVM
    - [Java内存区域](docs/javaSenior/JVM/memoryArea.md)
  - 其它         
    - [IO流](docs/javaSenior/ioStream.md) &nbsp;
      [JDK8](docs/javaSenior/JDK8.md) &nbsp;
      [网络](docs/javaSenior/network.md) &nbsp;
      [反射](docs/javaSenior/reflection.md)
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
  - [Jsp基础](docs/javaEE/jsp.md)
  - [Spring](#spring) &nbsp;
    [SpringMVC](#springmvc)
  - [Mybatis](#mybatis) &nbsp;
    [Hibernate](#hibernate)

##### 微服务
  - SpringBoot
    - [SpringBoot基础](docs/microService/springboot/springboot.md)
    - [SpringBoot整合Thymeleaf](docs/microService/springboot/springboot_thymeleaf.md)
  - [SpringCloud](#springcloud)

##### 操作系统
  - Linux
    - [Linux概述及常用命令](docs/operatingSystem/linuxBasic.md)
  - operating system  
    - [写给大忙人看的操作系统](docs/operatingSystem/os.md)



##### 认证授权
  - [Cookie/Session/Token基础知识](#)
  - [JWT基础知识](#)
  - [SpringSecurity](#)
  - [Shiro](#shiro)
  - [SSO单点登录](#)

##### 分布式
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

<!-- <span style="font-size:20px;">**为什么要做这个开源文档？**</span>

初始想法源于自己一段比较迷茫的经历。想抽时间整理自己的一个 java 知识体系。主要目的是为了加强自己的基本功, 同时也希望能帮助正在学习 java 的小伙伴。 -->
