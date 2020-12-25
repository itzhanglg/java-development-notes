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



