AOP本质：在不改变原有业务逻辑的情况下增强横切逻辑,横切逻辑代码往往是权限校验代码、日志代码、事务控制代码、性能监控代码。

## 一.AOP相关术语

Joinpoint（连接点）：它指的是那些可以用于把增强代码加入到业务主线中的点,那么由上图中我们可·以看出,这些点指的就是方法。在方法执行的前后通过动态代理技术加入增强的.代码。在Spring框架AOP思想的技术实现中,也只支持方法类型的连接点。

Pointcut（切入点）：它指的是那些已经把增强代码加入到业务主线进来之后的连接点。由上图中,我·们看出表现层transfer方法就只是连接点,因为判断访问权限的功能并没有对其增强。

Advice（通知/增强）：它指的是切面类中用于提供增强功能的方法。并且不同的方法增强的时机是不一样的。比如,开启事务肯定要在业务方法执行之前执行；提交事务要在业务方法正常执行之后执行,而回滚事务要在业务方法执行产生异常之后执行等等。那么这些就是通知的类型。其分类有：前置通知、后置通知、异常通知、最终通知和环绕通知。

Target（目标对象）：它指的是代理的目标对象。即被代理对象。

Proxy（代理）：它指的是一个类被AOP织入增强后，产生的代理类。即代理对象。

Weaving（织入）：它指的是把增强应用到目标对象来创建新的代理对象的过程。spring采用动态代理织入，而Aspect采用编译期织入和类装载期织入。

Aspect（切面）：它指定是增强的代码所关注的方面,把这些相关的增强代码定义到一个类中,这,个类就是切面类。例如,事务切面,它里面定义的方法就是和事务相关的,像开,启事务,提交事务,回滚事务等等,不会定义其他与事务无关的方法。我们前面的案例中TrasnactionManager就是一个切面。

总结：**锁定要在哪个地方插入什么横切逻辑代码**。

![image-20201006163013570](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201006163013570.png)

- **连接点**：方法开始时、结束时、正常运行完毕时、方法异常时等这些特殊的时机点,我们称之为连接点,项目中每个方法都有连接点,连接点是一种候选点。
- **切入点**：指定AOP思想想要影响的具体方法是哪些,描述感兴趣的方法。
- **Advice增强**：第一个层次指的是横切逻辑，第二个层次指方位点(在某一些连接点上加入横切逻辑,那么这些连接点就叫做方位点,描述的是具体的特殊时机)。
- **Aspect切面**：切面概念是对上述概念的一个综合。Aspect切面=切入点+增强=**切入点**(锁定方法) +**方位点**(锁定方法中的特殊时机) +**横切逻辑**。

## 二.AOP基本应用

Spring实现AOP思想使用的是**动态代理**技术。默认情况下, Spring会根据**被代理对象是否实现接口**来选择使用JDK还是CGLIB。当被代理对象没有实现,任何接口时, Spring会选择CGLIB；当被代理对象实现了接口, Spring会选择JDK官方的代理技术,不过我们可以通过配置的方式,让Spring强制使用CGLIB。

在Spring的AOP配置中,也和loC配置一样,支持3类配置方式。

- 使用XML配置
- 使用XML+注解组合配置
- 使用纯注解配置

### 1.XML模式

Spring是模块化开发的框架,使用aop就引入aop的jar。

坐标：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>5.1.12.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

AOP核心配置：

```xml
<!--
Spring基于XML的AOP配置前期准备：在Spring的配置文件中加入aop的约束：
    xmlns:aop="http://www.springframework.org/schema/aop"
    http://www.springframework.org/schema/aop
    https://www.springframework.org/schema/aop/spring-aop.xsd
Spring基于XML的AOP配置步骤：
    第⼀步：把通知Bean交给Spring管理     
    第⼆步：使⽤aop:config开始aop的配置
    第三步：使⽤aop:aspect配置切⾯
    第四步：使⽤对应的标签配置通知的类型
	⼊⻔案例采⽤前置通知，标签为aop:before
-->
<!--把通知bean交给spring来管理-->
<bean id="logUtil" class="com.fishleap.utils.LogUtil"></bean>

<!--开始aop的配置-->
<aop:config>
    <!--配置切⾯-->
    <aop:aspect id="logAdvice" ref="logUtil">
        <!--配置前置通知-->
        <aop:before method="printLog" pointcut="execution(public * com.fishleap.service.impl.TransferServiceImpl.updateAccountByCardNo(com.fishleap.pojo.Account))">
        </aop:before>
    </aop:aspect>
</aop:config>
```

#### 1.1 切⼊点表达式

上述配置实现了对 TransferServiceImpl 的 updateAccountByCardNo ⽅法进⾏增强，在其执⾏之前，输出了记录⽇志的语句。这⾥⾯，我们接触了⼀个⽐较陌⽣的名称：切⼊点表达式，它是做什么的呢？我们往下看。

切⼊点表达式，也称之为AspectJ切⼊点表达式，指的是遵循特定语法结构的字符串，其作⽤是⽤于对符合语法格式的连接点进⾏增强。它是AspectJ表达式的⼀部分。

AspectJ是⼀个基于Java语⾔的AOP框架，Spring框架从2.0版本之后集成了AspectJ框架中切⼊点表达式的部分，开始⽀持AspectJ切⼊点表达式。

切⼊点表达式使⽤示例：

```java
//全限定⽅法名：访问修饰符 返回值 包名.包名.包名.类名.⽅法名(参数列表)
public void com.fishleap.service.impl.TransferServiceImpl.updateAccountByCardNo(c om.fishleap.pojo.Account)

//访问修饰符可以省略：
void com.fishleap.service.impl.TransferServiceImpl.updateAccountByCardNo(c om.fishleap.pojo.Account)
//返回值可以使⽤*，表示任意返回值：
* com.fishleap.service.impl.TransferServiceImpl.updateAccountByCardNo(c om.fishleap.pojo.Account)
//包名可以使⽤.表示任意包，但是有⼏级包，必须写⼏个：
* ....TransferServiceImpl.updateAccountByCardNo(com.fishleap.pojo.Account)
//包名可以使⽤..表示当前包及其⼦包：
* ..TransferServiceImpl.updateAccountByCardNo(com.fishleap.pojo.Account)
//类名和⽅法名，都可以使⽤.表示任意类，任意⽅法：
* ...(com.fishleap.pojo.Account)

//参数列表，可以使⽤具体类型
//基本类型直接写类型名称 ： int
//引⽤类型必须写全限定类名：java.lang.String 
//参数列表可以使⽤*，表示任意参数类型，但是必须有参数：
* *..*.*(*)
//参数列表可以使⽤..，表示有⽆参数均可。有参数可以是任意类型：
* *..*.*(..)
//全通配⽅式：
* *..*.*(..)
```

#### 1.2 改变代理方式的配置

在前⾯我们已经说了，Spring在选择创建代理对象时，会根据被代理对象的实际情况来选择的。被代理对象实现了接⼝，则采⽤**基于接⼝的动态代理**。当被代理对象没有实现任何接⼝的时候，Spring会⾃动切换到**基于⼦类的动态代理**⽅式。

但是我们都知道，⽆论被代理对象是否实现接⼝，只要不是ﬁnal修饰的类都可以采⽤cglib提供的⽅式创建代理对象。所以Spring也考虑到了这个情况，提供了配置的⽅式实现强制使⽤ 基于⼦类的动态代理（即cglib的⽅式）。配置的⽅式有两种：

使⽤aop:conﬁg标签配置：

```xml
<aop:config proxy-target-class="true">
```

使⽤aop:aspectj-autoproxy标签配置：

```xml
<!--此标签是基于XML和注解组合配置AOP时的必备标签，表示Spring开启注解配置AOP的⽀持-->
<aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectj-autoproxy>
```

#### 1.3 五种通知类型

**前置通知**

配置⽅式：aop:before标签

```xml
<!--
作⽤：⽤于配置前置通知。
出现位置：它只能出现在aop:aspect标签内部。
属性：
    method：⽤于指定前置通知的⽅法名称。
	pointcut：⽤于指定切⼊点表达式。
    pointcut-ref：⽤于指定切⼊点表达式的引⽤
-->
<aop:before method="printLog" pointcut-ref="pointcut1">
</aop:before>
```

执⾏时机：前置通知永远都会在切⼊点⽅法（业务核⼼⽅法）执⾏之前执⾏。

细节：前置通知可以获取切⼊点⽅法的参数，并对其进⾏增强。

**正常执行时通知**

```xml
<!--
作⽤：⽤于配置正常执⾏时通知。
出现位置：它只能出现在aop:aspect标签内部。
属性：
    method:⽤于指定后置通知的⽅法名称。
    pointcut:⽤于指定切⼊点表达式。
    pointcut-ref:⽤于指定切⼊点表达式的引⽤。
-->
<aop:after-returning method="afterReturningPrintLog" pointcut- ref="pt1"></aop:after-returning>
```

**异常通知**

```xml
<!--
作⽤：⽤于配置异常通知。
出现位置：它只能出现在aop:aspect标签内部。
属性：
    method:⽤于指定后置通知的⽅法名称。
    pointcut:⽤于指定切⼊点表达式。
    pointcut-ref:⽤于指定切⼊点表达式的引⽤。
-->
<aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>
```

执行时机：异常通知的执⾏时机是在切⼊点⽅法（业务核⼼⽅法）执⾏产⽣异常之后，异常通知执⾏。如果切⼊点⽅法执⾏没有产⽣异常，则异常通知不会执⾏。

细节：异常通知不仅可以获取切⼊点⽅法执⾏的参数，也可以获取切⼊点⽅法执⾏产⽣的异常信息。

**最终通知**

```xml
<!--
作⽤：⽤于配置最终通知。
出现位置：它只能出现在aop:aspect标签内部。
属性：
    method:⽤于指定后置通知的⽅法名称。
    pointcut:⽤于指定切⼊点表达式。
    pointcut-ref:⽤于指定切⼊点表达式的引⽤。
-->
<aop:after method="afterPrintLog" pointcut-ref="pt1"></aop:after>
```

执行时机：最终通知的执⾏时机是在切⼊点⽅法（业务核⼼⽅法）执⾏完成之后，切⼊点⽅法返回之前执⾏。    换句话说，⽆论切⼊点⽅法执⾏是否产⽣异常，它都会在返回之前执⾏。

细节：最终通知执⾏时，可以获取到通知⽅法的参数。同时它可以做⼀些清理操作。

**环绕通知**

```xml
<!--
作⽤：⽤于配置环绕通知。
出现位置：它只能出现在aop:aspect标签内部。
属性：
    method:⽤于指定后置通知的⽅法名称。
    pointcut:⽤于指定切⼊点表达式。
    pointcut-ref:⽤于指定切⼊点表达式的引⽤。
-->
<aop:around method="aroundPrintLog" pointcut-ref="pt1"></aop:around>
```

环绕通知，它是有别于前⾯四种通知类型外的特殊通知。前⾯四种通知（前置，后置，异常和最终） 它们都是指定何时增强的通知类型。⽽环绕通知，它是Spring框架为我们提供的⼀种可以通过编码的⽅式，控制增强代码何时执⾏的通知类型。它⾥⾯借助的`ProceedingJoinPoint`接⼝及其实现类， 实现⼿动触发切⼊点⽅法的调⽤。

### 2.XML+注解模式

XML中开启Spring对AOP的支持：

```xml
<!--开启spring对注解aop的⽀持-->
<aop:aspectj-autoproxy/>
```

示例：

```java
// 模拟记录⽇志
@Component 
@Aspect
public class LogUtil {
    /**
	* 我们在xml中已经使⽤了通⽤切⼊点表达式，供多个切⾯使⽤，那么在注解中如何使⽤呢？
    *第⼀步：编写⼀个⽅法
	*第⼆步：在⽅法使⽤@Pointcut注解
	*第三步：给注解的value属性提供切⼊点表达式
	*细节：
	*1.在引⽤切⼊点表达式时，必须是⽅法名+()，例如"pointcut()"。
	*2.在当前切⾯中使⽤，可以直接写⽅法名。在其他切⾯中使⽤必须是全限定⽅法名。
	*/
    @Pointcut("execution(* com.lagou.service.impl.*.*(..))") 
    public void pointcut(){}
    
    @Before("pointcut()")
	public void beforePrintLog(JoinPoint jp){ 
        Object[] args = jp.getArgs();
		System.out.println("前置通知：beforePrintLog，参数是："+Arrays.toString(args));
	}

    @AfterReturning(value = "pointcut()",returning = "rtValue") 
    public void afterReturningPrintLog(Object rtValue){
    	System.out.println("后置通知：afterReturningPrintLog，返回值是："+rtValue);
    }

    @AfterThrowing(value = "pointcut()",throwing = "e") 
    public void afterThrowingPrintLog(Throwable e){
    	System.out.println("异常通知：afterThrowingPrintLog，异常是："+e);
    }

    @After("pointcut()")
    public void  afterPrintLog(){ 
        System.out.println("最终通知：afterPrintLog");
    }

    /**
    *环绕通知
    *@param pjp
    */ 
    @Around("pointcut()")
	public Object aroundPrintLog(ProceedingJoinPoint pjp){
        //定义返回值
        Object rtValue = null; 
        try{
            //前置通知
            System.out.println("前置通知");

            //1.获取参数
            Object[] args = pjp.getArgs();
            //2.执⾏切⼊点⽅法
			rtValue = pjp.proceed(args);
            
			// 后置通知
            System.out.println("后置通知");
		}catch (Throwable t){
			// 异常通知
            System.out.println("异常通知"); 
            t.printStackTrace();
		}finally {
			// 最终通知
            System.out.println("最终通知");
		}
		return rtValue;
    }
}    
```

### 3.注解模式

在使⽤注解驱动开发aop时，我们要明确的就是，是注解替换掉配置⽂件中的下⾯这⾏配置：

```xml
<!--开启spring对注解aop的⽀持-->
<aop:aspectj-autoproxy/>
```

在配置类中使⽤如下注解进⾏替换上述配置：

```java
@Configuration
@ComponentScan("com.fishleap") 
@EnableAspectJAutoProxy  //开启spring对注解AOP的⽀持
public class SpringConfiguration {
}
```

## 三.Spring中声明式事务

编程式事务：在业务代码中添加事务控制代码，这样的事务控制机制就叫做编程式事务。

声明式事务：通过xml或者注解配置的⽅式达到事务控制的⽬的，叫做声明式事务。

### 1.事务

#### 1.1 事务的概念

简单来说，**事务就是要保证一组数据库操作，要么全部成功，要么全部失败**。在MySQL中，事务支持是在引擎层实现的。

例如：A——B转帐，对应于如下两条sql语句:

```sql
/*转出账户减钱*/
update account set money=money-100 where name=‘a’;
/**转⼊账户加钱*/
update account set money=money+100 where name=‘b’;
```

这两条语句的执⾏，要么全部成功，要么全部不成功。

#### 1.2 事务的四大特性

![image-20201006180530511](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201006180530511.png)

原⼦性（Atomicity） 原⼦性是指事务是⼀个不可分割的⼯作单位，事务中的操作要么都发⽣，要么都不发⽣。从操作的⻆度来描述，事务中的各个操作**要么都成功要么都失败**.

⼀致性（Consistency） 事务必须使数据库从**⼀个⼀致性状态变换到另外⼀个⼀致性状态**。例如转账前A有1000，B有1000。转账后A+B也得是2000。⼀致性是从数据的⻆度来说的，（1000，1000） （900，1100），不应该出现（900，1000）

隔离性（Isolation） 事务的隔离性是多个⽤户并发访问数据库时，数据库为每⼀个⽤户开启的事务，**每个事务不能被其他事务的操作数据所⼲扰，多个并发事务之间要相互隔离**。⽐如：事务1给员⼯涨⼯资2000，但是事务1尚未被提交，员⼯发起事务2查询⼯资，发现⼯资涨了2000块钱，读到了事务1尚未提交的数据（脏读）

持久性（Durability）持久性是指⼀个事务⼀旦被提交，它对**数据库中数据的改变就是永久性的**，接下来即使数据库发⽣故障也不应该对其有任何影响。

#### 1.3 事务的隔离级别

事务的ACID(Atomicity, Consistency, Isolation, Durability)即原子性,一致性,隔离性,持久性.当数据库上有多个事务同时执行的时候，就可能出现脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题.

- **脏读**：⼀个线程中的事务读到了另外⼀个线程中**未提交**的数据。
- **不可重复读**：⼀个线程中的事务读到了另外⼀个线程中已经提交的`update`的数据（前后内容不⼀样）
  员⼯A发起事务1，查询⼯资，⼯资为1w，此时事务1尚未关闭; 财务⼈员发起了事务2，给员⼯A张了2000块钱，并且提交了事务; 员⼯A通过事务1再次发起查询请求，发现⼯资为1.2w，原来读出来1w读不到了，叫做不可重复读。
- **虚读**（幻读）：⼀个线程中的事务读到了另外⼀个线程中已经提交的`insert`或者`delete`的数据（前后条数不⼀样）
  事务1查询所有⼯资为1w的员⼯的总数，查询出来了10个⼈，此时事务尚未关闭; 事务2财务⼈员发起，新来员⼯，⼯资1w，向表中插⼊了2条数据，并且提交了事务; 事务1再次查询⼯资为1w的员⼯个数，发现有12个⼈，⻅了⻤了。

**不可重复读与幻读有什么区别?**

- **不可重复读**的重点是修改：在同一事务中，同样的条件，第一次读的数据和第二次读的「**数据不一样**」。（因为中间有其他事务提交了修改）
- **幻读**的重点在于新增或者删除：在同一事务中，同样的条件，第一次和第二次读出来的「**记录数不一样**」。（因为中间有其他事务提交了插入/删除）

SQL标准的事务隔离级别包括：读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（serializable ）。

- **读未提交**是指，一个事务还没提交时，它做的变更就能被别的事务看到。
- **读提交**是指，一个事务提交之后，它做的变更才会被其他事务看到。**可避免脏读**.
- **可重复读**是指，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。**可避免脏读,不可重复读**.
- **串行化**，顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。**都可避免. 级别最高,效率最低**.

实际上数据库里面会创建一个视图,访问时以视图逻辑结果为准。

- 串行化”隔离级别下直接用**加锁的方式**来避免并行访问
- 在“可重复读”隔离级别下，这个视图是在**事务启动时**创建的，整个事务存在期间都用这个视图
- 在“读提交”隔离级别下，这个视图是在**每个SQL语句开始执行**的时候创建的
- 读未提交”隔离级别下直接返回记录上的最新值，**没有视图概念**

MySQL的默认隔离级别时 REPEATABLE READ。查询当前使用的隔离级别:  `select @@tx_isolation;` 设置MySQL事务的隔离级别： `set session transaction isolation level xxx;`  （设置的是当前mysql连接会话的，并不是永久改变的）。也可以用show variables查看当前的值：`show variables like 'transaction_isolation';` 。

#### 1.4 事务的传播行为

事务往往在service层进⾏控制，如果出现service层⽅法A调⽤了另外⼀个service层⽅法B，A和B⽅法本身都已经被添加了事务控制，那么A调⽤B的时候，就需要进⾏事务的⼀些协商，这就叫做事务的传播⾏为。

A调⽤B，我们站在B的⻆度来观察来定义事务的传播⾏为：

![image.png](https://gitee.com/itzlg/mypictures/raw/master/img/1590334332971-e9a0af73-2560-4649-afdc-e4aca19bbea9.png)

### 2.Spring中事务的API

PlatformTransactionManager类：

```java
public interface PlatformTransactionManager {
    /**
    * 获取事务状态信息
	*/
	TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
    
    /**
    * 提交事务
    */
    void commit(TransactionStatus status) throws TransactionException;
    
    /**
    * 回滚事务
    */
    void rollback(TransactionStatus status) throws TransactionException;
```

此接⼝是Spring的事务管理器核⼼接⼝。Spring本身并不⽀持事务实现，只是负责提供标准，应⽤底层⽀持什么样的事务，需要提供具体实现类。此处也是策略模式的具体应⽤。在Spring框架中，也为我们内置了⼀些具体策略，例如：DataSourceTransactionManager , HibernateTransactionManager 等等。（ HibernateTransactionManager 事务管理器在spring-orm-5.1.12.RELEASE.jar 中）

- Spring JdbcTemplate（数据库操作⼯具）、Mybatis（mybatis-spring.jar）：DataSourceTransactionManager
- Hibernate框架：HibernateTransactionManager

DataSourceTransactionManager  归根结底是横切逻辑代码，声明式事务要做的就是使⽤Aop（动态代理）来将事务控制逻辑织⼊到业务代码。

### 3.Spring声明式事务配置

#### 1.纯XML模式

引入依赖：

```xml

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.12.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.1.12.RELEASE</version>
    </dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.1.12.RELEASE</version>
</dependency>
```

xml配置：

```xml
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <!--定制事务细节，传播⾏为、隔离级别等-->
    <tx:attributes>
        <!--⼀般性配置-->
        <tx:method name="*" read-only="false" propagation="REQUIRED" isolation="DEFAULT" timeout="-1"/>
        <!--针对查询的覆盖性配置-->
        <tx:method name="query*" read-only="true" propagation="SUPPORTS"/>
    </tx:attributes>
</tx:advice>

<aop:config>
    <!--advice-ref指向增强=横切逻辑+⽅位-->
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.fishleap.edu.service.impl.TransferServiceImpl.*(..))"/>
</aop:config>
```

#### 2.XML+注解模式

xml配置：

```xml
<!--配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>

<!--开启spring对注解事务的⽀持-->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

在接⼝、类或者⽅法上添加@Transactional注解：

```java
@Transactional(readOnly = true,propagation = Propagation.SUPPORTS)
```

#### 3.纯注解模式

Spring基于注解驱动开发的事务控制配置，只需要把 xml 配置部分改为注解实现。只是需要⼀个注解替换掉 xml 配置⽂件中的 `<tx:annotation-driven transaction- manager="transactionManager"/>` 配置。

在 Spring 的配置类上添加 @EnableTransactionManagement 注解即可：

```java
@EnableTransactionManagement//开启spring注解事务的⽀持
public class SpringConfiguration {
}
```

### 推荐链接

- [耗时3周！7000+字的Spring事务总结来啦！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486851&idx=1&sn=e249e4724781655278f751fd21c8b0e5&chksm=cea24248f9d5cb5e50893a60bc99da454d3f8e8e440cd065772015278b645ae43e9e0109ca20&scene=126&sessionid=1590252365&key=5c8778b7574a982da26a3de952f3cfeeee5ec71153157b65ad317c6d48781214b573729a52a0836f5389c0a90ed60103f5323b80880a83ca298a36572083c511e9c6338be7935e56cd05928be3fd386d&ascene=1&uin=MTU0ODQ4ODg4MQ%3D%3D&devicetype=Windows+10+x64&version=62090072&lang=zh_CN&exportkey=AVklc0wEsf9ulMeW5qZfo1k%3D&pass_ticket=BvMNelsjnT14X5f7rxMP%2BYofvyZHyPOv3Pcfx0hOiARUioOyXUC5al3YQToasWut)
- [太难了~面试官让我结合案例讲讲自己对Spring事务传播行为的理解](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486668&idx=2&sn=0381e8c836442f46bdc5367170234abb&chksm=cea24307f9d5ca11c96943b3ccfa1fc70dc97dd87d9c540388581f8fe6d805ff548dff5f6b5b&scene=126&sessionid=1590252372&key=89d850d4a03ade5807621c5fccdca22fa4f5fd93fb0633a6c0fa5e85d44cd4f9e2994d08f2b61bc6075a4df3a2900e3046383ef8d7490c72a158aa0300b3fdc173785b49fee7235603e7e1a76d82a07b&ascene=1&uin=MTU0ODQ4ODg4MQ%3D%3D&devicetype=Windows+10+x64&version=62090072&lang=zh_CN&exportkey=AWAhBtX9sAn%2Fh%2FoFMRTfwGo%3D&pass_ticket=BvMNelsjnT14X5f7rxMP%2BYofvyZHyPOv3Pcfx0hOiARUioOyXUC5al3YQToasWut)