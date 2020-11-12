### 一.ZAB协议

#### 1.概念

Zookeeper是使用的Zookeeper Atomic Broadcast（ZAB，Zookeeper原子消息广播协议）协议作为数据一致性的核心算法，并没有完全采用paxos算法。ZAB协议是一种专门为Zookeeper设计的**支持崩溃恢复的原子广播协议**。

基于该协议，Zookeeper实现了一种主备模式的系统架构来保持集群中各副本之间的数据一致性。

ZAB协议的核心是定义了**对于那些会改变Zookeeper服务器数据状态的事务请求的处理方式**。

> 即所有事务请求必须由一个全局唯一的服务器来协调处理,这样的服务器被称为**Leader服务器**,余下的服务器则称为**Follower服务器**, Leader服务器负责将一个客户端事务请求转化成一个事务Proposal (提·议) ,并将该Proposal分发给集群中所有的Follower服务器,之后Leader服务器需要等待所有Follower服务器的反馈,**一旦超过半数的Follower服务器进行了正确的反馈后**,那么Leader就会再次向所有的Follower服务器分发Commit消息,要求其将前一个Proposal进行提交。

![image-20201112211450309](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112211450309.png)

#### 2.ZAB协议两种基本模式

ZAB协议包括两种基本模式：崩溃恢复和消息广播。

**进入崩溃恢复模式**：

当整个服务框架启动过程中,或者是Leader服务器出现网络中断、崩溃退出或重启等异常情况时, ZAB协议就会进入崩溃恢复模式,同时选举产生新的Leader服务器。当选举产生了新的Leader服务器,同时集群中已经有过半的机器与该Leader服务器完成了状态同步之后, ZAB协议就会退出恢复模式,其中,所谓的状态同步就是指数据同步,用来保证**集群中过半的机器能够和Leader服务器的数据状态保持一致**。

**进入消息广播模式**：

当集群中已经有过半的Follower服务器完成了和Leader服务器的状态同步,那么整个服务框架就可以进入消息广播模式,当一台同样遵守ZAB协议的服务器启动后加入到集群中,如果此时集群中已经存在一个Leader服务器在负责进行消息广播,那么加入的服务器就会自觉地进入**数据恢复模式:找到Leader所在的服务器,并与其进行数据同步,然后一起参与到消息广播流程中去**。Zookeeper只允许唯一的一个Leader服务器来进行事务请求的处理, Leader服务器在接收到客户端的事务请求后,会生成对应的事务提议并发起一轮广播协议,而如果集群中的其他机器收到客户端的事务请求后,那么这些非Leader服务器会首先将这个事务请求转发给Leader服务器。

##### 2.1 消息广播

ZAB协议使用原子广播协议，类似于一个二阶段提交过程，对于客户端的事务请求，Leader服务器会为其生成对应的事务Proposal,并将其发送给集群中其余所有的机器,然后再分别收集各自的选票,最后进行事务提交。

![image-20201112213842570](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112213842570.png)

- 二阶段提交过程中，移出了中断逻辑，即在过半的Follower服务器已经反馈Ack后就可以开始提交事务Proposal了；
- ZAB采用崩溃恢复模式来解决因Leader服务器崩溃退出而带来的数据不一致性问题；
- 整个消息广播协议是基于具有FIFO特性的TCP协议来进行网络通信的，可以保证消息广播过程中消息接受与发送的顺序性。

在整个消息广播过程中, Leader服务器会为每个事务请求生成对应的Proposal来进行广播,并且在广播事务Proposal之前, Leader服务器会首先为这个事务Proposal分配一个**全局单调递增的唯一ID,称之为事务ID** (ZXID) ,由于ZAB协议需要保证每个消息严格的因果关系,因此必须将**每个事务Proposal按照其ZXID的先后顺序来进行排序和处理**。

具体的过程：**在消息广播过程中, Leader服务器会为每一个Follower服务器都各自分配一个单独的队列,然后将需要广播的事务Proposal依次放入这些队列中去,并且根据FIFO策略进行消息发送。每一个Follower服务器在接收到这个事务Proposal之后,都会首先将其以事务日志的形式写入到本地磁盘中去,并且在成功写入后反馈给Leader服务器一个Ack响应。当Leader服务器接收到超过半数Follower的Ack响应后,就会广播一个Commit消息给所有的Follower服务器以通知其进行事务提交,同时Leader自身也会完成对事务的提交,而每一个Follower服务器在接收到Commit消息后,也会完成对事务的提交**。

##### 2.2 崩溃恢复

一旦在Leader服务器出现崩溃，或者由于网络原因导致Leader服务器失去了过半Follower的联系，就会进入崩溃恢复模式。整个恢复过程结束后需要选举一个新的Leader服务器，因此，ZAB协议就需要一个高效且可靠的Leader选举算法来保证快速的选出新的Leader。

**基本特性**

ZAB协议规定了如果一个事务Proposal在一台机器上被处理成功,那么应该在所有的机器上都被处理成功,哪怕机器出现故障崩溃。

1. ZAB协议需要确保那些已经在Leader服务器上提交的事务最终被所有服务器都提交;
2. ZAB协议需要确保丢弃那些只在Leader服务器上被提出的事务。

ZAB协议必须设计这样一个Leader选举算法：**能够确保提交已经被Leader提交的事务Proposal,同时丢弃已经被跳过的事务Proposal**。针对这个要求,如果让Leader选举算法能够**保证新选举出来的Leader服务器拥有集群中所有机器最高编号(即ZXID最大)的事务Proposal**,那么就可以保证这个新选举出来的Leader一定具有所有已经提交的提案。如果让具有最高编号事务Proposal的机器来成为Leader,就可以省去Leader服务器检查Proposal的提交和丢弃工作的这一步操作了。

**数据同步**

完成Leader选举之后,在正式开始工作(即接收客户端的事务请求,然后提出新的提案)之前,Leader服务器会首先确认事务日志中的所有Proposal是否都已经被集群中过半的机器提交了,即是否完成数据同步。下面我们就来看看ZAB协议的数据同步过程。

**所有正常运行的服务器,要么成为Leader,要么成为Follower并和Leader保持同步。Leader服务器需要确保所有的Follower服务器能够接收到每一条事务Proposal,并且能够正确地将所有已经提交了的事务Proposal应用到内存数据库中去**。具体的, Leader服务器会为每一个Follower服务器都准备一个队列,并将那些没有被各Follower服务器同步的事务以Proposal消息的形式逐个发送给Follower服务器,并在每一个Proposal消息后面紧接着再发送一个Commit消息,以表示该事务已经被提交。等到Follower服务器将所有其尚未同步的事务Proposal都从Leader服务器上同步过来并成功应用到本地数据库中后, Leader服务器就会将该Follower服务器加入到真正的可用Follower列表中,并开始之后的其他流程。

#### 3.运行时状态分析

在ZAB协议的设计中，每个进程都有可能处于下面三种状态之一：

- **Looking**：Leader选举阶段；
- **Following**：Follower服务器和Leader服务器保持同步状态；
- **Leading**：Leader服务器作为主进程领导状态。

**所有进程初始状态都是LOOKING状态,此时不存在Leader,接下来,进程会试图选举出一个新的Leader,之后,如果进程发现已经选举出新的Leader了,那么它就会切换到FOLLOWING状态,并开始和Leader保持同步,处于FOLLOWING状态的进程称为Follower, LEADING状态的进程称为Leader,当Leader崩溃或放弃领导地位时,其余的Follower进程就会转换到LOOKING状态开始新一轮的Leader选举**。

一个Follower只能和一个Leader保持同步, Leader进程和所有的Follower进程之间都通过**心跳检测机制**来感知彼此的情况。若Leader能够在超时时间内正常收到心跳检测,那么Follower就会一直与该eader保持连接,而如果在指定时间内Leader无法从**过半的Follower进程那里接收到心跳检测,或者TCP连接断开,那么Leader会放弃当前周期的领导**,并转换到LOOKING状态,其他的Follower也会选择放弃这个Leader,同时转换到LOOKING状态,之后会进行新一轮的Leader选举。

#### 4.ZAB与Paxos联系和区别

联系：

- 都存在一个类似于**Leader进程的角色**,由其负责协调多个Follower进程的运行。
- Leader进程都会等待**超过半数的Follower做出正确的反馈**后,才会将一个提议进行提交。
- 在ZAB协议中,每个**Proposal**中都包含了一个epoch值,用来代表当前的Leader周期,在Paxos算法中,同样存在这样的一个标识,名字为Ballot。

区别：

- Paxos算法中,新选举产生的主进程会进行两个阶段的工作,第一阶段称为**读阶段**,新的主进程和其他进程通信来收集主进程提出的提议,并将它们提交。第二阶段称为**写阶段**,当前主进程开始提出自己的提议。
- ZAB协议在Paxos基础上添加了**同步阶段**,此时,新的Leader会确保存在过半的Follower已经提交了之前的Leader周期中的所有事务Proposal,这一同步阶段的引入,能够有效地**保证Leader在新的周期中提出事务Proposal之前,所有的进程都已经完成了对之前所有事务Proposal的提交**。

总的来说, ZAB协议和Paxos算法的本质区别在于,两者的设计目标不太一样, **ZAB协议主要用于构建一个高可用的分布式数据主备系统,而Paxos算法则用于构建一个分布式的一致性状态机系统**。

### 二.服务器角色

#### 1.Leader

Leader服务器是Zookeeper集群工作的核心，其主要工作有以下两个：

- **事务请求的唯一调度和处理者,保证集群事务处理的顺序性**。
- **集群内部各服务器的调度者**。

使用责任链来处理每个客户端的请求是Zookeeper的特色, Leader服务器的请求处理链如下:

![image-20201112224204488](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112224204488.png)

可以看到,从prepRequestProcessor到FinalRequestProcessor前后一共7个请求处理器组成了leader服务器的请求处理链。

- PrepRequestProcessor,**请求预处理器**。也是leader服务器中的第一个请求处理器。在Zookeeper中,那些会改变服务器状态的请求称为事务请求(创建节点、更新数据、删除节点、创建会话等). PrepRequestProcessor能够识别出当前客户端请求是否是事务请求。对于事务请求,PrepRequestProcessor处理器会对其进行一系列预处理,如创建请求事务头、事务体、会话检查、ACL检查和版本检查等。
- ProposalRequestProcessor,**事务投票处理器**。也是Leader服务器事务处理流程的发起者,对于非事务性请求, ProposalRequestProcessor会直接将请求转发到CommitProcessor处理器,不再做任何处理,而对于事务性请求,处理将请求转发到CommitProcessor外,还会根据请求类型创建对应的Proposal提议,并发送给所有的Follower服务器来发起一次集群内的事务投票。同时,ProposalRequestProcessor还会将事务请求交付给SyncRequestProcessor进行事务日志的记录。
- SyncRequestProcessor,**事务日志记录处理器**。用来将事务请求记录到事务日志文件中,同时会触发Zookeeper进行数据快照。
- AckRequestProcessor,负责在SyncRequestProcessor完成事务日志记录后,向Proposal的投票收集器发送ACK反馈,以通知投票收集器当前服务器已经完成了对该Proposal的事务日志记录。
- CommitProcessor,**事务提交处理器**。对于非事务请求,该处理器会直接将其交付给下一级处理器处理;对于事务请求,其会等待集群内针对Proposal的投票直到该Proposal可被提交,利用CommitProcessor,每个服务器都可以很好地控制对事务请求的顺序处理。
- ToBeCommitProcessor。该处理器有一个toBeApplied队列,用来存储那些已经被CommitProcessor处理过的可被提交的Proposal,其会将这些请求交付给FinalRequestProcessor处理器处理,待其处理完后,再将其从toBeApplied队列中移除。
- FinalRequestProcessor,用来进行客户端请求返回之前的操作,包括创建客户端请求的响应,针对事务请求,该处理器还会负责将事务应用到内存数据库中。

#### 2.Follower

Follower服务器是Zookeeper集群状态中的跟随者，其主要工作有以下三个：

- **处理客户端非事务性请求(读取数据) ,转发事务请求给Leader服务器**。
- **参与事务请求Proposal的投票**。
- **参与Leader选举投票**。

和leader一样, Follower也采用了责任链模式组装的请求处理链来处理每一个客户端请求,由于**不需要对事务请求的投票处理**,因此Follower的请求处理链会相对简单,其处理链如下：

![image-20201112225103738](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112225103738.png)

和Leader服务器的请求处理链最大的不同点在于, Follower服务器的第一个处理器换成了FollowerRequestProcessor处理器,同时由于不需要处理事务请求的投票,因此也没有了ProposalRequestProcessor处理器。

- FollowerRequestProcessor：其用作**识别当前请求是否是事务请求**,若是,那么Follower就会将该请求转发给Leader服务器,Leader服务器在接收到这个事务请求后,就会将其提交到请求处理链,按照正常事务请求进行处理。
- SendAckRequestProcessor：其承担了**事务日志记录反馈的角色**,在完成事务日志记录后,会向Leader服务器发送ACK消息以表明自身完成了事务日志的记录工作。

#### 3.Observer

Observer充当了一个观察者的角色——其**观察Zookeeper集群的最新状态变化并将这些状态变更同步过来**。Observer服务器在工作原理上和Follower基本是一致的,**对于非事务请求,都可以进行独立的处理,而对于事务请求,则会转发给Leader服务器进行处理**。

和Follower唯一的区别在于, **Observer不参与任何形式的投票,包括事务请求Proposal的投票和Leader选举投票**。简单地讲, Observer服务器只提供非事务服务,通常用于在不影响集群事务处理能力的前提下提升集群的非事务处理能力。另外, Observer的请求处理链路和Follower服务器也非常相近,其处理链如下：

![image-20201112225746605](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112225746605.png)

需要注意的是,虽然在图中可以看到, Observer服务器在初始化阶段会将SyncRequestProcessor处理器也组装上去,但是在实际运行过程中, **Leader服务器不会将事务请求的投票发送给Observer服务器**。

### 三.服务器启动

#### 1.服务端整体架构图

![image-20201112230122117](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112230122117.png)

Zookeeper服务器的启动，大致可以分为以下五个步骤：

1.配置文件解析 2.初始化数据管理器 3.初始化网络I/O管理器 4.数据恢复 5.对外服务。

#### 2.单机版服务器启动

![image-20201112230346526](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112230346526.png)

上图过程可分为预启动和初始化过程。

##### 2.1 预启动

1. 统一由**QuorumPeerMain作为启动类**。无论单机或集群,在zkServer.cmd和zkServer.sh中都配置了QuorumPeerMain作为启动入口类。
2. 解析**配置文件zoo.cfg**。zoo.cfg配置运行时的基本参数,如tickTime, dataDir,clientPort等参数。
3. 创建并启动**历史文件清理器DatadircleanupManager**。对事务日志和快照数据文件进行定时清理。
4. 判断当前是集群模式还是单机模式启动。若是单机模式,则委托给zooKeeperServerMain进行启动。
5. 再次进行**配置文件zoo.cfg的解析**。
6. 创建**服务器实例ZooKeeperServer**。zookeeper服务器首先会进行服务器实例的创建,然后对该服务器实例进行初始化,包括连接器、内存数据库、请求处理器等组件的初始化。

##### 2.2 初始化

1. 创建**服务器统计器ServerStats**。ServerStats是zookeeper服务器运行时的统计器。
2. 创建Zookeeper**数据管理器FilerxnSnapLog**。FilerxnSnapLog是zookeeper上层服务器和底层数据存储之间的对接层,提供了一系列操作数据文件的接口,如事务日志文件和快照数据文件。Zookeeper根据zoo.cfq文件中解析出的快照数据目录dataDir和事务日志目录dataLogDir来创建FilerxnSnapLog。
3. 设置服务器tickTime和会话超时时间限制。
4. **创建ServerCnxnFactory**。通过配置系统属性zookeper.serverCnxnFactory来指定使用zookeeper自己实现的NIO还是使用Netty框架作为zookeeper服务端网络连接工厂。
5. **初始化ServerCnxnFactory**。zookeeper会初始化Thread作为ServerCnxnFactory的主线程,然后再初始化NIO服务器。
6. **启动ServerCnxnFactory主线程**。进入Thread的run方法,此时服务端还不能处理客户端请求。
7. **恢复本地数据**。启动时,需要从本地快照数据文件和事务日志文件进行数据恢复。
8. **创建并启动会话管理器**。zookeeper会创建会话管理器SessionTracker进行会话管理。
9. **初始化zookeeper的请求处理链**。Zookeeper请求处理方式为责任链模式的实现。会有多个请求处理器依次处理一个客户端请求,在服务器启动时,会将这些请求处理器串联成一个请求处理链。
10. **注册JMX服务**。zookeeper会将服务器运行时的一些信息以JMX的方式暴露给外部。
11. **注册zookeeper服务器实例**。将zookeeper服务器实例注册给serverCnxnFactory,之后zookeeper就可以对外提供服务。

到此，单机版的Zookeeper服务器启动完毕。

#### 3.集群服务器启动

单机和集群服务器的启动在很多地方是一致的，流程图如下：

![image-20201112231913008](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112231913008.png)

上图过程可以分为预启动、初始化、Leader选举、Leader与Follower启动期交互、Leader与Follower启动等过程。

##### 3.1 预启动

1. 统一由QuorumPeerMain作为启动类。
2. 解析配置文件zoo.cfg。
3. 创建并启动历史文件清理器DatadircleanupFactory。
4. 判断当前是集群模式还是单机模式的启动。在集群模式中,在zoo.cfg文件中配置了多个服务器地址，可以选择集群启动。

##### 3.2 初始化

1. 创建ServerCnxnFactory。
2. 初始化serverCnxnFactory。
3. 创建zookeeper数据管理器FilerxnSnapLog。
4. 创建QuorumPeer实例。Quorum是集群模式下特有的对象,是zookeeper服务器实例(zooKeeperServer)的托管者, QuorumPeer代表了集群中的一台机器,在运行期间,QuorumPeer会不断检测当前服务器实例的运行状态,同时根据情况发起Leader选举。
5. 创建内存数据库ZKDatabase。ZKDatabase负责管理Zookeeper的所有会话记录以及Datarree和事务日志的存储。
6. 初始化QuorumPeer。将核心组件如FileTxnSnapLog、ServerCnxnFactory,zKDatabase注册到QuorumPeer中,同时配置QuorumPeer的参数,如服务器列表地址、Leader选举算法和会话超时时间限制等。
7. 恢复本地数据。
8. 启动serverCnxnFactory主线程。

##### 3.3 Leader选举

**1.初始化Leader选举**。

集群模式特有, zookeeper首先会根据自身的服务器ID (sID) 、最新的ZxID (lastLoggedzxid)和当前的服务器epoch (currentEpoch)来生成一个初始化投票,在初始化过程中,每个服务器都会给自己投票。然后,根据zoo.cfg的配置,创建相应Leader选举算法实现, zookeeper提供了三种默认算法(LeaderElection,AuthFastLeaderElection、FastLeaderElection) ,可通过zoo.cfg中的electionAlg属性来指定,但现只支持FastLeaderElection选举算法。在初始化阶段, zookeeper会创建Leader选举所需的网络I/O层QuorumCnxManager,同时启动对Leader选举端口的监听,等待集群中其他服务器创建连接。

**2.注册JMX服务**。

**3.检测当前服务器状态**。

运行期间, QuorumPeer会不断检测当前服务器状态。在正常情况下, zookeeper服务器的状态在LOOKING、 LEADING、FOLLOWING/OBSERVING之间进行切换。在启动阶段, QuorumPeer的初始状态是LOOKING,因此开始进行Leader选举。

**4.Leader选举**。

Zookeeper的Leader选举过程,简单地讲,就是一个集群中所有的机器相互之间进行一系列投票,选举产生最合适的机器成为Leader,同时其余机器成为Follower或是observer的集群机器角色初始化过程。关于Leader选举算法,简而言之,就是集群中哪个机器处理的数据越新(通常我们根据每个服务器处理过的最大ZxID来比较确定其数据是否更新) ,其越有可能成为Leader。当然,如果集群中的所有机器处理的zxID-致的话,那么sID最大的服务器成为Leader,其余机器称为Follower和observer。

##### 3.4 Leader与Follower启动期交互

到这里为止, ZooKeeper已经完成了Leader选举,并且集群中每个服务器都已经确定了自己的角色——通常情况下就分为**Leader和Follower两种角色**。下面我们来对Leader和Follower在启动期间的交互进行介绍,其大致交互流程如图所示。

![image-20201112235910592](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201112235910592.png)

1. **创建Leader服务器和Follower服务器**。完成Leader选举后,每个服务器会根据自己服务器的角色创建相应的服务器实例,并进入各自角色的主流程。
2. **Leader服务器启动Follower接收器LearnerCnxAcceptor**。运行期间, Leader服务器需要和所有其余的服务器(统称为Learner)保持连接以确集群的机器存活情况, LearnerCnxAcceptor负责接收所有非Leader服务器的连接请求。
3. **Learner服务器开始和Leader建立连接**。所有Learner会找到Leader服务器,并与其建立连接。
4. **Leader服务器创建LearnerHandler,** Leader接收到来自其他机器连接创建请求后,会创建一个earnerHandler实例,每个LearnerHandler实例都对应一个Leader与Learner服务器之间的连接,其负责Leader和Learner服务器之间几乎所有的消息通信和数据同步。
5. **向Leader注册**。Learner完成和Leader的连接后,会向Leader进行注册,即将Learner服务器的基本信息(Learnerlnfo) ,包括SID和ZXID,发送Leader服务器。
6. **Leader解析Learner信息，计算新的epoch**。Leader接收到Learner服务器基本信息后,会解析出该Learner的SID和ZXID,然后根据ZXID解析出对应的epoch_oflearner,并和当前Leader服务器的epoch_ofleader进行比较,如果该Learner的epoch_oflearner更大,则更新Leader的epoch_of leader = epoch_oflearner +1,然后LearnHandler进行等待,直到过半Learner已经句Leader进行了注册,同时更新了epoch_ofleader后, Leader就可以确定当前集群的epoch了。
7. **发送Leader状态**。计算出新的epoch后, Leader会将该信息以一个LEADERINFO消息的形式发送给Learner,并等待Learner的响应。
8. **Learner发送ACK消息**。Learner接收到LEADERINFO后,会解析出epoch和ZXID,然后向Leader反馈一个ACKEPOCH响应。
9. **数据同步**。Leader收到Learner的ACKEPOCH后,即可进行数据同步。
10. **启动Leader和Learner服务器**。当有过半Learner已经完成了数据同步,那么Leader和Learner服务器实例就可以启动了。

##### 3.5 Leader与Follower启动

1. 创建启动会话管理器。
2. 初始化Zookeeper请求处理链，集群模式的每个处理器也会在启动阶段串联请求处理链。
3. 注册JMX服务。

至此，集群版的Zookeeper服务器启动完毕。

### 四.leader选举

当Zookeeper集群中的一台服务器出现以下两种情况之一时，需要进入Leader选举。

- 服务器初始化启动。
- 服务器运行期间无法和Leader保持连接。

#### 1.服务器启动时期的Leader选举

若进行Leader选举,则至少需要两台机器,这里选取3台机器组成的服务器集群为例。在集群初始化阶段,当有一台服务器Server1启动时,其单独无法进行和完成Leader选,当第二台服务器Server2启动时,此时两台机器可以相互通信,每台机器都试图找到Leader,于是进入Leader选举过程。选举过程如下：

**1).每个Server发出一个投票**

由于是初始情况, Server1 (假设myid为1)和Server2假设myid为2)都会将自己作为Leader服务器来·进行投票,每次投票会包含所推举的服务器的myid和ZXID,使用(myid, ZXID)来表示,此时Server1的投票为(1, 0), Server2的投票为(2, 0),然后各自将这个投票发给集群中其他机器。

**2).接受来自各个服务器的投票**

集群的每个服务器收到投票后,首先判断该投票的有效性,如检查是否是本轮投票、是否来自LOOKING状态的服务器。

**3).处理投票**

针对每一个投票，服务器都需要将别人的投票和自己的投票进行PK，PK规则如下：

- **优先检查ZXID**。ZXID比较大的服务器优先作为Leader。
- **如果ZXID相同,那么就比较myid**。myid较大的服务器作为Leader服务器。

现在我们来看Server1和Server2实际是如何进行投票处理的。对于Server1来说,它自己的投票是(1, 0) ,而接收到的投票为(2, 0) 。首先会对比两者的ZXID,因为都是0,所以无法决定谁是Leader,接下来会对比两者的myid,很显然, Server1发现接收到的投票中的myid是2,大于自己,于是就会更新自己的投票为(2, 0) ,然后重新将投票发出去。而对于Server2来说,不需要更新自己的投票。

**4).统计投票**

每次投票后,服务器都会统计所有投票,判断是否已经有过半的机器接收到相同的投票信息。对于Server1和server2服务器来说,都统计出集群中已经有两台机器接受了(2, 0)这个投票信息。这里我们需要对"过半"的概念做一个简单的介绍。所谓"过半"就大于集群机器数一半,即大于或等于(n/2+1) 。对于这里由3台机器构成的集群,大于等于2台即为达到"过半"要求。

即当Server1和Server2都收到相同的投票信息（2，0）的时候，认为已经选出了Leader。

**5).改变服务器状态**

一旦确定了Leader,每个服务器就会更新自己的状态：如果是Follower,那么就变更为FOLLOWING,如果是Leader,那么就变更为LEADING。

#### 2.服务器运行时期的Leader选举

在ZooKeeper集群正常运行过程中,一旦选出一个Leader,那么所有服务器的集群角色一般不会再发生变化——也就是说, Leader服务器将一直作为集群的Leader,即使集群中有非Leader机器挂了或是有新机器加入集群也不会影响Leader。但是**一旦Leader所在的机器挂了,那么整个集群将暂时无法对外服务,而是进入新一轮的Leader选举**。服务器运行期间的Leader选举和启动时期的Leader选举基本过程是一致的。

还是假设当前正在运行的ZooKeeper机器由3台机器组成,分别是Server1、Server2和Server3, 当前的Leader是Server2。假设在某一个瞬间, Leader挂了,这个时候便开始了Leader选举。

**1).变更状态**

Leader挂后,余下的非Observer服务器都会将自己的服务器状态变更为LOOKING,然后开始进入Leader选举过程。

**2).每个Server会发出一个投票**

在运行期间,每个服务器上的ZXID可能不同,此时假定Server1的ZXID为123, Server3的ZXID为122;在第一轮投票中, Server1和Server3都会投自己,产生投票(1, 123), (3, 122),然后各自将投票发送给集群中所有机器。

**3).接收来自各个服务器的投票,与启动时过程相同**。

**4).处理投票。与启动时过程相同,此时,Server1将会成为Leader**。

**5).统计投票。与启动时过程相同**。

**6).改变服务器的状态。与启动时过程相同**。



<span style="color:red;">
**注：文章内容来源拉勾教育——Zookeeper课件文档**。
</span>