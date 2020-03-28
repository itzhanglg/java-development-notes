### 一.docker简介和理念

-   docker官网: [http://www.docker.com](http://www.docker.com)
-   docker中文网站: [https://www.docker-cn.com/](https://www.docker-cn.com/)
-   Docker Hub仓库: [https://hub.docker.com/](https://hub.docker.com/)
-   DaoCloud镜像市场: [https://hub.daocloud.io/](https://hub.daocloud.io/)

#### 1.docker是什么,为什么会有docker出现

**Docker**是一个开源的应用容器引擎；是一个轻量级容器技术；Docker支持将软件编译成一个镜像；然后在镜像中各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像；运行中的这个镜像称为容器，容器启动是非常快速的。

为什么会有docker出现呢?

一款产品从开发到上线，从操作系统，到运行环境，再到应用配置。作为开发+运维之间的协作我们需要关心很多东西，这也是很多互联网公司都不得不面对的问题，特别是各种版本的迭代之后，不同版本环境的兼容，对运维人员都是考验。

开发人员利用 Docker 可以消除协作编码时“在我的机器上可正常工作”的问题。安装的时候，把原始环境一模一样地复制过来。Docker镜像的设计，使得Docker得以打破过去「程序即应用」的观念。透过镜像(images)将作业系统核心除外，运作应用程式所需要的系统环境，由下而上打包，达到应用程式跨平台间的无缝接轨运作。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200315152028510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 2.docker的理念

Docker是基于Go语言实现的云开源项目。

Docker的主要目标是“Build，Ship and Run Any App,Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次封装，到处运行”。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200315152127375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

Linux 容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用运行在 Docker 容器上面，而 Docker 容器在任何操作系统上都是一致的，这就实现了跨平台、跨服务器。**只需要一次配置好环境，换到别的机子上就可以一键部署好，大大简化了操作**.

#### 3.以前的虚拟机技术

虚拟机（virtual machine）就是带环境安装的一种解决方案。

它可以在一种操作系统里面运行另一种操作系统，比如在Windows 系统里面运行Linux 系统。对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。  

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031515220757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

虚拟机的缺点：资源占用多, 冗余步骤多, 启动慢

#### 4.容器虚拟化技术

由于前面虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Containers，缩写为 LXC）。

Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。**有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置**。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200315152218311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

 比较了 Docker 和传统虚拟化方式的不同之处：

-   传统虚拟机技术是虚拟出一套硬件后，在其上**运行一个完整操作系统**，在该系统上再运行所需应用进程；
-   容器内的应用进程**直接运行于宿主的内核**，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。
-   **每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响**，能区分计算资源。

#### 5.docker优势和好处

开发或运维(DevOps)可以利用docker进行一次构建,到处运行:

-   更快速的应用交付和部署

    Docker化之后只需要**交付少量容器镜像文件**，在正式生产环境加载镜像并运行即可，应用安装配置在镜像里已经内置好，大大节省部署配置和测试验证时间。

-   更便捷的升级和扩缩容

    随着微服务架构和Docker的发展，大量的应用会通过微服务方式架构，应用的开发构建将变成搭乐高积木一样，每个Docker容器将变成一块“积木”，应用的升级将变得非常容易。当现有的容器不足以支撑业务处理时，可**通过镜像运行新的容器进行快速扩容**，使应用系统的扩容从原先的天级变成分钟级甚至秒级。

-   更简单的系统运维

    应用容器化运行后，生产环境运行的应用可与开发、测试环境的应用高度一致，**容器会将应用程序相关的环境和状态完全封装起来**，不会因为底层基础架构和操作系统的不一致性给应用带来影响，产生新的BUG。当出现程序异常时，也可以通过测试环境的相同容器进行快速定位和修复。

-   更高效的计算资源利用

    Docker是**内核级虚拟化**，其不像传统的虚拟化技术一样需要额外的Hypervisor支持，所以在一台物理机上可以运行很多个容器实例，可大大提升物理服务器的CPU和内存的利用率。

### 二.docker核心概念

#### 1.docker关键词

docker主机(Host)：安装了Docker程序的机器（Docker直接安装在操作系统之上）；

docker客户端(Client)：连接docker主机进行操作；

docker仓库(Registry)：用来保存各种打包好的软件镜像；

docker镜像(Images)：软件打包好的镜像；放在docker仓库中；

docker容器(Container)：镜像启动后的实例称为一个容器；容器是独立运行的一个或一组应用

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200315152327762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 2.使用Docker的步骤

1.  安装Docker
2.  去Docker仓库找到这个软件对应的镜像；
3.  使用Docker运行这个镜像，这个镜像就会生成一个Docker容器；
4.  对容器的启动停止就是对软件的启动停止；

#### 3.Docker的基本组成及架构图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200315152245346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

-   **镜像(image)**

    Docker 镜像（Image）就是一个只读的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。

-   **容器(container)**

    Docker 利用容器（Container）独立运行的一个或一组应用。容器是用镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。

-   **仓库(repository)**

    仓库（Repository）是集中存放镜像文件的场所。仓库(Repository)和仓库注册服务器（Registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag)。仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 [Docker Hub](https://hub.docker.com/)，存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云 、网易云 等

#### 4.docker核心概念总结

需要正确的理解**仓库/镜像/容器**这几个概念:

 Docker 本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就似乎 image镜像文件。只有通过这个镜像文件才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。

-   image 文件生成的容器实例，本身也是一个文件，称为镜像文件。
-   一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器
-   仓库，就是放了一堆镜像的地方，我们可以把镜像发布到仓储中，需要的时候从仓储中拉下来就可以了。

### 三.docker安装和配置镜像加速

#### 1.CentOS7上安装Docker

官网中文安装参考手册: [https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#prerequisites](https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#prerequisites)

简单安装:

```shell
#检查内核版本，必须是3.10及以上
uname -r
#安装docker
yum install docker
#输入y确认安装
#启动docker
[root@localhost ~]# systemctl start docker
[root@localhost ~]# docker -v
Docker version 1.12.6, build 3e8e77d/1.12.6
#开机启动docker
[root@localhost ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
#停止docker
systemctl stop docker
```

Centos7及以上版本确认: `cat /etc/redhat-release`

```shell
[root@atzlg3 ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
```

yum安装gcc相关: Centos7能上外网

```shell
yum -y install gcc
yum -y install gcc-c++
```

卸载旧版本:

```shell
yum -y remove docker docker-common docker-selinux docker-engine
2.18.3官网版本:
yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine
```

安装需要的软件包:

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

设置stable镜像仓库:

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

更新yum软件包索引:

```shell
yum makecache fast
```

安装,启动,测试,卸载docker:

```shell
#安装docker ce
yum -y install docker-ce
#启动docker
systemctl start docker
#测试docker
docker version
docker run hello-world
#配置镜像加速
mkdir -p /etc/docker
vim  /etc/docker/daemon.json
 #网易云
{"registry-mirrors": ["http://hub-mirror.c.163.com"] }
 #阿里云
{
  "registry-mirrors": ["https://｛自已的编码｝.mirror.aliyuncs.com"]
}
systemctl daemon-reload
systemctl restart docker
#卸载docker
systemctl stop docker 
yum -y remove docker-ce
rm -rf /var/lib/docker
```

测试docker时: `docker run hello-world`

docker里是没有hello-world镜像的,会先从仓库里pull下来再运行.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200315152350956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 2.配置镜像加速

**阿里云镜像加速**:

注册一个[阿里云](https://cr.console.aliyun.com/)的账号,可以使用淘宝账号登录;

获取[加速器地址](https://cr.console.aliyun.com/cn-shanghai/instances/mirrors):

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200315152419557.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

配置本机Docker运行镜像加速器:

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，可以配置加速器来解决，我使用的是阿里云的本人自己账号的镜像地址(需要自己注册有一个属于你自己的)： https://xxxx.mirror.aliyuncs.com

重启Docker后台服务: ` systemctl daemon-reload`, `systemctl restart docker`

**网易云镜像加速**:

```shell
vi /etc/docker/daemon.json

{
 "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

**国内加速站点**:

```java
https://registry.docker-cn.com	//Docker中国区官方镜像
http://hub-mirror.c.163.com		//网易
// 下面都需要自己注册账号后用自己私人的加速器
https://3laho3y3.mirror.aliyuncs.com	//阿里云
http://f1361db2.m.daocloud.io	//daocloud
https://mirror.ccs.tencentyun.com	//腾讯云
```

如果还是进行search很慢不起作用,可能时DNS解析的问题:

修改服务器DNS网络配置：

```shell
vi /etc/resolv.conf
# 把里面的内容清楚掉,改为或替换为自己网络的IPV4 DNS的地址
nameserver 8.8.8.8
nameserver 8.8.8.4
```

修改完后重启网络服务:

```shell
systemctl restart network
```

参考链接: [Docker Search 异常](https://blog.csdn.net/aotumemedazhao1996/article/details/102853164)

#### 3.docker底层原理

docker是怎么工作的:

Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上， 然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上， 然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。 **容器，是一个运行时环境，就是我们前面说到的集装箱**。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/202003151524300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

 为什么Docker比VM快:

-   docker有着比虚拟机更少的抽象层。由亍docker不需要Hypervisor实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。
-   docker利用的是宿主机的内核,而不需要Guest OS。因此,当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。仍而避免引寻、加载操作系统内核返个比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载Guest OS,返个新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,则省略了返个过程,因此新建一个docker容器只需要几秒钟。

### 四.docker常用命令

#### 1.帮助命令

```shell
docker -v
docker version
docker info
docker --help
```

#### 2.镜像命令

-   `docker images [OPTIONS]`: 列出本地主机上的镜像

    -   -a: 列出本地所有的镜像（含中间映像层）
    -   -q: 只显示镜像ID
    -   --digests: 显示镜像的摘要信息
    -   --no-trunc: 显示完整的镜像信息

    **REPOSITORY**：表示镜像的仓库源; **TAG**：镜像的标签; **IMAGE ID**：镜像ID; **CREATED**：镜像创建时间; **SIZE**：镜像大小;  同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，使用 `REPOSITORY:TAG` 来定义不同的镜像。如果不指定一个镜像的版本标签，docker 将默认使用 `mysql:latest` 镜像.

-   `docker search [OPTIONS] XXX`: 搜索xxx镜像

    -   --no-trunc : 显示完整的镜像描述
    -   -s : 列出收藏数不小于指定值的镜像
    -   --automated : 只列出 automated build类型的镜像

-   `docker pull XXX[:TAG]`: 下载xxx镜像, tag表示标签，可选, 多为软件的版本，默认是latest

-   `docker rmi XXX-id`: 删除镜像

    -   `docker rmi -f XXXID` : 删除单个
    -   `docker rmi -f XXX1:TAG XXX2:TAG` : 删除多个
    -   `docker rmi -f $(docker images -qa)` :删除全部

#### 3.容器命令

软件镜像（QQ安装程序）----运行镜像----产生一个容器（正在运行的软件，运行的QQ）,有镜像才能创建容器,这是根本前提.下载一个tomcat镜像演示: `docker pull tomcat` 

-   `docker run [OPTIONS] IMAGE名称 [COMMAND] [ARG...]`: 根据下载的镜像名称新建并启动容器
    -   --name: 为容器指定一个名称
    -   -d: 后台运行容器，并返回容器ID，也即启动守护式容器
    -   -i: 以**交互模式**运行容器，通常与 -t 同时使用
    -   -t: 为容器重新分配一个**伪输入终端**，通常与 -i 同时使用
    -   -P: 随机端口映射
    -   -p: 指定端口映射
        -   **hostPort:containerPort**
        -   ip::containerPort
        -   ip:hostPort:containerPort
        -   containerPort
    -   启动交互式容器: 在容器内执行/bin/bash命令
        -   `docker run -it centos /bin/bash` 
-   `docker ps [OPTIONS]` : 列出当前所有正在运行的容器
    -   -q :静默模式，只显示容器编号
    -   -a :列出当前所有正在运行的容器+历史上运行过的
    -   -l :显示最近创建的容器
    -   -n：显示最近n个创建的容器
    -   --no-trunc :不截断输出
-   退出容器:
    -   exit: 容器停止退出
    -   ctrl+P+Q: 容器不停止退出
-   `docker start containerID/containerName`: 启动容器
-   `docker restart containerID/containerName`: 重启容器
-   `docker stop containerID/containerName`: 停止容器
-   `docker kill containerID/containerName`: 强制停止容器
-   `docker rm containerID` : 删除已停止的容器
    -   `docker rm -f $(docker ps -a -q)` : 一次性删除多个容器
    -   `docker ps -a -q | xargs docker rm` : 一次性删除多个容器

**重要**:

-   `docker run -d containerName` : 以后台模式启动一个容器,启动守护式容器

    Docker容器后台运行,就必须有一个前台进程. 容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。

-   `docker logs -f -t --tail num containerId` :查看容器日志

    -   -t: 加入时间戳
    -   -f: 跟随最新的日志打印
    -   --tail num: 显示最后多少条

-   `docker top containerId` : 查看容器内运行的进程

-   `docker inspect containerId` : 查看容器内部细节

-   进入正在运行的容器并以命令行交互:

    -   `docker exec -it containerId bashShell(如:/bin/bash)` : 在容器中打开新的终端并且可以启动新的进程
    -   `docker attach containerId` : 直接进入容器启动命令终端,不会启动新的进程

-   `docker cp containerId:容器内路径 目的主机路径` : 从容器内拷贝文件到主机上

    eg: `docker cp f5fa5fad5dad7:/usr/local/mycptest/container.txt(容器内路径) /tmp/test.txt(主机目录)`

#### 4.总结

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200315152445622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)



### 五.docker示例安装

#### 1.安装tomcat示例

````shell
1、搜索镜像
[root@localhost ~]# docker search tomcat
2、拉取镜像
[root@localhost ~]# docker pull tomcat
3、根据镜像启动容器,下面没有做端口映射,无法直接通过浏览器访问容器中的tomcat
docker run --name mytomcat -d tomcat:latest
4、docker ps  
查看运行中的容器
5、 停止运行中的容器
docker stop  容器的id
6、查看所有的容器
docker ps -a
7、启动容器
docker start 容器id
8、删除一个容器
 docker rm 容器id
9、启动一个做了端口映射的tomcat
[root@localhost ~]# docker run -d -p 8888:8080 tomcat
-d：后台运行
-p: 将主机的端口映射到容器的一个端口    主机端口:容器内部的端口

10、为了演示简单关闭了linux的防火墙
service firewalld status ；查看防火墙状态
service firewalld stop：关闭防火墙
11、查看容器的日志
docker logs container-name/container-id

更多命令参看,可以参考每一个镜像的文档
https://docs.docker.com/engine/reference/commandline/docker/
````

#### 2.安装mysql示例

```shell
docker pull mysql
```

错误的启动

```shell
[root@localhost ~]# docker run --name mysql01 -d mysql
42f09819908bb72dd99ae19e792e0a5d03c48638421fa64cce5f8ba0f40f5846

# mysql自动退出了
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS               NAMES
42f09819908b        mysql               "docker-entrypoint.sh"   34 seconds ago      Exited (1) 33 seconds ago                            mysql01

# 错误日志
[root@localhost ~]# docker logs 42f09819908b
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD；这个三个参数必须指定一个
```

正确的启动

```shell
[root@localhost ~]# docker run --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
b874c56bec49fb43024b3805ab51e9097da779f2f572c22c695305dedd684c5f
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b874c56bec49        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        3306/tcp            mysql01
```

做了端口映射

```shell
[root@localhost ~]# docker run -p 3306:3306 --name mysql02 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
ad10e4bc5c6a0f61cbad43898de71d366117d120e39db651844c0e73863b9434
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ad10e4bc5c6a        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 2 seconds        0.0.0.0:3306->3306/tcp   mysql02
```

几个其他的高级操作

```shell
#把主机的/conf/mysql文件夹挂载到 mysqldocker容器的/etc/mysql/conf.d文件夹里面
#改mysql的配置文件就只需要把mysql配置文件放在自定义的文件夹下（/conf/mysql）
docker run --name mysql03 -v /conf/mysql:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

#指定mysql的一些配置参数
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```



参考链接:

-   [10分钟看懂Docker和K8S](https://zhuanlan.zhihu.com/p/53260098)
-   [docker基本概念解读](https://snailclimb.gitee.io/javaguide/#/docs/tools/Docker)
-   [从零开始入门 K8s：详解 K8s 容器基本概念](https://www.infoq.cn/article/te70FlSyxhltL1Cr7gzM)
-   [尚硅谷](http://www.atguigu.com/)
