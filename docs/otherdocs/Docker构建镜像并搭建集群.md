### 习题

作业：使用DockerFile构建镜像并搭建 swarm+compose集群。

![image-20201220203220991](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220203220991.png)

如图：

（1）Hot是应用程序(springboot)，打成jar包：Hot.jar 

（2）利用dockerfile将Hot.jar构建成镜像lgedu/hot:1.0 

（3）构建Swarm 集群

（4）在 Swarm 集群中使用 compose 文件 （docker-compose.yml） 来配置、启动多个服务 包括: Mysql、Redis以及应用程序Hot。

### 实现文档

### 1.将本地应用构建成镜像

将springboot应用(spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar)打包后上传到虚拟机上：

```shell
[root@linux101 dockerfile]# pwd
/opt/dockerfile
[root@linux101 dockerfile]# ls
dockerfile-demo  spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar  tb_article.sql
```

在dockerfile目录下编写dockerfile-demo文件：

```shell
#指定以openjdk:8-jre为基础镜像，来构建此镜像，可以理解为运行的需要基础环境
FROM java:8-alpine
#将当前spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar复制到容器根目录下
ADD spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar app.jar
#暴露容器端口为8080，Docker镜像告知Docker宿主机应用监听了8080端口
EXPOSE 8080
#配置容器启动后执行的命令
ENTRYPOINT ["java","-jar","/app.jar"]
```

 使用dockerfile-demo构建应用镜像：

```shell
# 构建
docker build -f dockerfile-demo -t fish/blog:v1 .
```

运行各个容器：

```shell
[root@linux101 dockerfile]# docker images
REPOSITORY   TAG                   IMAGE ID       CREATED        SIZE
fish/blog    v1                    e0a823b870c2   2 hours ago    187MB
redis        5.0.9                 987b553c835f   7 weeks ago    98.3MB
mysql        5.7.30                9cfcce23593a   6 months ago   448MB
java         8-alpine              3fd9dd82815c   3 years ago    145MB

# 运行Mysql，宿主机中：mkdir /docker/mysql/db
docker run --network host -e MYSQL_ROOT_PASSWORD=root -d -v /docker/mysql/db:/var/lib/mysql mysql:5.7.30
# 运行Redis，宿主机中：mkdir /docker/redis/data
docker run -p 6379:6379 -d -v /docker/redis/data:/data redis:5.0.9 redis-server --appendonly yes
# 运行本地应用
docker run --rm -d -p 8080:8080 fish/blog:v1
```

### 2.docker中mysql执行脚本

#### 2.1 mysql执行sql脚本

1.从本地数据库中导出tb_article.sql脚本，并上传到虚拟机中

```shell
[root@linux101 dockerfile]# pwd
/opt/dockerfile
[root@linux101 dockerfile]# ls
dockerfile-demo  spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar  tb_article.sql
```

2.启动docker，运行mysql容器

```shell
# 启动docker
systemctl start docker
# 运行mysql容器，持久化数据
docker run --network host -e MYSQL_ROOT_PASSWORD=root -d -v /docker/mysql/db:/var/lib/mysql mysql:5.7.30
```

3.将sql文件复制到mysql容器中的/home/目录下

```shell
# 查询mysql容器ID
[root@linux101 dockerfile]#  docker ps
3ab04abf33d2   mysql:5.7.30   "docker-entrypoint.s…"   16 minutes ago   Up 16 minutes 
# 复制文件到mysql容器中
[root@linux101 dockerfile]# docker cp tb_article.sql 3ab04abf33d2:/home/article.sql
```

4.进入mysql容器执行脚本

```shell
# 进入mysql容器
[root@linux101 ~]# docker exec -it 3ab04abf33d2 /bin/bash
root@linux101:/# cd home/
root@linux101:/home# ls
article.sql
root@linux101:/home# mysql -uroot -p
Enter password: 
mysql> create database blog;
Query OK, 1 row affected (0.00 sec)
mysql> use blog;
Database changed
mysql> source article.sql
```

#### 2.2 mysql中编码问题

```shell
mysql> show variables like "%char%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | latin1                     |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
mysql>  SHOW VARIABLES LIKE 'collation_%';
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | latin1_swedish_ci |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
```

解决方法：

```shell
root@linux101:/home# apt-get update
root@linux101:/home# apt-get install vim

root@linux101:/home# vim /etc/mysql/conf.d/mysql.cnf
----------------------------------------------------------
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
----------------------------------------------------------
mysql> SHOW VARIABLES LIKE 'character_set_%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
mysql> SHOW VARIABLES LIKE 'collation_%';
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | utf8_general_ci   |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
```

### 3.Swarm集群

对于Docker 1.12+版本，Swarm相关命令已经原生嵌入到了Docker Engine中。

#### 3.1 获取镜像并init集群

1.下载镜像

```shell
# 拉取镜像
docker pull swarm
# 查看版本
docker run --rm swarm -v

swarm version 1.2.9 (527a849)
```

2.Swarm集群

| IP             | 角色    |
| -------------- | ------- |
| 192.168.91.101 | Manager |
| 192.168.91.109 | Worker  |

3.创建集群

```shell
docker swarm init --advertise-addr 192.168.91.101
```

![image-20201221210047665](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221210047665.png)

#### 3.2 添加工作节点到集群

在工作节点192.168.91.109上安装docker和docker swarm。

```shell
# 安装yum源
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 安装docker
yum install -y docker-ce docker-ce-cli containerd.io
# 启动docker
systemctl start docker
# 查看信息
docker info

# 获取swarm镜像
docker pull swarm
# 查看版本
docker run --rm swarm -v
```

添加工作节点192.168.91.109到集群

```shell
docker swarm join --token SWMTKN-1-1m6ghlktyrtxgg29ic2jxxvr8bb32glk6if80ijek46tpf8l33-58rr8yn3eq7zk8tf9cabxl517 192.168.91.101:2377
```

![image-20201221205830215](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221205830215.png)

注：如果忘记token，可以管理节点192.168.91.101查看。

```shell
[root@linux101 ~]# docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1m6ghlktyrtxgg29ic2jxxvr8bb32glk6if80ijek46tpf8l33-5ou427c51h7vd5h9nrdz1ut8r 192.168.91.101:2377

[root@linux101 ~]# docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1m6ghlktyrtxgg29ic2jxxvr8bb32glk6if80ijek46tpf8l33-58rr8yn3eq7zk8tf9cabxl517 192.168.91.101:2377
```

添加工作节点192.168.91.109之后，在管理节点192.168.91.101上查看节点信息：

```shell
docker node ls
```

![image-20201221205736945](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221205736945.png)

### 4.Docker Compose安装

1.运行以下命令以下载Docker Compose的当前稳定版本

```shell
curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2.将可执行权限应用于二进制文件

```shell
chmod +x /usr/local/bin/docker-compose
```

3.添加到环境中

```shell
#ln -s ： 软链接
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

4.测试安装

```shell
docker-compose --version
```

![image-20201221213543117](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221213543117.png)

5.卸载Docker Compose

```shell
rm /usr/local/bin/docker-compose
```

### 5.编写docker-compose.yml

编写docker-compose.yml文件：

```shell
[root@linux101 dockerfile]# pwd
/opt/dockerfile
[root@linux101 dockerfile]# ls
dockerfile-demo  spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar  tb_article.sql
[root@linux101 dockerfile]# vim docker-compose.yml
```

文件内容如下：

```yml
# 使用docker swarm时版本必须是3.0
version: '3.0'
services:

  mysql:
    image: mysql:5.7.30 
    privileged: true
    #主备都存在
    volumes:
      - /docker/mysql/db:/var/lib/mysql
      #- ./mysql.cnf:/etc/mysql/conf.d/mysql.cnf
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
    ports:
      - "3306:3306"
    deploy:
      mode: replicated
      replicas: 2
    
  redis:
    image: redis:5.0.9
    privileged: true
    #主备都存在
    volumes:
      - /docker/redis/data:/data
    ports:
      - "6379:6379"
    deploy:
      mode: replicated
      replicas: 2
      #command: redis-server /usr/local/etc/redis/redis.conf
   
  blog:
    image: fish/blog:v1
    privileged: true
    #主备都存在
    volumes:
      - /etc/localtime:/etc/localtime
    ports:
      - "8080:8080"
    deploy:
      mode: replicated
      replicas: 2
```

拷贝文件

```shell
version: '3.0'
services:
  mysql:
    image: mysql:5.7.30 
    privileged: true
    volumes:
      - /docker/mysql/db:/var/lib/mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=root"
    ports:
      - "3306:3306"
    deploy:
      mode: replicated
      replicas: 2
  redis:
    image: redis:5.0.9
    privileged: true
    volumes:
      - /docker/redis/data:/data
    ports:
      - "6379:6379"
    deploy:
      mode: replicated
      replicas: 2
  blog:
    image: fish/blog:v1
    privileged: true
    volumes:
      - /etc/localtime:/etc/localtime
    ports:
      - "8080:8080"
    deploy:
      mode: replicated
      replicas: 2
```

启动服务：

```shell
docker stack deploy -c docker-compose.yml web
```

![image-20201221215600115](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221215600115.png)

查看服务：

```shell
# 查看服务
docker stack services web
```

![image-20201221215732658](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221215732658.png)

删除服务：

```shell
# 删除服务
docker stack down web
```

### 5.应用配置文件

```properties
# mysql数据库配置,需要指定时区serverTimezone
spring.datasource.url=jdbc:mysql://192.168.91.101:3306/blog?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root

# thymeleaf页面缓存设置false,开发中方便调试
spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates/client/
spring.thymeleaf.suffix=.html

# 开启驼峰命名法,数据库字段与属性自动映射
mybatis.configuration.map-underscore-to-camel-case=true

# mybatis-plus配置
mybatis-plus.mapper-locations=classpath:mapper/*Mapper.xml
# 实体扫描，多个package用逗号或者分号分隔,自己的实体类地址
mybatis-plus.type-aliases-package:com.fishleap.entity
# 这个配置会将执行的sql打印出来，在开发或测试的时候可以用
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl

# 配置国际化文件基础名
spring.messages.basename=i18n.login

# redis
spring.redis.host=192.168.91.101
spring.redis.port=6379
spring.redis.database=14
```

### 6.演示效果

window本地启动：

![image-20201221223003114](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221223003114.png)

Manager(192.168.91.101)节点访问：

![image-20201221223144909](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221223144909.png)

Worker(192.168.91.109)节点访问：

![image-20201221223221350](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201221223221350.png)

























