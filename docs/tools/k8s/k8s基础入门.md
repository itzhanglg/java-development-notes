## Kubernetes安装与配置

### 一.Centos7安装与配置

#### 1.Centos7安装

1.centos下载地址：推荐大家使用centos7.6以上版本。 [http://mirrors.aliyun.com/centos/7/isos/x86_64/](http://mirrors.aliyun.com/centos/7/isos/x86_64/)

![image-20201224004527513](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201224004527513.png)

查看centos系统版本命令：

```shell
cat /etc/centos-release
```

![image-20201223202659120](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223202659120.png)

2.配置阿里云yum源：

```shell
# 1.下载安装wget
yum install -y wget

# 2.备份默认的yum
mv /etc/yum.repos.d /etc/yum.repos.d.backup

# 3.设置新的yum目录
mkdir -p /etc/yum.repos.d

# 4.下载阿里yum配置到该目录中，选择对应版本
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 5.更新epel源为阿里云epel源
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

# 6.重建缓存
yum clean all
yum makecache

# 7.看一下yum仓库有多少包
#yum repolist
yum update
```

3.升级系统内核：

```shell
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install -y kernel-lt
grep initrd16 /boot/grub2/grub.cfg
grub2-set-default 0

reboot
```

![image-20201223202549313](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223202549313.png)

4.查看centos系统相关信息：

```shell
# 查看centos系统内核命令
uname -r
uname -a

# 查看CPU命令
lscpu

# 查看内存命令
free
free -h

# 查看硬盘信息
fdisk -l
```

![image-20201223203256267](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223203256267.png)

![image-20201223203515562](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223203515562.png)

#### 2.Centos7系统配置

1.关闭防火墙

```shell
systemctl stop firewalld
systemctl disable firewalld
```

2.关闭selinux

```shell
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
setenforce 0
```

3.网桥过滤

```shell
vi /etc/sysctl.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
net.ipv4.ip_forward=1
net.ipv4.ip_forward_use_pmtu = 0

# 生效命令
sysctl --system
# 查看效果
sysctl -a|grep "ip_forward"
```

![image-20201223203734068](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223203734068.png)

4.开启IPVS

```shell
# 安装IPVS
yum -y install ipset ipvsdm

# 编译ipvs.modules文件
vi /etc/sysconfig/modules/ipvs.modules

#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4

# 赋予权限并执行
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules &&lsmod | grep -e ip_vs -enf_conntrack_ipv4

# 重启电脑，检查是否生效
reboot
lsmod | grep ip_vs_rr
```

![image-20201223204006024](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223204006024.png)

![image-20201223211142770](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223211142770.png)

5.同步时间

```shell
# 安装软件
yum -y install ntpdate

# 向阿里云服务器同步时间
ntpdate time1.aliyun.com

# 删除本地时间并设置时区为上海
rm -rf /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 查看时间
date -R || date
```

![image-20201223205213692](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223205213692.png)

6.命令补全

```shell
# 安装bash-completion
yum -y install bash-completion bash-completion-extras

# 使用bash-completion
source /etc/profile.d/bash_completion.sh
```

7.关闭swap分区

```shell
# 临时关闭
swapoff -a

#永久关闭
vi /etc/fstab

#将文件中的/dev/mapper/centos-swap这行代码注释掉
#/dev/mapper/centos-swap swap swap defaults 0 0

# 确认swap已经关闭：若swap行都显示 0 则表示关闭成功
free -m
```

![image-20201223205651205](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223205651205.png)

8.hosts配置

```shell
vi /etc/hosts

#文件内容如下
#192.168.91.104 k8s-master01
#192.168.91.106 k8s-node01

cat <<EOF >>/etc/hosts
192.168.91.104 k8s-master01
192.168.91.106 k8s-node01
EOF

bash
```

![image-20201223210230652](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223210230652.png)

### 二.Docker安装与配置

开发者平台官网地址：可以参考阿里云官网提供的docker安装教程进行安装。 [https://developer.aliyun.com/article/110806](https://developer.aliyun.com/article/110806)

![image-20201224011231017](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201224011231017.png)

1.准备工作：

```shell
# 安装docker前置条件
yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加软件源信息
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新
yum makecache fast
```

2.安装docker最新版本：

```shell
# 查看docker更新版本
yum list docker-ce --showduplicates | sort -r

# 安装docker最新版本
yum -y install docker-ce
# 安装指定版本
#yum -y install docker-ce-18.09.8

# 开启docker服务
systemctl start docker
systemctl status docker

# 查看docker版本
docker version
```

![image-20201223212804161](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223212804161.png)

3.安装阿里云镜像加速器： [https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

```shell
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": ["自己的阿里云镜像加速地址"]
}
EOF
systemctl daemon-reload
systemctl restart docker

# 设置docker开启启动服务
systemctl enable docker
```

![image-20201223213549461](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223213549461.png)

4.修改Cgroup Driver：

```shell
# 修改daemon.json，新增
"exec-opts": ["native.cgroupdriver=systemd"]

# 重启docker服务
systemctl daemon-reload
systemctl restart docker

# 查看修改后状态
docker info | grep Cgroup
```

![image-20201223213642503](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223213642503.png)

> 修改cgroupdriver是为了消除安装k8s集群时的告警：
> [WARNING IsDockerSystemdCheck]:
> detected “cgroupfs” as the Docker cgroup driver. The recommended driver is “systemd”.
> Please follow the guide at https://kubernetes.io/docs/setup/cri/......

5.docker初始化完成后将虚拟机进行关机，然后进行拍摄快照【1.docker初始化完成】，方便后期异常时进行恢复。

![image-20201224012858593](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201224012858593.png)

### 三.使用kubeadm快速安装k8s

| 软件 | kubeadm                          | kubelet                                                   | kubectl                          | docker-ce             |
| ---- | -------------------------------- | --------------------------------------------------------- | -------------------------------- | --------------------- |
| 版本 | 初始化集群管理，集群版本：1.17.5 | 用于接收api-server指令，对pod生命周期进行管理版本：1.17.5 | 集群命令行管理，工具版本：1.17.5 | 推荐使用版本：19.03.8 |

#### 1.安装yum源

```shell
# 新建repo文件
vi /etc/yum.repos.d/kubernates.repo

[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
       
# 更新缓存
yum clean all
yum -y makecache

# 验证源是否可用
yum list | grep kubeadm
#如果提示要验证yum-key.gpg是否可用，输入y。查找到kubeadm。显示版本
```

![image-20201223215040666](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223215040666.png)

![image-20201223215302081](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223215302081.png)

#### 2.安装k8s

```shell
# 查看k8s版本
yum list kubelet --showduplicates | sort -r

# 安装k8s-1.17.5
yum install -y kubelet-1.17.5 kubeadm-1.17.5 kubectl-1.17.5
```

![image-20201223215608932](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223215608932.png)

#### 3.设置kubelet

```shell
# 增加配置信息
# 如果不配置kubelet，可能会导致K8S集群无法启动。为实现docker使用的cgroupdriver与kubelet使用的cgroup的一致性
vi /etc/sysconfig/kubelet

KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"

# 设置开机启动
systemctl enable kubelet
```

![image-20201223215915611](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223215915611.png)

### 四.初始化镜像

如果是第一次安装k8s，手里没有备份好的镜像，可以执行如下操作。

#### 1.下载镜像

```shell
# 查看安装集群需要的镜像
kubeadm config images list

k8s.gcr.io/kube-apiserver:v1.17.16
k8s.gcr.io/kube-controller-manager:v1.17.16
k8s.gcr.io/kube-scheduler:v1.17.16
k8s.gcr.io/kube-proxy:v1.17.16
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.5
```

![image-20201223221240880](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223221240880.png)

编写执行脚本：

```shell
mkdir -p /data
cd /data
vi images.sh
```

```shell
#!/bin/bash
# 下面的镜像应该去除"k8s.gcr.io"的前缀，版本换成kubeadm config images list命令获取到的版本
images=(
    kube-apiserver:v1.17.5
    kube-controller-manager:v1.17.5
    kube-scheduler:v1.17.5
    kube-proxy:v1.17.5
    pause:3.1
    etcd:3.4.3-0
    coredns:1.6.5
)
for imageName in ${images[@]} ;
do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```

执行脚本：

```shell
# 给脚本授权
chmod +x images.sh

# 执行脚本
./images.sh
```

查看下载的镜像：

```shell
[root@node104 data]# docker images
REPOSITORY                           TAG       IMAGE ID       CREATED         SIZE
k8s.gcr.io/kube-proxy                v1.17.5   e13db435247d   8 months ago    116MB
k8s.gcr.io/kube-controller-manager   v1.17.5   fe3d691efbf3   8 months ago    161MB
k8s.gcr.io/kube-apiserver            v1.17.5   f640481f6db3   8 months ago    171MB
k8s.gcr.io/kube-scheduler            v1.17.5   f648efaff966   8 months ago    94.4MB
k8s.gcr.io/coredns                   1.6.5     70f311871ae1   13 months ago   41.6MB
k8s.gcr.io/etcd                      3.4.3-0   303ce5db0e90   14 months ago   288MB
k8s.gcr.io/pause                     3.1       da86e6ba6ca1   3 years ago     742kB
```

![image-20201223222114110](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223222114110.png)

#### 2.备份镜像

idea的列编辑模式：alt+鼠标左键。

保存master节点所需要的镜像：

```shell
docker save -o k8s.1.17.5.tar \
k8s.gcr.io/kube-proxy:v1.17.5 \
k8s.gcr.io/kube-apiserver:v1.17.5 \
k8s.gcr.io/kube-controller-manager:v1.17.5 \
k8s.gcr.io/kube-scheduler:v1.17.5 \
k8s.gcr.io/coredns:1.6.5 \
k8s.gcr.io/etcd:3.4.3-0 \
k8s.gcr.io/pause:3.1 \
```

![image-20201223222706946](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223222706946.png)

保存node节点所需要的镜像：

```shell
docker save -o k8s.1.17.5.node.tar \
k8s.gcr.io/kube-proxy:v1.17.5 \
k8s.gcr.io/pause:3.1 \
```

![image-20201223222922065](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223222922065.png)

#### 3.导入镜像

```shell
# 导入master节点镜像tar包，master节点需要全部镜像
docker load -i k8s.1.17.5.tar

# 导入node节点镜像tar包，node节点需要 kube-proxy:v1.17.5和pause:3.1 镜像
docker load -i k8s.1.17.5.node.tar
```

### 五.初始化k8s集群

#### 1.下载calico所需要的镜像

官网下载地址：[https://docs.projectcalico.org/v3.14/manifests/calico.yaml](https://docs.projectcalico.org/v3.14/manifests/calico.yaml) ；

github地址：[https://github.com/projectcalico/calico](https://github.com/projectcalico/calico) 。

下载 calico.yaml 文件并上传到 master 节点，文件中搜索 image，可以发现文件中所需要下面4个镜像。

```shell
# 镜像下载
docker pull calico/cni:v3.14.2
docker pull calico/pod2daemon-flexvol:v3.14.2
docker pull calico/node:v3.14.2
docker pull calico/kube-controllers:v3.14.2
```

下载所需镜像后，查看镜像：

```shell
[root@node104 ~]# docker images
REPOSITORY                           TAG       IMAGE ID       CREATED         SIZE
calico/node                          v3.14.2   780a7bc34ed2   5 months ago    262MB
calico/pod2daemon-flexvol            v3.14.2   9dfa8f25b51c   5 months ago    22.8MB
calico/cni                           v3.14.2   e6189009f081   5 months ago    119MB
calico/kube-controllers              v3.14.2   4815e4106d26   5 months ago    52.8MB
k8s.gcr.io/kube-proxy                v1.17.5   e13db435247d   8 months ago    116MB
k8s.gcr.io/kube-apiserver            v1.17.5   f640481f6db3   8 months ago    171MB
k8s.gcr.io/kube-controller-manager   v1.17.5   fe3d691efbf3   8 months ago    161MB
k8s.gcr.io/kube-scheduler            v1.17.5   f648efaff966   8 months ago    94.4MB
k8s.gcr.io/coredns                   1.6.5     70f311871ae1   13 months ago   41.6MB
k8s.gcr.io/etcd                      3.4.3-0   303ce5db0e90   14 months ago   288MB
k8s.gcr.io/pause                     3.1       da86e6ba6ca1   3 years ago     742kB
[root@node104 ~]# 
```

![image-20201223231348588](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223231348588.png)

备份镜像：

```shell
docker save -o calico.3.14.2.tar \
calico/node:v3.14.2 \
calico/pod2daemon-flexvol:v3.14.2 \
calico/cni:v3.14.2 \
calico/kube-controllers:v3.14.2 \
```

![image-20201223232517078](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223232517078.png)

修改master主机名：

```shell
# 配置hostname
hostnamectl set-hostname k8s-master01
```

![image-20201223232857701](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223232857701.png)

#### 2.克隆当前节点

将当前节点关机，后进行拍摄快照【2.k8s集群初始化完成】。然后进行克隆。

以现有快照进行克隆：

![image-20201223233322212](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223233322212.png)

创建完整克隆：

![image-20201223233410647](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201223233410647.png)

克隆完成后，修改节点的hostname和ip：

```shell
# 配置hostname：
hostnamectl set-hostname node106
bash

# 配置ip地址
vim /etc/sysconfig/network-scripts/ifcfg-ens33
IPADDR="192.168.91.106"

# 重启网络
systemctl restart network 

# 查看ip是否修改完成
id addr
# 查看网络是否连接正常
ping www.baidu.com
# 重启
reboot
```

#### 3.配置k8s集群网络

初始化集群信息:calico网络：

```shell
kubeadm init --apiserver-advertise-address=192.168.91.104 --kubernetes-version v1.17.5 --service-cidr=10.1.0.0/16 --pod-network-cidr=10.81.0.0/16
```

![image-20201224002057350](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201224002057350.png)

执行配置命令：

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

node节点加入集群信息：

```shell
kubeadm join 192.168.91.104:6443 --token kyuzs8.y8821kjrk3djyo08 \
    --discovery-token-ca-cert-hash sha256:7365a454124635be00c64ff7fde8ba77959b20a96e6b3abb60de3c1aceeb6957
```

![image-20201224002517117](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201224002517117.png)

master上查看集群节点状态是否ready，此时节点状态都为NotReady：

![image-20201224003240233](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201224003240233.png)

master上应用 calico.yaml 文件：

![image-20201224003109519](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201224003109519.png)

再次查看集群状态信息，集群状态均正常，即集群网络配置成功。

![image-20201224003441433](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201224003441433.png)

#### 4.其它问题

kubectl命令自动补全：

```shell
echo "source <(kubectl completion bash)" >> ~/.bash_profile
source ~/.bash_profile
```

发送邮件问题：

```shell
# 在 bash 中设置当前 shell 的自动补全，要先安装 bash-completion 包。
echo "unset MAILCHECK">> /etc/profile
source /etc/profile

# 在你的 bash shell 中永久的添加自动补全
```

yum-key.gpg验证未通过：

```shell
wget https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
wget https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
rpm --import yum-key.gpg
rpm --import rpm-package-key.gpg
```

## Kubernetes集群实战

Mysql服务安装、springboot项目部署、浏览器客户端访问调用

（1）搭建mysql服务器，要求必须要挂载数据卷，挂载方式不限。可以使用deployment或者statefulSet方式部署

（2）将项目数据库导入mysql服务器。可以采用sqlyog、navicat。

（3）开发spirnboot项目。

（4）spirnboot项目打包测试。

（5）spirnboot项目制作成镜像。

（6）在k8s集群中spirnboot项目。

### 一.MySQL服务安装

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

### 二.SpringBoot项目部署

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

### 三.K8S部分

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



## Kubernetes之命令行



```shell
kubectl crate namespace lagou

kubectl get namespaces
kubectl get ns

kubectl delete namespaces lagou


kubectl get nodes
kubectl get pods
kubectl get pods --all-namespaces
kubectl get pod -A

kubectl get pod -o wide
```



```shell
docker pull tomcat:9.0.20-jre8-alpine
 # 创建一个pod和deployment
kubectl run tomcat-test --images=tomcat:9.0.20-jre8-alpine --port=8080

kubectl get pod
kubectl get deployment

# 删除pod
kubectl delete pod tomcat-test-asdfjaofij
# 删除deployment
kubectl delete deployments.apps tomcat-test
```



```shell
kubectl scale --replicas=10 deployment tomcat-test

# 8888:service对集群内其它应用暴露的端口号,NodePort(3000-3300)对集群外其它应用暴露的端口号
# 8080:pod内tomcat容器的端口
kubectl expose deployment tomcat-test --name=tomcat-svc --port=8888 --target-port=8080 --protocol=TCP --type=NodePort

kubectl get service/svc
kubectl get svc -o wide

kubectl get svc -n kube-system

kubectl describe pods tomcat-test-dfjalsfjsdfj-sdfao
```



```shell
kubectl exec -it tomcat-test-dfafasdfjlas-jfaoi sh

kubectl logs -f tomcat-test-dfafasdfjlas-jfaoi

kubectl delete pod podname --force--grace-period=0

```





## Kubernetes之资源文件



使用资源文件删除pod后,deployment不会自动再拉起一个pod。



