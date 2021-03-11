#### 需求

![image-20201023003112294](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201023003112294.png)

- 搭建Redis5.0集群，要求三主三从
- 能够添加一主一从（Master4和Slaver4）
- 能够通过JedisCluster向RedisCluster添加数据和取出数据

#### 环境搭建

##### 1.准备工作

```shell
#环境
#一台centos7最小安装，Ip:192.168.91.112

#安装redis编译的c环境
yum -y install wget gcc vim
#创建相应的文件夹
mkdir -p /opt/servers/redis-cluster/6379
mkdir -p /opt/applets
#切换到applets目录下
cd applets
```

##### 2.在线下载、解压、编译并安装到指定目录

```shell
#下载Redis5.0
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
#解压redis
tar -zxvf redis-5.0.5.tar.gz -C /opt/servers
#切换到src目录
cd /opt/servers/redis-5.0.5/src
#编译并安装到指定目录
make MALLOC=libc install PREFIX=/opt/servers/redis-cluster/6379
```

##### 3.复制并修改配置文件

```shell
#复制配置文件
cp /opt/servers/redis-5.0.5/redis.conf /opt/servers/redis-cluster/6379/bin/

#修改配置文件：cd /opt/servers/redis-cluster/6379/bin/ 并 vim redis.conf

# bind 127.0.0.1  #用网络ip连接
protected-mode no  #关闭保护模式
port 6379  #指定端口号
daemonize yes  #开启后台启动
cluster-enabled yes  #开启集群
```

##### 4.复制6379到6380、6381、6382、6383、6384、6385、6386并修改对应配置文件的端口

> cp -R 6379/ 6380
>
> vim 6380/bin/redis.conf

##### 5.编写启动脚本：start.sh

```shell
cd 6379/bin
./redis-server redis.conf
cd ..
cd ..
cd 6380/bin
./redis-server redis.conf
cd ..
cd ..
cd 6381/bin
./redis-server redis.conf
cd ..
cd ..
cd 6382/bin
./redis-server redis.conf
cd ..
cd ..
cd 6383/bin
./redis-server redis.conf
cd ..
cd ..
cd 6384/bin
./redis-server redis.conf
cd ..
cd ..
```

添加用户user执行权限：chmod u+x start.sh

启动脚本：./start.sh

##### 6、配置Redis集群

```shell
#切换到 6379/bin目录下：cd 6379/bin
./redis-cli --cluster create 192.168.91.112:6379 192.168.91.112:6380 192.168.91.112:6381 192.168.91.112:6382 192.168.91.112:6383 192.168.91.112:6384 --cluster-replicas 1
```

![image-20201021214755306](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201021214755306.png)

![image-20201021215201638](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201021215201638.png)

查看集群状态：`./redis-cli -p 6379 -c`  进入命令行界面

```shell
[root@atzlg8 bin]# ./redis-cli -p 6379 -c
127.0.0.1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
......
127.0.0.1:6379> cluster nodes
dee105876dc8c839025272f3662e6110b03374c3 192.168.91.112:6379@16379 myself,master - 0 1603288388000 1 connected 0-5460
e3b251cfd25dc2cdc85e5de0d5040011b9d7c4b1 192.168.91.112:6380@16380 master - 0 1603288389974 2 connected 5461-10922
......
```

![image-20201021220012141](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201021220012141.png)

##### 7、添加主节点

```shell
#新开一个窗口用来启动6385节点
cd /opt/servers/redis-cluster/6385/bin
./redis-server redis.conf

#在6385/bin下向集群中添加6385新节点
./redis-cli --cluster add-node 192.168.91.112:6385 192.168.91.112:6379
```

![image-20201021221638261](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201021221638261.png)

使用原来第一个窗口的命令行再次查看cluster nodes变化：

![image-20201021221311593](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201021221311593.png)

##### 8、hash槽(slot)重新分配

`./redis-cli --cluster reshard 192.168.91.112:6385`

```shell
How many slots do you want to move (from 1 to 16384)? 1000 #设置slot数1000  
What is the receiving node ID? 2b0ecf03697738c8582121d29b0bcafe480694cb #新节点node id 
Please enter all the source node IDs.  
  Type 'all' to use all the nodes as source nodes for the hash slots.  
  Type 'done' once you entered all the source nodes IDs.  
Source node #1: all #表示全部节点重新洗牌
#......
Do you want to proceed with the proposed reshard plan (yes/no)? yes #确认重新分 
```

![image-20201021224042408](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201021224042408.png)

##### 9、添加从节点

添加6386从结点，将6386作为6385的从结点：

```shell
#启动6386节点
cd /opt/servers/redis-cluster/6386/bin
./redis-server redis.conf

#添加6386从节点
./redis-cli --cluster add-node 192.168.91.112:6386 192.168.91.112:6385 --cluster-slave --cluster-master-id 2b0ecf03697738c8582121d29b0bcafe480694cb
```

![image-20201021225922151](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201021225922151.png)

再次查看集群节点：

![image-20201021225519765](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201021225519765.png)

##### 10、方便连接关闭防火墙

```shell
#查看防火墙状态
firewall-cmd --state
#停止firewall
systemctl stop firewalld
#禁止firewall开机启动
systemctl disable firewalld
```

#### JedisCluster操作数据

##### 1、引入依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.RELEASE</version>
    <relativePath/>
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>11</java.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <exclusions>
            <exclusion>
                <groupId>io.lettuce</groupId>
                <artifactId>lettuce-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>3.1.0</version>
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

</dependencies>
```

##### 2、配置文件

```yaml
spring:
  redis:
    cluster:
      #设置key的生存时间，当key过期时，它会被自动删除；
      expire-seconds: 120
      #设置命令的执行时间，如果超过这个时间，则报错;
      command-timeout: 5000
      #设置redis集群的节点信息，其中namenode为域名解析，通过解析域名来获取相应的地址;
      nodes: 192.168.91.112:6379,192.168.91.112:6380,192.168.91.112:6381,192.168.91.112:6382,192.168.91.112:6383,192.168.91.112:6384,192.168.91.112:6385,192.168.91.112:6386
```

##### 3、redis配置

RedisProperties类

```java
/**
 * @author zlg
 * 使用ConfigurationProperties注解读取yml文件中的字段值，并使用Component注入到spring容器中
 */
@Component
@ConfigurationProperties(prefix = "spring.redis.cluster")
public class RedisProperties {

    private int expireSeconds;
    private int commandTimeout;
    private String nodes;

    public int getExpireSeconds() {
        return expireSeconds;
    }

    public void setExpireSeconds(int expireSeconds) {
        this.expireSeconds = expireSeconds;
    }

    public int getCommandTimeout() {
        return commandTimeout;
    }

    public void setCommandTimeout(int commandTimeout) {
        this.commandTimeout = commandTimeout;
    }

    public String getNodes() {
        return nodes;
    }

    public void setNodes(String nodes) {
        this.nodes = nodes;
    }

    @Override
    public String toString() {
        return "RedisProperties{" +
                "expireSeconds=" + expireSeconds +
                ", commandTimeout=" + commandTimeout +
                ", nodes='" + nodes + '\'' +
                '}';
    }
}
```

RedisConfig类

```java
/**
 * @author zlg
 * 根据注入的RedisProperties对象来获取JedisCluster对象
 */
@Configuration
public class RedisConfig {

    @Autowired
    private RedisProperties redisProperties;

    @Bean
    public JedisCluster getJedisCluster() {
        //获取redis集群的ip及端口号等相关信息
        String[] nodeArray = redisProperties.getNodes().split(",");
        Set<HostAndPort> nodes = new HashSet<>();
        //遍历nodeArray到HostAndPort中
        for (String node : nodeArray) {
            String[] ipPortArray = node.split(":");
            String host = ipPortArray[0].trim();
            int port = Integer.parseInt(ipPortArray[1].trim());
            HostAndPort hostAndPort = new HostAndPort(host, port);
            nodes.add(hostAndPort);
        }
        //构建对象并返回
        return new JedisCluster(nodes, redisProperties.getCommandTimeout());
    }

}
```

##### 4、jedis操作

JedisClient类

```java
/**
 * @author zlg
 * 操作redis集群的接口
 */
public interface JedisClient {

    String set(String key, String value);

    String get(String key);

    Boolean exists(String key);

    Long expire(String key, int seconds);

    Long ttl(String key);

    Long incr(String key);

    Long hset(String key, String field, String value);

    String hget(String key, String field);

    Long hdel(String key, String... field);
}
```

JedisClientCluster类

```java
/**
 * @author zlg
 * 对JedisClient进行实现，实现类中写了对redis操作的主要的方法
 */
@Component
public class JedisClientCluster implements JedisClient {

    @Autowired
    private JedisCluster jedisCluster;

    @Override
    public String set(String key, String value) {
        return jedisCluster.set(key, value);
    }

    @Override
    public String get(String key) {
        return jedisCluster.get(key);
    }

    @Override
    public Boolean exists(String key) {
        return jedisCluster.exists(key);
    }

    @Override
    public Long expire(String key, int seconds) {
        return jedisCluster.expire(key, seconds);
    }

    @Override
    public Long ttl(String key) {
        return jedisCluster.ttl(key);
    }

    @Override
    public Long incr(String key) {
        return jedisCluster.incr(key);
    }

    @Override
    public Long hset(String key, String field, String value) {
        return jedisCluster.hset(key, field, value);
    }

    @Override
    public String hget(String key, String field) {
        return jedisCluster.hget(key, field);
    }

    @Override
    public Long hdel(String key, String... field) {
        return jedisCluster.hdel(key, field);
    }

}
```

##### 5、controller类

RedisController类

```java
@RestController
@RequestMapping("redis")
public class RedisController {

    @Autowired
    private RedisProperties redisProperties;
    @Autowired
    private RedisConfig redisConfig;
    @Autowired
    private JedisClientCluster jedisClientCluster;

    /**
     * http://localhost:8080/redis/getRedisInfo
     * @return String
     */
    @RequestMapping("/getRedisInfo")
    public String getRedisValue() {
        return redisProperties.toString() + "/n" + redisConfig.getJedisCluster().getClusterNodes();
    }

    /**
     * http://localhost:8080/redis/find?key=name:01
     * 查询redis集群
     * @param key 键
     * @return 值：zlg
     */
    @GetMapping("/find")
    public String findRedis(@RequestParam String key) {
        return jedisClientCluster.get(key);
    }

    /**
     * http://localhost:8080/redis/save?key=name:02&value=jack
     * return：OK
     */
    @GetMapping("/save")
    public String saveRedis(@RequestParam String key, @RequestParam String value){
        return jedisClientCluster.set(key, value);
    }

}
```

##### 6、启动类添加注解

```java
@SpringBootApplication
@EnableCaching
public class ApplicationMain {

    public static void main(String[] args) {
        SpringApplication.run(ApplicationMain.class, args);
    }
}
```





参考链接：

- [redis cluster 添加 删除 重分配 节点](http://blog.51yip.com/nosql/1726.html)



单机安装补充：

```
redis在Linux上的安装
1）安装redis编译的c环境，yum install gcc-c++
2）将redis-2.6.16.tar.gz上传到Linux系统中
3）解压到/usr/local下  tar -xvf redis-2.6.16.tar.gz -C /usr/local
4）进入redis-2.6.16目录 使用make命令编译redis
5）在redis-2.6.16目录中 使用make PREFIX=/usr/local/redis install命令安装redis到/usr/local/redis中
6）拷贝redis-2.6.16中的redis.conf到安装目录redis/bin中cp redis.conf /usr/local/redis/bin
7）启动redis服务，在bin下执行命令./redis-server redis.conf
8) 启动redis客户端./redis-cli
9) 关闭redis服务./redis-cli shutdown
8）如需远程连接redis，需配置redis端口6379在linux防火墙中开发
/sbin/iptables -I INPUT -p tcp --dport 6379 -j ACCEPT
/etc/rc.d/init.d/iptables save
```



