## 一.习题

**一**.在项目现有的系统架构基础上，完成课程模块的遗留功能，主要包括后台管理系统对课程信息的新增、编辑、列表展示、以及课程的上架和下架服务。

需求描述：

1. 课程模块的功能作为服务单独部署

2. 应用注册中心Eureka，配置中心SpringCloud Config，网关SpringCloud Gateway

3. 所有课程功能都通过boss服务统一远程访问

4. 使用Postman工具访问测试课程服务

参考视频：课程模块-->课程基础功能实现。

**二**.课程上架成功的同时，以异步方式向当前系统的所有登录用户推送消息，浏览器显示“xxx课程新上线，欢迎试学”。

需求描述：

1. 整合消息中间件RockeMQ实现消息异步推送

2. 使用netty-socketio实现消息的远程推送

3. 最终效果为，课程上线后，浏览器自动显示该提示信息

参考视频：消息推送模块-->netty-socketio功能实现、消息推送功能实现。

## 二.项目架构

后台架构:

![image-20210208032353382](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210208032353382.png)

服务间的关系:

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20210208032951645.png" alt="image-20210208032951645" style="zoom:70%;" />



## 三.作业说明

课程操作接口说明：

- 课程列表：分页查询课程信息（/boss/course/getQueryCourses）。通过课程状态和课程名称来分页查询课程，如果没有填写状态则查询全部状态，如果没有名称则查询所有名称的课程，如果分页中没有指定每页的数量则默认查询 30 页，查询的课程按照排序字段倒序排列。
- 课程上下架：通过课程 Id 和课程状态来更改课程的状态上架和下架（/boss/course/changeState）。
- 保存或者更新课程信息：（/boss/course/saveOrUpdateCourse）。根据 Id 来判断更新还是保存，如果是有 ID 则更新，没有课程 Id 则直接保存。
- 通过课程 Id 获取课程信息：（/boss/course/getCourseById）。

接口测试：

| 接口描述                 | 网关路径                                                     | 请求类型 |
| ------------------------ | ------------------------------------------------------------ | -------- |
| 通过课程 Id 获取课程信息 | http://localhost:9001/boss/course/getCourseById?courseId=7   | GET      |
| 分页查询课程列表         | http://localhost:9001/boss/course/getQueryCourses            | POST     |
| 新增或者更新课程信息     | http://localhost:9001/boss/course/saveOrUpdateCourse         | POST     |
| 课程上下架               | http://localhost:9001/boss/course/changeState?courseId=7&status=0 | GET      |



### 1.注册中心Eureka

#### 1.1 引入依赖

```xml
<parent>
    <groupId>com.lagou</groupId>
    <artifactId>edu-bom</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>

    <!--java.lang.TypeNotPresentException: Type javax.xml.bind.JAXBContext not present-->
    <dependency>
        <groupId>javax.xml.bind</groupId>
        <artifactId>jaxb-api</artifactId>
        <version>2.3.0</version>
    </dependency>
    <dependency>
        <groupId>com.sun.xml.bind</groupId>
        <artifactId>jaxb-impl</artifactId>
        <version>2.3.0</version>
    </dependency>
    <dependency>
        <groupId>org.glassfish.jaxb</groupId>
        <artifactId>jaxb-runtime</artifactId>
        <version>2.3.0</version>
    </dependency>
    <dependency>
        <groupId>javax.activation</groupId>
        <artifactId>activation</artifactId>
        <version>1.1.1</version>
    </dependency>
</dependencies>
```

#### 1.2 配置文件

```yaml
server:
  port: 8761

spring:
  application:
    name: edu-eureka-boot

eureka:
  instance:
    hostname: localhost
  client:
    fetch-registry: false
    register-with-eureka: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

#### 1.3 启动类

```java
@SpringBootApplication
@EnableEurekaServer
public class LagouEurekaServer {
    public static void main(String[] args) {
        SpringApplication.run(LagouEurekaServer.class,args);
    }
}
```

### 2.配置中心Config

#### 2.1 引入依赖

```xml
<parent>
    <groupId>com.lagou</groupId>
    <artifactId>edu-bom</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>
```

#### 2.2 配置文件

```yaml
server:
  port: 8090

spring:
  application:
    name: edu-config-boot
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/xxxxxx/lagou-edu-repo.git
          username: xxxxxx
          password: xxxxxx
          default-label: master
```

#### 2.3 启动类

```java
@SpringBootApplication
@EnableConfigServer
public class LagouConfigServer { 
    public static void main(String[] args) {
        SpringApplication.run(LagouConfigServer.class,args);
    }
}
```

### 3.网关GateWay

#### 3.1 引入依赖

```xml
<parent>
    <groupId>com.lagou</groupId>
    <artifactId>edu-bom</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

#### 3.2 配置文件

```yaml
server:
  port: 9001

spring:
  application:
    name: edu-gateway-boot
  cloud:
    gateway:
      routes:
        - id: lagou-edu-front
          uri: lb://edu-front-boot
          predicates:
            - Path=/front/**
          filters:
            - StripPrefix=1
        - id: lagou-edu-boss
          uri: lb://edu-boss-boot
          predicates:
            - Path=/boss/**
          filters:
            - StripPrefix=1

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}
```

#### 3.3 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class LagouGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(LagouGatewayApplication.class, args);
    }
}
```

### 4.课程模块Couse

#### 4.1 api对外服务

> edu-course-boot-api

##### 4.1.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-extension</artifactId>
    <version>3.3.2</version>
    <scope>compile</scope>
</dependency>
```

##### 4.1.2 远程服务接口

```java
@FeignClient(name = "edu-course-boot", path = "/course")
public interface CourseRemoteService {

    /**
     * 保存或者更新课程
     * @param courseDTO
     * @return
     */
    @PostMapping(value = "/saveOrUpdateCourse",consumes = "application/json")
    ResponseDTO saveOrUpdateCourse(@RequestBody CourseDTO courseDTO);
    
    /**
     * 分页获取课程列表
     * @param courseQueryParam
     * @return
     */
    @PostMapping(value = "/getQueryCourses",consumes = "application/json")
    Page<CourseDTO> getQueryCourses(@RequestBody CourseQueryParam courseQueryParam);
    
    /**
     * 根据Id获取课程详情
     * @param courseId
     * @return
     */
    @GetMapping("/getCourseById")
    CourseDTO getCourseById(@RequestParam("courseId") Integer courseId);
    
    /**
     * 根据ID修改课程的上下架
     * @param courseId
     * @param status
     * @return
     */
    @GetMapping("/updateCourseStatusById")
    ResponseDTO updateCourseStatusById(@RequestParam("courseId") Integer courseId,@RequestParam("status")Integer status);
}
```

分页查询参数：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CourseQueryParam implements Serializable {
     Integer currentPage;
     Integer pageSize;
     String courseName;
     Integer status;
}
```

#### 4.2 服务主要实现模块

> edu-course-boot-impl

##### 4.2.1 引入依赖

```xml
<dependency>
    <groupId>com.lagou</groupId>
    <artifactId>edu-course-boot-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>


<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.21</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.2</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.3.2</version>
</dependency>

<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>com.lagou</groupId>
    <artifactId>edu-common</artifactId>
</dependency>
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.5.2</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

##### 4.2.2 配置文件

```yaml
server:
  port: 8003

spring:
  application:
    name: edu-course-boot
  cloud:
    config:
      uri: http://localhost:8090
      label: master
      profile: dev  #lagou-edu-order-(dev).yml
      name: lagou-edu-course  #(lagou-edu-order)-dev.yml

#注册到Eureka服务中心
eureka:
  client:
    service-url:
      # 注册到集群，就把多个Eurekaserver地址使用逗号连接起来即可；注册到单实例（非集群模式），那就写一个就ok
      defaultZone: http://127.0.0.1:8761/eureka/
  instance:
    prefer-ip-address: true  #服务实例中显示ip，而不是显示主机名（兼容老的eureka版本）
    # 实例名称： 192.168.1.103:lagou-service-resume:8080，我们可以自定义它
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}

# 配置日志
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

##### 4.2.3 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
@MapperScan("com.lagou.edu.course.impl.mapper")
public class LagouEduCourseApplication {
    public static void main(String[] args) {
        SpringApplication.run(LagouEduCourseApplication.class, args);
    }
}
```

##### 4.2.4 服务实现类

```java
// @FeignClient(name = "edu-course-boot", path = "/course")
@RestController
@RequestMapping("/course")//远程调用时指定path路径，这里必须要加上映射；若没有指定path可以不加
public class CourseService implements CourseRemoteService {
    @Autowired
    private ICourseService courseService;

    // 根据ID查询
    @Override
    public CourseDTO getCourseById(@RequestParam("courseId") Integer courseId) {
        final Course course = courseService.getById(courseId);
        final CourseDTO courseDTO = new CourseDTO();
        BeanUtil.copyProperties(course, courseDTO);
        return courseDTO;
    }
    // 更改上下架状态
    @Override
    public ResponseDTO updateCourseStatusById(final Integer courseId, final Integer status) {
        Course course = new Course();
        course.setId(courseId);
        course.setStatus(status);
        ResponseDTO responseDTO = null;
        try {
            courseService.updateById(course);
            responseDTO = ResponseDTO.success();
        } catch (Exception e) {
            responseDTO = ResponseDTO.ofError(e.getMessage());
            e.printStackTrace();
        }
        return responseDTO;
    }
    // 新增或保存
    @Override
    public ResponseDTO saveOrUpdateCourse(final CourseDTO courseDTO) {
        final Course course = ConvertUtils.convert(courseDTO, Course.class);
        System.out.println(courseDTO.getCreateTime());
        // 新增课程
        if (course.getId() == null) {
            course.setCreateTime(LocalDateTime.now());
            course.setUpdateTime(LocalDateTime.now());
        } else {  // 修改课程
            course.setUpdateTime(LocalDateTime.now());
        }
        ResponseDTO responseDTO = null;
        try {
            courseService.saveOrUpdate(course);
            responseDTO = ResponseDTO.success();
        } catch (Exception e) {
            responseDTO = ResponseDTO.ofError(e.getMessage());
            e.getStackTrace();
        }
        return responseDTO;
    }
    // 分页查询
    @Override
    public Page<CourseDTO> getQueryCourses(final CourseQueryParam courseQueryParam) {
        Integer currentPage = courseQueryParam.getCurrentPage();
        Integer pageSize = courseQueryParam.getPageSize();
        String courseName = courseQueryParam.getCourseName();
        Integer status = courseQueryParam.getStatus();
        
        QueryWrapper<Course> queryWrapper = new QueryWrapper<>();
        if (StringUtils.isNotBlank(courseName)) {
            queryWrapper.like("course_name", courseName);
        }
        if (status != null) {
            queryWrapper.eq("status", status);
        }
        // 查询总记录数
        final int count = courseService.count(queryWrapper);
        queryWrapper.orderByDesc("id");
        
        Page<Course> page = new Page<>(currentPage, pageSize == null ? 30 : pageSize);
        final IPage<Course> coursePage = courseService.getBaseMapper().selectPage(page, queryWrapper);
        
        List<CourseDTO> courseDTOList = new ArrayList<>();
        coursePage.getRecords().forEach( course -> {
            final CourseDTO courseDTO = new CourseDTO();
            BeanUtil.copyProperties(course, courseDTO);
            courseDTOList.add(courseDTO);
        });
        
        final Page<CourseDTO> coursePageResultDTO = new Page<>();
        BeanUtil.copyProperties(coursePage, coursePageResultDTO);
        coursePageResultDTO.setRecords(courseDTOList);
        coursePageResultDTO.setTotal(count);  //总记录数需要单独查询
        return coursePageResultDTO;
    }
}
```

### 5.统一后台入口Boss

#### 5.1 引入依赖

```xml
<parent>
    <groupId>com.lagou</groupId>
    <artifactId>edu-bom</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<dependencies>
    <dependency>
        <groupId>com.lagou</groupId>
        <artifactId>edu-course-boot-api</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>

    <dependency>
        <groupId>com.lagou</groupId>
        <artifactId>edu-common</artifactId>
    </dependency>
</dependencies>
```

#### 5.2 配置文件

```yaml
server:
  port: 8082

spring:
  application:
    name: edu-boss-boot

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}
```

#### 5.3 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients("com.lagou.edu")  //远程调用,扫描远程调用接口
public class LagouBossApplication {
    public static void main(String[] args) {
        SpringApplication.run(LagouBossApplication.class, args);
    }
}
```

#### 5.4 远程调用课程服务

```java
@RestController
@RequestMapping("/course")
public class CourseController {
    @Autowired
    private CourseRemoteService courseRemoteService;
    // 新增或保存课程信息
    @PostMapping("/saveOrUpdateCourse")
    public ResponseDTO saveOrUpdateCourse(@RequestBody CourseDTO courseDTO){
        return courseRemoteService.saveOrUpdateCourse(courseDTO);
    }
    // 根据ID查询课程信息
    @GetMapping("/getCourseById")
    public ResponseDTO<CourseDTO> getCourseById(@RequestParam("courseId") Integer courseId)  {
        final CourseDTO courseDTO = courseRemoteService.getCourseById(courseId);
        return ResponseDTO.success(courseDTO);
    }
    // 根据ID修改课程上下架
    @GetMapping("/changeState")
    public ResponseDTO changeState(@RequestParam("courseId") Integer courseId,
                                @RequestParam("status") Integer status)  {
        return courseRemoteService.updateCourseStatusById(courseId, status);
    }
	// 分页查询课程信息
    @PostMapping("/getQueryCourses")
    public ResponseDTO getQueryCourses(@RequestBody CourseQueryParam courseQueryParam)  {
        final Page<CourseDTO> queryCourses = courseRemoteService.getQueryCourses(courseQueryParam);
        return ResponseDTO.success(queryCourses);
    }
}
```

### 6.代码生成器MyBatisGenerator

#### 6.1 手动输入表名

```java
public class Generator {
    
    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }
    
    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();
        
        // 全局配置
        GlobalConfig gc = new GlobalConfig();
//        String projectPath = System.getProperty("user.dir");
        String projectPath = "E:\\develop\\lagou-edu\\edu-ad-boot\\edu-ad-boot-impl";
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setFileOverride(true);
        gc.setActiveRecord(false);// 不需要ActiveRecord特性的请改为false
        gc.setEnableCache(false);// XML 二级缓存
        gc.setBaseResultMap(true);// XML ResultMap
        gc.setBaseColumnList(true);// XML columList
        gc.setAuthor("zlg");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        mpg.setGlobalConfig(gc);
        
        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3308/edu_ad?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=Asia/Shanghai");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("root");
        mpg.setDataSource(dsc);
        
        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("模块名"));
        pc.setParent("com.lagou.edu");
        mpg.setPackageInfo(pc);
        
        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        
        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";
        
        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        /*
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // 判断自定义文件夹是否需要创建
                checkDir("调用默认方法创建的目录，自定义目录用");
                if (fileType == FileType.MAPPER) {
                    // 已经生成 mapper 文件判断存在，不想重新生成返回 false
                    return !new File(filePath).exists();
                }
                // 允许生成模板文件
                return true;
            }
        });
        */
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);
        
        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();
        
        // 配置自定义输出模板
        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();
        
        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);
        
        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        //strategy.setSuperEntityClass("你自己的父类实体,没有就不用设置!");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
        //strategy.setSuperControllerClass("你自己的父类控制器,没有就不用设置!");
        // 写于父类中的公共字段
        //strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}
```

#### 6.2 代码中写死表名

```java
public class MpGenerator {

    // 需要生成表的名称数组
    private static final String[] TABLE_NAME_ARRAY =
        new String[]{"tb_baseline", "tb_product", "tb_test_event", "tb_android_benchmark",
            "tb_apk_start_statistic", "tb_apk_start_statistic_noload"};

    /**
     * <p>
     * MySQL 代码生成
     * </p>
     */
    public static void main(String[] args) {
        AutoGenerator mpg = new AutoGenerator();
        // 选择 freemarker 引擎，默认 Veloctiy
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        //String projectPath = System.getProperty("user.dir");
        String projectPath = "E:\\develop\\lagou-edu\\edu-ad-boot\\edu-ad-boot-impl";
        // 生成代码的路径
        gc.setOutputDir(projectPath + "/course/src/main/java");
        gc.setFileOverride(true);
        gc.setActiveRecord(false);// 不需要ActiveRecord特性的请改为false
        gc.setEnableCache(false);// XML 二级缓存
        gc.setBaseResultMap(true);// XML ResultMap
        gc.setBaseColumnList(true);// XML columList
        // .setKotlin(true) 是否生成 kotlin 代码
        gc.setAuthor("zlg");
        gc.setOpen(false);
        // 自定义文件命名，注意 %s 会自动填充表实体属性！
        // gc.setMapperName("%sDao");
        // gc.setXmlName("%sDao");
        // gc.setServiceName("MP%sService");
        // gc.setServiceImplName("%sServiceDiy");
        // gc.setControllerName("%sAction");
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setDbType(DbType.MYSQL);
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("reporter");
        dsc.setPassword("Reporter01");
        dsc.setUrl("jdbc:mysql://10.150.154.190:3306/BasePerformance?useUnicode=true&characterEncoding=utf8&useSSL=false&useLegacyDatetimeCode=false&serverTimezone=UTC&createDatabaseIfNotExist=true");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setParent("com.transsion.microservice.baselineperf");
        pc.setModuleName("");
        pc.setController("web.rest.controller");
//        pc.setEntity("entity");
        pc.setMapper("dao");
//        pc.setService("service");
//        pc.setServiceImpl("service.impl");
        pc.setXml("dao.xml");
        mpg.setPackageInfo(pc);

        // 注入自定义配置，可以在 VM 中使用 cfg.abc 【可无】
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
//                Map<String, Object> map = new HashMap<String, Object>();
//                map.put("abc", this.getConfig().getGlobalConfig().getAuthor() + "-mp");
//                this.setMap(map);
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        //String entityPath = "/templates/entity.java.ftl";
//        String mapperPath = "/templates/mapper.java.ftl";


        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<FileOutConfig>();
        // 自定义 xxList.jsp 生成
//        focList.add(new FileOutConfig("/template/list.jsp.vm") {
//            @Override
//            public String outputFile(TableInfo tableInfo) {
//                // 自定义输入文件名称
//                return "D://my_" + tableInfo.getEntityName() + ".jsp";
//            }
//        });
//        cfg.setFileOutConfigList(focList);
//        mpg.setCfg(cfg);
        // 调整 xml 生成目录演示
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                //+ pc.getModuleName()
                return projectPath + "/baselineperf/src/main/resources/mapper/" +
                    tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
//        //dao数据访问对象
//        focList.add(new FileOutConfig(mapperPath) {
//            @Override
//            public String outputFile(TableInfo tableInfo) {
//                // 自定义输出文件名 ，将entity文件实体类放到model中
//                return projectPath + "/src/main/java/com/transsion/microservice/baselineperf/dao"
//                    + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_JAVA;
//            }
//        });
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 关闭默认 xml 生成，调整生成 至 根目录
        TemplateConfig tc = new TemplateConfig();
        // 配置自定义输出模板
        // 指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
//        tc.setEntity("templates/entity.java");
        tc.setXml(null);
        mpg.setTemplate(tc);

        // 自定义模板配置，可以 copy 源码 mybatis-plus/src/main/resources/templates 下面内容修改，
        // 放置自己项目的 src/main/resources/templates 目录下, 默认名称一下可以不配置，也可以自定义模板名称
        // TemplateConfig tc = new TemplateConfig();
        // tc.setController("...");
        // tc.setEntity("...");
        // tc.setMapper("...");
        // tc.setXml("...");
        // tc.setService("...");
        // tc.setServiceImpl("...");
        // 如上任何一个模块如果设置 空 OR Null 将不生成该模块。
        // mpg.setTemplate(tc);

        // 策略配置：数据库表配置
        StrategyConfig strategy = new StrategyConfig();
        // strategy.setCapitalMode(true);// 全局大写命名 ORACLE 注意
        //strategy.setTablePrefix(new String[] { "tlog_", "tsys_" });// 此处可以修改为您的表前缀
        strategy.setNaming(NamingStrategy.underline_to_camel);// 表名生成策略
        // 列名生成策略(下划线转驼峰命名)
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        // 需要生成的表
        strategy.setInclude(TABLE_NAME_ARRAY);
        strategy.setRestControllerStyle(true); //生成RestController控制器
        // 驼峰转连字符
        strategy.setControllerMappingHyphenStyle(false); //@RequestMapping("//test-event")
//        strategy.setTablePrefix(pc.getModuleName() + "_");

        // strategy.setExclude(new String[]{"test"}); // 排除生成的表
        // 自定义实体父类
        // strategy.setSuperEntityClass("com.baomidou.demo.TestEntity");
        // 自定义实体，公共字段
        // strategy.setSuperEntityColumns(new String[] { "test_id", "age" });
        // 自定义 mapper 父类
        // strategy.setSuperMapperClass("com.baomidou.demo.TestMapper");
        // 自定义 service 父类
        // strategy.setSuperServiceClass("com.baomidou.demo.TestService");
        // 自定义 service 实现类父类
        // strategy.setSuperServiceImplClass("com.baomidou.demo.TestServiceImpl");
        // 自定义 controller 父类
        // strategy.setSuperControllerClass("com.baomidou.demo.TestController");
        // 【实体】是否为lombok模型（默认 false)
        strategy.setEntityLombokModel(true);
        strategy.setTablePrefix("tb_"); // 生成实体时去掉表名的前缀，如 tb_
        // 【实体】是否为链式模型（默认 false）
        // public User setName(String name) {this.name = name; return this;}
        strategy.setChainModel(true);
        // 是否生成实体时，生成字段注解（默认为 true）
        strategy.setEntityTableFieldAnnotationEnable(true);
        // Boolean类型字段是否移除is前缀（默认 false）
        strategy.setEntityBooleanColumnRemoveIsPrefix(true);
        // 【实体】是否生成字段常量（默认 false）
        // public static final String ID = "test_id";
        strategy.setEntityColumnConstant(false);
        mpg.setStrategy(strategy);


        // 执行生成
        mpg.execute();

        // 打印注入设置【可无】
//        System.err.println(mpg.getCfg().getMap().get("abc"));
    }

}
```

