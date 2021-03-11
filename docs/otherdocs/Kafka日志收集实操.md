### 一.习题

使用Kafka做日志收集。

一.需要收集的信息：

1、用户ID（user_id）

2、时间（act_time）

3、操作（action，可以是：点击：click，收藏：job_collect，投简历：cv_send，上传简历：cv_upload）

4、对方企业编码（job_code）

二.工作流程

![image-20201206213149046](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201206213149046.png) 

1、HTML可以理解为拉勾的职位浏览页面

2、Nginx用于收集用户的点击数据流，记录日志access.log

3、将Nginx收集的日志数据发送到Kafka主题：tp_individual

三.架构：

HTML+Nginx+[ngx_kafka_module](https://github.com/brg-liuwei/ngx_kafka_module)+Kafka

ngx_kafka_module网址：https://github.com/brg-liuwei/ngx_kafka_module

注意问题：由于使用ngx_kafka_module，只能接收POST请求，同时一般Web服务器不会和数据收集的Nginx在同一个域名，会涉及到使用ajax发送请求的跨域问题，可以在nginx中配置跨域来解决。

四.实战步骤：

1、 安装Kafka

2、安装Nginx

3、配置ngx_kafka_module，注意跨域配置

4、开发HTML页面

### 二.实现步骤

1. 安装依赖

    ```shell
    # 在虚拟机192.168.91.109服务器上安装相关依赖和nginx模块依赖
    # 按照 git 和 gcc 依赖
    yum install wget git -y
    yum install gcc-c++ -y
    
    # 下载librdkafka
    git clone https://github.com/edenhill/librdkafka
    # 或者下载后上传，再解压
    tar -zxf librdkafka-1.5.2.tar.gz -C /opt/
    cd /opt
    mv librdkafka-1.5.2/ librdkafka
    cd librdkafka
    # 配置、编译、安装
    ./configure
    make
    sudo make install
    ```

2. 下载nginx

    ```shell
    # 下载nginx
    wget http://nginx.org/download/nginx-1.17.8.tar.gz
    # 解压
    tar -zxf nginx-1.17.8.tar.gz -C /opt
    cd /opt/nginx-1.17.8
    # 安装相关依赖
    yum install gcc zlib zlib-devel openssl openssl-devel pcre pcre-devel -y
    ```

3. 下载ngx_kafka_module

    ```shell
    # 下载ngx_kafka_module后再上传
    cd ~
    git clone https://github.com/brg-liuwei/ngx_kafka_module.git
    # 上传后解压
    tar -zxf ngx_kafka_module-0.9.1.tar.gz -C /opt/
    mv ngx_kafka_module-0.9.1/ ngx_kafka_module
    
    # 切换到nginx根目录下
    cd nginx-1.17.8
    ./configure --add-module=/opt/ngx_kafka_module
    make
    sudo make install
    ```

4. 配置nginx：nginx.conf

    ```shell
    # 切换到 nginx 安装的配置目录下
    cd /usr/local/nginx/conf
    # 编辑nginx.conf文件
    vim nginx.conf
    
    http {
    
        # some other configs
    
        kafka;
    
        kafka_broker_list 192.168.91.112:9092; # host:port ...
    
        server {
    		location = /log {
        		add_header 'Access-Control-Allow-Origin' $http_origin;
       		 	add_header 'Access-Control-Allow-Credentials' 'true';
       		 	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
       		 	add_header 'Access-Control-Allow-Headers' 'DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        		add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
        		if ($request_method = 'OPTIONS') {
          		  	add_header 'Access-Control-Max-Age' 1728000;
         		   	add_header 'Content-Type' 'text/plain; charset=utf-8';
         		   	add_header 'Content-Length' 0;
         		   	return 204;
       		 	}
       		 	kafka_topic tp_log_01;
    		}
    	}	
    }
    ```

5. 让操作系统加载模块：

    ```shell
    echo "/usr/local/lib" >> /etc/ld.so.conf
    ldconfig
    ```

6. 启动Kafka

    ```shell
    # 在 192.168.91.112 启动 zk 和 kafka，zk也可以单独部署到另一台服务器上
    # 启动zk
    zkServer.sh start
    # 启动kafka
    kafka-server-start.sh -daemon /opt/kafka_2.12-1.0.2/config/server.properties
    
    # 列出现有的主题
    kafka-topics.sh --zookeeper localhost:2181/mykafka --list
    # 创建主题
    kafka-topics.sh --zookeeper localhost:2181/mykafka --create --topic tp_log_01 --partitions 1 --replication-factor 1
    # 查看指定主题的详细信息
    [root@atzlg8 ~]# kafka-topics.sh --zookeeper localhost:2181/mykafka --describe --topic tp_log_01
    Topic:tp_log_01	PartitionCount:1	ReplicationFactor:1	Configs:
    	Topic: tp_log_01	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
    
    # 开启消费者
    [root@atzlg8 ~]# kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic tp_log_01 --from-beginning
    ```

7. 启动nginx：

    ```shell
    # 在 192.168.91.109 启动nginx
    /usr/local/nginx/sbin/nginx
    #在网页上访问 192.168.91.109:80 显示nginx欢迎页面
    ```

8. 测试：

    ```shell
    # 在192.168.91.109上测试发送 post 请求：http://localhost/log
    [root@atzlg5 ~]# curl localhost/log -d "hello ngx_kafka_module" -v
    * About to connect() to localhost port 80 (#0)
    *   Trying ::1...
    * 拒绝连接
    *   Trying 127.0.0.1...
    * Connected to localhost (127.0.0.1) port 80 (#0)
    > POST /log HTTP/1.1
    > User-Agent: curl/7.29.0
    > Host: localhost
    > Accept: */*
    > Content-Length: 22
    > Content-Type: application/x-www-form-urlencoded
    > 
    * upload completely sent off: 22 out of 22 bytes
    < HTTP/1.1 204 No Content
    < Server: nginx/1.17.8
    < Date: Sun, 06 Dec 2020 15:49:51 GMT
    < Connection: keep-alive
    < Access-Control-Allow-Credentials: true
    < Access-Control-Allow-Methods: GET, POST, OPTIONS
    < Access-Control-Allow-Headers: DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range
    < Access-Control-Expose-Headers: Content-Length,Content-Range
    < 
    * Connection #0 to host localhost left intact
    ```

9. 使用Idea的静态项目直接打开访问即可：

    <img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201207000134024.png" alt="image-20201207000134024" />

10. 效果

    ![image-20201207000319138](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201207000319138.png)













```sh
cd /opt/zookeeper-3.4.14/conf
vim zoo.cfg
dataDir=/var/fishleap/zookeeper/data
server.1=192.168.91.109:2881:3881
server.2=192.168.91.112:2881:3881


mkdir -p /var/fishleap/zookeeper/data
echo 1 > /var/fishleap/zookeeper/data/myid

export ZOOKEEPER_PREFIX=/opt/zookeeper-3.4.14
export PATH=$PATH:$ZOOKEEPER_PREFIX/bin
export ZOO_LOG_DIR=/var/fishleap/zookeeper/log

echo 2 > /var/fishleap/zookeeper/data/myid

listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://192.168.91.112:9092
zookeeper.connect=192.168.91.112:2181,192.168.91.109:2181/myKafka
log.dirs=/var/fishleap/kafka/kafka-logs

listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://192.168.91.109:9092
zookeeper.connect=192.168.91.112:2181,192.168.91.109:2181/myKafka
log.dirs=/var/fishleap/kafka/kafka-logs

kafka-server-start.sh /opt/kafka_2.12-1.0.2/config/server.properties
kafka-server-start.sh /opt/servers/kafka_2.12-1.0.2/config/server.properties


[2020-12-06 23:30:03,883] INFO Cluster ID = ZFfedpSfQ7q3fFqJkLnk8A (kafka.server.KafkaServer)
```

