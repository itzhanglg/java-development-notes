

# Lagou-edu微服务部署

## 需求说明

将课程功能进行容器化发布和Skywalking监控。

- 容器化发布：可以使用Dockerfile单个单个的部署，也可以使用Docker compose在单节点上多个一起部署；
- Skywalking监控：需要在每个服务构建镜像的Dockerfile文件中添加【启动jar包时的配置参数】，不需要引入额外依赖。

软件环境说明：

| 软件                   | 版本                 |
| ---------------------- | -------------------- |
| centos                 | 7.9                  |
| adoptopenjdk/openjdk11 | jdk-11.0.8_10-alpine |
| mysql                  | 5.7.31               |
| docker                 | 20.10.3              |
| elasticsearch          | 7.9.0                |
| skywalking             | 8.1.0                |
| redis                  | 5.0.5                |
| rocketmq               | 4.5.1                |
| xshell                 | 6.0                  |
| xftp                   | 6.0                  |

节点功能说明：

| 节点           | 描述                   |
| -------------- | ---------------------- |
| 192.168.91.105 | Skywalking服务部署     |
| 192.168.91.107 | 项目微服务部署、数据库 |
| 192.168.91.100 | Redis、RocketMQ部署    |

## 1.docke安装与部署

1.卸载历史版本

```shell
# 查看安装
yum list installed | grep docker
# 卸载
yum -y remove containerd.io.x86_64
yum -y remove docker-ce.x86_64
yum -y remove docker-ce-cli.x86_64
# 删库
rm -rf /var/lib/docker
```

2.安装官方yum源

```shell
[root@linux101 ~]# yum install -y yum-utils
[root@linux101 ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
```

3.安装Docker引擎

```shell
[root@linux101 ~]# yum install -y docker-ce docker-ce-cli containerd.io

已安装:
  containerd.io.x86_64 0:1.4.3-3.1.el7    docker-ce.x86_64 3:20.10.1-3.el7   
  docker-ce-cli.x86_64 1:20.10.1-3.el7 
```

4.启动docker

```shell
# 开机启动
[root@linux101 ~]# systemctl enable docker
# 启动
[root@linux101 ~]# systemctl start docker
# 查看Docker状态
[root@linux101 ~]# docker info

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.1
 Storage Driver: overlay2
```

5.配置镜像加速

注册一个 [阿里云](https://cr.console.aliyun.com/) 的账号，可以使用淘宝账号登录。获取 [加速器地址](https://cr.console.aliyun.com/cn-shanghai/instances/mirrors) 

![在这里插入图片描述](https://gitee.com/itzlg/mypictures/raw/master/img/20200315152419557.png)

配置本机Docker运行镜像加速器：

```shell
[root@linux101 ~]# vim /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://xxxxxx.mirror.aliyuncs.com"
  ]
}

# 重启Docker后台服务
[root@linux101 ~]# systemctl daemon-reload 
[root@linux101 ~]# systemctl restart docker
[root@linux101 ~]# docker info
Registry Mirrors:
  https://xxxxxx.mirror.aliyuncs.com/
```

## 2.制作项目镜像

### 2.1拉取基础镜像

使用docker拉取openjdk11、mysql5.7基础镜像到本地

```shell
# docker pull openjdk:8-alpine3.9
[root@node107 ~]# docker pull openjdk:11
11: Pulling from library/openjdk
b9a857cbf04d: Pull complete 
d557ee20540b: Pull complete 
3b9ca4f00c2e: Pull complete 
667fd949ed93: Pull complete 
661d3b55f657: Pull complete 
511ef4338a0b: Pull complete 
a56db448fefe: Pull complete 
Digest: sha256:6527dd97857129e5873ed3aa50220d1fb7998571799564c62e12a348dc840ef4
Status: Downloaded newer image for openjdk:11
docker.io/library/openjdk:11
[root@node107 ~]# docker pull adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
jdk-11.0.8_10-alpine: Pulling from adoptopenjdk/openjdk11
188c0c94c7c5: Pull complete 
8f3dd6423c69: Pull complete 
dbc799903e9d: Pull complete 
Digest: sha256:5d1d97b7834fb7d78fcbffe29e99a79147b658dac94c8d5a3b370bd54625a8b7
Status: Downloaded newer image for adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
docker.io/adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine

[root@node107 ~]# docker pull mysql:5.7.31
5.7.31: Pulling from library/mysql
bb79b6b2107f: Pull complete 
49e22f6fb9f7: Pull complete 
842b1255668c: Pull complete 
9f48d1f43000: Pull complete 
c693f0615bce: Pull complete 
8a621b9dbed2: Pull complete 
0807d32aef13: Pull complete 
6d2fc69dfa35: Pull complete 
56153548dd2c: Pull complete 
3bb6ba940303: Pull complete 
3e1888da91a7: Pull complete 
Digest: sha256:b3dc8d10307ab7b9ca1a7981b1601a67e176408be618fc4216d137be37dae10b
Status: Downloaded newer image for mysql:5.7.31
docker.io/library/mysql:5.7.31

[root@node107 ~]# docker images
REPOSITORY               TAG                    IMAGE ID       CREATED        SIZE
openjdk                  11                     1eec9f9fe101   4 weeks ago    628MB
adoptopenjdk/openjdk11   jdk-11.0.8_10-alpine   0590fa97a55b   3 months ago   342MB
mysql                    5.7.31                 42cdba9f1b08   3 months ago   448MB
[root@node107 ~]# 
```

### 2.2创建项目结构

创建项目结构：

创建sh脚本initlagouedu-bom.sh ：

```sh
#!/bin/bash
echo '创建根项目edu-bom目录'
cd /data
mkdir -p edu-bom
cd edu-bom
echo '创建每个子项目目录'
mkdir -p mysql edu-eureka-boot edu-config-boot edu-gateway-boot edu-boss-boot edu-ad-boot edu-course-boot edu-message-boot
```

修改脚本权限和执行：

```sh
chmod 777 initlagouedu-bom.sh

./initlagouedu-bom.sh
```

项目结构创建成功后，可以将各个微服务应用进行打包上传到对应的目录下，在各自的服务 `pom.xml` 文件中添加如下构建，然后再进行打包。

```xml
<build>
    <finalName>${project.name}</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal><!--可以把依赖的包都打包到生成的Jar包中-->
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-clean-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### 2.3MySQL数据库

初始化mysql容器时自动导入项目所需要的数据库

编写Dockerfile文件

```dockerfile
# 基础镜像
FROM mysql:5.7.31
# 编写作者信息和日期亚洲上海
MAINTAINER mysql from date UTC By Asia/Shanghai "zlg@lagou.com"
# 修改时区东八区
ENV TZ Asia/Shanghai
# 将脚本复制到指定目录下,mysql容器初始化后自动导入数据库
COPY edu_ad.sql /docker-entrypoint-initdb.d
COPY edu_authority.sql /docker-entrypoint-initdb.d
COPY edu_comment.sql /docker-entrypoint-initdb.d
COPY edu_course.sql /docker-entrypoint-initdb.d
COPY edu_message.sql /docker-entrypoint-initdb.d
COPY edu_oauth.sql /docker-entrypoint-initdb.d
COPY edu_order.sql /docker-entrypoint-initdb.d
COPY edu_pay.sql /docker-entrypoint-initdb.d
COPY edu_user.sql /docker-entrypoint-initdb.d
```

将Dockerfile上传到下图的文件下：

![image-20210210000744804](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210000744804.png)

mysql目录下制作镜像：

```sh
docker build --rm -t lagou/mysql:5.7 .
docker images

[root@node107 mysql]# docker images
REPOSITORY               TAG                    IMAGE ID       CREATED         SIZE
lagou/mysql              5.7                    e72ea74ce727   4 minutes ago   449MB
openjdk                  11                     1eec9f9fe101   4 weeks ago     628MB
adoptopenjdk/openjdk11   jdk-11.0.8_10-alpine   0590fa97a55b   3 months ago    342MB
mysql                    5.7.31                 42cdba9f1b08   3 months ago    448MB

# 我们要使用mysql目录进行数据卷挂载，需要删除mysql目录中所有文件。
rm -rf *
```

运行镜像：

```sh
docker run -itd --name mysql --restart always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v /data/edu-bom/mysql:/var/lib/mysql lagou/mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

[root@node107 mysql]# docker run -itd --name mysql --restart always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v /data/edu-bom/mysql:/var/lib/mysql lagou/mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
7b25e5a807270a790d897343a635eae91db71944467ce5fee9ced89a24e99187

# 查看日志有错误信息
docker logs -f mysql
[root@node107 mysql]# docker logs -f mysql
2021-02-10 00:31:18+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_ad.sql
2021-02-10 00:31:18+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_authority.sql
2021-02-10 00:31:19+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_comment.sql
2021-02-10 00:31:19+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_course.sql
2021-02-10 00:31:19+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_message.sql
2021-02-10 00:31:19+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_oauth.sql
2021-02-10 00:31:19+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_order.sql
2021-02-10 00:31:19+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_pay.sql
2021-02-10 00:31:19+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/edu_user.sql
```

navicat客户端测试：

```sh
192.168.91.107
username:root
password:root
```

### 2.4部署edu-eureka-boot项目

打包edu-eureka-boot项目

```sh
mvn clean package
```

编写Dockerfile文件

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-eureka-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai
COPY edu-eureka-boot.jar app.jar
EXPOSE 8761
ENTRYPOINT ["java","-jar","/app.jar"]
```

上传Dockerfile文件和eureka服务jar包到下图目录下

![image-20210210005102469](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210005102469.png)

制作镜像

```sh
cd /data/edu-bom/edu-eureka-boot
# 构建镜像
docker build --rm -t lagou/edu-eureka-boot:1.0 .

[root@node107 edu-eureka-boot]# docker images
REPOSITORY               TAG                    IMAGE ID       CREATED          SIZE
lagou/edu-eureka-boot    1.0                    fff3cd21a1a4   43 seconds ago   395MB
lagou/mysql              5.7                    86c9f6530326   21 minutes ago   449MB
adoptopenjdk/openjdk11   jdk-11.0.8_10-alpine   0590fa97a55b   3 months ago     342MB
mysql                    5.7.31                 42cdba9f1b08   3 months ago     448MB
```

运行镜像

```sh
# 运行镜像
docker run -itd --name lagoueureka -p 8761:8761 lagou/edu-eureka-boot:1.0

# 查看启动日志
docker logs -f lagoueureka

# 可以将镜像镜像打包到本地
docker save lagou/edu-eureka-boot:1.0 -o lagou.eureka.tar
docker save lagou/mysql:5.7 -o lagou.mysql.5.7.tar
```

测试项目

```html
http://192.168.91.107:8761/
```

![image-20210210010346132](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210010346132.png)

### 2.5部署edu-config-boot项目

打包edu-config-boot项目

```sh
mvn clean package
```

编写Dockerfile文件

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-config-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai
COPY edu-config-boot.jar app.jar
EXPOSE 8090
ENTRYPOINT ["java","-jar","/app.jar"]
```

上传文件和jar包到下图目录下

![image-20210210011852046](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210011852046.png)

制作镜像

```sh
cd /data/edu-bom/edu-config-boot
# 制作镜像
docker build --rm -t lagou/edu-config-boot:1.0 .
```

运行镜像

```sh
docker run -itd --name lagouconfig -p 8090:8090 lagou/edu-config-boot:1.0

docker logs -f lagouconfig
```

### 2.6部署edu-gateway-boot项目

打包edu-gateway-boot项目

```sh
mvn clean package
```

编写Dockerfile文件

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-gateway-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai
COPY edu-gateway-boot.jar app.jar
EXPOSE 9001
ENTRYPOINT ["java","-jar","/app.jar"]
```

上传文件和jar包到下图目录下

![image-20210210012304634](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210012304634.png)

制作镜像

```sh
cd /data/edu-bom/edu-gateway-boot

docker build --rm -t lagou/edu-gateway-boot:1.0 .
```

运行镜像

```sh
docker run -itd --name lagougateway -p 9001:9001 lagou/edu-gateway-boot:1.0

docker logs -f lagougateway
```

测试项目

```html
http://192.168.91.107:8761/
```

![image-20210210012613216](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210012613216.png)

到此查看构建的镜像

![image-20210210013047096](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210013047096.png)

容器中存在的镜像

![image-20210210013257098](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210013257098.png)

### 2.7部署edu-ad-boot-impl项目

打包edu-ad-boot-impl项目

```sh
# 安装edu-ad-boot项目
mvn clean install
# 打包edu-ad-boot-api项目
mvn clean install
# 打包edu-ad-boot-impl项目
mvn clean package
```

编写Dockerfile文件

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-ad-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai
COPY edu-ad-boot-impl.jar app.jar
EXPOSE 8001
ENTRYPOINT ["java","-jar","/app.jar"]
```

上传文件和jar包到下图目录下

![image-20210210205036089](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210205036089.png)

制作镜像

```sh
cd /data/edu-bom/edu-ad-boot

# 构建镜像
docker build --rm -t lagou/edu-ad-boot:1.0 .
```

运行镜像

```sh
docker run -itd --name lagouad -p 8001:8001 lagou/edu-ad-boot:1.0

# 查看日志
docker logs -f lagouad
```

测试项目

```html
http://192.168.91.107:8001/ad/getAllAds
```

![image-20210210204926516](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210204926516.png)

### 2.8部署edu-boss-boot项目

打包edu-boss-boot项目

```sh
mvn clean package
```

编写Dockerfile文件

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-boss-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai
COPY edu-boss-boot.jar app.jar
EXPOSE 8082
ENTRYPOINT ["java","-jar","/app.jar"]
```

上传文件和jar包到下图目录下

![image-20210210210502715](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210210502715.png)

制作镜像

```sh
cd /data/edu-bom/edu-boss-boot

# 构建镜像
docker build --rm -t lagou/edu-boss-boot:1.0 .
```

运行镜像

```sh
docker run -itd --name lagouboss -p 8082:8082 lagou/edu-boss-boot:1.0

# 查看日志信息
docker logs -f lagouboss
```

测试项目：使用gateway访问项目

```html
http://192.168.91.107:9001/boss/ad/space/getAllSpaces
```

![image-20210210210256991](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210210256991.png)

### 2.9所有镜像

```sh
[root@node107 edu-bom]# docker images
REPOSITORY               TAG                    IMAGE ID       CREATED        SIZE
lagou/edu-boss-boot      1.0                    fb701ee57cf4   3 hours ago    422MB
lagou/edu-ad-boot        1.0                    0850f60f2361   4 hours ago    431MB
lagou/edu-gateway-boot   1.0                    88a773493f5c   23 hours ago   389MB
lagou/edu-config-boot    1.0                    565f4deb6b60   23 hours ago   379MB
lagou/edu-eureka-boot    1.0                    fff3cd21a1a4   23 hours ago   395MB
lagou/mysql              5.7                    86c9f6530326   24 hours ago   449MB
adoptopenjdk/openjdk11   jdk-11.0.8_10-alpine   0590fa97a55b   3 months ago   342MB
mysql                    5.7.31                 42cdba9f1b08   4 months ago   448MB
```

## 3.保存项目镜像

### 3.1保存/导入项目镜像

保存镜像

```sh
docker save \
	lagou/edu-front-boot:1.0 \
	lagou/edu-boss-boot:1.0 \
	lagou/edu-ad-boot:1.0 \
	lagou/edu-gateway-boot:1.0 \
	lagou/edu-config-boot:1.0 \
	lagou/edu-eureka-boot:1.0 \
	lagou/mysql:5.7 \
-o lagou-bom.tar
```

可以在别的节点导入镜像

```sh
docker load -i lagou-bom.tar
```

### 3.2删除项目容器

编写shell脚本：vi stopdocker.sh

```sh
#!/bin/bash
echo '停止所有运行容器'
docker stop $(docker ps -qa)
echo '删除所有的容器'
docker rm $(docker ps -aq)
echo '删除所有的镜像'
docker rmi $(docker images -q)
```

对sh脚本进行授权

```sh
chmod 777 stopdocker.sh
```

运行脚本

```sh
./stopdocker.sh
```

## 4.skywalking监控

### 4.1基础镜像

在节点 `192.168.91.105` 上安装docker并部署，拉取下列基础镜像

![image-20210210231550836](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210231550836.png)

拉取镜像

```sh
docker pull elasticsearch:7.9.0
# 默认es存储数据镜像
docker pull apache/skywalking-oap-server:8.1.0-es7
# webUI界面镜像
docker pull apache/skywalking-ui:8.1.0
# 制作微服项目镜像
#docker pull openjdk:8-alpine3.9
docker pull adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
```

备份镜像

```sh
docker save apache/skywalking-oap-server:8.1.0-es7 apache/skywalking-ui:8.1.0 elasticsearch:7.9.0 -o skywalking8.1.0.tar
```

导入镜像

```sh
docker load -i skywalking8.1.0.tar
```

### 4.2下载压缩包

相关网址

```sh
https://hub.docker.com/r/apache/skywalking-oap-server
# 官网
http://skywalking.apache.org/
# 中文网址
http://skywalking.apache.org/zh/
```

下载地址：可以通过官网链接地址，使用清华大学镜像地址进行下载

```sh
https://mirrors.tuna.tsinghua.edu.cn/apache/skywalking/8.1.0/apache-skywalking-apm-8.1.0.tar.gz
```

下载后的压缩包上传到 `192.168.91.107` 节点下的 `/opt` 目录下

![image-20210210231231737](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210231231737.png)

### 4.3前置条件

在节点 `192.168.91.105` 服务上修改相关配置。

**文件创建数**：修改Linux系统的限制配置，将文件创建数修改为65536个 。

```sh
vi /etc/security/limits.conf

# 新增如下内容在limits.conf文件中
es soft nofile 65536
es hard nofile 65536
es soft nproc 4096
es hard nproc 4096
```

**系统控制权限**：修改系统控制权限，ElasticSearch需要开辟一个65536字节以上空间的虚拟内存。Linux默认不允许任何用户和应用程序直接开辟这么大的虚拟内存。

```sh
vi /etc/sysctl.conf

# 添加参数:新增如下内容在sysctl.conf文件中，当前用户拥有的内存权限大小
vm.max_map_count=262144
```

重启生效：让系统控制权限配置生效。

```sh
sysctl -p

[root@node105 opt]# sysctl -p
vm.max_map_count = 262144
```

### 4.4docker-compose安装

在节点 `192.168.91.105` 和 `192.168.91.107`上均安装 Docker compose。

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

### 4.5安装skywalking

在节点 `192.168.91.105` 上使用docker-compose方式安装skywalking，根据 [https://github.com/apache/skywalking](https://github.com/apache/skywalking) 官网地址进入docker目录中，修改docker-compose.yml文件内容。

修改后的docker-compose文件内容如下：

```sh
version: '3.5'
services:
  elasticsearch:
    image: elasticsearch:7.9.0
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
    environment:
      discovery.type: single-node
      TZ: Asia/Shanghai
    ulimits:
      memlock:
        soft: -1
        hard: -1
  oap:
    image: apache/skywalking-oap-server:8.1.0-es7
    container_name: oap
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    restart: always
    ports:
      - 11800:11800
      - 12800:12800
    environment:
      SW_STORAGE: elasticsearch7  #指定ES版本
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
      TZ: Asia/Shanghai
  ui:
    image: apache/skywalking-ui:8.1.0
    container_name: ui
    depends_on:
      - oap
    links:
      - oap
    restart: always
    ports:
      - 8080:8080
    environment:
      SW_OAP_ADDRESS: oap:12800
      TZ: Asia/Shanghai
```

文件修改后上传到下图所示目录下

![image-20210210235435171](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210235435171.png)

启动、查看服务

```sh
# 启动服务（后台运行）
docker-compose up -d
# 查看服务
docker-compose ps
# 查看日志
docker-compose logs

# 重启服务
docker-compose restart
# 停止服务
docker-compose down
```

![image-20210210235706614](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210210235706614.png)

服务启动成功后，使用google浏览器访问skywalking-ui界面进行测试：http://192.168.91.105:8080/ ，UI端口号配置的 8080，若发生冲突可以在配置中修改。（elasticsearch启动时间比较长，需要耐心等待几分钟）

![image-20210211000936273](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211000936273.png)

## 5.启动Redis和Rocketmq服务

在 `192.168.91.100` 节点上启动Redis和Rocketmq服务。对于Redis和RocketMQ服务的安装在这里就不过多叙述了。

### 5.1Redis启动与关闭

```sh
cd /opt/servers/redis/bin
# 启动redis服务
redis-server redis.conf
# 关闭redis服务
redis-cli shutdown
```

![image-20210211164459139](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211164459139.png)

### 5.2RocketMQ的启动与关闭

1.启动rocketmq

```shell
# 1.先启动NameServer
[root@atzlg8 rocket]# mqnamesrv
# 2.再启动Broker
[root@atzlg8 rocket]# mqbroker -n localhost:9876

# 动态查看namesrv和broker的启动日志
[root@atzlg8 rocket]# tail -f ~/logs/rocketmqlogs/namesrv.log
[root@atzlg8 rocket]# tail -f ~/logs/rocketmqlogs/broker.log
```

2.关闭rocketmq

```shell
# 1.先关闭broker
[root@linux100 rocketmqlogs]# mqshutdown broker
# 2.再关闭namesrv
[root@linux100 rocketmqlogs]# mqshutdown namesrv
```

## 6.整合SpringCloud工程

整合其实很简单，不需要引入依赖，也不需要添加任何代码，我们只需要在启动jar包时配置参数即可。edu-bom项目采用docker-compose方式进行一键式部署。

在 `192.168.91.107` 节点上部署SpringCloud项目。**在gitee仓库的配置文件中数据库连接的IP和PORT需要做对应修改**。

### 6.1拉取基础镜像

```sh
# 制作微服项目镜像
#docker pull openjdk:8-alpine3.9
docker pull adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# mysql基础镜像
docker pull mysql:5.7.31
```

### 6.2创建项目结构

也可以如上文中使用脚本进行创建相关项目结构。

```sh
mkdir -p /data
cd /data
mkdir -p edu-bom
cd edu-bom
mkdir -p mysql edu-eureka-boot edu-config-boot edu-gateway-boot edu-boss-boot edu-course-boot edu-message-boot edu-ad-boot
cd mysql
mkdir -p docker-entrypoint-initdb.d lagou
```

项目结构创建成功后，可以将各个微服务应用进行打包上传到对应的目录下，在各自的服务 `pom.xml` 文件中添加如下构建，然后再进行打包。

```xml
<build>
    <finalName>${project.name}</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal><!--可以把依赖的包都打包到生成的Jar包中-->
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-clean-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

每个服务目录下存放文件如下

![image-20210211192450967](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211192450967.png)

### 6.3配置agent

将上传在 `/opt` 目录下的 apache-skywalking-apm-8.1.0.tar.gz 压缩包进行解压到 `/data` 目录下

```sh
[root@node107 opt]# ls
apache-skywalking-apm-8.1.0.tar.gz  containerd

# 将skywalking压缩包解压到 /data 目录下
tar zxf apache-skywalking-apm-8.1.0.tar.gz -C /data/
```

将skywalking下的agent复制到各个微服务目录下

```sh
cp -r /data/apache-skywalking-apm-bin/agent/ /data/edu-bom/edu-ad-boot/
cp -r /data/apache-skywalking-apm-bin/agent/ /data/edu-bom/edu-course-boot/
cp -r /data/apache-skywalking-apm-bin/agent/ /data/edu-bom/edu-message-boot/
cp -r /data/apache-skywalking-apm-bin/agent/ /data/edu-bom/edu-boss-boot/
cp -r /data/apache-skywalking-apm-bin/agent/ /data/edu-bom/edu-config-boot/
cp -r /data/apache-skywalking-apm-bin/agent/ /data/edu-bom/edu-eureka-boot/
cp -r /data/apache-skywalking-apm-bin/agent/ /data/edu-bom/edu-gateway-boot/
```

### 6.4各个服务的Dockerfile文件

每个服务的Dockerfile文件都放在各自对应的自己目录下，如下图

![image-20210211163400872](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211163400872.png)

#### mysql

将所有sql脚本上传到 `/data/edu-bom/mysql/docker-entrypoint-initdb.d` 目录下，再编写Dockerfile文件

```sh
# 基础镜像
FROM mysql:5.7.31
# 编写作者信息和日期亚洲上海
MAINTAINER mysql from date UTC By Asia/Shanghai "zlg@lagou.com"
# 修改时区东八区
ENV TZ Asia/Shanghai
# 将脚本复制到指定目录下,mysql容器初始化后自动导入数据库
COPY docker-entrypoint-initdb.d/edu_ad.sql /docker-entrypoint-initdb.d
COPY docker-entrypoint-initdb.d/edu_authority.sql /docker-entrypoint-initdb.d
COPY docker-entrypoint-initdb.d/edu_comment.sql /docker-entrypoint-initdb.d
COPY docker-entrypoint-initdb.d/edu_course.sql /docker-entrypoint-initdb.d
COPY docker-entrypoint-initdb.d/edu_message.sql /docker-entrypoint-initdb.d
COPY docker-entrypoint-initdb.d/edu_oauth.sql /docker-entrypoint-initdb.d
COPY docker-entrypoint-initdb.d/edu_order.sql /docker-entrypoint-initdb.d
COPY docker-entrypoint-initdb.d/edu_pay.sql /docker-entrypoint-initdb.d
COPY docker-entrypoint-initdb.d/edu_user.sql /docker-entrypoint-initdb.d
```

#### edu-eureka-boot

在 `/opt` 目录下创建 `skyagent` 目录

```sh
mkdir -p /opt/skyagent/
```

Dockerfile文件内容

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-eureka-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai
# 将agent目录下内容复制到skyagent目录下
COPY agent/ /opt/skyagent/
COPY edu-eureka-boot.jar app.jar

EXPOSE 8761
# 指定jvm内存，skywalking-agent运行jar包，服务名称，skywalking服务ip和port
ENTRYPOINT ["java","-Xmx512m","-javaagent:/opt/skyagent/skywalking-agent.jar","-Dskywalking.agent.service_name=edu-eureka-boot","-Dskywalking.collector.backend_service=192.168.91.105:11800","-jar","/app.jar"]
```

#### edu-config-boot

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-config-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai

COPY agent/ /opt/skyagent/
COPY edu-config-boot.jar app.jar

EXPOSE 8090
ENTRYPOINT ["java","-Xmx512m","-javaagent:/opt/skyagent/skywalking-agent.jar","-Dskywalking.agent.service_name=edu-config-boot","-Dskywalking.collector.backend_service=192.168.91.105:11800","-jar","/app.jar"]
```

#### edu-boss-boot

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-boss-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai

COPY agent/ /opt/skyagent/
COPY edu-boss-boot.jar app.jar

EXPOSE 8082
CMD sleep 30
ENTRYPOINT ["java","-Xmx512m","-javaagent:/opt/skyagent/skywalking-agent.jar","-Dskywalking.agent.service_name=edu-boss-boot","-Dskywalking.collector.backend_service=192.168.91.105:11800","-jar","/app.jar"]
```

#### edu-gateway-boot

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-gateway-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai

COPY agent/ /opt/skyagent/
COPY edu-gateway-boot.jar app.jar

EXPOSE 9001
CMD sleep 60
ENTRYPOINT ["java","-Xmx512m","-javaagent:/opt/skyagent/skywalking-agent.jar","-Dskywalking.agent.service_name=edu-gateway-boot","-Dskywalking.collector.backend_service=192.168.91.105:11800","-jar","/app.jar"]
```

#### edu-course-boot

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-course-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai

COPY agent/ /opt/skyagent/
COPY edu-course-boot-impl.jar app.jar

EXPOSE 8003
CMD sleep 20
ENTRYPOINT ["java","-Xmx512m","-javaagent:/opt/skyagent/skywalking-agent.jar","-Dskywalking.agent.service_name=edu-course-boot","-Dskywalking.collector.backend_service=192.168.91.105:11800","-jar","/app.jar"]
```

#### edu-message-boot

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-message-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai

COPY agent/ /opt/skyagent/
COPY edu-message-boot-impl.jar app.jar

EXPOSE 8006
CMD sleep 25
ENTRYPOINT ["java","-Xmx512m","-javaagent:/opt/skyagent/skywalking-agent.jar","-Dskywalking.agent.service_name=edu-message-boot","-Dskywalking.collector.backend_service=192.168.91.105:11800","-jar","/app.jar"]
```

#### edu-ad-boot

```sh
# 从dockerhub拉取基础镜像
FROM adoptopenjdk/openjdk11:jdk-11.0.8_10-alpine
# 作者信息
MAINTAINER zlg Docker edu-ad-boot "zlg@lagou.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories
# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai

COPY agent/ /opt/skyagent/
COPY edu-ad-boot-impl.jar app.jar

EXPOSE 8001
CMD sleep 20
ENTRYPOINT ["java","-Xmx512m","-javaagent:/opt/skyagent/skywalking-agent.jar","-Dskywalking.agent.service_name=edu-ad-boot","-Dskywalking.collector.backend_service=192.168.91.105:11800","-jar","/app.jar"]
```

### 6.5编写docker-compose文件

编写docker-compose文件，后上传到 `/data/edu-bom` 文件夹下

![image-20210211175057784](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211175057784.png)

docker-compose.yml文件内容如下

```sh
version: '3.3'
services:
  lagou-mysql:
    build:
      context: ./mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
    restart: always
    container_name: lagou-mysql
    volumes:
      - /data/edu-bom/mysql/lagou:/var/lib/mysql
    image: lagou/mysql:5.7
    ports:
      - 3306:3306
  lagou-eureka:
    build:
      context: ./edu-eureka-boot
    restart: always
    ports:
      - 8761:8761
    container_name: edu-eureka-boot
    hostname: edu-eureka-boot
    image: lagou/edu-eureka-boot:1.0
    depends_on:
      - lagou-mysql
  lagou-config:
    build:
      context: ./edu-config-boot
    restart: always
    ports:
      - 8090:8090
    container_name: edu-config-boot
    hostname: edu-config-boot
    image: lagou/edu-config-boot:1.0
    depends_on:
      - lagou-eureka
  lagou-ad:
    build:
      context: ./edu-ad-boot
    restart: always
    ports:
      - 8001:8001
    container_name: edu-ad-boot
    hostname: edu-ad-boot
    image: lagou/edu-ad-boot:1.0
    depends_on:
      - lagou-config
  lagou-course:
    build:
      context: ./edu-course-boot
    restart: always
    ports:
      - 8003:8003
    container_name: edu-course-boot
    hostname: edu-course-boot
    image: lagou/edu-course-boot:1.0
    depends_on:
      - lagou-config
  lagou-message:
    build:
      context: ./edu-message-boot
    restart: always
    ports:
      - 8006:8006
    container_name: edu-message-boot
    hostname: edu-message-boot
    image: lagou/edu-message-boot:1.0
    depends_on:
      - lagou-config
  lagou-boss:
    build:
      context: ./edu-boss-boot
    restart: always
    ports:
      - 8082:8082
    container_name: edu-boss-boot
    hostname: edu-boss-boot
    image: lagou/edu-boss-boot:1.0
    depends_on:
      - lagou-ad
  lagou-gateway:
    build:
      context: ./edu-gateway-boot
    restart: always
    ports:
      - 9001:9001
    container_name: edu-gateway-boot
    hostname: edu-gateway-boot
    image: lagou/edu-gateway-boot:1.0
    depends_on:
      - lagou-boss
```

### 6.6构建镜像

```sh
# 制作镜像，通过docker-compose文件内容的build：context指定目录下的Dockerfile文件构建各个服务内容
docker-compose build
# 启动服务
docker-compose up -d
# 删除镜像, 挂载角，所有镜像
docker-compose down -v --rmi all

# 查看服务
docker-compose ps
# 查看日志
docker-compose logs
# 重启服务
docker-compose restart

# 根据容器名称查看单个服务的启动日志
docker logs -f containerName
# 查看运行的容器
docker ps
# 查看镜像
docker images
```

### 6.7所有服务启动成功后情况

`docker-compose ps`

![image-20210211184545497](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211184545497.png)

`docker images`

![image-20210211184617158](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211184617158.png)

`docker ps`

![image-20210211184713999](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211184713999.png)

### 6.8测试项目

可以根据ad服务来测试项目是否部署成功

```sh
# 访问Eureka
http://192.168.91.107:8761/
# 访问ad服务
http://192.168.91.107:8001/ad/space/getAllSpaces
# 通过网关访问ad服务
http://192.168.91.107:9001/boss/ad/space/getAllSpaces
# WebSocket测试链接（通过message服务访问测试界面）
http://192.168.91.107:8006/index
```

### 6.9效果演示

**Eureka服务**：http://192.168.91.107:8761/

![image-20210211175911033](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211175911033.png)

通过**课程服务**指定端口来访问指定课程：http://192.168.91.107:8003/course/getCourseById?courseId=7

![image-20210211180431380](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211180431380.png)

通过**网关服务**指定端口和路由来访问指定课程：http://192.168.91.107:9001/boss/course/getCourseById?courseId=7

![image-20210211180635427](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211180635427.png)

SkyWalking UI界面效果：

- 主界面

![image-20210211180810378](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211180810378.png)

- 拓补图

![image-20210211181034188](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211181034188.png)

- 链路追踪

![image-20210211181302691](https://gitee.com/itzlg/mypictures/raw/master/img/image-20210211181302691.png)



