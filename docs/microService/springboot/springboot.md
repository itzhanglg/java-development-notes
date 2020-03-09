
### 一.SpringBoot介绍

#### 1.Spring

Spring是重量级企业开发框架 **Enterprise JavaBean（EJB）** 的替代品，Spring为企业级Java开发提供了一种相对简单的方法，通过 **依赖注入** 和 **面向切面编程** ，用简单的 **Java对象（Plain Old Java Object，POJO）** 实现了EJB的功能。

**虽然Spring的组件代码是轻量级的，但它的配置却是重量级的（需要大量XML配置）** 。Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。

#### 2.SpringBoot

**简而言之，从本质上来说，Spring Boot就是Spring，它做了那些没有它你自己也会去做的Spring Bean配置。**

Spring Framework旨在简化J2EE企业应用程序开发。Spring Boot Framework旨在简化Spring开发。

**主要优点：**

- Spring Boot不需要编写大量样板代码、XML配置和注释。
- Spring Boot遵循“固执己见的默认配置”，以减少开发工作（默认配置可以修改）。
- Spring Boot 应用程序提供嵌入式HTTP服务器，如Tomcat和Jetty，可以轻松地开发和测试web应用程序。（这点很赞！普通运行Java程序的方式就能运行基于Spring Boot web 项目，省事很多）
- Spring Boot提供命令行接口(CLI)工具，用于开发和测试Spring Boot应用程序，如Java或Groovy。
- Spring Boot提供了多种插件，可以使用内置工具(如Maven和Gradle)开发和测试Spring Boot应用程序。

### 二.SpringBoot开发环境要求

#### 1.JDK

截止到目前Spring Boot 的最新版本：2.1.8.RELEASE 要求 JDK 版本在 1.8 以上，所以确保你的电脑已经正确下载安装配置了 JDK（推荐 JDK 1.8 版本）。

#### 2.构建工具

| Build Tool | Version |
| :--------: | :-----: |
|   Maven    |  3.3+   |
|   Gradle   |  4.4+   |

#### 3.开发工具

IDEA

#### 4.Web服务器

Spring Boot支持以下嵌入式servlet容器:

|     Name     | ServletVersion |
| :----------: | :------------: |
|  Tomcat 9.0  |      4.0       |
|  Jetty 9.4   |      3.1       |
| Undertow 2.0 |      4.0       |

### 三.HelloWorld项目结构分析

#### 1.创建SpringBoot项目的两种方式

通过 [https://start.spring.io/](https://start.spring.io/) 网站生成一个 SpringBoot 项目。

勾选上 Spring Web 这个模块，这是我们所必需的一个依赖。当所有选项都勾选完毕之后，点击下方的按钮 Generate 下载这个 Spring Boot 的项目。下载完成并解压之后，我们直接使用 IDEA 打开即可。

也可以直接通过 IDEA 来生成一个 Spring Boot 的项目，具体方法和上面类似：`File->New->Project->Spring Initializr`。实质上也是通过 https://start.spring.io 这个来生成的。

#### 2.SpringBoot项目结构分析

- src：存放工程代码
- static：存放静态资源，图片、css、js
- templates：存放模板文件，jsp、thymeleaf
- application.properties：配置文件
- test：测试文件
- Application为后缀的java类：SpringBoot 启动类

注意：**Spring Boot 的启动类是需要在最外层的，不然可能导致一些类无法被正确扫描到，导致一些奇怪的问题。** 

**项目结构：**

- HelloApplication.java：项目的启动类
- domain：用于实体(Entity)与数据访问层(Repository)
- mapper：类似dao层，用于处理sql
- service：业务类代码
- controller：页面访问控制
- config：存放一些配置类
- util：存放工具类
- common：存放公用类，自定义异常类等

#### 3.注解 @SpringBootApplication 分析

大概可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。根据 SpringBoot官网，这三个注解的作用分别是：

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的bean，注解默认会扫描该类所在的包下所有的类。
- `@Configuration`：允许在上下文中注册额外的bean或导入其他配置类。

实质 `@SpringBootApplication` 就是几个重要的注解组合，使用它可以避免每次开发 Spring Boot 项目都要写一些必备的注解。这一点在我们平时开发中也经常用到，比如我们通常会提一个**测试基类，这个基类包含了我们写测试所需要的一些基本的注解和一些依赖。**

```java
// Application类：启动类
@SpringBootApplication
public class ReportsApplication {
	public static void main(String[] args) {
		SpringApplication.run(ReportsApplication.class, args);
	}
}

// @SpringBootApplication注解
@SpringBootConfiguration	// @Configuration
@EnableAutoConfiguration	// @AutoConfigurationPackage
@ComponentScan( ... )
public @interface SpringBootApplication { ... }

// @SpringBootConfiguration
@Configuration	// @Component
public @interface SpringBootConfiguration { ... }

// @EnableAutoConfiguration
@AutoConfigurationPackage	// @Import({Registrar.class})
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration { ... }
```

### 四.开发RestFul web服务

#### 1.介绍

**传统的 MVC 模式开发会直接返回给客户端一个视图，但是 RESTful Web 服务一般会将返回的数据以 JSON 的形式返回，这也就是现在所推崇的前后端分离开发。**

#### 2.Lombok优化代码

依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.18</version>
</dependency>
```

需要下载 IDEA 中支持 lombok 的插件。

实体类：

```java
@Data
public class Book {
    private String name;
    private String description;
}
```

控制层：

```java
@RestController
@RequestMapping("/api")
public class BookController {

    List<Book> books = new ArrayList<>();

    @PostMapping("/book")
    public ResponseEntity<List<Book>> add(@RequestBody Book book){
        books.add(book);
        return ResponseEntity.ok(books);
    }

    @DeleteMapping("/book/{id}")
    public ResponseEntity del(@PathVariable("id") int id){
        books.remove(id);
        return ResponseEntity.ok(books);
    }

    @GetMapping("/book")
    public ResponseEntity find(@RequestParam("name") String name){
        List<Book> result = this.books.
                stream().
                filter(book -> book.getName().equals(name)).
                collect(Collectors.toList());

        return ResponseEntity.ok(result);
    }
}
```

1. `@RestController` **将返回的对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中。**绝大部分情况下都是直接以 JSON 形式返回给客户端，很少的情况下才会以 XML 形式返回。转换成 XML 形式还需要额为的工作，上面代码中演示的直接就是将对象数据直接以 JSON 形式写入 HTTP 响应(Response)中。
2. `@RequestMapping` :上面的示例中没有指定 GET 与 PUT、POST 等，因为**@RequestMapping默认映射所有HTTP Action**，你可以使用`@RequestMapping(method=ActionType)`来缩小这个映射。
3. `@PostMapping`实际上就等价于 `@RequestMapping(method = RequestMethod.POST)`，同样的 `@DeleteMapping` ,`@GetMapping`也都一样，常用的 HTTP Action 都有一个这种形式的注解所对应。
4. `@PathVariable` :取url地址中的参数。`@RequestParam` url的查询参数值。
5. `@RequestBody`:可以**将 HttpRequest body 中的 JSON 类型数据反序列化为合适的 Java 类型。**
6. `ResponseEntity`: **表示整个HTTP Response：状态码，标头和正文内容**。我们可以使用它来自定义HTTP Response 的内容。

#### 3.@Controller返回一个页面

当我们需要直接在后端返回一个页面的时候，Spring 推荐使用 Thymeleaf 模板引擎。Spring MVC中`@Controller`中的方法可以直接返回模板名称，接下来 Thymeleaf 模板引擎会自动进行渲染,模板中的表达式支持Spring表达式语言（Spring EL)。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

HelloController.java

```java
@Controller
public class HelloController {
    @GetMapping("/hello")
    public String greeting(@RequestParam(name = "name", required = false, defaultValue = "World") String name, Model model) {
        model.addAttribute("name", name);
        return "hello";
    }
}
```

hello.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Serving Web Content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>
<p th:text="'Hello, ' + ${name} + '!'"/>
</body>
</html>
```

访问：http://localhost:8080/hello?name=team-c 输出：`Hello, team-c!`

### 五.SpringBoot 异常处理

#### 1.使用 @ControllerAdvice 和 @ExceptionHandler 处理全局异常




