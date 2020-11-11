## 一.Zookeeper基本概念

分布式系统是同时跨越多个物理主机，独立运行的多个软件所组成的系统。分布式系统的协调工作就是通过某种方式，让某个节点的信息能够同步和共享。这依赖于服务进程之间的通信。通信方式有两种：

- 通过网络进行信息共享
- 通过共享存储

**Zookeeper是作为分布式系统的分布式协同服务**。Zookeeper对分布式系统的协调使用的是第二种方式，共享存储（存储和网络通信）。Zookeeper存储了任务的分配、完成情况等共享信息。每个分布式应用的节点就是组员，订阅这些共享信息。**当leader对某个节点的分工信息作出改变时，zookeeper会通知相关订阅的从节点获取自己最新的任务分配（从节点需要在关心的数据节点上设置观察点才能获取zookeeper的更新通知**）。**完成工作后会把完成情况存储到zookeeper。zookeeper会通知订阅该任务完成情况信息的leader。zookeeper是一个典型的分布式数据一致性zookeeper保证了分布式系统信息的一致性**。

![image-20200701073427244](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200701073427244.png)

### 1.基本概念

**集群角色**

- leader：领导者
- follower：跟随着
- observer：不参与Leader选举过程，不参与写操作的过半写成功策略，也就是降低了投票的压力。同时也保证了性能（读写操作）

**会话**

- 一个客服端连接是指客户端和服务端之间的一个TCP长连接，Zookeeper对外的服务端口默认是2181。

**数据节点**

- 机器节点：构成集群的机器
- 数据节点：指数据模型中的数据单元

Zookeeper将所有数据存储在内存中，数据模型是一棵树（ZNode Tree），每个ZNode都会保存自己的数据内容和一系列属性信息。

**版本**

Zookeeper会为每个ZNode维护一个Stat的数据结构，Stat记录了这个ZNode的三个数据版本，分别是version（当前ZNode版本）、cversion（当前ZNode子节点版本）、aversion（当前ZNode的ACL版本）。

**事件监听器（Watcher）**

在指定节点上注册一些Watcher，这些事件监听器触发时，Zookeeper服务端会将事件通知给感兴趣的客户端，这机制是Zookeeper实现分布式协调服务的重要特性。

**ACL**

ACL（Access Control Lists）权限控制策略。如下五种权限：

- CREATE：创建**子节点**的权限
- READ：获取节点数据和子节点列表的权限
- WRITE：更新节点数据的权限
- DELETE：删除**子节点**的权限
- ADMIN：设置节点ACL的权限

### 2.环境搭建

Zookeeper安装方式有三种,单机模式和集群模式以及伪集群模式.

- 单机模式：Zookeeper在一台服务器上运行，适合测试环境
- 集群模式：Zookeeper运行在一个集群上，适合生成环境，计算机群被称为一个“集合体”
- 伪集群模式：一台服务器上运行多个Zookeeper实例

环境搭建推荐：

- [【ZooKeeper系列】1.ZooKeeper单机版、伪集群和集群环境搭建](https://juejin.im/post/5df71960f265da33e228ff41#heading-6)
- [zookeeper安装及配置](https://blog.csdn.net/weixin_41558061/article/details/80597174?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

工具准备：Mac上工具 文件上传：ZenTermLite   虚拟机：Parallels Desktop   系统：CentOS7

```
// 伪集群环境搭建时
server.1=192.168.91.105:2881:3881
server.2=192.168.91.105:2882:3882
server.3=192.168.91.105:2883:3883
```

## 二.Zookeeper的基本使用

### 1.Zookeeper系统模型

在Zookeeper中数据信息被保存在一个个数据节点上,这些节点被称为ZNode。ZNode是zookeeper中最小数据单位，类似文件系统的层级树状结构。

![image-20200701074846031](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200701074846031.png)

#### 1.1 ZNode的类型

数据节点ZNode的节点类型分类：持久性节点（Persistent）、临时性节点（Ephemeral）、顺序性节点（Sequential）。

创建节点时可以通过组合生成以下四种节点类型：持久节点、持久顺序节点、临时节点、临时顺序节点。**不同类型的节点会有不同的生命周期**。

- 持久节点：节点被创建后会一直存在服务器，知道删除操作主动清除
- 持久顺序节点：创建节点时会在节点名后面加上一个数字后缀来表示顺序，和持久节点特性一样
- 临时节点：生命周期和客户端会话绑在一起，客户端会话结束，节点会被删除；不能创建子节点
- 临时顺序节点：有顺序的临时节点

在zookeeper中，事务是指能够改变zookeeper服务器状态的操作，称为事务操作和更新操作。一般包括数据节点的创建与删除、数据节点内容更新等操作。

每一次事务请求，zk都会为其分配一个全局唯一的事务id，用ZXID表示，通常是一个64位数字。每一个ZXID对应一次更新更新操作。

#### 1.2 ZNode的状态信息

![image-20200701074954030](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200701074954030.png)

整个ZNode节点内容包括两部分：节点数据内容和节点状态信息。quota是数据内容，其它属于状态信息。

- cZxid 就是 Create ZXID，表示节点被创建时的事务ID。
- ctime 就是 Create Time，表示节点创建时间。
- mZxid 就是 Modified ZXID，表示节点最后⼀次被修改时的事务ID。
- mtime 就是 Modified Time，表示节点最后⼀次被修改的时间。
- pZxid 表示该节点的⼦节点列表最后⼀次被修改时的事务 ID。只有⼦节点列表变更才会更新 pZxid，⼦节点内容变更不会更新。
- cversion 表示⼦节点的版本号。
- dataVersion 表示内容版本号。
- aclVersion 标识acl版本。
- ephemeralOwner 表示创建该临时节点时的会话 sessionID，如果是持久性节点那么值为 0。
- dataLength 表示数据⻓度。
- numChildren 表示直系⼦节点数。

#### 1.3 Watcher数据变更通知

Zookeeper使用Watcher机制实现分布式数据的发布/订阅功能。多个订阅者同时监听某一个主题对象，主题对象状态发生变化时，会通知所有的订阅者做出相应处理。

Zookeeper允许客户端向服务端注册一个Watcher监听，当服务端的指定事件触发了Watcher，就会向指定客服端发送一个事件通知。

![image-20200701075035071](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200701075035071.png)

Zookeeper的Watcher机制包括**客户端线程、客服端WatcherManager、Zookeeper服务器**三部分。

**具体流程**：客户端向zk服务器注册的同时会将Watcher对象存储在客户端的WatcherManager中，当Zookeeper服务器触发Watcher使劲按后回向客服端发送通知，客户端线程从WatcherManager中取出对应的Watcher对象来执行回调逻辑。

#### 1.4 ACL保障数据的安全

在Zookeeper中提供了一套完善的ACL（Access Control List）权限控制机制来保障数据的安全。

通常会使用“权限模式（scheme）：授权对象（id）：权限（permission）”来标志一个有效的ACL信息。

- **权限模式**用来确定权限验证中使用的检验策略
- **授权对象**指的是权限赋予的用户或一个指定实体
- **权限**指通过权限检查后可以被允许执行的操作

权限与授权对象的关系：

| 权限模式 | 授权对象                                                     |
| -------- | ------------------------------------------------------------ |
| IP       | 通常使用IP地址或IP段。例如：192.168.91.105（IP）或 192.168.91.1/24（网段） |
| Digest   | 自定义，通常是username：BASE64（SHA-1（username：password））进行加密再编码 |
| World    | 只有一个ID：anyone                                           |
| Super    | 超级用户                                                     |

权限分为五大类：CREATE（子节点）、DELETE（子节点）、READ、WRITE、ADMIN（对节点进行ACL设置）。简称为CDRWA。

### 2.命令行操作

对节点的增删改查常用命令：输入help后，回显示可用的Zookeeper命令

- create 【-s】【-e】path data acl：创建节点（顺序/临时） /zlg  123
- ls path：显示节点的所有直系子节点
- get path：显示节点的内容和属性信息
- ls2 path：显示节点的直系子节点列表和属性信息
- set path data 【version】：更新指定节点的数据内容，data表示更新的内容，version表示数据版本
- delete path 【version】：删除指定的节点，version表示数据版本（dataVersion），若删除节点存在子节点，就无法删除该节点，必须先删除子节点，再删除父节点

### 3.相关客户端api使用

有Zookeeper的原生API、ZkClient、Curator 三种使用方式。下面对这三种使用方式的API做下对比：

引入依赖：

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.14</version>
</dependency>

<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.2</version>
</dependency>

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.12.0</version>
</dependency>
```

原生部分API：

- 创建会话：new Zookeeper(connectString,sesssionTimeOut,Watcher)
- 创建节点：zookeeper.create(path,data,acl,createMode)
- 删除节点：zookeeper.delete(path,version) 
- 获取数据：zk.getData(path, watch, stat)
- 获取子节点列表：zooKeeper.getChildren(path, watch)
- 更新数据：zooKeeper.setData(path, data,version)

ZkClient客户端：

- 创建会话：new ZkClient(serverString)
- 创建节点：zkClient.createPersistent(String path, boolean createParents)
- 删除节点：zkClient.deleteRecursive(String path)
- 获取数据：zkClient.readData(path)
- 获取子节点列表：zkClient.getChildren(path)
- 更新数据：zkClient.writeData(path, object)
- 事件监听：
    - zkClient.subscribeChildChanges(path, new IZkChildListener() {...})
    - zkClient.subscribeDataChanges(path, new IZkDataListener() {...}

Curator客户端：

- 创建会话：public static CuratorFramework newClient(String connectString, int sessionTimeoutMs, int connectionTimeoutMs, RetryPolicy retryPolicy)
- 创建节点：client.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPa
    th(path,data);  data需要序列化 data = serializer.serialize(new ServerInfo(ip, port, 0L))
- 删除节点：client.delete().deletingChildrenIfNeeded().forPath(path)
- 获取数据：client.getData().forPath(path)得到的字节数组bytes需要反序列化; serializer.deserialize(ServerInfo.class, bytes);
- 获取子节点列表：client.getChildren().forPath(path)
- 更新数据：client.setData().forPath(path, data)

#### CuratorUtils工具类

推荐：[Zookeeper开源客户端Curator之事件监听详解](https://blog.csdn.net/wo541075754/article/details/70167841)

```java
/**
 * @author zlg
 * 封装Curator客户端相关Zookeeper API操作
 */
@Slf4j
public class CuratorUtils {
    // 基础睡眠时间
    private static final int BASE_SLEEP_TIME = 1000;
    // 重大重试次数
    private static final int MAX_RETRIES = 3;
    // server地址
    private static final String CONNECT_STRING = "127.0.0.1:2181";
    // 会话超时时间
    private static final int SESSION_TIMEOUT_MS = 5000;
    // 连接超时时间
    private static final int CONNECTION_TIMEOUT_MS = 30000;
    // 独立命名空间
    public static final String NAMESPACE = "mysql-config";
    // 服务地址,key是serviceName,value是子节点列表
    private static Map<String, List<String>> serviceAddressMap = new ConcurrentHashMap<>();
    // 注册中心
    private static Set<String> registeredPathSet = ConcurrentHashMap.newKeySet();
    // Zookeeper客户端
    public static CuratorFramework zkClient;

    static {
        zkClient = getZkClient();
    }

    /**
     * 1.使用Fluent风格创建Zk客户端
     * @return CuratorFramework Zk客户端
     */
    private static CuratorFramework getZkClient() {
        // 重试策略,重试三次,会增加重试之间的睡眠时间
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(BASE_SLEEP_TIME, MAX_RETRIES);
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(CONNECT_STRING)  // server地址
                .sessionTimeoutMs(SESSION_TIMEOUT_MS)     // 会话超时时间
                .connectionTimeoutMs(CONNECTION_TIMEOUT_MS)  // 连接超时时间
                .retryPolicy(retryPolicy)   // 重试策略
                .namespace(NAMESPACE)  // 独立命名空间
                .build();
        client.start();
        log.info("Zookeeper session established. ");
        System.out.println("Zookeeper session established. ");
        return client;
    }

    /**
     * 2.创建持久化节点。不同于临时节点，持久化节点不会因为客户端断开连接而被删除
     * @param path 创建节点的路径,即名称
     */
    public static void createPersistentNode(String path, String data) {
        try {
            if (registeredPathSet.contains(path) || zkClient.checkExists().forPath(path) != null) {
                log.info("持久化节点已经存在，节点为:[{}]", path);
                System.out.println("持久化节点已经存在，节点为:["+path+"]");
            } else {
                //eg: /zdy-rpc/com.fishleap.service.IUserService/127.0.0.1:8888
                zkClient.create().creatingParentsIfNeeded()
                        .withMode(CreateMode.PERSISTENT)
                        .forPath(path,data.getBytes());
                log.info("持久化节点创建成功，节点为:[{}]", path);
                System.out.println("持久化节点创建成功，节点为:["+path+"]");
            }
            // 将创建的节点信息保存到set中
            registeredPathSet.add(path);
        } catch (Exception e) {
            e.getMessage();
        }
    }

    /**
     * 创建临时节点,临时节点会因为客户端断开连接而被删除
     * @param path 创建节点的路径,即名称
     */
    public static void createEphemeralNode(String path) {
        try {
            if (registeredPathSet.contains(path) || zkClient.checkExists().forPath(path) != null) {
                log.info("临时节点已经存在，节点为:[{}]", path);
                System.out.println("临时节点已经存在，节点为:["+path+"]");
            } else {
                //eg: /zdy-rpc/com.fishleap.service.IUserService/127.0.0.1:8888
                zkClient.create().creatingParentsIfNeeded()
                        .withMode(CreateMode.EPHEMERAL)
                        .forPath(path);
                log.info("临时节点创建成功，节点为:[{}]", path);
                System.out.println("临时节点创建成功，节点为:["+path+"]");
            }
            // 将创建的节点信息保存到set中
            registeredPathSet.add(path);
        } catch (Exception e) {
            e.getMessage();
        }
    }

    /**
     * 3.获取某个字节下的子节点,也就是获取所有提供服务的生产者的地址
     * @param serviceName 服务对象接口名 eg:com.fishleap.service.IUserService
     * @return List<String> 指定字节下的所有子节点
     */
    public static List<String> getChildrenNodes(String serviceName) {
        // 判断map中是否有该serviceName的key
        if (serviceAddressMap.containsKey(serviceName)) {
            return serviceAddressMap.get(serviceName);
        }
        List<String> result = null;
        String servicePath = "/" + serviceName;
        try {
            result = zkClient.getChildren().forPath(servicePath);
            log.info("当前 {} 节点的子节点列表为 {}", servicePath, result);
            System.out.println("当前 "+servicePath+" 节点的子节点列表为 "+result);
            serviceAddressMap.put(serviceName, result);
            // 注册节点子节点监听
            registerWatcher(zkClient, serviceName);
        } catch (Exception e) {
            e.getMessage();
        }
        return result;
    }

    /**
     * 4.获取某一节点的数据信息
     * @param path 节点的路径,不带"/"
     * @return 返回节点数据字符串
     */
    public static String getNodeData(String path) {
        Stat stat = new Stat();
        byte[] bytes = new byte[0];
        try {
            bytes = zkClient.getData().storingStatIn(stat).forPath(path);
            // 注册节点监听
            registerWatcherNodeData(zkClient, path);
        } catch (Exception e) {
            e.getMessage();
        }
        return new String(new String(bytes));
    }

    /**
     * 修改指定节点的数据
     * @param path 节点路径
     * @param data 节点数据
     */
    public static void setNodeData(String path, String data) {
        try {
            Stat stat = zkClient.setData().withVersion(-1).forPath(path, data.getBytes());
            if (stat != null) {
                System.out.println("修改Zookeeper上数据库配置信息成功!");
            }
        } catch (Exception e) {
            e.getMessage();
        }
    }

    /**
     * 5.清空注册中心的数据
     */
    public static void clearRegistry() {
        // 遍历注册中心 path 集合
        registeredPathSet.stream().parallel().forEach(path -> {
            try {
                zkClient.delete().forPath(path);
            } catch (Exception e) {
                e.getMessage();
            }
        });
        log.info("服务端（Provider）所有注册的服务都被清空:[{}]", registeredPathSet.toString());
        System.out.println("服务端（Provider）所有注册的服务都被清空:["+registeredPathSet.toString()+"]");
    }


    /**
     * 对指定节点本身内容进行监听
     * @param zkClient zk客户端
     * @param path 节点路径
     */
    private static void registerWatcherNodeData(CuratorFramework zkClient, String path) {
        final NodeCache nodeCache = new NodeCache(zkClient, path);
        NodeCacheListener nodeCacheListener = new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                System.out.println("监听到数据库配置信息节点内容变化为：===>>>" +
                        new String(nodeCache.getCurrentData().getData()));

                String newNodeData = new String(nodeCache.getCurrentData().getData());
                Map map = (Map)JSON.parse(newNodeData);
                // 创建新的连接池
                DruidDataSource dataSource = (DruidDataSource)DruidDataSourceFactory.createDataSource(map);
                DruidUtils.setDataSource(dataSource);
                // 销毁旧的连接池
//                DruidUtils.getDataSource().close();

                // 测试数据库连接
                UserController.list();
            }
        };
        nodeCache.getListenable().addListener(nodeCacheListener);
        try {
            nodeCache.start();
        } catch (Exception e) {
            e.getMessage();
        }
    }

    /**
     * 注册监听指定节点的子节点列表
     * @param zkClient 客服端对象
     * @param serviceName 服务对象接口名 eg:com.fishleap.service.IUserService
     */
    private static void registerWatcher(CuratorFramework zkClient, String serviceName) {
        String servicePath = "/" + serviceName;
        PathChildrenCache pathChildrenCache = new PathChildrenCache(zkClient, servicePath, true);
        PathChildrenCacheListener pathChildrenCacheListener = new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent pathChildrenCacheEvent) throws Exception {
                List<String> serviceAddresses = curatorFramework.getChildren().forPath(servicePath);
                log.info("监听到 {} 节点的子节点列表变化为 {}", servicePath, serviceAddresses);
                System.out.println("监听到 "+servicePath+" 节点的子节点列表变化为 "+serviceAddresses);
                serviceAddressMap.put(serviceName, serviceAddresses);
            }
        };
        pathChildrenCache.getListenable().addListener(pathChildrenCacheListener);
        try {
            pathChildrenCache.start();
        } catch (Exception e) {
            e.getMessage();
        }
    }
}
```

### 三.zookeeper应用场景

利用 ZooKeeper 可以非常方便构建一系列分布式应用中都会涉及到的核心功能。

1. 数据发布/订阅
2. 负载均衡
3. 命名服务
4. 分布式协调/通知
5. 集群管理
6. Master 选举
7. 分布式锁
8. 分布式队列

多个开源项目中都应用到了 ZooKeeper，例如 HBase, Spark, Flink, Storm, Kafka, Dubbo 等等。

Zookeeper应用场景文章推荐：

- [ZooKeeper 的应用场景](https://zhuanlan.zhihu.com/p/59669985)
- [图解ZooKeeper的典型应用场景](https://www.javazhiyin.com/28435.html)
- [ZooKeeper应用场景及方案介绍](https://www.jianshu.com/p/2e970fe35c3f)
- [Zookeeper系列（6）-- Zookeeper的典型应用场景](https://blog.csdn.net/u013679744/article/details/79371022)
- [Zookeeper应用场景](https://xiaoxiami.gitbook.io/zookeeper/chapter1)

**下面主要看下Zookeeper在分布式锁上的应用场景**

分布式锁是控制分布式系统之间同步访问共享资源的一种方式。如果不同的系统或是同一个系统的不同主机之间共享了一个或一组资源,那么访问这些资源的时候,往往需要通过一些互斥手段来防止彼此之间的干扰,以保证一致性,在这种情况下,就需要使用分布式锁了。下面看下**Zookeeper如何实现排他锁和共享锁两类分布式锁**。

#### 1.排他锁

排他锁(Exclusive Locks,简称×锁) ,又称为写锁或独占锁,是一种基本的锁类型。如果事务T1对数据对象o1加上了排他锁,那么在整个加锁期间,只允许事务T1对o1进行读取和更新操作,其他任何事务都不能再对这个数据对象进行任何类型的操作——直到T1释放了排他锁。

排他锁的核心是如何保证当前有且仅有一个事务获,得锁,并且锁被释放后,所有正在等待获取锁的事务都能够被通知到。

Zookeeper实现排他锁步骤如下：

**1.定义锁**

Java并发编程中可以使用synchronized机制和ReentrantLock来定义锁。Zookeeper中需要定义一个**临时节点**来表示一个锁，如/exclusive_lock/lock节点可以被定义为一个锁。如图：

![image-20201112002600677](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112002600677.png)

**2.获取锁**

在需要获取排他锁时，**所有客户端**都试图在/exclusive_lock节点下通过调用create()接口**创建临时子节点**。谁创建成功了就表示谁获取了排他锁，同时所有**没有获取到锁**的客户端就需要在/exclusive_lock节点上**注册一个子节点变更的Watcher监听**，来监听子节点的变化情况。

**3.释放锁**

当获取锁的客户端发生宕机或正常执行完业务逻辑后，临时节点都会被移出，也就释放了锁。Zookeeper就会通知所有在/exclusive_lock节点上注册监听的客户端，客户端接收到通知后再次发起分布式锁获取。

![image-20201112003645536](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112003645536.png)

#### 2.共享锁

共享锁(Shared Locks,简称S锁) ,又称为读锁,同样是一种基本的锁类型。如果事务T1对数据对象o1加上了共享锁,那么当前事务只能对o1进行读取操作,其他事务也只能对这个数据对象加共享锁——直到该数据对象上的所有共享锁都被释放。

共享锁和排他锁最根本的区别在于：**加上排他锁后,数据对象只对一个事务可见,而加上共享锁后,数据对所有事务都可见**。

Zookeeper实现共享锁步骤如下：

**1.定义锁**

Zookeeper上通过定义一个**临时顺序节点**来代表一个共享锁。如/share_lock/host1-R-0000000001，如图所示：

![image-20201112004211314](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112004211314.png)

**2.获取锁**

在需要获取共享锁时,所有客户端都会到/shared lock这个节点下面创建一个临时顺序节点,如果当前是读请求,那么就创建例如/shared lock/host1-R-000000000节点;如果是写请求,那么就创建例如/sharedlock/host2-W-0000000002的节点。

判断读写顺序：通过Zookeeper来去确定分布式读写顺序，大致分为四步。

>1,创建完节点后,获取/shared-lock节点下所有子节点,并对该节点变更注册监听。
>
>2,确定自己的节点序号在所有子节点中的顺序。
>
>3,对于读请求:若没有比自己序号小的子节点或所有比自己序号小的子节点都是读请求,那么表明自己已经成功获取到共享锁,同时开始执行读取逻辑,若有写请求,则需要等待。对于写请求:若自己不是序号最小的子节点,那么需要等待。
>
>4,接收到Watcher通知后,重复步骤1

**3.释放锁**，流程与排他锁一致。

#### 3.羊群效应

上面的共享锁的实现大体能满足集群规模不是特别大的场景。当规模扩大之后，会出现什么问题呢？结合下图看下实际运行情况：

![image-20201112005701385](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112005701385.png)

可以看到，host1客户端移出自己的共享锁后，只对host2产生影响了，对其它机器没有任何作用。大量的Watcher通知和子节点列表获取两个操作会重复运行,这样不仅会对zookeeper服务器造成巨大的性能影响影响和网络开销,更为严重的是,如果同一时间有多个节点对应的客户端完成事务或是事务中断引起节点消失, ZooKeeper服务器就会在短时间内向其余客户端发送大量的事件通知,这就是所谓的羊群效应。

上面的分布式锁竞争过程,它的核心逻辑在于:判断自己是否是所有子节点中序号最小的。于时可以联想到**每个节点对应的客户端只需要关注比自己序号小的那个相关节点的变更情况**就可以了--而不需要关注全局的子列表变更情况。

#### 4.改进后的分布式锁实现

上面提到的共享锁实现,从整体思路上来说完全正确。这里主要的改动在于:**每个锁竞争者,只需要关注/sharedlock节点下序号比自己小的那个节点是否存在即可**,具体实现如下。

1. 客户端调用create接口常见类似于/shared_lock/[Hostname]-请求类型-序号的临时顺序节点。
2. 客户端调用getChildren接口获取所有已经创建的子节点列表(不注册任何Watcher)。
3. 如果无法获取共享锁,就调用exist接口来对比自己小的节点注册Watcher,对于读请求:向比自己序号小的最后一个写请求节点注册Watcher监听。对于写请求:向比自己序号小的最后一个节点注册Watcher监听。
4. 等待Watcher通知,继续进入步骤2。

![image-20201112010535009](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112010535009.png)

如同在多线程并发编程实践中,我们会去尽量缩小锁的范围——对于分布式锁实现的改进其实也是同样的思路。根据具体的业务场景和集群规模来选择适合自己的分布式锁实现:在**集群规模不大、网络资源丰富的情况**下,第一种分布式锁实现方式是简单实用的选择;而如果**集群规模达到一定程度**,并且希望能够**精细化地控制分布式锁机制**,那么就可以试试改进版的分布式锁实现。





