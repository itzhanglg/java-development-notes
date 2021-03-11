#### 1.Mybatis动态sql是做什么的？都有哪些动态sql？简述一下动态sql的执行原理？

**Mybatis动态sql**: 可以在 `*mapper.xml `sql映射文件中以xml标签编写sql,根据表达式的值进行逻辑判断并拼接成不同的sql;

**动态sql标签**: `<if>`, `<choose>`,  `<when>`, `<otherwise>`, `<trim>`, `<where>`, `<set>`, `<foreach>`,  `<bind>` ;

- set,if标签:

    ```xml
    <set>
      <if test="name != null">
        name=#{name},
      </if>
    </set>
    ```

- where,if标签:

    ```xml
    <where>
    	<if test="name != null"> 
            and name = #{name}
        </if>
    </where>
    ```

- foreach标签:

    ```xml
    <foreach collection="array" item="item" separator="," open="id in (" close=")">
      	#{item}
    </foreach>
    ```

- choose,when,otherwise标签:

    ```xml
    <choose>
      <when test="type == 1">
          startTime &gt; #{startTime}
      </when>
      <when test="type == 2">
          startTime &lt; #{startTime}
      </when>
      <otherwise>
          startTime = #{startTime}
      </otherwise> 
    </choose>
    ```

**动态sql的执行原理**: 从sql传入的参数对象中计算OGNL表达式的值,完成逻辑判断并动态拼接sql功能.



#### 2.Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？

Mybatis支持延迟加载,,可以在核心配置文件做相关配置,但仅支持关联对象查询时,也就是association(一对一)关联对象和collection(一对多)关联集合对象的延迟加载.

> 延迟加载定义: 当真正需要数据时,才真正执行数据加载操作,是为了避免一些无谓的性能开销而提出来的.
>
> mybatis中的延迟加载: 当执行查询语句是,并不是直接到数据库中执行查询语句,而是根据配置好的延迟策略将查询延迟来减轻数据库服务器的压力.
>
> Mybatis支持延迟加载的条件:
>
> - 只能对关联对象(resultMap中使用association或collection)进行查询时使用延迟加载策略;对于主对象直接加载即可
> - 只能使用多表单独查询,不能使用多表连接查询(多表连接查询直接可以查询到全部信息),也就是需要两个`select`语句来完成
>
> Mybatis中延迟加载策略分类:  demo参考: [Mybatis之延迟加载机制](https://www.cnblogs.com/BlueStarWei/p/9463316.html)
>
> - 直接加载: 执行对象的select语句，完成对主加载马上执行对关联对象的select查询。
> - 侵入式延迟加载(立即加载): 执行对主加载对象的查询时，不会执行对关联对象的查询。但是当要访问主加载对象的详情时马上执行对关联对象的select查询。即对关联对象的执行查询，侵入到了主加载对象的访问详情中。
> - 深入延迟加载: 执行对主加载对象的查询时，不会执行对关联对象的查询。访问主加载对象的详情时也不会执行关联对象的select查询。只有当真正访问关联对象的详情时，才会执行对关联对象的select查询。

在Mybatis核心配置文件中配置延迟加载:

```xml
<configuration>
    <!--在此标签下面-->
    <settings>
        <!--延迟加载的总开关，默认是深度延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!--侵入式延迟加载的开关，在总开关打开时才起作用-->
        <!--<setting name="aggressiveLazyLoading" value="true"/>-->
    </settings>
</configuration>
```

也可以在resultMap关联中配置: 在`<association>` 和 `<collection>` 标签的属性中配置:

```xml
<!--
	fetchType: 数据加载方式可选为lazy(深度延迟加载)和eager(侵入式延迟加载);
	该配置会覆盖全局的 lazyLoadingEnabled 配置.
-->
<association property="course"
             javaType="Course"
             fetchType="lazy"
             select="selectCourseById"
             column="courseId"/>
```

**实现原理**:

延迟加载的实现原理简单来说就是将关联查询分成了两次单表查询,关联对象查询的时机根据延迟加载策略情况而定.使用CGLIB创建目标对的代理对象,当调用目标方法时,进入拦截器方法.

参考: 

- [Mybatis中延迟加载策略](https://www.cnblogs.com/yjc1605961523/p/11671803.html)
- [Mybatis的延迟加载](https://www.cnblogs.com/thegarden/p/11827248.html)



#### 3.Mybatis都有哪些Executor执行器？它们之间的区别是什么？

![Executor](https://img-blog.csdnimg.cn/20200428055429260.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

从上图可以看出Mybatis主要由三种执行器,它们都继承了BaseExecutor类.

- **SimpleExecutor**: 默认的执行器, 对每条sql进行预编译->设置参数->执行等操作
- **ReuseExecutor**: REUSE 执行器会重用预处理语句 (prepared statements)
- **BatchExecutor**: 批量执行器, 对相同sql进行一次预编译, 然后设置参数, 最后统一执行操作
- **CachingExecutor**: 在 BaseExecutor 实现上包装了一层二级缓存，cacheEnabled=true时并且配置cache则覆盖全局的缓存

Mybatis支持全局修改执行器,参数名为 `defaultExecutorType` ,但一般不建议这样做,建议在获取sqlSession对象时设置.

- **局部设置**: 获取sqlSession时设置,若选择的是批量执行器时,需要手工提交事务.

    ```java
    // 获取指定执行器的sqlSession
    SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH)
    
    // 获取批量执行器时, 需要手动提交事务
    sqlSession.commit();
    ```

- **全局配置**: 不推荐这种方式.

    ```xml
    <settings>
        <setting name="defaultExecutorType" value="BATCH" />
    </settings>
    ```

**三者之间区别**: 对于单条sql语句执行,不同的执行器没有太大的差异.当选择批量执行器时,即使在获取sqlSession时,设置了自动提交事务,也需要手动提交事务.在做批量操作时,使用批量执行器性能会有很大提升.下面是做批量插入操作测试不同执行器的不同行为方式:

- **SIMPLE方式**: 每次插入操作, 都会执行编译, 设置参数, 执行sql操作.(每次创建一个statement对象执行sql,用完立即关闭)
- **REUSE方式**: 只有第一次插入操作,执行了sql编译步骤, 对其它插入操作执行了设置参数, 执行sql的操作.(将statement存储在Map对象中,同一请求复用statement)
- **BATCH方式**: 只对第一次插入操作执行了sql编译操作, 对其它插入操作仅执行了设置参数操作, 最后统一执行.(所有sql添加到批处理,等待统一执行)

参考:

- [Mybatis的三种执行器](https://blog.csdn.net/zongf0504/article/details/100104029?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-9&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-9)
- [Mybatis源码解读-执行器](https://blog.csdn.net/qingtian211/article/details/81838042?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-8&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-8)



#### 4.简述下Mybatis的一级、二级缓存（分别从存储结构、范围、失效场景。三个方面来作答）？

![03](验证资料/pictures/03.jpg)

一级缓存是**SqlSession级别的缓存**.不同的sqlSession之间的缓存数据区域(**HashMap**)是互不影响的.Mybatis**默认开启**一级缓存.在同一个sqlSession中执行两次相同的sql,第一次执行会将查询的数据写入缓存(**LocalCache**),第二次会**判断缓存是否存在,若存在则直接从缓存中获取值.当发生增删改和commit时,缓存会被清除**.

![04](验证资料/pictures/04.jpg)

二级缓存是实现的**全局缓存**.**多个sqlSession共享二级缓存**,是基于**namespace**的.配置中**cacheEnabled**开启二级缓存,被**CachingExecutor**进行包装.在接口添加**CacheNamespace**注解或mapper中添加**cache**标签.使用二级缓存的**pojo类也需要实现序列化接口**.Mybatis采用**PerpetualCache实现Cache接口**实现二级缓存,也可以整合redis来实现二级缓存.同样也是实现了Cache接口.发生**增删改和commit时,缓存会被清除**.

**总结**: 

- 一级缓存与二级缓存底层数据结构都是HashMap结构,都是`PerpetualCache.cache` Map结构
- 一级缓存: 
    - SqlSession中Map集合,存在于每个SqlSession中;
    - **缓存存储的是查询结果的引用**,判断引用是否相等是true;
    - 一级缓存默认时开启的
    - 当sqlSession调用方法发生增删改并commit时,缓存被清空
- 二级缓存:
    - 基于`*Mapper.xml`的**namespace**,同一个mapper.xml的所有sqlsession共享二级缓存;
    - **缓存存储的是查询结果的数据,会重新将数据封装到新的对象中**,判断引用是否相等是false;
    - 二级缓存需要在``sqlMapConfig.xml`文件中配置settings标签**cacheEnabled**开启缓存,接口中使用注解 `@CacheNamespace` 开启二级缓存,类上注解`@CacheNamespace(implementation = PerpetualCache.class)` ,使用二级缓存的pojo的类也需要**实现序列化**.
    - 同一个mapper.xml映射文件中发生增删改并commit,二级缓存会失效

参考:

- [Mybatis的一级缓存和二级缓存](https://blog.csdn.net/xiaojin21cen/article/details/105689189)
- [mybatis中的一级缓存和二级缓存](https://blog.csdn.net/qq_35688140/article/details/89848539)



#### 5.简述Mybatis的插件运行原理，以及如何编写一个插件？

**Mybatis允许拦截下面4种接口的方法**:

- `ParameterHandler(getParameterObject, setParameters等)`: 参数设置处理
- `ResultSetHandler(handlerResultSets, handlerOutputParameters等)`: 结果封装处理
- `Statementhandler(prepare, parameterize, batch, update, query等)`: SQL语法构建器
- `Executor(update, query, commit, rollback等)`: 执行器

Mybatis使用JDK动态代理,代理拦截的接口生成代理对象.然后实现接口的拦截方法.即当执行需要拦截的接口方法时,会进入拦截方法.每个创建的对象是`interceptorChain.pluginAll(parameterHandler);` 方法获取所有的Interceptor,然后遍历最终得到target包装的对象.

**编写插件步骤**:

- 编写`Interceptor`接口的实现类
- 使用`@Intercepts`和`@Signature`注解设置插件的签名,显示要拦截哪个对象的哪个方法
- 在全局配置文件中配置插件

示例:

```java
@Intercepts({   // 可以配置多个@Signature对多个地方拦截
    @Signature(type= StatementHandler.class,    // 拦截哪个接口
        method = "prepare", // 拦截接口中的指定方法
        args = {Connection.class,Integer.class})    // 拦截方法的入参,确定方法的唯一,可能方法重载
})
public class MyPlugin implements Interceptor {

    /**
     * 拦截方法: 每次执行StatementHandler.prepare方法都之前都会执行下列方法
     * 3.拦截方法：只要被拦截的目标对象的目标方法被执行时，每次都会执行intercept方法
     * @param invocation
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println("在sql执行之前对方法进行增强");
        return invocation.proceed();   //调用原方法;
    }

    /**
     * 包装目标对象,为目标对象创建代理对象
     * 主要为了把当前的拦截器生成代理存到拦截器链中
     * 2.在拦截方法之前,每次调用都会先将当前拦截器代理存到拦截器链中
     * @param target    要拦截的对象
     * @return  代理对象
     */
    @Override
    public Object plugin(Object target) {
        System.out.println("将当前拦截器生成代理存到拦截器链中");
        Object wrap = Plugin.wrap(target, this);
        return wrap;
    }

    /**
     * 获取sqlMapConfig文件中插件配置参数
     * 插件初始化时调用,也只调用一次
     * 1.应用启动时最先执行下面方法获取插件的配置参数
     * @param properties
     */
    @Override
    public void setProperties(Properties properties) {
        System.out.println("获取sqlMapConfig文件中配置参数"+ properties);
    }
}

```

sqlMapConfig.xml中全局配置插件: 需要注意标签的顺序

```xml
<!--配置插件-->
<plugins>
	<!--配置自定义插件-->
	<plugin interceptor="com.fishleap.plugin.MyPlugin">
		<property name="username" value="jack"/>
	</plugin>
</plugins>
```





参考:

- [mybatis 面试题](https://blog.csdn.net/xsj_blog/article/details/80016602)






