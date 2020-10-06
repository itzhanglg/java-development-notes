## 一.Spring概述及核心思想

### 1.Spring概述

Spring 是分层的 full-stack（全栈） 轻量级开源框架，以 IoC 和 AOP 为内核，提供了展现层 SpringMVC 和业务层事务管理等众多的企业级应用技术，还能整合开源世界众多著名的第三⽅框架和类库，已经成为使⽤最多的 Java EE 企业应用开源框架。

Spring 官网地址：[http://spring.io](http://spring.io/) 。Spring 其实指的是Spring Framework（spring 框架）。

Spring的优势:

- **方便解耦**，简化开发： Spring提供的IoC容器，对象的创建和管理
- **AOP编程的支持**：面向切面编程
- **声明式事务的支持**：进行事物管理
- 方便程序的测试
- 方便集成各种优秀框架
- 降低JavaEE API的使用难度
- 源码是经典的 Java 学习范例

Spring核心结构：

![image-20201005030434142](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201005030434142.png)

- Spring核心容器（Core Container）是Spring框架最核心的部分，管理着bean的创建，配置和管理。Spring bean工厂为Spring提供了DI的功能。
- 面向切面编程（AOP），AOP可以将横切逻辑进行解耦。
- 数据访问与集成（Data Access/Integration），该模块由JDBC、Transactions、ORM、OXM和JMS等模块组成。
- Web模块提供了SpringMVC框架给Web应用。
- Test模块方便测试模块。

### 2.Spring核心思想

Spring很好的实现了IoC和AOP两个思想。

#### 2.1 IoC思想

**什么是IoC**

IoC：Inversion of Control (控制反转/反转控制)，注意它是一个技术思想，不是一个技术实现。描述的事情：**Java开发领域对象的创建，管理的问题**。

- 传统开发方式：比如类A依赖于类B，往往会在类A中new一个B的对象。
- IoC思想下开发方式：我们不用自己去new对象了，而是由IoC容器（Spring框架）去帮助我们实例化对象并且管理它，我们需要使用哪个对象，去问IoC容器要即可。这样我们丧失了一个权利（创建、管理对象的权利）,得到了一个福利（不用考虑对象的创建、管理等一系列事情）。

为什么叫做控制反转？

- **控制**：**指的是对象创建（实例化、管理）的权利**
- **反转**：**控制权交给外部环境了（spring框架、IoC容器**）

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588924110765-c49023dd-706e-432c-ab51-320ca1f6f742.png)

**IoC解决了什么问题**：**IoC解决了对象的耦合问题**

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588924794749-5cbc3b3e-23c4-49b2-a319-1223e2214c80.png)
**IoC和DI的区别**

DI：Dependancy Injection（依赖注入）。IOC和DI描述的是同一件事情，只不过⻆度不一样罢了。

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588924923836-c4b3d4cf-291d-47d2-a98d-be680f2ebe7a.png)

#### 2.2 AOP思想

**AOP**: Aspect oriented Programming 面向切面编程/面向方面编程 。AOP是OOP的延续，从OOP说起。
OOP三大特征：封装、继承和多态。**OOP是一种垂直继承体系**。

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588925264926-e53e7273-35c6-4052-9d25-24c69552c3cc.png)

OOP思想可以解决大多数代码重复问题，但在一个类的多个方法中相同位置出现了重复代码，OOP就没法解决了。比如性能监控代码（程序运行的时间），日志，事物等横切逻辑。

**横切逻辑代码**：在多个纵向（顺序）流程中出现的相同子流程代码。使用场景：事物控制，权限校验，日志。存在什么问题：

- 横切代码重复问题
- 横切逻辑代码和业务代码混杂在一起，代码臃肿，维护不方便

**AOP提出横向抽取机制，将横切逻辑代码和业务逻辑代码分开**。需要悄无声息的把横切逻辑代码应用到原有的业务逻辑中，达到和原来一样的效果，这个是是较难的。AOP在不改变原有业务逻辑情况下，增强横切逻辑代码，根本上解耦合，避免横切逻辑代码重复。

**为什么叫面向切面编程**：

- 切：指的是横切逻辑，不改变原有业务情况下操作横切逻辑代码，所以面向横切逻辑
- 面：横切逻辑代码往往要影响的是很多个方法，每一个方法都如同一个点，多个点构成面

## 二.简化版IOC和AOP框架应用

### 1.案例问题分析

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588926413942-e908a2b4-2554-490b-b67e-aa4bef197b46.png)

上面两个问题实质上就是IoC解决对象耦合问题和AOP事务管理问题。

### 2.对象耦合问题解决思路

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588926748254-e661fa5d-907b-4adf-b176-31aa6b7117a1.png)

### 3.事务管理解决思路

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588926871127-8523ad34-76b3-488b-99d2-8a26af002762.png)

### 4.问题解决流程图

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588618179491-6b8ea191-9575-4243-a599-a3f7059c0747.jpeg)

采用**beans.xml**配置文件和**BeanFactory**来处理对象耦合问题：

- beans.xml：用来配置需要加载到Spring容器中实例对象
- BeanFactory：解析beans.xml文件，通过反射进行生成实例对象和相关的依赖注入

采用**ProxyFactory**和**TransactionManager**来处理事务问题：

- ProxyFactory：用来动态代理需要事务的类，生成代理对象，增强事务
- TransactionManager：用来处理事务，横切逻辑代码的封装



