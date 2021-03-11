### 习题

1.在MySQL数据中建立lagou_db数据库, 将position.sql中的数据导入到mysql 数据中。

2.选择一种合理的方式(可以使用上课中使用的方式 也可以选择自己熟悉的插件等) 将mysql中的数据导入到ES中。

3.使用SpringBoot 访问ES 使用positionName 字段检索职位信息 如果检索到的职位信息不够5条 则需要启用poitionAdvantage 查找 美女多、员工福利好 的企业职位信息进行补充够5条。

提示：搜索一次查询不能满足要求时 发起第二次搜索请求。搜索一次满足需求不需要发送第二次搜索请求。

### 代码实现

#### 1.环境准备

SpringBoot工程中引入依赖：

```xml
<properties>
    <elasticsearch.version>7.3.0</elasticsearch.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>${elasticsearch.version}</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.elasticsearch/elasticsearch -->
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>${elasticsearch.version}</version>
    </dependency>

    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.3.0</version>
        <exclusions>
            <exclusion>
                <groupId>org.elasticsearch</groupId>
                <artifactId>elasticsearch</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.3.0</version>
    </dependency>
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
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- HttpClient -->
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
        <version>4.5.3</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.58</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.9</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <!--devtools热部署-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
        <scope>true</scope>
    </dependency>
</dependencies>
```

application.yml

```yaml
spring:
  devtools:
    restart:
      enabled: true  #设置开启热部署
      additional-paths: src/main/java #重启目录
      exclude: WEB-INF/**
    freemarker:
      cache: false    #页面不加载缓存，修改即时生效
  elasticsearch:
    rest:
      uris: 192.168.91.100:9200
      
server:
  port: 8080

logging:
  level:
    root: info
    com.xdclass.search: debug
```

#### 2.配置文件

数据库工具类：

```java
public class DBHelper {
    public static final String url = "jdbc:mysql://localhost:3306/lagou_position?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai";
    public static final String name = "com.mysql.cj.jdbc.Driver";
    public static final String user = "root";
    public static final String password = "root";
    private  static Connection  connection = null;

    public  static   Connection  getConn(){
        try {
            Class.forName(name);
            connection = DriverManager.getConnection(url,user,password);
        } catch (Exception e){
            e.printStackTrace();
        }
        return  connection;
    }
}
```

Elasticsearch配置类：

```java
@Configuration
public class EsConfig {
    
    @Value("${spring.elasticsearch.rest.uris}")
    private  String  hostlist;
    
    @Bean
    public RestHighLevelClient client() {
        //解析hostlist配置信息
        String[] split = hostlist.split(",");
        //创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for(int i=0;i<split.length;i++){
            String item = split[i];
            System.out.println(item);
            httpHostArray[i] = new HttpHost(item.split(":")[0], Integer.parseInt(item.split(":")[1]), "http");
        }
        //创建RestHighLevelClient客户端
        return new RestHighLevelClient(RestClient.builder(httpHostArray));
    }
}
```

#### 3.控制类

```java
@Controller
public class PositionController {
    
    @Autowired
    private PositionService service;

    // 测试页面
    @GetMapping({"/","/index"})
    public String indexPage(){
        return "index";
    }
    
    // 根据关键字分页查询
    @GetMapping("/search/{field}/{keyword}/{pageNo}/{pageSize}")
    @ResponseBody
    public List<Map<String,Object>> searchPosition(@PathVariable("field") String field,@PathVariable("keyword") String keyword,@PathVariable("pageNo")int pageNo,@PathVariable("pageSize")int pageSize) throws IOException{
        List<Map<String,Object>> list = service.searchPos(field, keyword, pageNo, pageSize);
        return  list;
    }
    
    // mysql数据库数据导入es
    @RequestMapping("/importAll")
    @ResponseBody
    public String importAll(){
        try {
            service.importAll();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return  "success";
    }
}
```

#### 4.服务类

服务接口：

```java
public interface PositionService {
    /*分页查询*/
    public List<Map<String,Object>> searchPos(String field,String keyword,int pageNo,int pageSize) throws  IOException;
    
    /*导入数据*/
    void importAll() throws IOException;
}
```

实现类：

```java
@Service
public class PositionServiceImpl  implements PositionService {
    
    private  static  final Logger  logger = LogManager.getLogger(PositionServiceImpl.class);
    
    @Autowired
    private RestHighLevelClient  client;
    private  static  final  String  POSITION_INDEX = "position";

    @Override
    public List<Map<String, Object>> searchPos(String field, String keyword, int pageNo, int pageSize) throws IOException {
        if (pageNo <= 1){
            pageNo = 1;
        }
        // 搜索
        SearchRequest  searchRequest = new SearchRequest(POSITION_INDEX);

        SearchSourceBuilder  searchSourceBuilder = new SearchSourceBuilder();
        // 分页设置
        searchSourceBuilder.from((pageNo-1)*pageSize);
        searchSourceBuilder.size(pageSize);
        QueryBuilder builder = QueryBuilders.matchQuery(field, keyword);
        searchSourceBuilder.query(builder);
        searchSourceBuilder.timeout(new TimeValue(60,TimeUnit.SECONDS));
        // 执行搜索
        searchRequest.source(searchSourceBuilder);
        SearchResponse  searchResponse = client.search(searchRequest,RequestOptions.DEFAULT);
        ArrayList<Map<String,Object>>  list = new ArrayList<>();
        SearchHit[]  hits = searchResponse.getHits().getHits();
        for (SearchHit hit:hits){
            list.add(hit.getSourceAsMap());
        }

        return   list;
    }

    @Override
    public void importAll() throws IOException {
         writeMySQLDataToES("position");
    }
    
    private void writeMySQLDataToES(String tableName){
        BulkProcessor  bulkProcessor  = getBulkProcessor(client);
        Connection  connection = null;
        PreparedStatement  ps = null;
        ResultSet  rs = null;
        try {
            connection = DBHelper.getConn();
            logger.info("start handle data :" + tableName);
            String  sql = "select * from " + tableName;
            ps = connection.prepareStatement(sql,ResultSet.TYPE_FORWARD_ONLY,ResultSet.CONCUR_READ_ONLY);
            // 根据自己需要设置 fetchSize
            ps.setFetchSize(20);
            rs = ps.executeQuery();
            ResultSetMetaData  colData = rs.getMetaData();
            ArrayList<HashMap<String,String>> dataList = new ArrayList<>();
            HashMap<String,String>  map = null;
            int  count = 0;
            // c 就是列的名字   v 就是列对应的值
            String  c = null;
            String  v = null;
            while(rs.next()){
                count ++;
                map = new HashMap<String,String>(128);
                for (int i=1;i< colData.getColumnCount();i++){
                    c = colData.getColumnName(i);
                    v = rs.getString(c);
                    map.put(c,v);
                }
                dataList.add(map);
                // 每1万条 写一次   不足的批次的数据 最后一次提交处理
                if (count % 10000 == 0){
                    logger.info("mysql handle data  number:"+count);
                    // 将数据添加到 bulkProcessor
                    for (HashMap<String,String> hashMap2 : dataList){
                        bulkProcessor.add(new IndexRequest(POSITION_INDEX).source(hashMap2));
                    }
                    // 每提交一次 清空 map 和  dataList
                    map.clear();
                    dataList.clear();
                }
            }
            // 处理 未提交的数据
            for (HashMap<String,String> hashMap2 : dataList){
                bulkProcessor.add(new IndexRequest(POSITION_INDEX).source(hashMap2));
            }
            bulkProcessor.flush();

        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            try {
                rs.close();
                ps.close();
                connection.close();
                boolean  terinaFlag = bulkProcessor.awaitClose(150L,TimeUnit.SECONDS);
                logger.info(terinaFlag);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    private BulkProcessor getBulkProcessor(RestHighLevelClient client) {

        BulkProcessor bulkProcessor = null;
        try {
            BulkProcessor.Listener listener = new BulkProcessor.Listener() {
                @Override
                public void beforeBulk(long executionId, BulkRequest request) {
                    logger.info("Try to insert data number : "
                            + request.numberOfActions());
                }

                @Override
                public void afterBulk(long executionId, BulkRequest request,
                                      BulkResponse response) {
                    logger.info("************** Success insert data number : "
                            + request.numberOfActions() + " , id: " + executionId);
                }

                @Override
                public void afterBulk(long executionId, BulkRequest request, Throwable failure) {
                    logger.error("Bulk is unsuccess : " + failure + ", executionId: " + executionId);
                }
            };

            BiConsumer<BulkRequest, ActionListener<BulkResponse>> bulkConsumer = (request, bulkListener) -> client
                    .bulkAsync(request, RequestOptions.DEFAULT, bulkListener);

            BulkProcessor.Builder builder = BulkProcessor.builder(bulkConsumer, listener);
            builder.setBulkActions(5000);
            builder.setBulkSize(new ByteSizeValue(100L, ByteSizeUnit.MB));
            builder.setConcurrentRequests(10);
            builder.setFlushInterval(TimeValue.timeValueSeconds(100L));
            builder.setBackoffPolicy(BackoffPolicy.constantBackoff(TimeValue.timeValueSeconds(1L), 3));
            // 注意点：让参数设置生效
            bulkProcessor = builder.build();

        } catch (Exception e) {
            e.printStackTrace();
            try {
                bulkProcessor.awaitClose(100L, TimeUnit.SECONDS);
            } catch (Exception e1) {
                logger.error(e1.getMessage());
            }
        }
        return bulkProcessor;
    }
}
```

#### 5.实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Position {
    //主键
    private String id;

    //公司名称
    private String companyName;

    //职位名称
    private String positionName;

    //职位诱惑
    private String  positionAdvantage;

    //薪资
    private String salary;

    //薪资下限
    private int salaryMin;

    //薪资上限
    private int salaryMax;

    //学历
    private String education;

    //工作年限
    private String workYear;

    //发布时间
    private String publishTime;

    //工作城市
    private String city;

    //工作地点
    private String workAddress;

    //创建时间
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;

    //工作模式
    private String jobNature;
}
```

#### 6.前端首页

```html
<!DOCTYPE html>
<html xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <link rel="stylesheet" href="layout_8a42639.css">
    <link rel="stylesheet"  href="main.html_aio_2_29de31f.css">
    <link rel="stylesheet"  href="main.html_aio_16da111.css">
    <link rel="stylesheet"  href="widgets_03f0f0e.css">

    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
</head>
<body>
<div id="top_bannerC">
    <a rel="nofollow" href="https://zhuanti.lagou.com/qiuzhaozhc.html"
       style="background: url(https://www.lgstatic.com/i/image/M00/3C/48/CgqCHl8nkiOAOihGAAC5a9EQ2rU358.PNG) center center no-repeat"
       target="_blank" data-lg-tj-id="bd00" data-lg-tj-no="idnull" data-lg-tj-cid="11127" class=""></a>
</div>

<div id="lg_header" >
    <!-- 页面主体START -->
    <div id="content-container">

        <div class="search-wrapper" style="height: 180px;">
            <div id="searchBar" class="search-bar" style="padding-top: 30px;">
                <div class="tab-wrapper" style="display: block;">
                    <a id="tab_pos" class="active" rel="nofollow" href="javascript:;">职位 ( <span>500+</span> ) </a>
                    <a id="tab_comp" class="disabled" rel="nofollow" href="javascript:;">公司 ( <span>0</span> ) </a>
                </div>
                <div class="input-wrapper" data-lg-tj-track-code="search_search" data-lg-tj-track-type="1">
                    <div class="keyword-wrapper">
                        <span role="status" aria-live="polite" class="ui-helper-hidden-accessible"></span><input
                            type="text" id="keyword" v-model="keyword" autocomplete="off" maxlength="64" placeholder="搜索职位、公司或地点"
                            value="java" class="ui-autocomplete-input"   @keyup.enter="searchPosition">
                    </div>
                    <input type="button" id="submit"  value="搜索" @click="searchPosition">
                </div>
            </div>
        </div>

        <!-- 搜索输入框模块 -->
        <div id="main_container">
            <!-- 左侧 -->
            <div class="content_left">
                <div class="s_position_list " id="s_position_list">
                    <ul class="item_con_list" style="display: block;">
                        <li class="con_list_item default_list"  v-for="element in results">
                            <span class="top_icon direct_recruitment"></span>
                            <div class="list_item_top">
                                <div class="position">
                                    <div class="p_top">
                                        <h3 style="max-width: 180px;">{{element.positionName}}</h3>
                                        <span class="add">[<em>{{element.workAddress}}</em>]</span>
                                        <span class="format-time"> {{element.createTime}} 发布</span>
                                    </div>
                                    <div class="p_bot">
                                        <div class="li_b_l">
                                            <span class="money">{{element.salary}}</span>
                                            经验 {{element.workYear}} 年 / {{element.education}}
                                        </div>
                                    </div>
                                </div>
                                <div class="company">
                                    <div class="company_name">
                                        <a href="#" target="_blank">{{element.companyName}}</a>
                                        <i class="company_mark"><span>该企业已经上传营业执照并通过资质验证审核</span></i>
                                    </div>
                                    <div class="industry">
                                        福利 【{{element.positionAdvantage}}】
                                    </div>
                                </div>
                                <div class="com_logo">
                                    <img src="//www.lgstatic.com/thumbnail_120x120/i/image2/M01/79/70/CgotOV1aS4qAWK6WAAAM4NTpXws809.png"
                                         alt="拉勾渠道" width="60" height="60">
                                </div>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</div>
</div>
</body>
<script>
    new Vue({
        el: "#lg_header",
        data: {
            keyword: "",  //搜索关键字
            results: []  //查询结果
        },
        methods: {
            searchPosition(){
                var keyword = this.keyword;
                let field = "positionName";
                axios.get('http://localhost:8080/search/' + field + '/' + keyword + '/1/5').then(response =>{
                    this.results = response.data;
                    //如果返回结果不够5条，根据职位优势推荐查询数据补全5条
                    if (this.results.length < 5){
                        field = "positionAdvantage";
                        let fieldValue = "美女多,员工福利好";
                        let num = 5 - this.results.length;
                        axios.get('http://localhost:8080/search/' + field +'/'+ fieldValue + '/1/' + num).then(response =>{
                            response.data.forEach(item => {
                                this.results.push(item);
                            })
                        });
                    }
                });
            }
        }
    });
</script>

</html>
```

#### 7.效果图

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219222642525.png" alt="image-20201219222642525" />

### 数据库脚本

lagou_position数据库

```sql
/*
SQLyog Ultimate v12.09 (64 bit)
MySQL - 5.7.31 : Database - lagou_position
*********************************************************************
*/


/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`lagou_position` /*!40100 DEFAULT CHARACTER SET latin1 */;

USE `lagou_position`;

/*Table structure for table `position` */

DROP TABLE IF EXISTS `position`;

CREATE TABLE `position` (
  `companyName` varchar(300) DEFAULT NULL,
  `id` double DEFAULT NULL,
  `positionAdvantage` varchar(300) DEFAULT NULL,
  `companyId` double DEFAULT NULL,
  `positionName` varchar(240) DEFAULT NULL,
  `salary` varchar(120) DEFAULT NULL,
  `salaryMin` double DEFAULT NULL,
  `salaryMax` double DEFAULT NULL,
  `salaryMonth` double DEFAULT NULL,
  `education` varchar(60) DEFAULT NULL,
  `workYear` varchar(60) DEFAULT NULL,
  `jobNature` varchar(120) DEFAULT NULL,
  `chargeField` blob,
  `createTime` datetime DEFAULT NULL,
  `email` varchar(300) DEFAULT NULL,
  `publishTime` varchar(150) DEFAULT NULL,
  `isEnable` double DEFAULT NULL,
  `isIndex` double DEFAULT NULL,
  `city` varchar(150) DEFAULT NULL,
  `orderby` double DEFAULT NULL,
  `isAdvice` double DEFAULT NULL,
  `showorder` double DEFAULT NULL,
  `publishUserId` double DEFAULT NULL,
  `workAddress` varchar(300) DEFAULT NULL,
  `generateTime` datetime DEFAULT NULL,
  `bornTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `isReward` double DEFAULT NULL,
  `rewardMoney` varchar(60) DEFAULT NULL,
  `isExpired` double DEFAULT NULL,
  `positionDetailPV` double DEFAULT NULL,
  `offlineTime` datetime DEFAULT NULL,
  `positionDetailPV_cnbeta` double DEFAULT NULL,
  `adviceTime` datetime DEFAULT NULL,
  `comeFrom` varchar(150) DEFAULT NULL,
  `receivedResumeCount` double DEFAULT NULL,
  `refuseResumeCount` double DEFAULT NULL,
  `markCanInterviewCount` double DEFAULT NULL,
  `haveNoticeInterCount` double DEFAULT NULL,
  `isForbidden` double DEFAULT NULL,
  `reason` varchar(768) DEFAULT NULL,
  `verifyTime` datetime DEFAULT NULL,
  `adWord` double DEFAULT NULL,
  `adRankAndTime` varchar(120) DEFAULT NULL,
  `adTimes` double DEFAULT NULL,
  `adStartTime` datetime DEFAULT NULL,
  `adEndTime` datetime DEFAULT NULL,
  `adBeforeDetailPV` double DEFAULT NULL,
  `adAfterDetailPV` double DEFAULT NULL,
  `adBeforeReceivedCount` double DEFAULT NULL,
  `adAfterReceivedCount` double DEFAULT NULL,
  `adjustScore` double DEFAULT NULL,
  `weightStartTime` datetime DEFAULT NULL,
  `weightEndTime` datetime DEFAULT NULL,
  `isForward` bit(1) DEFAULT NULL,
  `forwardEmail` varchar(300) DEFAULT NULL,
  `isSchoolJob` bit(1) DEFAULT NULL,
  `type` tinyint(4) DEFAULT NULL,
  `prolong_offline_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `position` */

insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('百度',1,'lianhfisnhdh',141823,'java','15k-25k',15,25,0,'本科','3-5年','全职',NULL,'2018-12-06 17:17:24','',NULL,1,0,'青岛',NULL,0,0,100013344,NULL,'2018-11-22 14:58:49','2020-09-06 11:52:45',0,NULL,1,0,'2018-12-06 17:21:26',0,NULL,'andorid',0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,'2020-07-27 14:55:59','2020-09-30 00:00:58','\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('阿里',3,'Web测试工程师1',141956,'Web测试工程师1','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2017-08-24 07:14:11','anan1@lagou.com','0',0,0,'青岛',65,0,1373869361621,100013340,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,828,'2018-05-02 16:00:05',0,NULL,'andorid',0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('腾讯',4,'美女多222',147,'Web测试工程师','10k-12k',10,12,0,'本科','4','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','2',0,0,'北京',65,0,1373870620501,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,752,'2017-10-28 00:00:05',0,NULL,'lagouOpen',0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'','alan@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('360',5,'手机测试工程师111',147,'手机测试工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','2',0,0,'北京',65,0,1373875824893,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,788,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('小米',6,'前端工程师（页面制作）',147,'前端工程师（页面制作）','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','6',0,0,'北京',65,0,1373876343375,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,723,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('京东',7,'PHP软件工程师',147,'PHP软件工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','0',0,0,'北京',65,0,1373877071767,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,634,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('亚马逊',8,'PHP软件工程师',147,'PHP软件工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','2',0,0,'北京',65,0,1373877366356,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,774,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('口袋购物',9,'IPhone手机开发工程师',147,'IPhone手机开发工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','0',0,0,'北京',65,0,1373877881539,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,650,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('当当网',10,'产品经理',147,'产品经理','2k-20k',2,20,0,'本科','3-5年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','6',0,0,'北京',65,0,1373878138496,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,299,'2017-01-16 09:41:34',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('聚美优品',11,'扁平管理',147,'C++高级软件工程师（界面开发）','2k-20k',2,20,0,'本科','不限','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','0',0,0,'北京',100,0,1373878590050,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,34,'2017-01-16 09:41:34',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('美团',12,'产品运营',147,'产品运营','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2019-01-29 11:18:13','fengshao@lagou.com','7',0,0,'北京',65,0,1373878652026,100011815,'北京','2015-08-18 12:05:24','2019-01-30 12:02:33',0,NULL,1,327,'2017-01-16 09:41:34',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('拉手网',13,'制度和环境比较人性化',5,'Flash开发工程师','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:24','fengshao@lagou.com','0',0,0,'上海',100,0,1373878742151,2,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,40,'2018-10-20 00:00:19',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('明星衣橱',14,'扁平管理',5,'PHP开发工程师','2k-20k',2,20,0,'大专','3-5年','全职',NULL,'2020-06-15 18:06:24','fengshao@lagou.com','1',0,0,'上海',100,0,1373878926196,2,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,26,'2018-10-20 00:00:19',1,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('美丽说',15,'帅哥美女多',4,'社区运营经理','2k-20k',2,20,0,'大专','1-3年','全职',NULL,'2020-06-15 18:06:08','6666@qq.com','3',0,0,'上海',65,0,1373878937637,1,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,234,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('去哪儿',16,'扁平管理',5,'产品经理','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2020-06-15 18:06:24','fengshao@lagou.com','18',0,0,'上海',100,0,1373879131552,2,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,63,'2018-10-20 00:00:20',2,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('艺龙',17,'员工福利好',4,'UI交互设计师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2020-06-15 18:06:09','6666@qq.com','8',0,0,'上海',65,0,1373879215143,1,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,243,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('豆瓣',18,'员工福利好',4,'视觉设计师','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:09','6666@qq.com','8',0,0,'上海',65,0,1373879318142,1,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,243,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('知乎',19,'员工福利好',5,'JAVA开发工程师','2k-20k',2,20,0,'大专','1-3年','全职',NULL,'2020-06-15 18:06:24','fengshao@lagou.com','12',0,0,'上海',100,0,1373879768887,2,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,6,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('Nice',20,'员工福利好',4,'数据编辑','2k-20k',2,20,0,'大专','不限','全职',NULL,'2020-06-15 18:06:09','6666@qq.com','3',0,0,'上海',65,0,1373879543712,1,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,214,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('脉脉',21,'帅哥多',5,'测试工程师（移动端）','2k-20k',2,20,0,'大专','1-3年','全职',NULL,'2020-06-15 18:06:25','fengshao@lagou.com','1',0,0,'上海',100,0,1373879757316,2,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,5,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('果壳',22,'员工福利好',4,'网站运营专员','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2020-06-15 18:06:10','6666@qq.com','4',0,0,'上海',65,0,1373879661503,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,315,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('陌陌',23,'美女多、员工福利好',5,'IOS 开发工程师','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:25','fengshao@lagou.com','2',0,0,'上海',100,0,1373879747360,2,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,3,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('锤子手机',24,'节日礼物',5,'Android开发工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2020-06-15 18:06:25','fengshao@lagou.com','3',0,0,'上海',100,0,1373879738700,2,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,8,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('凤凰网',25,'员工福利好',4,'BD','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:10','6666@qq.com','0',0,0,'上海',65,0,1373879769461,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,360,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('豌豆荚',26,'美女多、员工福利好',5,'PHP开发工程师（游戏网站）','2k-20k',2,20,0,'大专','1-3年','全职',NULL,'2020-06-15 18:06:25','fengshao@lagou.com','1',0,0,'上海',100,0,1373880113172,2,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,6,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('友盟',27,'员工福利好',6,'产品经理','2k-20k',2,20,0,'不限','不限','全职',NULL,'2020-06-15 18:06:11','6666@qq.com','6',0,0,'上海',55,0,1373881057811,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,210,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('新浪',28,'员工福利好',8,'编辑助理','2k-20k',2,20,0,'不限','不限','全职',NULL,'2020-06-15 18:06:11','6666@qq.com','3',0,0,'上海',15,0,1373882286658,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,360,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('网易',29,'员工福利好',8,'美工、网页设计','2k-20k',2,20,0,'不限','不限','全职',NULL,'2020-06-15 18:06:11','6666@qq.com','1',0,0,'上海',15,0,1373882357697,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,329,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('今日头条',30,'职位年薪：15-30万',7,'js前端高级工程师','12k-25k',12,25,0,'本科','3-5年','全职',NULL,'2016-07-22 10:24:18','test@testtest01.com','1',1,0,'北京',50,0,1373882639044,183,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,379,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('极客公园',31,'薪资构成：基本薪资 + 奖金/提成',7,'UI设计师','2k-20k',2,20,0,'不限','不限','全职',NULL,'2016-07-22 10:24:18','test@testtest01.com','25',1,0,'北京',50,0,1373883039599,183,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,311,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('微软',32,'通信交通：有补助',7,'android程序员','12k-15k',12,15,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','test@testtest01.com','3',1,0,'北京',50,0,1373883142193,183,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,270,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('36氪',33,'居住福利：住房补贴',7,'IOS程序员','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2016-07-22 10:24:18','test@testtest01.com','13',1,0,'北京',50,0,1373883247413,183,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,301,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,'2019-04-11 00:00:00','2019-04-18 23:59:59','\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('搜房网',34,'国际企业',9,'技术开发工程师','7k-10k',7,10,0,'不限','不限','全职',NULL,'2020-06-15 18:06:12','6666@qq.com','1',0,0,'北京',40,0,1373884037734,1,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,337,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('央视网',35,'员工福利好',9,'Flash 资深设计师','10k-15k',10,15,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','0',0,0,'北京',40,0,1373884229000,1,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,371,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('汽车之家',36,'员工福利好',9,'数码分析师','7k-10k',7,10,0,'本科','不限','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','3',0,0,'北京',40,0,1373884393050,1,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,290,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('印象笔记',37,'员工福利好',9,'数据顾问','7k-10k',7,10,0,'本科','不限','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','7',0,0,'北京',40,0,1373884954342,1,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,379,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('金山',38,'如果你只想跟聪明的人一起工作，就加入我们',10,'软件工程师','7k-7k',7,7,0,'本科','不限','全职',NULL,'2016-07-22 10:24:18','anming@hsgh.com','1',1,0,'北京',65,0,1373885055935,193,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,292,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('滴滴',39,'优秀者，可解决北京市户口',11,'手机测试工程师','2k-5k',2,5,0,'大专','1-3年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','9',1,0,'北京',40,0,1373886219984,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,348,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('猎豹移',40,'优秀者，可解决北京市户口',11,'美术UI','5k-10k',5,10,0,'本科','3-5年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','1',1,0,'北京',40,0,1373886377328,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,345,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('联想',41,'员工福利好',12,'系统工程师','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','1',0,0,'上海',15,0,1373890215550,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,315,'2018-10-20 00:00:21',2,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,42,'员工福利好',12,'前端工程师','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','3',0,0,'上海',15,0,1373890347725,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,545,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('APUS',43,'员工福利好',12,'Android开发工程师','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','6',0,0,'上海',15,0,1373890444399,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,296,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,44,'员工福利好',12,'IOS开发工程师','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:14','6666@qq.com','7',0,0,'上海',15,0,1373890590732,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,327,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,45,'优秀者，可解决北京市户口',11,'ios海外推广','5k-10k',5,10,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','8',1,0,'北京',40,0,1373890665844,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,332,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,46,'优秀者，可解决北京市户口',11,'美术特效','5k-10k',5,10,0,'不限','3-5年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','1',1,0,'北京',40,0,1373890778792,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,326,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,47,'员工福利好',12,'数据库工程师','2k-20k',2,20,0,'不限','不限','全职',NULL,'2020-06-15 18:06:14','6666@qq.com','0',0,0,'上海',15,0,1373890917741,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,190,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,48,'优秀者，可解决北京市户口',11,'测试实习','2k-5k',2,5,0,'本科','应届毕业生','实习',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','13',1,0,'北京',40,0,1373890841711,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,327,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,49,'员工福利好',12,'后端工程师','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:14','6666@qq.com','0',0,0,'上海',15,0,1373890902610,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,162,'2018-10-20 00:00:22',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,50,'优秀者，可解决北京市户口',11,'.NET开发工程师','5k-10k',5,10,0,'不限','1-3年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','5',1,0,'北京',40,0,1373890960056,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,329,'2018-10-20 00:00:22',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,51,'优秀者，可解决北京市户口',11,'美术实习','2k-5k',2,5,0,'硕士','1年以下','实习',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','1',1,0,'北京',40,0,1373891110617,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,267,'2018-10-20 00:00:22',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
commit;
/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```



