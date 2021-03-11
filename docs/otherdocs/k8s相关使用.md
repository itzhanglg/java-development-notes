### 习题

Mysql服务安装、springboot项目部署、浏览器客户端访问调用

（1）搭建mysql服务器，要求必须要挂载数据卷，挂载方式不限。可以使用deployment或者statefulSet方式部署

（2）将项目数据库导入mysql服务器。可以采用sqlyog、navicat。

（3）开发spirnboot项目。

（4）spirnboot项目打包测试。

（5）spirnboot项目制作成镜像。

（6）在k8s集群中spirnboot项目。

考察技能点：

（1）mysql数据卷挂载。deployment的yml文件编写。

（2）java项目基础开发能力

（3）springboot项目至少要完成一张表查询所有记录的功能。推荐完成单表的CRUD。可以使用postman等工具进行测试。不需要编写页面。

（4）Dockerfile基本技能

（5）mysql容器与springboot项目容器相互访问模式：以下方式任选其一。

- 采用SVC的IP地址方式
- 采用SVC的名称方式 

### MySQL服务安装

#### 1.基础镜像

```shell
# 拉取镜像
docker pull mysql:5.7.31
docker pull openjdk:8-alpine3.9

# 备份镜像
docker save mysql:5.7.31 -o mysql.5.7.31.tar
docker save openjdk:8-alpine3.9 -o openjdk8.tar

# 导入镜像
docker load -i mysql.5.7.31.tar
docker load -i openjdk8.tar
```

#### 2.运行镜像

```shell
# 运行镜像
docker run -itd --name mysql --restart always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.7.31

# 查看启动日志
docker logs -f mysql
```

使用Navicat测试连接，连接上后执行下列sql脚本【tb_article.sql】：

```sql
CREATE DATABASE IF NOT EXISTS blog DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
USE blog;

DROP TABLE IF EXISTS `tb_article`;
CREATE TABLE `tb_article` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(100) NOT NULL COMMENT '文章标题',
  `content` longtext COMMENT '文章具体内容',
  `created` date NOT NULL COMMENT '发表时间',
  `modified` date DEFAULT NULL COMMENT '修改时间',
  `categories` varchar(200) DEFAULT '默认分类' COMMENT '文章分类',
  `tags` varchar(200) DEFAULT NULL COMMENT '文章标签',
  `allow_comment` tinyint(1) NOT NULL DEFAULT '1' COMMENT '是否允许评论',
  `thumbnail` varchar(200) DEFAULT NULL COMMENT '文章缩略图',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;

INSERT INTO `tb_article` VALUES ('1', '2019新版Java学习路线图', 'Java学习路线图具体内容具体内容具体内容具体内容具体内容具体内容具体内容', '2019-10-10', null, '默认分类', '‘2019,Java,学习路线图', '1', null);
INSERT INTO `tb_article` VALUES ('2', '2019新版Python学习线路图', '据悉，Python已经入驻小学生教材，未来不学Python不仅知识会脱节，可能与小朋友都没有了共同话题~~所以，从今天起不要再找借口，不要再说想学Python却没有资源，赶快行动起来，Python等你来探索', '2019-10-10', null, '默认分类', '‘2019,Java,学习路线图', '1', null);
INSERT INTO `tb_article` VALUES ('3', 'JDK 8——Lambda表达式介绍', ' Lambda表达式是JDK 8中一个重要的新特性，它使用一个清晰简洁的表达式来表达一个接口，同时Lambda表达式也简化了对集合以及数组数据的遍历、过滤和提取等操作。下面，本篇文章就对Lambda表达式进行简要介绍，并进行演示说明', '2019-10-10', null, '默认分类', '‘2019,Java,学习路线图', '1', null);
INSERT INTO `tb_article` VALUES ('4', '函数式接口', '虽然Lambda表达式可以实现匿名内部类的功能，但在使用时却有一个局限，即接口中有且只有一个抽象方法时才能使用Lamdba表达式代替匿名内部类。这是因为Lamdba表达式是基于函数式接口实现的，所谓函数式接口是指有且仅有一个抽象方法的接口，Lambda表达式就是Java中函数式编程的体现，只有确保接口中有且仅有一个抽象方法，Lambda表达式才能顺利地推导出所实现的这个接口中的方法', '2019-10-10', null, '默认分类', '‘2019,Java,学习路线图', '1', null);
INSERT INTO `tb_article` VALUES ('5', '虚拟化容器技术——Docker运行机制介绍', 'Docker是一个开源的应用容器引擎，它基于go语言开发，并遵从Apache2.0开源协议。使用Docker可以让开发者封装他们的应用以及依赖包到一个可移植的容器中，然后发布到任意的Linux机器上，也可以实现虚拟化。Docker容器完全使用沙箱机制，相互之间不会有任何接口，这保证了容器之间的安全性', '2019-10-10', null, '默认分类', '‘2019,Java,学习路线图', '1', null);
```

脚本运行完后，修改本地应用数据库连接的ip和端口号，启动本地项目测试是否可以显示数据库数据。

#### 3.导入数据库

1.查看初始化目录：在用docker创建mysql容器的时，我们期望容器启动后数据库和表已经自动建好，初始化数据也已自动录入。一般都将sql文件放置在 `/docker-entrypoint-initdb.d` 目录中。

```shell
[root@k8s-master01 data]# docker ps | grep mysql
8257c108804b   mysql:5.7.31           "docker-entrypoint.s…"   13 minutes ago   Up 13 minutes   0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
[root@k8s-master01 data]# docker exec -it mysql bash
root@8257c108804b:/# whoami
root
root@8257c108804b:/# date
Thu Dec 24 16:06:44 UTC 2020
root@8257c108804b:/# ls
bin   dev			  entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint-initdb.d  etc		 lib   media  opt  root  sbin  sys  usr
root@8257c108804b:/# exit
exit
```

2.删除测试的mysql容器：

```shell
# 不推荐下面强制删除
[root@k8s-master01 data]# docker rm -f mysql 
mysql

docker stop mysql
docker rm mysql

mkdir -p /data/mysqldocker
cd /data/mysqldocker
```

3.编写Dockerfile文件，使用docker部署时，需要编写my.cnf来防止乱码；若使用k8s部署时，不需要指定，在service配置文件中可设置字符编码参数来防止乱码。

先编写my.cnf文件，避免乱码：

```shell
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
collation-server=utf8_general_ci
character-set-server=utf8
init-connect='SET NAMES utf8'
```

Dockerfile文件：将tb_article.sql文件复制到容器根目录下docker-entrypoint-initdb.d的文件夹内。

```dockerfile
FROM mysql:5.7.31
# 作者信息
MAINTAINER mysql from date UTC by Asia/Shanghai "zlg@fish.com"
ENV TZ Asia/Shanghai
COPY tb_article.sql /docker-entrypoint-initdb.d
COPY my.cnf /etc/mysql/conf.d/mysqlutf8.cnf
CMD ["mysqld", "--character-set-server=utf8", "--collation-server=utf8_unicode_ci"]
```

#### 4.自定义mysql镜像

将 tb_article.sql 和 Dockerfile文件上传到 /data/mysqldocker 目录中，制作镜像：

```shell
[root@k8s-master01 data]# cd mysqldocker/
[root@k8s-master01 mysqldocker]# ls
Dockerfile  tb_article.sql

# 构建mysql镜像
docker build --rm -t lagou/mysql:5.7 .

docker images

# 备份镜像，k8s作业中需要用到镜像
docker save lagou/mysql:5.7 -o lagou.mysql.5.7.tar
# 将镜像上传windows系统备份
sz lagou.mysql.5.7.tar
```

运行镜像：

```shell
docker run -itd --name mysql --restart always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root lagou/mysql:5.7

docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root  lagou/mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
# 生效
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d lagou/mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

docker logs -f mysql
# 在mysql容器启动日志中可以看到执行了对应的 tb_article.sql 脚本
2020-12-25 00:21:17+08:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/tb_article.sql
```

使用Navicat客户端进行测试连接，后更改本地项目数据库连接的ip和端口，启动本地项目测试是否显示数据。

删除测试容器和镜像：

```shell
# 强制删除mysql容器
docker rm -f mysql

docker stop mysql
docker rm mysql

# 删除镜像
docker rmi lagou/mysql:5.7
```

### SpringBoot项目部署

#### 1.项目打包

application.yml

```properties
server.port=8080
# mysql数据库配置,需要指定时区serverTimezone
spring.datasource.url=jdbc:mysql://192.168.91.104:3306/blog?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
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
```

启动本地项目，显示数据成功后，将项目打包。

#### 2.构建应用镜像

将打好的应用包上传到 /data/dockerdemo 目录下。

![image-20201226014851337](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226014851337.png)

编写Dockerfile文件：

```dockerfile
FROM openjdk:8-alpine3.9
# 作者信息
MAINTAINER zlg Docker springboot "zlg@fish.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories

# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai

COPY spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
```

构建镜像并运行：

```shell
# 构建镜像
docker build --rm -t lagou/dockerdemo:1.0 .
# 查看镜像
docker images

# 运行镜像
docker run -itd --name dockerdemo -p 8080:8080 lagou/dockerdemo:1.0
# 查看启动日志
docker logs -f dockerdemo

# 可以将镜像镜像打包到本地
docker save lagou/dockerdemo:1.0 -o lagou.dockerdemo.tar
docker save lagou/mysql:5.7 -o lagou.mysql.5.7.tar
```

启动完成后可以在浏览器上访问：http://192.168.91.104:8080 来测试应用镜像是否运行成功。

### K8S部分

#### 1.基础镜像

每台服务器上都需要下面的基础镜像。

Master节点拉取镜像：

```shell
# nfs动态存储
docker pull vbouchaud/nfs-client-provisioner:v3.1.1
# 测试nfs动态存储是否成功
docker pull nginx:1.19.3-alpine
# dockerdemo项目自定义mysql镜像
docker pull lagou/mysql:5.7
# 微服项目需要的基础镜像
docker pull openjdk:8-alpine3.9
```

传输或导入镜像到Node节点：

```shell
# 备份镜像
docker save vbouchaud/nfs-client-provisioner:v3.1.1 -o vbouchaud.nfs.client.provisioner.3.1.1.tar
docker save nginx:1.19.3-alpine -o nginx.alpine.1.19.3.tar
docker save openjdk:8-alpine3.9 -o openjdk.alpine.8.tar
docker save lagou.mysql:5.7 -o lagou.mysql.5.7.tar

# 复制本地镜像到Node节点
scp lagou.mysql.5.7.tar root@192.168.91.106:/data/

# Node节点上执行下面语句，导入基础镜像
docker load -i lagou.mysql.5.7.tar
```

#### 2.NFS服务

| 主机名       | 功能描述                                       | IP地址         |
| ------------ | ---------------------------------------------- | -------------- |
| k8s-master01 | 1.k8s集群master节点。<br/>2.NFS服务-server端。 | 192.168.91.104 |
| k8s-node01   | 工作节点。安装NFS服务。                        | 192.168.91.106 |

安装NFS服务：

```shell
yum install -y nfs-utils rpcbind

# 在master-156节点创建目录
mkdir -p /nfs
chmod 777 /nfs
# 更改归属组与用户
chown -R nfsnobody:nfsnobody /nfs
# 配置共享目录
echo "/nfs *(insecure,rw,sync,no_root_squash)" > /etc/exports
# 创建mysql共享目录
mkdir -p /nfs/mysql
# 所有节点设置启动服务
systemctl start rpcbind && systemctl start nfs
# 所有节点设置开启启动
systemctl enable rpcbind && systemctl enable nfs

# master节点检查配置是否生效
exportfs
# worker节点检查配置是否生效
showmount -e 192.168.91.104
```

master节点安装NFS服务：

![image-20201225233129404](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201225233129404.png)

worker节点安装NFS服务：

![image-20201225233227878](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201225233227878.png)

测试NFS挂载：

在 /data/nginx 目录下编写 nginx.yml 文件：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vol-nfs
  namespace: default
spec:
  volumes:
    - name: html
      nfs:
        path: /nfs
        server: 192.168.91.104
  containers:
    - name: myapp
      image: nginx:1.19.3-alpine
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html/
```

测试pod：使用模板创建名为vol-nfs的Pod

```shell
# 使用模板创建名为vol-nfs的Pod
kubectl apply -f nginx.yml

# 查看Pod详细信息
kubectl get pods -o wide

# 发送nginx请求
curl 10.81.85.203
# 无法访问nginx首页，因为nfs共享目录中没有indxe.html页面。在nfs服务共享目录中新建index.html页面
echo "111111" > /nfs/index.html
# 需要等待片刻才能正常访问到index.html的页面信息
curl 10.81.85.203

# 卸载pod
kubectl delete -f nginx.yml
```

![image-20201225234353893](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201225234353893.png)

#### 3.MySQL相关模板文件

编写下列4个yaml模板文件来部署MySQL服务。可以使用IDEA编写后上传到Master节点指定目录下。

![image-20201226021248313](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226021248313.png)

##### 3.1 RBAC

mysqlrbac.yml清单：

```yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default      #替换成要部署NFS Provisioner的namespace
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default      #替换成要部署NFS Provisioner的namespace
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

##### 3.2 storageClass

mysqlstorageclass.yml模板：

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    #设置升级策略为删除再创建(默认为滚动更新)
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          #由于quay.io仓库部分镜像国内无法下载，所以替换为其他镜像地址
          image: vbouchaud/nfs-client-provisioner:v3.1.1
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: nfs-client   #nfs-provisioner的名称，以后设置的storageclass要和这个保持一致
            - name: NFS_SERVER
              value: 192.168.91.104 #NFS服务器地址，与volumes.nfs.servers保持一致
            - name: NFS_PATH
              value: /nfs/mysql #NFS服务共享目录地址，与volumes.nfs.path保持一致
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.91.104    #NFS服务器地址，与spec.containers.env.value保持一致
            path: /nfs/mysql          #NFS服务器目录，与spec.containers.env.value保持一致。使用NFS4版本进行多级目录挂载

---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage-mysql
  annotations:
    storageclass.kubernetes.io/is-default-class: "true" #设置为默认的storageclass
#动态卷分配者名称，必须和创建的"provisioner"变量中设置的name一致
provisioner: nfs-client
parameters:
  archiveOnDelete: "true"     #设置为"false"时删除PVC不会保留数据,"true"则保留数据
mountOptions:
  - hard                      #指定为硬挂载方式
  - nfsvers=4                 #指定NFS版本，这个需要根据 NFS Server 版本号设置
```

##### 3.3 PVC

mysqlpvc.yml模板：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  # pvc名称
  name: mysqlbpvc
spec:
  # 使用的存储类
  storageClassName: nfs-storage-mysql
  # 读写权限
  accessModes:
    - ReadWriteOnce
  # 定义容量
  resources:
    requests:
      storage: 5Gi
```

##### 3.4 services

mysqlservice.yml模板：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deploy
  labels:
    app: mysql-deploy
spec:
  replicas: 1
  template:
    metadata:
      name: mysql-deploy
      labels:
        app: mysql-deploy
    spec:
      containers:
        - name: mysql-deploy
          image: lagou/mysql:5.7
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              #这是mysqlroot用户的密码
              value: root
            - name: TZ
              value: Asia/Shanghai
          args:
            - "--character-set-server=utf8mb4"
            - "--collation-server=utf8mb4_unicode_ci"
          volumeMounts:
            - mountPath: /var/lib/mysql #容器内的挂载目录
              name: volume-mysql
      restartPolicy: Always
      volumes:
        - name: volume-mysql
          persistentVolumeClaim:
            claimName: mysqlbpvc
  selector:
    matchLabels:
      app: mysql-deploy
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-svc
spec:
  selector:
    # 标签选择必须是template.labels.app
    app: mysql-deploy
  ports:
    - port: 3306  # 对集群内其它服务暴露端口号
      targetPort: 3306  # 容器端口号
      nodePort: 30036  # 集群外其它服务暴露端口号
  type: NodePort
```

#### 4.部署MySQL

在4个yaml模板文件目录下部署mysql服务：

```shell
# 部署服务
kubectl apply -f .
```

![image-20201226001422453](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226001422453.png)

查看创建的Pod：下图mysql服务已部署成功。

![image-20201226001743083](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226001743083.png)

部署成功后，可以使用Navicat软件进行连接，查看脚本数据是否导入集群。

其它操作：

```shell
# 删除服务
kubectl delete -f .

# 获取动态PV
kubectl get pv
```

#### 5.将本地项目部署到K8S

本地测试，调整application.yml的数据库url连接地址，浏览器上访问是否正常。将项目打包上传到K8S集群master节点。制作好镜像后分发到集群每一个节点。

将项目的application.yml文件url地址修改为**部署的mysql服务名称**，**端口号为pod容器端口号**。其余部分不用调整。

```properties
spring.datasource.url=jdbc:mysql://mysql-svc:3306/blog?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
```

![image-20201226022809510](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226022809510.png)

修改好后打包项目并上传到Master节点指定目录下。编写Dockerfile文件：

![image-20201226023938331](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226023938331.png)

```dockerfile
FROM openjdk:8-alpine3.9
# 作者信息
MAINTAINER zlg Docker springboot "zlg@fish.com"
# 修改源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories

# 安装需要的软件，解决时区问题
RUN apk --update add curl bash tzdata && \
    rm -rf /var/cache/apk/*
#修改镜像为东八区时间
ENV TZ Asia/Shanghai

COPY spring-boot-thymeleaf-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
```

制作项目镜像并分发镜像到其它节点：

```shell
# 构建镜像
docker build --rm -t lagou/k8sdemo:1.0 .

# 分发镜像到其它节点
docker save lagou/k8sdemo:1.0 -o blog.k8sdemo.tar
scp blog.k8sdemo.tar root@192.168.91.106:/data/

# 106 worker节点导入镜像
docker load -i blog.k8sdemo.tar
```

部署应用项目：编写k8sdemo.yml文件模板创建Pod、Deployment和Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-demo
  labels:
    app: k8s-demo
spec:
  replicas: 1
  template:
    metadata:
      name: k8s-demo
      labels:
        app: k8s-demo
    spec:
      containers:
        - name: k8s-demo
          image: lagou/k8sdemo:1.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
      restartPolicy: Always
  selector:
    matchLabels:
      app: k8s-demo
---
apiVersion: v1
kind: Service
metadata:
  name: fish-svc
spec:
  selector:
    # 标签选择必须是template.labels.app
    app: k8s-demo
  ports:
    - port: 8080  # 对集群内其它服务暴露端口号
      targetPort: 8080  # 容器端口号
      nodePort: 30080  # 集群外其它服务暴露端口号
  type: NodePort
```

部署项目应用到k8s集群：

```shell
kubectl apply -f k8sdemo.yml
```

![image-20201226024800337](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226024800337.png)

如显示上图信息，则表示应用部署到k8s集群成功。使用浏览器测试项目：ip为服务器IP，端口为service文件中配置的**nodePort端口号**（集群外其它服务暴露端口号）。

Master节点： http://192.168.91.104:30080

![image-20201226024956429](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226024956429.png)

Worker节点： http://192.168.91.106:30080

![image-20201226025024555](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201226025024555.png)





