## 一.Spring中IOC基础及应用

### 1.ioc基础知识

从自定义ioc来看,有两个重要的组件：

- `beans.xml` : 定义需要实例化对象的类的全限定类名以及类之间依赖关系描述
- `BeanFactory.java` : IOC容器: 通过反射技术来实例化对象并维护对象之间的依赖关系

Spring框架中IOC实现: 各种bean定义模式下IOC容器启动方式：

1.纯xml(bean信息定义全部配置在xml中) : 

- JavaSE应用: 
    ```java
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    // 或
    ApplicationContext context = new FileSystemXmlApplicationContext("c:/beans.xml");
    ```
- JavaWeb应用: **ContextLoaderListener**(监听器加载xml)

2.xml+注解(部分bean使用xml定义,部分bean使用注解定义) : 与纯xml一致

3.纯注解模式(所有bean都是用注解来定义的) : 

- JavaSE应用: 
    ```java
    ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
    ```
- JavaWeb应用: **ContextLoaderListener**(监听器加载注解配置类)

### 2.BeanFactory与ApplicationContext区别

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588582701320-4a3ad92d-bba0-46b6-98d5-e07fe2ff1225.png)

BeanFactory是Spring框架中IoC容器的顶层接口,用来定义规范,ApplicationContext是它的⼀个⼦接⼝,具备BeanFactory的全部功能。通常称BeanFactory为SpringIOC的基础容器，ApplicationContext是容器的⾼级接⼝,拥有更多的功能如 国际化⽀持和资源访问（xml，java配置类）等。

**启动 IoC 容器的方式**

1.Java环境下启动IoC容器

- **ClassPathXmlApplicationContext**：从类的根路径下加载配置⽂件（推荐使⽤）
- FileSystemXmlApplicationContext：从磁盘路径上加载配置⽂件
- **AnnotationConfigApplicationContext**：纯注解模式下启动Spring容器

2.Web环境下启动IoC容器

- 从**xml**启动容器

```xml
<!DOCTYPE web-app PUBLIC
"-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
"http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <!--配置Spring ioc容器的配置⽂件-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <!--使⽤监听器启动Spring的IOC容器-->
    <listener>
    	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

   - 从**配置类**启动容器

```xml
<!DOCTYPE web-app PUBLIC
"-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
"http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <!--告诉ContextloaderListener知道我们使⽤注解的⽅式启动ioc容器-->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
         org.springframework.web.context.support.AnnotationConfigWebApplicationContext
      	</param-value>
    </context-param>
    <!--配置启动类的全限定类名-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.fishleap.SpringConfig</param-value>
    </context-param>
    <!--使⽤监听器启动Spring的IOC容器-->
    <listener>
    	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

### 3.纯xml模式

applicationContext.xml中的head部分:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
https://www.springframework.org/schema/beans/spring-beans.xsd">
```

#### 3.1 实例化Bean的三种方式

- **使用无参构造函数** : 通过反射调⽤⽆参构造函数来创建对象。如果类中没有⽆参构造函数，将创建失败。

```xml
<!--配置service对象-->
<bean id="userService" class="com.fishleap.service.impl.TransferServiceImpl"></bean>
```

- **使用静态方法创建** : 提供⼀个创建对象的⽅法,恰好这个⽅法是static修饰的⽅法. 如jdbc中getConnection方法

```xml
<!--使⽤静态⽅法创建对象的配置⽅式: factory-method值为静态方法名称-->
<bean id="userService" class="com.fishleap.factory.BeanFactory" 
      factory-method="getTransferService"></bean>
```

- **使用实例化方法创建** : 获取对象的⽅法不再是static修饰的

```xml
<!--使⽤实例⽅法创建对象的配置⽅式: factory-bean值为引用对象 factory-method值为引用对象的成员方法-->
<bean id="beanFactory" class="com.fishleap.factory.instancemethod.BeanFactory"></bean>
<bean id="transferService" factory-bean="beanFactory" factory-method="getTransferService"></bean>
```

#### 3.2 Bean的作用范围及生命周期

在spring框架管理Bean对象的创建时，**Bean对象默认都是单例的**，但是它⽀持配置的⽅式改变作⽤范围。

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588598827272-4f70f52e-08ca-4c97-b0c1-0caea61936b6.png)

具体在xml中配置:

```xml
<!--配置service对象: id标识对象，class是类的全限定类名, scope值为作用范围-->
<!--scope：定义bean的作用范围
    singleton：单例，IOC容器中只有一个该类对象，默认为singleton
    prototype：原型(多例)，每次使用该类的对象（getBean），都返回给你一个新的对象，Spring只创建对象，不管理对象
-->
<bean id="transferService" class="com.fishleap.service.impl.TransferServiceImpl" 
      scope="singleton">
</bean>
```

不同作⽤范围的⽣命周期:

- **singleton** : 单例模式的bean对象⽣命周期与容器相同
    - 对象出⽣：当创建容器时，对象就被创建了
    - 对象活着：只要容器在，对象⼀直活着
    - 对象死亡：当销毁容器时，对象就被销毁了
- **prototype** : 原型模式的bean对象，spring框架只负责创建，不负责销毁
    - 对象出⽣：当使⽤对象时，创建新的对象实例
    - 对象活着：只要对象在使⽤中，就⼀直活着
    - 对象死亡：当对象⻓时间不⽤时，被java的垃圾回收器回收了

#### 3.3 Bean的标签属性

在xml的IOC配置中,bean标签表示容器中的一个对象.需要在xml配置bean标签来让spring进行管理.

- **id属性**： ⽤于给bean提供⼀个唯⼀标识。在⼀个标签内部，标识必须唯⼀
- **class属性**：⽤于指定创建Bean对象的全限定类名
- name属性：⽤于给bean提供⼀个或多个名称。多个名称⽤空格分隔
- **factory-bean属性**：⽤于指定创建当前bean对象的⼯⼚bean的唯⼀标识。当指定了此属性之后，class属性失效
- **factory-method属性**：⽤于指定创建当前bean对象的⼯⼚⽅法，如配合factory-bean属性使⽤，则class属性失效。如配合class属性使⽤，则⽅法必须是static的
- **scope属性**：⽤于指定bean对象的作⽤范围。通常情况下就是singleton。当要⽤到多例模式时，可以配置为prototype
- init-method属性：⽤于指定bean对象的初始化⽅法，此⽅法会在bean对象装配后调⽤。必须是⼀个⽆参⽅法
- destory-method属性：⽤于指定bean对象的销毁⽅法，此⽅法会在bean对象销毁前执⾏。它只能为scope是singleton时起作⽤

**init-method和destory-method属性使用示例**:

applicationContext.xml:

```xml
<bean id="accountDao" class="com.fishleap.dao.impl.JdbcTemplateDaoImpl" scope="singleton" init-method="init" destroy-method="destory"></bean>
```

JdbcAccountDaoImpl类:

```java
public void init() {
    System.out.println("初始化方法.....");
}

public void destory(){
    System.out.println("销毁方法.....");
}
```

#### 3.4 DI依赖注入的xml配置

##### 按注入的方式分类

1.**set方法注入** : 它是通过类成员的set⽅法实现数据的注⼊. 利⽤字段的set⽅法实现赋值的注⼊⽅式,是开发中使用最多的注入方式.必须有无参的构造函数。

在使⽤set⽅法注⼊时，需要使⽤ `property` 标签，该标签属性如下：

   - **name**：指定注⼊时调⽤的set⽅法名称. 注:不包含set这三个字⺟,druid连接池指定属性名称
   - **value**：指定注⼊的数据。它⽀持基本类型和String类型
   - **ref**：指定注⼊的数据。它⽀持其他bean类型。写的是其他bean的唯⼀标识

applicationContext.xml:

```xml
<bean id="accountDao" class="com.fishleap.dao.impl.JdbcTemplateDaoImpl">
  	<!--使用set方法注入:类中必须有无参构造函数-->
    <!--set注入使用property标签，name是字段的名称. 如果注入的是另外一个bean那么使用ref属性，
      如果注入的是普通值那么使用的是value属性-->
  	<!--set+ name 之后锁定到传值的set方法了，通过反射技术可以调用该方法传入对应的值-->
    <property name="connectionUtils" ref="connectionUtils"/>
    <property name="name" value="zhangsan"/>
    <property name="sex" value="1"/>
    <property name="money" value="100.3"/>
</bean>  
```

JdbcTemplateDaoImpl.java:

```java
private ConnectionUtils connectionUtils;
private String name;
private int sex;
private float money;

public void setConnectionUtils(ConnectionUtils connectionUtils) {
    this.connectionUtils = connectionUtils;
}
public void setName(String name) {
    this.name = name;
}
public void setSex(int sex) {
    this.sex = sex;
}
public void setMoney(float money) {
    this.money = money;
}
```

2.**构造函数注⼊** : 利⽤带参构造函数实现对类成员的数据赋值.

在使⽤构造函数注⼊时，涉及的标签是 `constructor-arg`  ，该标签有如下属性：

   - **name**：⽤于给构造函数中指定名称的参数赋值
   - **value**：⽤于指定基本类型或者String类型的数据
   - **ref**：⽤于指定其他Bean类型的数据。写的是其他bean的唯⼀标识
   - **index**：⽤于给构造函数中指定索引位置的参数赋值

applicationContext.xml:

```xml
<bean id="accountDao" class="com.fishleap.dao.impl.JdbcTemplateDaoImpl">
    <!--name：按照参数名称注入，index按照参数索引位置注入-->
    <constructor-arg name="connectionUtils" ref="connectionUtils"/>
    <constructor-arg name="name" value="zhangsan"/>
    <constructor-arg name="sex" value="1"/>
    <constructor-arg name="money" value="100.6"/>
</bean>  
```

JdbcTemplateDaoImpl.java:

```java
private ConnectionUtils connectionUtils;
private String name;
private int sex;
private float money;

public JdbcAccountDaoImpl(ConnectionUtils connectionUtils, String name, int sex, float money) {
    this.connectionUtils = connectionUtils;
    this.name = name;
    this.sex = sex;
    this.money = money;
}
```

##### 按注入的数据类型分类

- **基本类型和String** : 注⼊的数据类型是基本类型或字符串类型的数据
- **其它Bean类型** : 注⼊的数据类型是对象类型
- **复杂类型(集合类型)**  : 注⼊的数据类型是Aarry，List，Set，Map，Properties中的⼀种类型. 集合分为两类,⼀类是List结构（数组结构），⼀类是Map接⼝（键值对）

集合类型注入: 示例选用set方法注入。

JdbcTemplateDaoImpl.java:

```java
private String[] myArray;
private Map<String,String> myMap;
private Set<String> mySet;
private Properties myProperties;

public void setMyArray(String[] myArray) {
    this.myArray = myArray;
}
public void setMyMap(Map<String, String> myMap) {
    this.myMap = myMap;
}
public void setMySet(Set<String> mySet) {
    this.mySet = mySet;
}
public void setMyProperties(Properties myProperties) {
    this.myProperties = myProperties;
}
```

applicationContext.xml:

```xml
<bean id="accountDao" class="com.fishleap.dao.impl.JdbcTemplateDaoImpl">
		<!--set注入注入复杂数据类型-->
    <property name="myArray">
      <array>
        <value>array1</value>
        <value>array2</value>
        <value>array3</value>
      </array>
    </property>
    <property name="myMap">
      <map>
        <entry key="key1" value="value1"/>
        <entry key="key2" value="value2"/>
      </map>
    </property>
    <property name="mySet">
      <set>
        <value>set1</value>
        <value>set2</value>
      </set>
    </property>
    <property name="myProperties">
      <props>
        <prop key="prop1">value1</prop>
        <prop key="prop2">value2</prop>
      </props>
    </property>
</bean>
```

在**List结构**的集合数据注⼊时， array , list , set 这三个标签通⽤，另外注值的 value 标签内部可以直接写值，也可以使⽤ bean 标签配置⼀个对象，或者⽤ ref 标签引⽤⼀个已经配合的bean的唯⼀标识。

在**Map结构**的集合数据注⼊时， map 标签使⽤ entry ⼦标签实现数据注⼊， entry 标签可以使⽤key和value属性指定存⼊map中的数据。使⽤value-ref属性指定已经配置好的bean的引⽤。同时 entry 标签中也可以使⽤ ref 标签，但是不能使⽤ bean 标签。⽽ property 标签中不能使⽤ ref 或者 bean 标签引⽤对象.

### 4.xml+注解模式

- 实际企业开发中，纯xml模式使⽤已经很少了
- 引⼊注解功能，不需要引⼊额外的jar
- xml+注解结合模式，xml⽂件依然存在，所以，spring IOC容器的启动仍然从加载xml开始
- 哪些bean的定义写在xml中，哪些bean的定义使⽤注解

注: 第三⽅jar中的bean定义在xml，⽐如德鲁伊数据库连接池; ⾃⼰开发的bean定义使⽤注解。

#### 4.1 xml中标签与注解的对应(IoC)

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588616955327-d9ab75a3-7bb3-47a5-a086-a9a73b109296.png)

#### 4.2 依赖注⼊的注解实现⽅式(DI)

1.**@Autowired** (推荐使⽤) : 为Spring提供的注解。

- 需要导⼊包 `org.springframework.beans.factory.annotation.Autowired` 
- 采取的策略为按照类型注⼊: spring容器中找到类型为如AccountDao的类，然后将其注⼊进来. 当⼀个类型有多个bean值的时,需要配合着`@Qualifier`使⽤。

2.**@Qualifier** 告诉Spring具体去装配哪个对象

示例:

```java
public class TransferServiceImpl {
    // @Autowired 按照类型注入 ,如果按照类型无法唯一锁定对象，可以结合@Qualifier指定具体的id
    @Autowired
    @Qualifier(name="jdbcAccountDaoImpl")
    private AccountDao accountDao;
}
```

这个时候可以通过类型和名称定位到我们想注⼊的对象.

3.**@Resource**

- @Resource 注解由 J2EE 提供，需要导⼊包 javax.annotation.Resource
- @Resource 默认按照 ByName ⾃动注⼊
- @Resource 在 Jdk 11中已经移除，如果要使⽤，需要单独引⼊jar包

```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

示例:

```java
public class TransferService {
    @Resource
    private AccountDao accountDao;
    @Resource(name="studentDao")	// 上下文中查找id为studentDao的bean
    private StudentDao studentDao;
    @Resource(type="TeacherDao")	// 从上下⽂中找到类型匹配的唯⼀bean进⾏装配
    private TeacherDao teacherDao;
    @Resource(name="manDao",type="ManDao")	// 从Spring上下⽂中找到唯⼀匹配的bean进⾏装配
    private ManDao manDao;
}
```

单元测试:

```java
// 通过读取classpath下的xml文件来启动容器（xml模式SE应用下推荐）
// 启动容器调用xml配置文件中相关init方法
ClassPathXmlApplicationContext classPathXmlApplicationContext =
    new ClassPathXmlApplicationContext("classpath:applicationContext.xml");

// 不推荐使用
//ApplicationContext applicationContext1 =
//      new FileSystemXmlApplicationContext("文件系统的绝对路径");

// 第一次getBean该对象
AccountDao accountDao = (AccountDao) classPathXmlApplicationContext.getBean("accountDao");
accountDao.queryAccountByCardNo("111111");
System.out.println("accountDao" + accountDao);

// 关闭容器, ApplicationContext没有close方法
// 调用xml文件中相关destroy方法
classPathXmlApplicationContext.close();
```

### 5.纯注解模式

改造xm+注解模式，将xml中遗留的内容全部以注解的形式迁移出去，最终删除xml，从Java配置类启动。

对应注解:

- `@Configuration`  注解，表明当前类是⼀个**配置类**
- `@ComponentScan`  注解，**注解扫描**. 替代 context:component-scan
- `@PropertySource` : 引⼊**外部属性配置⽂件**
- `@Import`  引⼊**其他配置类**
- `@Value`  对变量赋值，可以直接赋值，也可以使⽤ `${}`  读取资源配置⽂件中的信息
- `@Bean`  将⽅法返回对象加⼊ SpringIOC 容器

原xml文件内容:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--跟标签beans，里面配置一个又一个的bean子标签，每一个bean子标签都代表一个类的配置-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启注解扫描，base-package指定扫描的包路径-->
    <context:component-scan base-package="com.fishleap" />

    <!--引入外部资源文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--第三方jar中的bean定义在xml中-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
    </bean>
</beans>
```

将xml中注解扫描,引入外部资源,第三方jar配置成注解形式，SpringConfig.java配置类:

```java
// @Configuration 注解表明当前类是一个配置类
@Configuration
@ComponentScan({"com.fishleap"})	// 注解扫描
@PropertySource({"classpath:jdbc.properties"})	// 引入外部资源
//@Import()	// 导入其它多个配置类
public class SpringConfig {

    @Value("${jdbc.driver}")	// 注入外部资源文件中的值
    private String driverClassName;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;

    @Bean	// 将方法的返回值注入IoC容器中
    public DruidDataSource getDruidDataSource() {
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName(driverClassName);
        druidDataSource.setUrl(url);
        druidDataSource.setUsername(username);
        druidDataSource.setPassword(password);
        return druidDataSource;
    }
}
```

javaSE启动:

```java
// 通过读取classpath下的xml文件来启动容器（xml模式SE应用下推荐）
ApplicationContext applicationContext =
    new AnnotationConfigApplicationContext(SpringConfig.class);

AccountDao accountDao = (AccountDao) applicationContext.getBean("accountDao");
System.out.println("accountDao" + accountDao);
```

javaWeb启动: web.xml配置

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <!--告诉ContextloaderListener知道我们使用注解的方式启动ioc容器-->
  <context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
  </context-param>
  <!--配置启动类的全限定类名-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>com.fishleap.SpringConfig</param-value>
  </context-param>
  <!--使用监听器启动Spring的IOC容器-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
</web-app>
```

## 二.Spring中IoC高级特性

### 1.lazy-Init 延迟加载

Bean的延迟加载（延迟创建）: **ApplicationContext 容器的默认⾏为是在启动服务器时将所有 singleton bean 提前进⾏实例化**。提前实例化意味着作为初始化过程的⼀部分，ApplicationContext 实例会创建并配置所有的singleton bean。

如果不想让⼀个singleton bean 在 ApplicationContext实现初始化时被提前实例化，那么可以将bean设置为延迟实例化:

```xml
<!--
lazy-init值为false时表示在spring启动时,立刻进行实例化;
lazy-init值为true时,bean是第⼀次向容器通过 getBean 索取 bean 时实例化的
-->
<bean id="testBean" calss="com.fishleap.LazyBean" lazy-init="true" />
```

也可以在容器层次中通过在 元素上使⽤ "default-lazy-init" 属性来控制延时初始化:

```xml
<!--全局配置:所有注入的bean开启延迟加载,当单个bean也配置时优先级更高-->
<beans default-lazy-init="true">
		<!-- no beans will be eagerly pre-instantiated... -->
</beans>
```

**如果⼀个 bean 的 scope 属性为 scope="pototype" 时，即使设置了 lazy-init="false"，容器启动时也不会实例化bean，⽽是调⽤ getBean ⽅法实例化的**。

若是注解形式时: 只需要在当前类上加上 `@lazy` 注解即可.

### 2.FactoryBean 和 BeanFactory

BeanFactory接⼝是容器的顶级接⼝，定义了容器的⼀些基础⾏为，负责⽣产和管理Bean的⼀个⼯⼚，具体使⽤它下⾯的⼦接⼝类型，⽐如ApplicationContext；

Spring中Bean有两种，⼀种是普通Bean，⼀种是⼯⼚Bean（FactoryBean），FactoryBean可以⽣成某⼀个类型的Bean实例（返回给我们），也就是说我们可以借助于它⾃定义Bean的创建过程。

Bean创建的三种⽅式中的静态⽅法和实例化⽅法和FactoryBean作⽤类似，FactoryBean使⽤较多，尤其在Spring框架⼀些组件中会使⽤，还有其他框架和Spring框架整合时使⽤。

FactoryBean接口:

```java
// 可以让我们⾃定义Bean的创建过程（完成复杂Bean的定义）
public interface FactoryBean<T> {
    @Nullable
    // 返回FactoryBean创建的Bean实例，如果isSingleton返回true，
    // 则该实例会放到Spring容器的单例对象缓存池中Map
    T getObject() throws Exception;
    
    @Nullable
    // 返回FactoryBean创建的Bean类型
    Class<?> getObjectType();
    
    // 返回作⽤域是否单例
    default boolean isSingleton() {
    	return true;
    }
}
```

Company类:

```java
public class Company {
    private String name;
    private String address;
    private int scale;
    // ......
}    
```

CompanyFactoryBean类：

```java
public class CompanyFactoryBean implements FactoryBean<Company> {
    private String companyInfo; // 公司名称,地址,规模
    public void setCompanyInfo(String companyInfo) {
    	this.companyInfo = companyInfo;
    }
    
    @Override
    public Company getObject() throws Exception {
        // 模拟创建复杂对象Company
        Company company = new Company();
        String[] strings = companyInfo.split(",");
        company.setName(strings[0]);
        company.setAddress(strings[1]);
        company.setScale(Integer.parseInt(strings[2]));
        return company;
    }
    
    @Override
    public Class<?> getObjectType() {
    	return Company.class;
    }
    
    @Override
    public boolean isSingleton() {
    	return true;
    }
}
```

xml配置:

```xml
<bean id="companyBean" class="com.fishleap.factory.CompanyFactoryBean">
	<property name="companyInfo" value="阿里,杭州,600"/>
</bean>
```

测试，获取FactoryBean产⽣的对象:

```java
Object companyBean = applicationContext.getBean("companyBean");
System.out.println("bean:" + companyBean);
// 结果如下
bean:Company{name='阿里', address='杭州', scale=600}
```

测试，获取FactoryBean，需要在id之前添加“&”：

```java
// 在id前加上 & 符号,获取CompanyFactoryBean
Object companyBean = applicationContext.getBean("&companyBean");
System.out.println("bean:" + companyBean);
// 结果如下
bean:com.lagou.edu.factory.CompanyFactoryBean@53f6fd09
```

### 3.后置处理器

Spring提供了两种后处理bean的扩展接⼝，分别为 BeanPostProcessor 和BeanFactoryPostProcessor，两者在使⽤上是有所区别的。⼯⼚初始化（BeanFactory）—> Bean对象。

- 在BeanFactory初始化之后可以使⽤BeanFactoryPostProcessor进⾏后置处理做⼀些事情; 
- 在Bean对象实例化（并不是Bean的整个⽣命周期完成）之后可以使⽤BeanPostProcessor进⾏后置处理做⼀些事情。

注意：**对象不⼀定是springbean，⽽springbean⼀定是个对象**。

Bean的生命周期:

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588671269712-5ca809f4-384c-49e7-b339-ca42f5a47256.png)

BeanDefinition接口:
![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588671357389-c2200c86-f956-4784-aea8-72f56424bf31.png)

#### 3.1 BeanPostProcessor

BeanPostProcessor是针对**Bean**级别的处理，可以针对某个具体的Bean。

```java
public interface BeanPostProcessor {
    // 第一个参数是每个bean的实例，第二个参数是每个bean的name或者id属性的值
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
	// 第一个参数是每个bean的实例，第二个参数是每个bean的name或者id属性的值
    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

该接⼝提供了两个⽅法，分别**在Bean的初始化方法前和初始化方法后执⾏**，具体这个初始化⽅法指的是什么⽅法，类似我们在定义bean时，定义了init-method所指定的⽅法。

定义⼀个类实现了BeanPostProcessor，默认是会对整个Spring容器中所有的bean进⾏处理。如果要对具体的某个bean处理，可以通过方法参数判断，两个类型参数分别为Object和String，第一个参数是每个bean的实例，第二个参数是每个bean的name或者id属性的值。所以我们可以通过第⼆个参数，来判断我们将要处理的具体的bean。

注意：**处理是发生在Spring容器的实例化和依赖注入之后**。

#### 3.2 BeanFactoryPostProcessor

BeanFactory级别的处理，是针对整个Bean的⼯⼚进⾏处理，典型应⽤：PropertyPlaceholderConfigurer。

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory var1) throws BeansException;
}
```

此接口只提供了一个方法，方法参数为ConfigurableListableBeanFactory，该参数类型定义了一些方法:

![img](https://cdn.nlark.com/yuque/0/2020/png/626958/1588671980597-63ffddc6-52f7-409e-889c-9280a85fabe3.png)

其中有个方法名为getBeanDefinition的方法，我们可以根据此方法，找到我们定义bean 的BeanDefinition对象。然后我们可以对定义的属性进行修改，以下是BeanDefinition中的方法:

![img](https://gitee.com/itzlg/mypictures/raw/master/img/1588672440063-638c7f4d-9cda-4bc9-a64b-3a3976fbb014.png)

方法名字类似我们bean标签的属性，setBeanClassName对应bean标签中的class属性，所以当我们拿到BeanDefinition对象时，我们可以主动修改bean标签中所定义的属性值。

**BeanDefinition对象**：我们在XML中定义的bean标签，Spring解析bean标签成为一个JavaBean，这个JavaBean 就是 BeanDefinition。

注意：**调用 BeanFactoryPostProcessor 方法时，这时候bean还没有实例化，此时bean刚被解析成BeanDefinition对象**。

