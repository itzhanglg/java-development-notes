

## 一.Docker安装与部署

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

## 二.Docker操作

### 1.配置镜像加速

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

如果search很慢不起作用，可能时DNS解析的问题，修改服务器DNS网络配置：

```shell
vi /etc/resolv.conf
# 把里面的内容清楚掉,改为或替换为自己网络的IPV4 DNS的地址
nameserver 8.8.8.8
nameserver 8.8.8.4
# 重启网络服务
systemctl restart network
```

### 2.使用Docker镜像

#### 2.1 常用命令

查看镜像：

```shell
# 查看镜像信息
# man docker-images
[root@linux101 ~]# docker images

# 查看镜像详细信息
# docker inspect NAME[：TAG]
[root@linux101 ~]# docker inspect mysql:5.7.30
# 只查看其中一项内容时， 可以使用参数-f来指定
[root@linux101 ~]# docker inspect mysql:5.7.30 -f {{".Architecture"}}
```

获取镜像：

```shell
# 搜寻镜像，dockerhub仓库镜像, 私有仓库无法搜索到
# docker search 名称
[root@linux101 ~]# docker search mysql

# 获取镜像，如果tag号缺省，默认latest
# docker pull NAME[：TAG]
[root@linux101 ~]# docker pull mysql:5.7.30

# 添加镜像标签，如果tag号缺省，默认latest
# docker tag [原镜像名:tag号] [目标镜像名:tag号] 
[root@linux101 ~]# docker tag mysql:5.7.30 mysql5
```

删除镜像：

```shell
# 使用镜像ID删除镜像
# 如果有容器正在运行该镜像，则不能删除；如果想强行删除用 -f (不推荐)
# docker rmi IMAGE ID
[root@linux101 ~]# docker rmi 9cfcce23593a

# 使用镜像Name:Tag删除镜像
# 当镜像只剩下一个标签的时候，使用docker rmi命令会彻底删除镜像；
# 当同一个镜像拥有多个标签的时候，docker rmi命令只是删除该镜像多个标签中的指定标签而已，
# 并不影响镜像文件
# docker rmi NAME[：TAG]
[root@linux101 ~]# docker rmi mysql:5.7.30
```

上传镜像：

```shell
# 默认上传到Docker Hub官方仓库（需要登录）
# docker push NAME[:TAG]
[root@linux101 ~]# docker push mysql5:latest
```

#### 2.2 示例

```shell
[root@linux101 ~]# docker pull mysql:5.7.30
5.7.30: Pulling from library/mysql
# 镜像文件一般由若干层（layer）组成 层的唯一id镜像文件一般由若干层（layer）组成
8559a31e96f4: Pull complete
d51ce1c2e575: Pull complete
c2344adc4858: Pull complete
fcf3ceff18fc: Pull complete
16da0c38dc5b: Pull complete
b905d1797e97: Pull complete
4b50d1c6b05c: Pull complete
d85174a87144: Pull complete
a4ad33703fa8: Pull complete
f7a5433ce20d: Pull complete
3dcd2a278b4a: Pull complete

[root@linux101 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mysql        5.7.30    9cfcce23593a   6 months ago   448MB
# 添加镜像标签,如果tag号缺省，默认latest
# docker tag [原镜像名:tag号] [目标镜像名:tag号]
[root@linux101 ~]# docker tag mysql:5.7.30 mysql5
[root@linux101 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mysql5       latest    9cfcce23593a   6 months ago   448MB
mysql        5.7.30    9cfcce23593a   6 months ago   448MB

# 查看镜像详细信息：docker inspect NAME[：TAG]
[root@linux101 ~]# docker inspect mysql:5.7.30
# 只查看其中一项内容时
[root@linux101 ~]# docker inspect mysql:5.7.30 -f {{".Architecture"}}
amd64

# 搜索镜像：docker search 名称
[root@linux101 ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10281     [OK]       
mariadb                           MariaDB is a community-developed fork of MyS…   3801      [OK]       
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   750                  [OK]

# 删除镜像：docker rmi NAME[：TAG]
[root@linux101 ~]# docker rmi mysql5:latest 
Untagged: mysql5:latest
[root@linux101 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mysql        5.7.30    9cfcce23593a   6 months ago   448MB
[root@linux101 ~]# docker rmi 9cfcce23593a
```

### 3.操作Docker容器

查看容器：

```shell
# docker ps 查看运行的容器
#查看运行的容器
[root@linux101 ~]# docker ps
#查看所有容器
[root@linux101 ~]# docker ps -a

# 查看容器的详细信息
# docker inspect [容器ID]
[root@linux101 ~]# docker inspect f5230a1a1125

# 查看容器的状态
# docker stats [容器ID]
[root@linux101 ~]# docker stats f5230a1a1125
```

创建，启动容器：

```shell
# docker create NAME[:TAG]
# 可以加选项参数，-i 交互模式，-t 伪终端，-d 后台运行，-rm 容器退出后是否自动删除
[root@linux101 ~]# docker create -it nginx

# docker start 容器id
[root@linux101 ~]# docker start f5230a1a1125
```

新建并启动容器：相当于 `docker create + docker start`

```shell
# docker run NAME[:TAG]
# man docker run 或 docker run --help
# --network host 使用宿主机IP地址，-d 后端启动，--rm 终止时直接卸载掉
[root@linux101 ~]# docker run -it --rm --network host nginx
```

终止容器：

```shell
# 首先向容器发送SIGTERM信号，等待一段超时时间（默认为10 秒）后，再发送SIGKILL信号来终止容器
# docker stop 容器id -t 时间 (默认10秒)
[root@linux101 ~]# docker stop f5230a1a1125 -t 5

# docker kill命令会直接发送SIGKILL信号来强行终止容器
# docker kill 容器id
[root@linux101 ~]# docker kill f5230a1a1125

# docker restart命令会将一个运行态的容器先终止，然后再重新启动
[root@linux101 ~]# docker restart f5230a1a1125
```

容器启动后进入容器：无论在容器内进行何种操作，依据镜像创建的其他容器都不会受影响(由于namespace的隔离)（将数据持久化的除外）。

```shell
# exec 可以在容器内直接执行任意命令，操作为容器ID后边的命令，-it: 以伪终端模式
# docker exec -it [容器ID] /bin/bash
[root@linux101 ~]# docker exec -it 44be2190d011 /bin/bash
```

删除容器：要直接删除一个运行中的容器，可以添加-f参数（不推荐）。Docker会先发送SIGKILL信号给容器，终止其中的应用，之后强行删除。

```shell
# docker rm命令只能删除处于终止或退出状态的容器，并不能删除还处于运行状态的容器
# docker rm [容器ID]
[root@linux101 ~]# docker rm f5230a1a1125
```

### 4.访问Docker仓库

Docker Hub是最大的公共镜像仓库(https://hub.docker.com/)。在公共仓库中注册一个账号，每ID可以免费拥有1个私有镜像，

![image-20201220142815018](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220142815018.png)

登陆仓库：默认登陆的是docker hub

```shell
[root@linux101 ~]# docker login -u gavinli80s -p ljp123465
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
```

登录成功的用户可以上传个人制造的镜像。用户无需登录即可通过docker search命令来查找官方仓库中的镜 像，并利用docker pull命令来将它下载到本地。

登出仓库：可以同时登陆多个docker仓库，因此此命令一般不执行。

```shell
[root@linux101 ~]# docker logout
Removing login credentials for https://index.docker.io/v1/
```

认证文件：Mac/Win机器上的是隐藏密码的，但是在Linux下是显示密码的，只不过进行了base64编码, 只要拷贝
此文件到其他机器指定目录下(/root/.docker/config.json)即可免登录。

```shell
[root@linux101 ~]# vim /root/.docker/config.json
{
    "auths": {
        "https://index.docker.io/v1/": {
        	"auth": "Z2F2aW5saTgwczpsanAxMjM0NjU="
        }
    },
    "HttpHeaders": {
    	"User-Agent": "Docker-Client/19.03.12 (linux)"
    }
}
```

### 5.常用软件的容器化部署

#### 5.1 MySQL

```shell
# 获取镜像
docker pull mysql:5.7.30

# 根据镜像新建容器并启动
# --network host : 宿主机IP 不能再使用端口映射 -p 宿主机端口:容器端口 只能使用容器端口
# --rm:当容器停止后，对容器及其申请的卷执行删除操作
# -e key=value: 指定环境变量（此处指定了mysql root密码的环境变量，密码为root）
# -d :后台运行
docker run --network host -e MYSQL_ROOT_PASSWORD=root -d --rm mysql:5.7.30
```

访问MySQL：

```shell
[root@linux101 ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS     NAMES
06530b1ca4aa   mysql:5.7.30   "docker-entrypoint.s…"   About a minute ago   Up About a minute             sleepy_elbakyan

# 本机访问
[root@linux101 ~]# docker exec -it 0653 mysql -uroot -p
Enter password: 
mysql>

# 远端访问，在另外一台安装MySQL服务的机器上访问
[root@atzlg5 ~]# mysql -h192.168.91.101 -uroot -proot
mysql> 
```

#### 5.2 Tomcat

```shell
# 获取镜像
docker pull tomcat:8.5.56-jdk8-openjdk
# 根据镜像新建容器并启动
# -it ： 交互式伪客户端
# --rm:当容器停止后，对容器及其申请的卷执行删除操作
# --network host：宿主机IP
docker run -it --rm --network host -d tomcat:8.5.56-jdk8-openjdk
```

访问tomcat：

![image-20201220151932499](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220151932499.png)

```shell
[root@linux101 ~]# curl http://192.168.91.101:8080/
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/8.5.56</h3></body></html>
```

进入到运行中的容器tomcat:8.5.56-jdk8-openjdk：

```shell
[root@linux101 ~]# docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS     NAMES
a4d830e10e87   tomcat:8.5.56-jdk8-openjdk   "catalina.sh run"        2 minutes ago    Up 2 minutes              practical_wilbur
[root@linux101 ~]# docker exec -it a4d830e10e87 /bin/bash
root@linux101:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@linux101:/usr/local/tomcat# cd webapps
root@linux101:/usr/local/tomcat/webapps# mkdir app1
root@linux101:/usr/local/tomcat/webapps# cd app1/
root@linux101:/usr/local/tomcat/webapps/app1# echo Hello Docker > index.html
```

![image-20201220152012553](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220152012553.png)

#### 5.3 Nginx

```shell
# 获取镜像
docker pull nginx
# 根据镜像新建容器并启动
# --name:运行的容器名称
docker run --name nginx1 --network host -d nginx
```

访问Nginx：

![image-20201220152357461](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220152357461.png)

#### 5.4 Redis

```shell
# 获取镜像
docker pull redis:5.0.9
# 根据镜像新建容器并启动
docker run --network host -d redis:5.0.9
# 若宿主端口6379被占用，需要使用-p将容器端口映射为宿主端口
docker run -p 16379:6379 -d redis:5.0.9
```

访问Redis：

```shell
[root@linux101 ~]# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS     NAMES
77ac2ac7d0fc   redis:5.0.9   "docker-entrypoint.s…"   46 seconds ago   Up 45 seconds             magical_davinci

# 本机访问
[root@linux101 ~]# docker exec -it 77ac2ac7d0fc redis-cli
127.0.0.1:6379> 

# 远端访问
[root@localhost ~]# cd /var/redis-cluster/7001/bin/
[root@localhost bin]# ./redis-cli -h 192.168.91.101
192.168.72.129:6379>
```

### 6.Docker命令图谱

常用命令图：

![image-20201220153633379](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220153633379.png)

命令流程图：

![image-20201220153907526](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220153907526.png)

## 三.DockerFile

DockerFile是一个文本格式的配置文件，用户可以使用DockerFile来快速创建自定义的镜像。

### 1.DockerFile基本结构

Dockerfile由一行行命令语句组成，并且支持以 `#` 开头的注释行。Dockerfile分为四部分：基础镜像信息、维护者信息、 镜像操作指令和容器启动时执行指令。

```shell
# FROM:指明所基于的镜像名称，从ubuntu:18.04Docker镜像创建一个图层
FROM ubuntu:18.04
# MAINTAINER:维护者信息，docker_user维护（可以不写）
MAINTAINER docker_user docker_user@email.com
# COPY:从Docker客户端的当前目录添加文件，镜像操作指令
COPY . /app
# RUN:构建镜像时执行make命令，每运行一条RUN指令，镜像就添加新的一层，并提交（添加可写层）
RUN make /app
# CMD:指定在容器中运行什么命令，用来指定运行容器时的操作命令
CMD python /app/app.py
```

Docker镜像由只读层组成，每个只读层代表一个Dockerfile指令。这些层是堆叠的，每个层都是上一层的变化的增量。Docker可以通过读取Dockerfile指令来自动构建镜像。

示例：

```shell
# 在一个空目录下，新建一个名为 Dockerfile 文件
mkdir /usr/dockerfile -p
# 编辑 dockerfile
vim dockerfile-demo
--------------------------------------------------------------------
# 基础镜像
FROM nginx:latest
# 维护者 可以省略
MAINTAINER gavinli gavinli@docker.com
# 启动容器
RUN mkdir /usr/share/nginx/html/ -p
RUN echo Hello DockerFile! > /usr/share/nginx/html/demo.html
# run command
--------------------------------------------------------------------
# 构建镜像 . : 根据当前上下文环境构建
docker build -f dockerfile-demo -t fish/nginx:v1 .
# 运行
docker run --rm -d -it --network host fish/nginx:v1
```

浏览器访问nginx：

![image-20201220194615046](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220194615046.png)

### 2.DockerFile指令详解





### 3.DockerFile创建镜像





### 4.DockerFile模板







## 四.Docker应用实战

使用DockerFile构建镜像并搭建 swarm+compose集群。

![image-20201220203220991](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201220203220991.png)

- Hot是应用程序(springboot)，打成jar包：spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar；
- 利用dockerfile将Hot.jar构建成镜像fish/blog:v1
- 构建Swarm集群
- 在 Swarm 集群中使用 compose 文件 （docker-compose.yml） 来配置、启动多个服务 包括: Mysql、Redis以及应用程序Hot。

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
    #主备都存在，目录若不存在需要mkdir
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





