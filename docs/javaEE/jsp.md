### 一.框架发展过程简单了解

在刚接触javaEE时,学的jsp做前端页面,当时还只知道 Jsp+Servlet+JavaBean 架构模式开发.

在这里简单了解一下后端框架的发展过程:

-   Jsp+Javabean 模式: 效率高,逻辑混乱,适合小项目
-   Jsp+Servlet+JavaBean 模式: 也就是现在所谓的MVC模式(模型,视图,控制器),便于分工,适合大项目,易于维护和扩展
-   Struts+Spring+Hibernate 模式: 也就是ssh框架,struts后来又发展为struts2
-   SpringMVC+Spring+MyBatis 模式: 也就是ssm框架
-   Springboot+MyBatis/JPA 模式
-   SpringCloud全家桶,依赖于Springboot,Springboot依赖于Spring

从以前的 单体应用 到 前后端分离 再到 微服务架构.

回归到jsp层面上,jsp属于模板引擎中的一种,比如现在springboot工程推荐用的thymeleaf.还有其它freemarker等都是模板引擎,这些都用来编写前端页面,说到前端框架,以前大家常用的是JQuery、Bootstrap框架,现在形成React、Vue、Angular三大主流框架的天下了.

使用MVC设计模式的开发要点:

-   jsp:做数据的展示.尽量不写java小脚本
-   servlet:对用户输入数据的封装(request.getParamter()),对业务处理结果的设置(request.setAttribute()),控制页面的流向
-   javabean:做相关的业务处理

### 二.web相关知识了解

#### 1.网络应用程序分类

C/S	客服端(client)/服务器(server) 典型应用:QQ,YY 

B/S 浏览器(browser)/服务器(server) 典型应用:sina,baidu

c/s的优点:

-   个性化更容易实现
-   更安全
-   占用网络资源少

B/S的优点:

-   更新方便
-   使用方便,到处可以使用
-   几乎不占用本地资源

#### 2.Web应用程序的目录结构

Web应用程序: 一般将工程打成war/jar包部署在 D:\tomcat7\webapps目录下.

WEB-INF目录:**web应用配置目录,不能被客户端访问,即将jsp放在该目录下时不能直接被客户端访问,属于安全目录**

-   classes目录:存放java字节码的文件
-   lib目录:存放web应用所需的jar包
-   web.xml文件:存放web应用的部署描述文件,该文件包含web应用的源数据信息

#### 3.http协议

url通常由4个部分组成,url格式如下:

-   应用层协议:// 主机ip地址或web服务器域名:协议端口号 / 资源所在路径 / 文件名
-   端口:一个ip地址的端口可达65536个之多,端口号只有整数,范围从0到65535,修改过端口号后需要重新启动tomcat服务器.80端口是http协议默认的端口,在访问网络地址时可以省略该端口号; tomcat默认的服务端口为8080
-   IP:用来识别主机
-   端口号:用来主机中应用程序

http协议:如何在网络上传输超文本(html文档)的协议; 浏览器发出http请求,服务器做出http响应

http处理流程:

-   客户端和web服务器建立连接
-   发送请求
-   接受请求,生成http响应并发送给客户端
-   服务器关闭连接,客户端接受服务器端的响应,恢复页面(关闭连接后,不再存储连接信息,即http协议称为无状态协议)

http请求方式:

-   get请求方式:
    -   仅能传送文本给服务器
    -   提交的数据会暴露再地址栏,不安全
    -   提交的数据不能超过2kb
-   post请求方式:
    -   可以传送二进制数据,如音频,视频等文件
    -   提交的数据不会暴露再地址栏,安全性高
    -   提交的数据无限制
-   表单通过method属性来指明使用哪种请求方式,默认是get请求方式

#### 4.分析tomcat目录结构

-   bin:存放常用的命令文件
-   conf:存放配置文件
-   lib:存放jar包
-   logs:存放日志文件
-   temp:存放临时文件
-   webapps:存放web应用程序(默认web应用发布目录)
-   work:存放由各种JSP所生成的Servlet文件(存放编译,运行后的文件)
-   conf里的配置文件:
    -   context.xml 配置上下文环境,如JNDI,连接池
    -   server.xml 配置服务器的信息,还有一些应用信息如端口号,虚拟路径
    -   web.xml web应用服务的部署文件(查)
    -   tomcat-users.xml 配置tomcat用户

#### 5.如何在tomcat中部署一个应用

直接将静态项目放入webapps目录下,直接访问即可 或 将项目打成jar/war包放在webapps目录下,启动服务会自动解析war包

http://localhost:8080/exam/zhuye.html
http://127.0.0.1:8080/exam/zhuye.html

HTTP:协议 localhost:服务器ip地址或域名 8080:端口号 exam:项目名 zhuye.html:主页名

服务器部署一般是上面的方法或通过idea直接部署到服务器上. 在tomcat中启动应用时,一般在开发工具里配置tomcat,直接在开发工具里启动应用.

#### 6.配置虚拟路径

配置tomcat虚拟发布目录,默认发布目录是webapps文件夹.需要修改:Conf/server.xml文件

在倒数第四行内,在`<host>` 标签下配置`<Context>` 子元素,使用子元素中相关属性配置tomcat虚拟发布目录:

`<Context path="/pro" docBase="E:\javaWeb\HTML\HtmlProject\project" />`

-   path指定访问web应用的url入口(url中的项目名)
-   docBase指明项目存放位置

然后清理浏览器缓存:ctrl+shift+delete,再访问项目

### 三.servlet介绍和使用

#### 1.概念及执行流程

servlet就是一个java类,服务器端的小程序,用来处理用户请求的.

用户发起一个请求后,由服务器接受处理,根据web.xml文件中的配置信息,查找所请求的资源(访问路径)是否存在,如果不存在则返回错误(404);当找到资源后(servlet),检查该Servlet对象是否存在,如果不存在则创建该对象,如果
存在则执行相应的处理方法.处理方法执行以后将返回处理结果给web服务器,web服务器根据结果进行相关处理后,返回给浏览器,浏览器显示相应的处理结果.

#### 2.servlet生命周期

Servlet 生命周期：**Servlet 加载--->实例化--->服务--->销毁**。

`init()` : 在Servlet的生命周期中，仅执行一次init()方法. 它是在服务器载入Servlet时执行的，负责初始化Servlet对象。可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。无论有多少客户机访问Servlet，都不会重复执行init（）。

`service()` : 它是Servlet的核心，负责响应客户的请求。每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”(ServletResponse)对象作为参数。在HttpServlet中已存在Service()方法。默认的服务功能是调用与HTTP请求的方法相应的do功能。

`destroy()` : 仅执行一次，在服务器端停止且卸载Servlet时执行该方法。当Servlet对象退出生命周期时，负责释放占用的资源。一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确认在调用destroy()方法时，这些线程已经终止或完成。

创建Servlet对象的时机：

-   Servlet容器启动时, 加载Servlet类,读取web.xml配置文件中的信息，创建ServletConfig对象, 构造指定的Servlet对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法。

-   在Servlet容器启动后,客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet 对象的service方法进行处理,然后交由相应的doPost()方法或doGet()方法进行处理.

-   注:Servlet容器在启动时自动创建Servlet，这是由在web.xml文件中为Servlet设置的`<load-on-startup>` 属性决定的。从中我们也能看到同一个类型的Servlet对象在Servlet容器中以单例的形式存在。

    ```xml
    <servlet>
      <servlet-name>Init</servlet-name>
      <servlet-class>org.xl.servlet.InitServlet</servlet-class>
      <load-on-startup>1</load-on-startup>
    </servlet>
    ```

生命周期总结:

-   初始化:
    -   当Tomcat服务器加载servlet时会产生一个.class
    -   生成一个改servlet的servletconfig对象
    -   当第一次请求到达时创建servlet实例对象（只执行一次）
    -   调用init方法进行初始化（只执行一次）
-   服务:
    -   当请求到达时Service方法会判断用户的请求方法
    -   如果get请求：调用doGet方法
    -   如果post请求：调用doPost方法
-   销毁:
    -   关闭服务器时销毁servlet实例对象

#### 3.servlet开发

##### Servlet配置文件说明

每写一个selvlet类,就需要配置一个servlet. 每个servlet都有三个名字:类名,servlet-name,访问名. servlet-name:唯一(与类名相似). 访问名:访问路径(可以有多个)

通知Tomcat服务器对于哪个请求调用哪一个Servlet对象进行处理，对Servlet起到注册的作用.

```xml
//声明Servlet对象
<servlet>
  //指定Servlet的名称，可以自定义名称，但要唯一
  <servlet-name>FirstServlet</servlet-name>
  //指定Servlet对象的完整位置：类的全限定名称：包名+类名
  <servlet-class>com.book.servlet.FirstServlet</servlet-class>
</servlet>

//映射Servlet
<servlet-mapping>
  //与上面的元素内容保持一致
  <servlet-name>FirstServlet</servlet-name>
  //用于映射访问Servlet的url
  <url-pattern>/FristServlet</url-pattern>
</servlet-mapping>
```

##### Servlet运行原理

Tomcat接受一个http请求时,根据请求内容创建Servlet实例的步骤: 

用户(url) --> tomcat服务器(url中资源名) --> tomcat服务器(资源名与url-pattern内容匹配)--> tomcat服务器(取出servlet-name的内容值)  --> 与sevlet标签中servlet-name匹配  -->tomcat服务器实例化该Servlet

Servlet运行原理:

-   服务器接受请求 --> Servlet实例是否存在? --> 存在:直接调用service方法
-   --> 不存在:装载Servlet类并创建实例  -->调用init方法初始化  --> 再调用service方法

#### 4.Servlet API常用接口和类

HttpServletRequest接口:

-   `String getParameter(String name)` 	//获取指定名称的参数值
-   `String[] getParameterValues(String name)`	//获取相同名称参数的数组值

HttpServletResponse接口

-   `PrintWriter getWriter()`	//获取打印流对象
-   `void sendRedirect(String path)`	//将请求重定向到指定位置
-   `void setCharacterEncoding(String enc)`	//设置响应编码
-   `void addCookie()`	//向响应中添加cookie对象

ServletConfig接口:类似局部变量，某一个servlet所私有的

-   `String getInitParameter(String path)`	//获取web.xml指定Servlet的初始化参数值

ServletContext接口:类似全局变量，整个web应用所共享的

-   `String getInitParameter(String name)`	//返回web应用范围内匹配的初始化参数值

#### 5.重定向和请求转发

重定向: 响应对象(resp)调用`sendRedirect(String path)` 方法. 也可以在path后手动配置请求参数和值(我请Jack买东西,Jack没时间,我就再请Tom买,Tom再响应给我)

请求转发: 请求对象(req)调用`getRequestDispatcher(String url)` 方法获取实例,RequestDispatcher实例调用`forward(req,resp)` 方法(我请Jack买东西,Jack找Tom帮忙买,Jack再响应给我)

重定向：(访问站外资源时用重定向)

-   浏览器URL会发生改变
-   两次请求，两次响应，所以request不同，不能通过request传递参数
-   可以访问站外资源

请求转发：(传递请求数据用请求转发)

-   浏览器URL不会发生改变
-   请求转发通用同一个request对象，可以通过request传递参数
-   只能访问当前站点资源

#### 6.利用请求域属性传递对象数据

请求对象中存储对象的常用方法:

-   `void setAttribute(String name,Object obj)` 	//将对象存储到请求对象中
-   `Object getAttribute(String name)` 	//获取存储在请求对象中的对象
-   `void removeAttribute(String name)` 	//从请求对象中删除指定名称的属性

存储在请求实例中的数据称为请求域属性,处于同一请求过程的多个处理模块之间,可以通过请求域属性来传递对象数据.

-   请求转发 可以通过 请求域属性 来传递数据
-   重定向 不可以通过 请求域属性 来传递数据

请求对象接口分为两个空间,一个存储页面提交的数据,一个存储setAttribute的数据

-   `String getParameter(String name)` 	//获取页面提交的请求参数值
-   `Object getAttribute(String name)` 	//获取存储再在请求对象中的对象

### 四.Cookie的概念和使用

cookie指浏览器能永久存储的一种数据,仅仅时浏览器实现的一种数据存储功能.**cookie由服务器生成,发送给浏览器,浏览器把cookie以key-value形式保存到某个目录下的文本文件内,下一次请求同一网站会把该cookie发送给服务器.cookie存放在客服端,一般用来保存用户信息**.

**cookie在服务端的使用**:

```java
//1.cookie的创建和存储
    //创建cookie对象
    Cookie cookie = new Cookie(String name, String value);
    //设置有效期
    cookie.setMaxAge(int seconds);
    //设置有效路径
    cookie.setPath(String uri);
    //响应cookie信息给客户端
    response.addCookie(cookie);
//2.cookie的获取
    //获取cookie信息数组
    Cookie[] cookies = request.getCookies();
    //遍历数组获取cookies信息
    if(cookies != null){
      //增强for循环写法
      for(Cookie c:cookies){
        String name = c.getName();
        String value = c.getValue();
        System.out.println(name+":"+value);
      }
      //java8写法
      Arrays.stream(cookies).map(c->c.getName()+":"+c.getValue())
        .collect(Collectors.joining(", "));
    }
//3.spring框架注解 @CookieValue 获取特定的cookie的值
    public void readCookie(@CookieValue(value="name",defaultValue="Jack")String name){
    }
```

注意: 一个Cookie对象存储一条数据。多条数据，可以多创建几个Cookie对象进行存储。

特点:

-   浏览器端的数据存储技术。
-   存储的数据声明在服务器端。
-   临时存储:存储在浏览器的运行内存中，浏览器关闭即失效。
-   定时存储:设置了Cookie的有效期，存储在客户端的硬盘中，在有效期内符合路径要求的请求都会附带该信息。
-   默认cookie信息存储好之后，每次请求都会附带，除非设置有效路径

### 五.Session的概念和使用

session是浏览器与服务器之间的一次会话,包含多个请求,session是服务器为每个客户在服务器端开辟的一块空间.当每个用户都有自己不同的数据时,像购物车,就要使用session. session和浏览器服务器都有关,存放在服务器.

**session保存数据状态,进行身份认证方式**:

客服端和服务端第一次建立请求时,也就是第一次登录成功时,服务器会为该用户创建一个session对象,并且指派一个sessionID.当服务器响应客服端时,会将这个sessionID以cookie方式写入浏览器的内存中,当用户再一次发送请求时,就将该sessionID传给服务器,服务器根据接受的sessionID去查找该客服端对应的session信息进行比较验证身份,响应客服端信息时附带用户当前的状态.

![session](../../media/pictures/session.png)

-- 图片来源: [认证授权基础](https://snailclimb.gitee.io/javaguide/#/docs/system-design/authority-certification/basis-of-authority-certification?id=7-%e4%bb%80%e4%b9%88%e6%98%afoauth-20%ef%bc%9f)

注意: 使用session时确保客服端开启了cookie(依赖cookie技术),注意session的过期时间(默认存储时间时30分钟).

**session在服务端的使用**:

```java
//创建或获取session对象: 当请求中有sessionID标志时就是获取session对象,没有就是创建session对象
HttpSession session = request.getSession();
//存储数据: 一般用户在登陆web项目时会将用户的个人信息存储到Sesion中,供该用户的其他请求使用
session.setAttribute(String name, Object value);
//获取数据: 存储的动作和取出的动作发生在不同的请求中，但是存储要先于取出执行
session.getAttribute(String name);
//移除数据: 移除用户
session.removeAttribute(String name);
//设置session存储时间: 在指定的时间内session对象没有被使用则销毁,如果使用了则重新计时
session.setMaxInactiveInterval(int seconds);
//设置session强制失效
session.invalidate();
```

**注意**: **浏览器关闭后,session没有删除(服务器),再打开浏览器不能找到原来服务器里的session对象(sessionID存储在浏览器的cookie临时存储空间中)**

**session何时被删除**: 超时,强制失效,服务器关闭或停止. 

**问题**: 没有cookie话session还可以用嘛? 一般是通过cookie保存sessionID.但也并不是不能用,可以将sessionID放在请求的url里面.









