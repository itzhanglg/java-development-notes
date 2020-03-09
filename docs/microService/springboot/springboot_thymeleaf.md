
### 一.概述

JavaEE领域有几种常用的模板引擎: Jsp, Thymeleaf, Freemarker, Velocity等.对于前端页面渲染效率来说 JSP 其实还是最快的, Velocity次之.Thymeleaf虽然渲染效率不是很快,但语法比较轻巧.

Thymeleaf 支持html5标准, Thymeleaf页面无需部署到servlet开发到服务器上,以 .html 后缀结尾,可直接通过浏览器就能打开.可完全替代JSP(前后端分离不是很好).

Thymeleaf可以让美工在浏览器查看页面的静态效果,也可以让程序员在服务器查看带数据的动态页面效果.(支持html原型,在html标签增加额外的属性来达到 模板+数据 的展示方式).浏览器解锁html时会忽略未定义的标签属性,模板可以静态运行;当有数据返回到页面时,Thymeleaf标签会动态的替换静态内容,使页面动态显示.

Thymeleaf提供标准和spring标准两种方言,可以直接套用模板实现JSTL,OGNL表达式效果.提供spring标准方言和一个与springMVC完美集成的可选模块,可以快速实现表单绑定,属性编辑器,国际化等功能.

### 二.springboot集成thymeleaf

springboot项目默认查找文件位置:

-   /src/java/resources/**static**
-   /src/java/resources/**public**
-   /src/java/resources/**templates**
-   /src/java/resources/**META-INF/resources**
-   /src/java/**resources** ：Maven的资源文件目录

#### 1.引入依赖

```java
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.1.4.RELEASE</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- set thymeleaf version -->
  	<thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
    <thymeleaf-layout-dialect.version>2.3.0</thymeleaf-layout-dialect.version>
    <!--set java version-->
  	<java.version>1.8</java.version>
</properties>
// 依赖中直接引入
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

#### 2.配置视图解析器

具体配置的参数可以查看`org.springframework.boot.autoconfigure.thymeleaf.ThymeleafProperties`这个类.

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    private Charset encoding;
    private boolean cache;
    ...
}
```

在application.properties中可以配置thymeleaf模板解析器属性.就像使用springMVC的 JSP解析器配置一样.下面这些配置实际上就是注入到**ThymeleafProperties类**中的属性值.

```properties
#配置thymeleaf缓存开发期间先关闭，否则影响测试
spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.mode=HTML
spring.thymeleaf.check-template-location=true
spring.thymeleaf.servlet.content-type=text/html
```

#### 3.编写demo

**controller:**

```java
@RestController
@RequestMapping("/thymeleaf")
public class HelloController {

    private Logger logger = LoggerFactory.getLogger(HelloController.class);

    @GetMapping("index")
    public ModelAndView index(ModelMap modelMap){
        modelMap.put("userName","Jack");
        modelMap.put("date",new Date());
        List<Map<String,Object>> list = new ArrayList<>();
        Map<String,Object> map1 = new HashMap<>();
        map1.put("id",1);
        map1.put("name","Tom");
        list.add(map1);
        Map<String,Object> map2 = new HashMap<>();
        map2.put("id",2);
        map2.put("name","Marry");
        list.add(map2);
        modelMap.put("list", list);
        return new ModelAndView("hello", modelMap);
    }
}
```

**hello.html:**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>springboot-thymeleaf</title>
</head>
<body>
    <h3>字符串属性</h3>
    <p th:text="${userName}" />
    <h3>日期属性</h3>
    <p th:text="${#dates.format(date, 'yyyy-MM-dd hh:mm:ss')}" />
    <h3>循环</h3>
    <div th:each="user,userStat :${list}">
        <p th:text="'第' + ${userStat.count} + '个用户'">
        <p>ID:<span th:text="${user.id}"></span></p>
        <p>名字:<span th:text="${user.name}"></span></p>
    </div>
    <h3>判断1</h3>
    <div th:switch="${userName}">
        <p th:case="'Jack'">存在Jack</p>
        <p th:case="'Mack'">存在Mack</p>
        <p th:case="*">不存在任何人</p>
    </div>
    <h3>判断3</h3>
    <span th:if="${userName} == 'Jack'" th:text="${userName}">  </span><br />
</body>
</html>
```

#### 4.效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200217225730815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

### 三.thymeleaf基础语法

#### 1.引入标签

在使用thymeleaf时首先需要在html标签里引入`xmlns:th="http://www.thymeleaf.org"` 才能使用 `th:*` 的语法.

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

指令优先级:

| Order | Feature                         | Attributes                            | remark                                   |
| ----- | ------------------------------- | ------------------------------------- | ---------------------------------------- |
| 1     | Fragment inclusion              | th:insert 、th:replace                 | 片段包含: `jsp:include`                      |
| 2     | Fragment iteration              | th:each                               | 遍历: `c:forEach`                          |
| 3     | Conditional evaluation          | th:if 、th:unless、th:switch、 th:case   | 条件判断: `c:if`                             |
| 4     | Local variable definition       | th:object 、th:with                    | 声明变量: `c:set`                            |
| 5     | General attribute modification  | th:attr、 th:attrprepend、th:attrappend | 任意属性修改支持prepend,append                   |
| 6     | Specific attribute modification | th:value、 th:href、 th:src...          | 修改指定属性默认值                                |
| 7     | Text (tag body modification)    | th:text 、th:utext                     | 修改标签体内容. `th:text`转义特殊字符, `th:utext`不转义特殊字符 |
| 8     | Fragment specification          | th:fragment                           | 声明片段                                     |
| 9     | Fragment removal                | th:remove                             | 删除片段                                     |

一些常用指令的使用说明:

| 指令          | 备注                             |
| ----------- | ------------------------------ |
| th:text     | 替换原来text中的文本(p,span,div,table) |
| th:value    | 替换原来value的值(form)              |
| th:object   | 替换标签的对象, `th:object="对象"`      |
| th:field    | 填充,若外层有对象,可以直接用 `*{属性} 取值`     |
| th:checked  | 当值为true时为选中                    |
| th:remove   | 删除                             |
| th:href     | 用 `@{...}`                     |
| th:if       | 值为true时才显示标签                   |
| th:unless   | 值为false时才显示标签                  |
| th:each     | 循环遍历结果集                        |
| th:style    | 替换原有样式                         |
| th:class    | 替换原有class样式                    |
| th:action   | 替换action地址,用 `@{...}` 取地址      |
| th:alt      | 用 `@{...}` 取地址                 |
| th:fragment | 定义一个fragment模板,后面再引用它          |

#### 2.获取变量值

```html
<p th:text="'hello, ' + ${name} + '!'">666666</p>
```

需要获取实体类中的属性值时,可以使用 `${对象.属性}` 方式获取,这个学JSP时EL表达式一样.注意: `$` 表达式只在标签内部生效.`th:text="${对象.属性名}"` 动态显示数据,替换静态值.原先静态值是开发前端时做展示用的.很好的做到了前后端分离.

#### 3.引入URL

Thymeleaf通过`@{...}` 语法来处理URL链接.标签属性有 `th:href` 和 `th:src`.

```html
<a th:href="@{http://www.zhangligong.xyz}">绝对路径</a>
<a th:href="@{/}">相对路径</a>
<a th:href="@{css/bootstrap.css}">Content路径,默认访问static下文件</a>
```

#### 4.字符串替换

有时只需要对一段文字中某一处地方进行替换,可以采用字符串拼接 `'...'+${对象模.属性}+'...' ` 或 `|... ${对象名.属性}...|`. 注意: `|...|` 只能包含变量表达式 `${...}` ,不能包含其它常量,条件表达式等

```html
<p th:text="'姓名: ' + ${user.name} + '!'">Jack</p>
<p th:text="|年龄: ${user.age}!|">18</p>
```

#### 5.运算符

一些算术运算也可以用: +, -, *, div(/) 和 mod(%).

```html
<div th:with="isEven=(${prodStat.count} % 2 == 0)">
```

这些运算符也可以在OGNL变量表达式内部应用(由OGNL执行):

```html
<div th:with="isEven= ${prodStat.count % 2 == 0}">
```

逻辑运算符gt(>), lt(<), le(<=), ge(>=), eq(==), neq/ne(!=)都可以使用，XML规定，不得在属性值中使用`<`和`>`符号，因此应将它们替换为`&lt;`和`&gt;`。

```html
<div th:if="${prodStat.count} &gt; 1">
<span th:text="'Execution mode is '+((${execMode}=='dev')?'Development':'Production')" />
```

#### 6.条件

##### if/unless

`th:if` 当值为true时标签才显示; `th:unless` 当值为false时标签才显示.

```html
<a th:href="@{/login}" th:unless=${session.user != null}>Login</a>
```

##### switch/case

thymeleaf支持switch结构,默认属性(default)用*表示.

```html
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="#{roles.manager}">User is a manager</p>
  <p th:case="*">User is some other thing</p>
</div>
```

#### 7.循环

对数据集进行遍历使用 `th:each` 标签:

```html
<table>
    <tr>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
    </tr>
    <tr th:each="prod : ${prods}">
      <td th:text="${prod.name}">Onions</td>
      <td th:text="${prod.price}">2.41</td>
      <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
</table>
```

#### 8.Expression Utility对象

Thymeleaf提供了一系列Utility对象(内置于Context中),可以通过`#` 直接访问.具体API可访问: [usingthymeleaf-utility](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression-utility-objects)

-   **#execInfo**：表达式对象，提供有关Thymeleaf标准表达式中正在处理的模板的有用信息。
-   **#messages**：实用程序方法，用于获取变量表达式内的外部化消息，其方式与使用`#{...}`语法获得消息的方式相同。
-   **#uris**：在Thymeleaf标准表达式内执行URI / URL操作（尤其是转义/转义）的实用程序对象。
-   **#conversions**：实用程序对象，允许在模板的任何位置执行*转换服务*
-   **#dates**：`java.util.Date`对象的实用程序方法
-   **#calendars**：类似于`#dates`，但对于`java.util.Calendar`对象
-   **#numbers**：用于数字对象的实用方法
-   **#strings**：`String`对象的实用方法
-   **#objects**：一般对象的实用方法
-   **#bools**：用于布尔值评估的实用方法
-   **#arrays**：数组的实用方法
-   **#lists**：列表的实用方法
-   **#sets**：set集合的实用方法
-   **#maps**：map集合的实用方法
-   **#aggregates**：在数组或集合上创建聚合的实用程序方法
-   **#ids**：用于处理`id`可能重复（例如，由于迭代的结果）的属性的实用方法

### 四.thymeleaf示例

#### 1.表达式

##### 标准变量表达式

```html
<table>
  <tr>
    <td th:text="${user.id}">1</td>
    <td th:text="${user.name}">Jack</td>
    <td th:text="${user.phone}">136</td>
  </tr>
</table>
```

##### 选择变量表达式

```html
<table>
    <tr th:object="${user}">
        <td th:text="*{id}">1</td>
        <td th:text="*{name}">Jack</td>
        <td th:text="*{phone}">136</td>
    </tr>
</table>
```

##### url表达式

将后台传入的数据拼接到url中:

```html
<a href="info.html" th:href="@{/user/info(id=${user.id})}">参数拼接</a>
<a href="info.html" th:href="@{/user/info(id=${user.id},phone=${user.phone})}">多参数拼接</a>
<a href="info.html" th:href="@{/user/info/{uid}(uid=${user.id})}">restful风格</a>
<a href="info.html" th:href="@{/user/info/{uid}/abc(uid=${user.id})}">restful风格</a>
```

##### 字符串拼接

方式一:

`<span th:text="'当前是第'+${page}+'页 ,共'+${page}+'页'"></span>` 

方式二: 使用"|"减少了字符串的拼接

`<span th:text="|当前是第${page}页，共${page}页|"></span>` 

#### 2.常用属性

##### 循环list

```html
<table>
    <tr th:each="user, interStat : ${userList}">
        <td th:text="${interStat.index}"></td>
        <td th:text="${user.id}"></td>
        <td th:text="${user.name}"></td>
        <td th:text="${user.phone}"></td>
    </tr>
</table>
```

这里的interStat类似于jstl里面foreach的varStatus，可以获取到当前的迭代信息。interStat里面一些属性的含义：

-   index: 当前迭代对象的index（从0开始计算）
-   count: 当前迭代对象的个数（从1开始计算）
-   size: 被迭代对象的大小
-   current: 当前迭代变量
-   even/odd: 布尔值，当前循环是否是偶数/奇数（从0开始计算）
-   first: 布尔值，当前循环是否是第一个
-   last: 布尔值，当前循环是否是最后一个

##### 循环map

myMapVal.key相当于map的键，myMapVal.value相当于map中的值。

```html
<div th:each="myMapVal : ${userMap}">
    <span th:text="${myMapValStat.count}"></span>
    <span th:text="${myMapVal.key}"></span>
    <span th:text="${myMapVal.value.name}"></span>
    <span th:text="${myMapVal.value.phone}"></span>
    <br/>
</div>
```

##### 循环array

```html
<div th:each="myArrayVal : ${userArray}">
    <div th:text="${myArrayVal.name}"></div>
    <div th:text="${myArrayVal.phone}"></div>
</div>
```

##### if判断

条件判断，比如后台传来一个变量，判断该变量的值，0为男，1为女：

```html
<span th:if="${sex} == 0" >
    男：<input type="radio" name="sex"  th:value="男" />
</span>
<span th:if="${sex} == 1">
    女：<input type="radio" name="sex" th:value="女"  />
</span>
```

##### switch判断

`*` 表示默认，当下面的case都是false的时候，会执行默认的内容。

```html
<div th:switch="${sex}">
  <p th:case="0">性别：男</p>
  <p th:case="1">性别：女</p>
  <p th:case="*">性别：未知</p>
</div>
```

##### 三目运算符

`<span th:text="${sex eq 0} ? '男' : '女'">未知</span>` 

##### id属性

动态设置html标签中的id属性

```html
<span th:id="${uid}">id</span>
```

从后台传入的uid的值，然后将这个值作为id的值。

##### value属性

类似html标签中的value属性，能对某元素的value属性进行赋值，比如：

```html
<input type="hidden" id="userId" name="userId" th:value="${userId}">
```

##### th:inline用法

th:inline有三个取值类型:

-   text: 从后台取出数据展示

    ```html
    <span th:inline="text">Hello, [[${user.nick}]]</span>
    等同于：
    <span>Hello, <span th:text="${user.nick}"></span></span>
    ```

-   none: 希望在html中直接显示[[1,2,3],[4,5]],可以使用none

    ```html
    <p th:inline="none"> [[1, 2, 3], [4, 5]]!</p>
    ```

-   javascript :希望在JavaScript中获取后台相应的数据

    ```html
    <script th:inline="javascript" type="text/javascript">
          var msg  = "Hello," + [[${user.phone}]];
          alert(msg);
    </script>
    ```

#### 3.内置对象

内置对象有#号开始引用.

`#request` 相当于HttpServletRequest对象, `${#request.getContextPath()}` .

`#session` 相当于HttpSession对象, `${#session.getAttribute("name")}` .

除了上面的对象外,thymeleaf还提供了功能性对象来处理:

-   **#dates**：java.util.Date对象的实用方法，可以调用里面的方法
-   **#numbers**：格式数学对象的实用方法
-   **#strings**：字符串对象的实用方法
-   **#lists**：list的实用方法
-   **#aggregates**：对数组或集合创建聚合的实用方法
-   **#objects**：对objects操作的实用方法
