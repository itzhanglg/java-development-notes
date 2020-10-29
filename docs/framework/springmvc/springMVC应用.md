### 一.SpringMVC应用

#### 1.MVC体系结构

**三层架构**

在B/S架构中，系统标准的三层架构包括：表现层、业务层、持久层。

- 表现层：也就是web层。负责接受客户端请求，向客户端响应结果。表现层包括展示层（负责结果的展示）和控制层（负责接受请求）。表现层的设计一般使用MVC模型（和其他层没有关系）。
- 业务层：也就是service层。负责业务逻辑处理，若要对数据持久化需要保证事务一致性（事务应该放到业务层来控制）。
- 持久层：也就是dao层。负责数据持久化，和数据库交互，对数据库表进行增删改查。

**MVC设计模式**

MVC 全名是 Model View Controller，是 模型(model)－视图(view)－控制器(controller) 的缩写， 是⼀ 种⽤于设计创建 Web 应⽤程序表现层的模式。MVC 中每个部分各司其职：

- **Model**（模型）：模型包含**业务模型和数据模型**，数据模型⽤于封装数据，业务模型⽤于处理业务。
- **View**（视图）： 通常指的就是我们的 jsp 或者 html。作⽤⼀般就是**展示数据**的。通常视图是依据模型数据创建的。
- **Controller**（控制器）： 是应⽤程序中处理⽤户交互的部分。作⽤⼀般就是**处理程序逻辑**的。 MVC提倡：每⼀层只编写⾃⼰的东⻄，不编写任何其他的代码；分层是为了解耦，解耦是为了维 护⽅便和分⼯协作。

SpringMVC和Struts2一样，都是为了解决表现层问题的web框架，都是基于MVC设计模式的。表现层框架的主要职责就是处理前端HTTP请求。SpringMVC本质可以认为是对servlet的封装，简化了servlet的开发。SpringMVC用来接受请求，返回响应，跳转页面。

![image-20201027222647986](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201027222647986.png)

#### 2.SpringMVC工作流程

SpringMVC架构分析：

- 一个中心：前端控制器  (由springmvc提供)
- 三个基本点：处理器映射器  处理器适配器  视图解析器  (由springmvc提供)
- 两个开发：Handler处理器  jsp视图  (由程序员书写)

**SpringMVC请求处理流程**

![image.png](https://gitee.com/itzlg/mypictures/raw/master/img/1589730129678-6732a2cd-1782-4ffc-9b24-682a0461c9eb.png)

流程说明：

- 第⼀步：⽤户发送请求⾄前端控制器DispatcherServlet
- 第⼆步：DispatcherServlet收到请求调⽤HandlerMapping处理器映射器
- 第三步：处理器映射器根据请求Url找到具体的Handler（后端控制器），⽣成处理器对象及处理器拦截器(如果有则⽣成)⼀并返回DispatcherServlet
- 第四步：DispatcherServlet调⽤HandlerAdapter处理器适配器去调⽤Handler
- 第五步：处理器适配器执⾏Handler
- 第六步：Handler执⾏完成给处理器适配器返回ModelAndView
- 第七步：处理器适配器向前端控制器返回 ModelAndView，ModelAndView 是SpringMVC 框架的⼀个 底层对 象，包括 Model 和 View
- 第⼋步：前端控制器请求视图解析器去进⾏视图解析，根据逻辑视图名来解析真正的视图。
- 第九步：视图解析器向前端控制器返回View
- 第⼗步：前端控制器进⾏视图渲染，就是将模型数据（在 ModelAndView 对象中）填充到 request 域
- 第⼗⼀步：前端控制器向⽤户响应结果

**小结**：

用户请求到DispatcherServlet，DispatcherServlet让处理器映射器根据rul去找具体的处理器中请求映射的路径,对应的方法,返回找到的方法(包名+类名+方法名)；DispatcherServlet通过处理器适配器执行此方法(执行前绑定参数),返回ModelAndView ；DispatcherServlet让视图解析器去解析ModelAndView,返回具体View；DispatcherServlet对View进行渲染视图(即将模型数据填充至视图中)；DispatcherServlet渲好的html响应给用户。

**SpringMVC九大组件**

- HandlerMapping（处理器映射器）：Handler负责具体实际的请求处理，请求到达后，找到请求相应的处理器Handler和Interceptor。
- HandlerAdapter（处理器适配器）：让Servlet处理方法调用Handler来进行处理（Servlet方法结构都是doService(HttpServletRequest req,HttpServletResponse resp)形式的）。
- HandlerExceptionResolver（异常解析）：处理Handler产生的异常情况。根据异常设置ModelAndView，之后交给渲染方法进行渲染。
- ViewResolver（视图解析器）：将String类型的视图名和Locale解析成为View类型的视图，只有一个resolveViewName(ViewName，Locale)方法。视图解析器找到渲染所用的模板和所用的技术并填入参数。
- RequestToViewNameTranslator：当Handler处理完成后没有设置View或ViewName时，可以通过这个组件从请求中获取ViewName。
- LocaleResolver（区域解析器）：从请求中解析出Locale，用来表示一个区域。
- ThemeResolver（主题解析器）：用来解析主题的。主题是样式、图片及它们所形成的显示效果集合。
- MultipartResolver（上传解析）：用于上传请求，将普通请求包装成MultipartHttpServletRequest来实现文件上传功能。
- FlashMapManager：FlashMap用于重定向时的参数传递。只需要在重定向之前将要传递的数据写入请求的属性（OUTPUT_FLASH_MAP_ATTRIBUTE）中。FlashMapManager用来管理FlashMap的。

#### 3.请求参数绑定

SpringMVC如何接受请求参数？SpringMVC对Servlet进行了封装，简化了很多操作。参数绑定（取出参数值绑定到handler方法的形参上）。

**默认支持ServletAPI作为方法参数**

当需要使用HttpServletRequest、HttpServletResponse、HttpSession等原生servlet对象时，直接在handler方法中声明形参即可。

**绑定简单类型参数**

八种基本数据类型及包装类，参数推荐使用包装类型。对于布尔类型参数，请求参数值为true和false，或1和0。绑定简单数据类型参数，只需要直接声明形参即可(形参参数名和传递的参数名要保持一致,建议使用包装类型,当形参参数名和传递参数名不一致时可以使用@RequestParam注解进行手动映射)

**绑定Pojo类型参数**

Pojo类型参数，直接声明形参即可，类型就是Pojo的类型，形参名无所谓。但是要求传递的参数名必须和Pojo的属性名一致。

**绑定Pojo包装对象参数**

绑定参数时直接声明即可。传递的参数名必须和Pojo的属性名一致，如果不能定位数据项，需要通过属性名+“."的方式进一步锁定数据。例如：`url:/demo/handle?user.id=1&user.username=zhangsan`

**绑定日期类型参数(需要配置自定义类型转换器)**

绑定日期类型参数，需要定义一个SpringMVC的类型转换器接口，将日期字符串转为日期类型。如：

```java
// 自定义类型转换器
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {
        // 完成字符串向日期的转换
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        try {
            Date parse = simpleDateFormat.parse(source);
        	return parse;
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

SpringMVC中进行注册自定义类型转换器：

```xml
<!-- 自动注册合适的处理器映射器(调用handler方法) -->
<mvc:annotation-driven conversion-service="conversionServiceBean"/>

<!-- 注册自定义类型转换器 -->
<bean id="conversionServiceBean" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
		<set>
    		<bean class="com.fishleap.converter.DateConverter"></bean>
    	</set>
	</property>
</bean>
```

**小结**

- 默认参数绑定：Request 、Response、Session、Model(接口)->ModelMap(实现类)
- 简单类型参数绑定：方法的形参上(`@RequestParam("name")Integer/String/Double/Boolean...`)，表单参数与形参一致(不一致可以用@RequestParam注解)
- pojo类型：方法的形参上(@ModelAttribute("user")User user)，表单参数与实体属性一致，可以使用@ModelAttribute注解:将请求参数封装到对象user中,并以key=user存储到request作用域中
- QueryVo包装类(里面item)：表单参数与包装类中item对象的属性一致，QueryVo vo.item(实体属性)  --  item.name(表单参数)
- 自定义参数类型： 转换日期，如：
    ```xml
    <!-- springmvc.xml里配置转换器的工厂 
    converters -> list/set/array -> <bean class="自定义转换器类"/> -->
    <bean id="conversionServiceFactoryBean" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <!-- 配置多个转换器-->
        <property name="converters">
        	<list>
        		<bean class="com.springmvc.conversion.DateConveter"/>
            </list>
        </property>
    </bean>
    ```
    创建自定义转换器类：实现Converter<S,T>，S:页面传递过来的类型，T:转换后的类型。

#### 4.Restful风格请求和Ajax Json交互

**RESTful支持**

Restful就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。

SpringMVC对rest风格请求提供了@PathVariable注解，可以从uri中取出参数。从url上获取商品id，步骤如下:

```java
//使用注解@RequestMapping("item/{id}")声明请求的url,{xxx}叫做占位符，请求的URL可以是“item /1”或“item/2”
//使用(@PathVariable() Integer id)获取url上的数据

@RequestMapping(value = "/handle/{id}",method = {RequestMethod.GET})
public ModelAndView handleGet(@PathVariable("id") Integer id) {
    Date date = new Date();
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.addObject("date",date);
    modelAndView.setViewName("success");
    return modelAndView;
}
```

如果@RequestMapping中表示为"item/{id}"，id和形参名称一致，@PathVariable不用指定名称。如果不一致，例如"item/{ItemId}"则需要指定名称@PathVariable("itemId")。

注意：

- @PathVariable是获取url上数据的。@RequestParam获取请求参数的（包括post表单提交）
- 如果加上@ResponseBody注解，就不会走视图解析器，不会返回页面，返回目前的json数据。如果不加，就走视图解析器，返回页面。

**json数据交互**

- 前端到后台：前端ajax发送json格式字符串,后台直接接收为pojo参数,使用注解@RequstBody；
- 后台到前端：后台直接返回pojo对象,前端直接接收为json对象或者字符串,使用注解@ResponseBody。

`@RequestBody`注解用于读取http请求的内容(字符串)，通过springmvc提供的HttpMessageConverter接口，将读到的内容（json数据）转换为java对象并绑定到Controller方法的参数上。

`@ResponseBody`注解用于将Controller的方法返回的对象，通过springmvc提供的HttpMessageConverter接口，转换为指定格式的数据如：json,xml等，通过Response响应给客户端。注意:在使用此注解之后不会再走视图处理器,而是直接将数据写入到输入流中,他的效果等同于通过response对象输出指定格式的数据。

springmvc支持json包：jackson-annotations-2.4.0.jar、jackson-core-2.4.2.jar、jackson-databind-2.4.2.jar。

前端Ajax代码：

```js
$(function () {
    $("#ajaxBtn").bind("click",function () {
        // 发送ajax请求
        $.ajax({
            url: '/demo/handle',
            type: 'POST',
            data: '{"id":"1","name":"jack"}',
            contentType: 'application/json;charset=utf-8',
            dataType: 'json',
            success: function (data) {
            	alert(data.name);
            }
        })
    })
})
```

后台handler方法：

```java
@RequestMapping("/handle")
public @ResponseBody User handle(@RequestBody User user) {
    user.setName("tom");
    return user;
}
```

### 二.SpringMVC高级技术

#### 1.拦截器(Inteceptor)使用

**监听器、过滤器和拦截器对比**

**Servlet**：处理Request请求和Response响应。

**过滤器**（Filter）：对Request请求起到过滤的作⽤，作⽤在Servlet之前，如果配置为/*可以对所有的资源访问（servlet、js/css静态资源等）进⾏过滤处理。

**监听器**（Listener）：实现了javax.servlet.ServletContextListener 接⼝的服务器端组件，它随 Web应⽤的启动⽽启动，只初始化⼀次，然后会⼀直运⾏监视，随Web应⽤的停⽌⽽销毁。

- 作⽤⼀：做⼀些初始化⼯作，web应⽤中spring容器启动ContextLoaderListener。
- 作⽤⼆：监听web中的特定事件，⽐如HttpSession,ServletRequest的创建和销毁；变量的创建、 销毁和修改等。可以在某些动作前后增加处理，实现监控，⽐如统计在线⼈数，利⽤ HttpSessionLisener等。

**拦截器**（Interceptor）：是SpringMVC、Struts等表现层框架⾃⼰的，不会拦截 jsp/html/css/image的访问等，只会拦截访问的控制器⽅法（Handler）。 从配置的⻆度也能够总结发现：serlvet、filter、listener是配置在web.xml中的，⽽**interceptor是配置在表现层框架⾃⼰的配置⽂件中**的：

- 在Handler业务逻辑执⾏之前拦截⼀次；
- 在Handler逻辑执⾏完毕但未跳转⻚⾯之前拦截⼀次；
- 在跳转⻚⾯之后拦截⼀次。

![image.png](https://gitee.com/itzlg/mypictures/raw/master/img/1589731104903-22a877d8-c38b-414d-b5ca-f8f4ff99267a.png)

**拦截器的执行流程**

在运行程序时，拦截器的执行是有一定顺序的，该顺序与配置文件中所定义的拦截器的顺序相关。单个拦截器在程序中的执行流程如下图所示：

![image-20201029202040278](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201029202040278.png)

1. 程序先执行preHandle()方法,如果该方法的返回值为true,则程序会继续向下执行处理器中的方法,否则将不再向下执行。
2. 在业务处理器(即控制器Controller类)处理完请求后,会执行postHandle()方法,然后会通过1DispatcherServlet向客户端返回响应。
3. 在DispatcherServlet处理完请求后,才会执行afterCompletion()方法。

**多个拦截器的执行流程**

多个拦截器（假设有两个拦截器Interceptor1和Interceptor2，并且在配置⽂件中， Interceptor1拦截 器配置在前），在程序中的执⾏流程如下图所示：

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/1589731180841-cba4d2e5-5f40-4024-bbcf-89369d5ffa07.png" alt="image.png" style="zoom: 67%;" />

从图可以看出，当有多个拦截器同时⼯作时，它们的preHandle()⽅法会按照配置⽂件中拦截器的配置顺序执⾏，⽽它们的postHandle()⽅法和afterCompletion()⽅法则会按照配置顺序的反序执⾏。

**自定义SpringMVC拦截器**

1)定义拦截器(实现HandlerInterceptor接口)

```java
public class MyIntercepter01 implements HandlerInterceptor {
    
    /**
    * 会在handler方法业务逻辑执行之前执行
    * 往往在这里完成权限校验工作
    * 返回值Boolean代表是否放行，true代表放行，false代表中止
    */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    	// 从request中获取session
        HttpSession session = request.getSession();
        // 从session中获取username
        Object username = session.getAttribute("username");
        // 判断username是否为null
        if (username != null) {
            // 如果不为空则放行
            return true;
        } else {
            // 如果为空则跳转到登录页面
            response.sendRedirect(request.getContextPath() + "/user/toLogin.action");
            return false;
        }
    }
    
    /**
    * 会在handler方法业务逻辑执行之后尚未跳转页面时执行
    * 参数ModelAndView封装了视图和数据，此时尚未跳转页面，可以针对返回的数据和视图信息进行修改
    */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    	System.out.println("MyIntercepter01 postHandle......");
    }
    
    /**
    * 页面已经跳转渲染完毕之后执行
    * 参数ex可以在这里捕获异常
    */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    	System.out.println("MyIntercepter01 afterCompletion......");
    }  
}    
```

2)在springmvc.xml中配置拦截器

```xml
<mvc:interceptors>
    <!--拦截所有handler-->
    <!--<bean class="com.lagou.edu.interceptor.MyIntercepter01"/>-->
    
    <mvc:interceptor>
		<!--配置当前拦截器的url拦截规则，**代表当前目录下及其子目录下的所有url-->
		<mvc:mapping path="/**"/>
		<!--exclude-mapping可以在mapping的基础上排除一些url拦截-->
		<!--<mvc:exclude-mapping path="/demo/**"/>-->
		<bean class="com.fishleap.interceptor.MyIntercepter01"/>
	</mvc:interceptor>
    
    <mvc:interceptor>
    	<mvc:mapping path="/**"/>
    	<bean class="com.fishleap.interceptor.MyIntercepter02"/>
    </mvc:interceptor>
</mvc:interceptors>
```

总结:

- preHandle按拦截器定义顺序调用；
- postHandler按拦截器定义逆序调用；在拦截器链内所有拦截器返成功调用。
- afterCompletion按拦截器定义逆序调用；只有preHandle返回true才调用(当第一个拦截器为不为true时都不会调用)。

#### 2.处理multipart形式的数据

引入依赖：

```xml
<!--⽂件上传所需jar坐标--> 
<dependency> 
  <groupId>commons-fileupload</groupId> 
  <artifactId>commons-fileupload</artifactId> 
  <version>1.3.1</version>
</dependency>
```

配置上传解析器(springmvc.xml)：

```xml
<!--配置⽂件上传解析器，id是固定的multipartResolver--> 
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"> 
  <!--设置上传⼤⼩，单位字节--> 
  <property name="maxUploadSize" value="1000000000"/>
</bean>
```

设置表单可以进行文件上传：`enctype="multipart/form-data"`。

```html
<form method="post" enctype="multipart/form-data" action="/demo/upload">
    <input type="file" name="uploadFile"/>
    <input type="submit" value="上传"/>
</form>
```

图片上传(形参:MultipartFile pictureFile)：

```java
@RequestMapping(value = "/upload")
public ModelAndView upload(MultipartFile uploadFile, HttpSession session) throws IOException {

    // 处理上传文件
    // 重命名，原名123.jpg ，获取后缀
    String originalFilename = uploadFile.getOriginalFilename();// 原始名称
    // 扩展名  jpg
    //String extName = oriName.substring(oriName.lastIndexOf("."));
    String ext = originalFilename.substring(originalFilename.lastIndexOf(".") + 1, originalFilename.length());
    String newName = UUID.randomUUID().toString() + "." + ext;

    // 存储,要存储到指定的文件夹，/uploads/yyyy-MM-dd，考虑文件过多的情况按照日期，生成一个子文件夹
    String realPath = session.getServletContext().getRealPath("/uploads");
    String datePath = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
    File folder = new File(realPath + "/" + datePath);

    if(!folder.exists()) {
        folder.mkdirs();
    }

    // 存储文件到目录
    uploadFile.transferTo(new File(folder,newName));

    // TODO 文件磁盘路径要更新到数据库字段

    Date date = new Date();
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.addObject("date",date);
    modelAndView.setViewName("success");
    return modelAndView;
}
```

#### 3.在控制器中处理异常

全局异常处理可以让我们优雅的捕获所有Controller对象handler方法抛出的异常。

```java
@ControllerAdvice   // 全局异常
public class HandlerException {
    
    @ExceptionHandler(ArithmeticException.class)
    public ModelAndView exceptionHandler(ArithmeticException e, HttpServletResponse response){
        // 异常处理逻辑
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("msg", e.getMessage());
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

#### 4.基于Flash属性跨重定向请求数据传递

重定向时请求参数会丢失,我们往往需要重新携带请求参数,我们可以进行手动参数拼接如下：

```java
return "redirect:handle01?name=" + name;
```

但是上述拼接参数的方法属于get请求,携带参数长度有限制,参数安全性也不高。我们可以使用SpringMVC提供的**flash属性机制,向上下文中添加flash属性,框架会在session中记录该属性值,当跳转到页面之后框架会自动删除flash属性**,不需要我们手动删除,通过这种方式进行重定向参数传递,参数长度和安全性都得到了保障,如下：

```java
/**
 * SpringMVC 重定向时参数传递的问题
 * 转发：A 找 B 借钱400，B没有钱但是悄悄的找到C借了400块钱给A
 *      url不会变,参数也不会丢失,一个请求
 * 重定向：A 找 B 借钱400，B 说我没有钱，你找别人借去，那么A 又带着400块的借钱需求找到C
 *      url会变,参数会丢失需要重新携带参数,两个请求
 */
@RequestMapping("/handleRedirect")
public String handleRedirect(String name, RedirectAttributes redirectAttributes) {
    //return "redirect:handle01?name=" + name;  // 拼接参数安全性、参数长度都有局限
    // addFlashAttribute方法设置了一个flash类型属性，该属性会被暂存到session中，在跳转到页面之后该属性销毁
    redirectAttributes.addFlashAttribute("name",name);
    return "redirect:handler01";
}
/**
 * url: http://localhost:8080/demo/handle01
 * ModelAttribute接受了Session中的name值并绑定到name属性上
 * ModelAttribute和RequestBody都可以绑定到字符串类型
 */
@RequestMapping(value = "/handler01")
public ModelAndView handler01(@ModelAttribute("name")String name) {
    Date date = new Date();
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.addObject("date",date);
    modelAndView.setViewName("success");
    return modelAndView;
}
```






