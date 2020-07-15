> 博客出处：[[Network] 计算机网络基础知识总结](https://www.cnblogs.com/maybe2030/p/4781555.html)

<p><span style="font-size: 16px;">　　计算机网络学习的核心内容就是网络协议的学习。网络协议是为计算机网络中进行数据交换而建立的规则、标准或者说是约定的集合。因为不同用户的数据终端可能采取的字符集是不同的，两者需要进行通信，必须要在一定的标准上进行。一个很形象地比喻就是我们的语言，我们大天朝地广人多，地方性语言也非常丰富，而且方言之间差距巨大。A地区的方言可能B地区的人根本无法接受，所以我们要为全国人名进行沟通建立一个语言标准，这就是我们的普通话的作用。同样，放眼全球，我们与外国友人沟通的标准语言是英语，所以我们才要苦逼的学习英语。</span></p>
<p><span style="font-size: 16px;">　　计算机网络协议同我们的语言一样，多种多样。而ARPA公司与1977年到1979年推出了一种名为ARPANET的网络协议受到了广泛的热捧，其中最主要的原因就是它推出了人尽皆知的TCP/IP标准网络协议。目前TCP/IP协议已经成为Internet中的“通用语言”，下图为不同计算机群之间利用TCP/IP进行通信的示意图。</span></p>
<p><span style="font-size: 16px;"><img style="display: block; margin-left: auto; margin-right: auto;" src="https://images2015.cnblogs.com/blog/764050/201509/764050-20150904094424185-2018280216.gif" alt="" width="530" height="167"></span></p>
<h3 class="First"><strong><span style="font-size: 16px;">1. 网络层次划分</span></strong></h3>
<div class="para"><span style="font-size: 16px;">　　为了使不同计算机厂家生产的计算机能够相互通信，以便在更大的范围内建立计算机网络，国际标准化组织（ISO）在1978年提出了“开放系统互联参考模型”，即著名的OSI/RM模型（Open System Interconnection/Reference Model）。它将计算机网络体系结构的通信协议划分为七层，自下而上依次为：物理层（Physics Layer）、数据链路层（Data Link Layer）、网络层（Network Layer）、传输层（Transport Layer）、会话层（Session Layer）、表示层（Presentation Layer）、应用层（Application Layer）。</span><span style="font-size: 16px;">其中第四层完成数据传送服务，上面三层面向用户。</span></div>
<div class="para"><span style="font-size: 16px;">　　除了标准的OSI七层模型以外，常见的网络层次划分还有TCP/IP四层协议以及TCP/IP五层协议，它们之间的对应关系如下图所示：</span></div>
<div class="para">&nbsp;</div>
<div class="para"><span style="font-size: 16px;"><img style="display: block; margin-left: auto; margin-right: auto;" src="https://images2015.cnblogs.com/blog/764050/201509/764050-20150904094019903-1923900106.jpg" alt="" width="494" height="310"></span></div>
<div class="para">&nbsp;</div>
<h3 class="First" style="font-size: 16px;">2. OSI七层网络模型</h3>
<div class="para"><span style="font-size: 16px;">　　TCP/IP协议毫无疑问是互联网的基础协议，没有它就根本不可能上网，任何和互联网有关的操作都离不开TCP/IP协议。不管是OSI七层模型还是TCP/IP的四层、五层模型，每一层中都要自己的专属协议，完成自己相应的工作以及与上下层级之间进行沟通。由于OSI七层模型为网络的标准层次划分，所以我们以OSI七层模型为例从下向上进行一一介绍。</span></div>
<div class="para"><span style="font-size: 16px;"><img style="display: block; margin-left: auto; margin-right: auto;" src="https://images2015.cnblogs.com/blog/764050/201509/764050-20150904095142060-1017190812.gif" alt="" width="510" height="273"></span></div>
<div class="para">&nbsp;</div>
<div class="para"><span style="font-size: 16px;">　　<strong>1）物理层（Physical Layer）</strong></span></div>
<div class="para">
<p><span style="font-size: 16px;">　　激活、维持、关闭通信端点之间的机械特性、电气特性、功能特性以及过程特性。<span style="color: #ff0000;"><strong>该层为上层协议提供了一个传输数据的可靠的物理媒体。简单的说，物理层确保原始的数据可在各种物理媒体上传输。</strong></span>物理层记住两个重要的设备名称，中继器（Repeater，也叫放大器）和集线器。<br></span></p>
<p><span style="font-size: 16px;">　　<strong>2）数据链路层（Data Link Layer）</strong></span></p>
<p><span style="font-size: 16px;">　　数据链路层在物理层提供的服务的基础上向网络层提供服务，其最基本的服务是将源自网络层来的数据可靠地传输到相邻节点的目标机网络层。为达到这一目的，数据链路必须具备一系列相应的功能，主要有：如何将数据组合成数据块，在数据链路层中称这种数据块为帧（frame），帧是数据链路层的传送单位；如何控制帧在物理信道上的传输，包括如何处理传输差错，如何调节发送速率以使与接收方相匹配；以及在两个网络实体之间提供数据链路通路的建立、维持和释放的管理。</span><span style="font-size: 16px;">数据链路层在不可靠的物理介质上提供可靠的传输。该层的作用包括：物理地址寻址、数据的成帧、流量控制、数据的检错、重发等。</span></p>
<p><span style="font-size: 16px;">　　有关数据链路层的重要知识点：</span></p>
<p><span style="font-size: 16px;">　　<strong><span style="color: #ff0000;">1&gt;&nbsp;</span><span style="color: #ff0000;">数据链路层为网络层提供可靠的数据传输；</span></strong></span></p>
<p><span style="font-size: 16px;"><strong><span style="color: #ff0000;">　　2&gt;&nbsp;基本数据单位为帧；</span></strong></span></p>
<p><span style="font-size: 16px;"><strong><span style="color: #ff0000;">　　3&gt; 主要的协议：以太网协议；</span></strong></span></p>
<p><span style="font-size: 16px;"><strong><span style="color: #ff0000;">　　4&gt; 两个重要设备名称：网桥和交换机。</span></strong></span></p>
<p><span style="font-size: 16px;">　　<strong>3）网络层（Network Layer）</strong></span></p>
<p><span style="font-size: 16px;">　　网络层的目的是实现两个端系统之间的数据透明传送，具体功能包括寻址和路由选择、连接的建立、保持和终止等。它提供的服务使传输层不需要了解网络中的数据传输和交换技术。如果您想用尽量少的词来记住网络层，那就是“路径选择、路由及逻辑寻址”。</span></p>
<p><span style="font-size: 16px;">　　网络层中涉及众多的协议，其中包括最重要的协议，也是TCP/IP的核心协议——IP协议。</span><span style="font-size: 16px;">IP协议非常简单，仅仅提供不可靠、无连接的传送服务。IP协议的主要功能有：无连接数据报传输、数据报路由选择和差错控制。与IP协议配套使用实现其功能的还有地址解析协议ARP、逆地址解析协议RARP、因特网报文协议ICMP、因特网组管理协议IGMP。具体的协议我们会在接下来的部分进行总结，有关网络层的重点为：</span></p>
<p><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　1&gt; 网络层负责对子网间的数据包进行路由选择。此外，网络层还可以实现拥塞控制、网际互连等功能；</span></strong></span></p>
<p><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　2&gt; 基本数据单位为IP数据报；</span></strong></span></p>
<p><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　3&gt; 包含的主要协议：</span></strong></span></p>
<p><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　IP协议（Internet Protocol，因特网互联协议）;</span></strong></span></p>
<p><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　</span></strong></span><span style="color: #ff0000;"><strong><span style="font-size: 16px;">ICMP协议（Internet Control Message Protocol，因特网控制报文协议）;</span></strong></span></p>
<p><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　</span></strong></span><span style="color: #ff0000;"><strong><span style="font-size: 16px;">ARP协议（Address Resolution Protocol，地址解析协议）;</span></strong></span></p>
<p><strong style="color: #ff0000; line-height: 1.5;"><span style="font-size: 16px;">　　RARP协议（Reverse Address Resolution Protocol，逆地址解析协议）。</span></strong></p>
<p><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　4&gt; 重要的设备：路由器。</span></strong></span></p>
<p><span style="font-size: 16px;">　　<strong>4）传输层（Transport Layer）</strong></span></p>
<p><span style="font-size: 16px;">　　第一个端到端，即主机到主机的层次。传输层负责将上层数据分段并提供端到端的、可靠的或不可靠的传输。此外，传输层还要处理端到端的差错控制和流量控制问题。</span></p>
<div class="para"><span style="font-size: 16px;">　　传输层的任务是根据通信子网的特性，最佳的利用网络资源，为两个端系统的会话层之间，提供建立、维护和取消传输连接的功能，负责端到端的可靠数据传输。在这一层，信息传送的协议数据单元称为段或报文。</span></div>
<div class="para"><span style="font-size: 16px;">　　网络层只是根据网络地址将源结点发出的数据包传送到目的结点，而传输层则负责将数据可靠地传送到相应的端口。</span></div>
<div class="para"><span style="font-size: 16px;">　　有关网络层的重点：</span></div>
<div class="para"><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　1&gt;&nbsp;传输层负责将上层数据分段并提供端到端的、可靠的或不可靠的传输以及端到端的差错控制和流量控制问题；</span></strong></span></div>
<div class="para"><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　2&gt; 包含的主要协议：TCP协议（Transmission Control Protocol，传输控制协议）、UDP协议（User Datagram Protocol，用户数据报协议）；</span></strong></span></div>
<div class="para"><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　3&gt; 重要设备：网关。</span></strong></span></div>
<p><span style="font-size: 16px;">　　<strong>5）会话层</strong></span></p>
<p><span style="font-size: 16px;">　　会话层管理主机之间的会话进程，即负责建立、管理、终止进程之间的会话。会话层还利用在数据中插入校验点来实现数据的同步。</span></p>
<p><span style="font-size: 16px;">　　<strong>6）表示层</strong></span></p>
<p><span style="font-size: 16px;">　　表示层对上层数据或信息进行变换以保证一个主机应用层信息可以被另一个主机的应用程序理解。表示层的数据转换包括数据的加密、压缩、格式转换等。</span></p>
<p><span style="font-size: 16px;">　　<strong>7）应用层</strong></span></p>
<p><span style="font-size: 16px;">　　为操作系统或网络应用程序提供访问网络服务的接口。</span></p>
<p><span style="font-size: 16px;">　　会话层、表示层和应用层重点：</span></p>
<p><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>1&gt; 数据传输基本单位为报文；</strong></span></span></p>
<p><span style="color: #ff0000;"><strong><span style="font-size: 16px;">　　2&gt; 包含的主要协议：FTP（文件传送协议）、Telnet（远程登录协议）、DNS（域名解析协议）、SMTP（邮件传送协议），POP3协议（邮局协议），HTTP协议（Hyper Text Transfer Protocol）。</span></strong></span></p>
<h3 class="First"><span style="font-size: 16px;">3. IP地址</span></h3>
<p><span style="font-size: 16px;">　　<strong>1）网络地址</strong></span></p>
<p><span style="font-size: 16px;">　　IP地址由网络号（包括子网号）和主机号组成，网络地址的主机号为全0，网络地址代表着整个网络。</span></p>
<p><span style="font-size: 16px;">　　<strong>2）广播地址</strong></span></p>
<p><span style="font-size: 16px;">　　广播地址通常称为直接广播地址，是为了区分受限广播地址。</span></p>
<p><span style="font-size: 16px;">　　广播地址与网络地址的主机号正好相反，广播地址中，主机号为全1。当向某个网络的广播地址发送消息时，该网络内的所有主机都能收到该广播消息。</span></p>
<p><span style="font-size: 16px;">　　<strong>3）组播地址</strong></span></p>
<p><span style="font-size: 16px;">　　D类地址就是组播地址。</span></p>
<p><span style="font-size: 16px;">　　先回忆下A，B，C，D类地址吧：</span></p>
<p><span style="font-size: 16px;">　　A类地址以0开头，第一个字节作为网络号，地址范围为：0.0.0.0~127.255.255.255；(<span style="color: #ff0000;"><strong>modified @2016.05.31</strong></span>)</span></p>
<p><span style="font-size: 16px;">　　B类地址以10开头，前两个字节作为网络号，地址范围是：128.0.0.0~191.255.255.255;</span></p>
<p><span style="font-size: 16px;">　　C类地址以110开头，前三个字节作为网络号，地址范围是：192.0.0.0~223.255.255.255。</span></p>
<p><span style="font-size: 16px;">　　D类地址以1110开头，地址范围是224.0.0.0~239.255.255.255，D类地址作为组播地址（一对多的通信）；</span></p>
<p><span style="font-size: 16px;">　　E类地址以1111开头，地址范围是240.0.0.0~255.255.255.255，E类地址为保留地址，供以后使用。</span></p>
<p><span style="font-size: 16px;">　　注：只有A,B,C有网络号和主机号之分，D类地址和E类地址没有划分网络号和主机号。</span></p>
<p><span style="font-size: 16px;">　　<strong>4）255.255.255.255</strong></span></p>
<p><span style="font-size: 16px;">　　该IP地址指的是受限的广播地址。受限广播地址与一般广播地址（直接广播地址）的区别在于，受限广播地址只能用于本地网络，路由器不会转发以受限广播地址为目的地址的分组；一般广播地址既可在本地广播，也可跨网段广播。例如：主机192.168.1.1/30上的直接广播数据包后，另外一个网段192.168.1.5/30也能收到该数据报；若发送受限广播数据报，则不能收到。</span></p>
<p><span style="font-size: 16px;">　　注：一般的广播地址（直接广播地址）能够通过某些路由器（当然不是所有的路由器），而受限的广播地址不能通过路由器。</span></p>
<p><span style="font-size: 16px;">　　<strong>5）0.0.0.0</strong></span></p>
<p><span style="font-size: 16px;">　　常用于寻找自己的IP地址，例如在我们的RARP，BOOTP和DHCP协议中，若某个未知IP地址的无盘机想要知道自己的IP地址，它就以255.255.255.255为目的地址，向本地范围（具体而言是被各个路由器屏蔽的范围内）的服务器发送IP请求分组。</span></p>
<p><span style="font-size: 16px;">　　<strong>6）回环地址</strong></span></p>
<p><span style="font-size: 16px;">　　127.0.0.0/8被用作回环地址，回环地址表示本机的地址，常用于对本机的测试，用的最多的是127.0.0.1。</span></p>
<p><span style="font-size: 16px;">　　<strong>7）A、B、C类私有地址</strong></span></p>
<p><span style="font-size: 16px;">　　私有地址(private address)也叫专用地址，它们不会在全球使用，只具有本地意义。</span></p>
<p><span style="font-size: 16px;">　　A类私有地址：10.0.0.0/8，范围是：10.0.0.0~10.255.255.255</span></p>
<p><span style="font-size: 16px;">　　B类私有地址：172.16.0.0/12，范围是：172.16.0.0~172.31.255.255</span></p>
<p><span style="font-size: 16px;">　　C类私有地址：192.168.0.0/16，范围是：192.168.0.0~192.168.255.255</span></p>
<h3 class="First"><span style="font-size: 16px;">4. 子网掩码及网络划分</span></h3>
<p><span style="font-size: 16px;">　　</span><span style="font-size: 16px;">随着互连网应用的不断扩大，原先的IPv4的弊端也逐渐暴露出来，即网络号占位太</span><span style="font-size: 16px;">多，而主机号位太少，所以其能提供的主机地址也越来越稀缺，目前除了使用NAT</span><span style="font-size: 16px;">在企业内部利用保留地址自行分配以外，通常都对一个高类别的IP地址进行再划</span><span style="font-size: 16px;">分，以形成多个子网，提供给不同规模的用户群使用。</span></p>
<p><span style="font-size: 16px;">　　这里主要是为了在网络分段情况下有效地利用IP地址，通过对主机号的高位部分取</span><span style="font-size: 16px;">作为子网号，从通常的网络位界限中扩展或压缩子网掩码，用来创建某类地址的更</span><span style="font-size: 16px;">多子网。但创建更多的子网时，在每个子网上的可用主机地址数目会比原先减少。</span></p>
<p><span style="font-size: 16px;">　　<strong>什么是子网掩码？</strong></span></p>
<p><span style="font-size: 16px;">　　子网掩码是标志两个IP地址是否同属于一个子网的，也是32位二进制地址，其每一个为1代表该位是网络位，为0代表主机位。它和IP地址一样也是使用点式十进制来表示的。如果两个IP地址在子网掩码的按位与的计算下所得结果相同，即表明它们共属于同一子网中。</span></p>
<p><span style="font-size: 16px;">　　<strong><span style="color: #ff0000;">在计算子网掩码时，我们要注意IP地址中的保留地址，即“ 0”地址和广播地址，它们是指主机地址或网络地址全为“ 0”或“ 1”时的IP地址，它们代表着本网络地址和广播地址，一般是不能被计算在内的。</span></strong></span></p>
<p><span style="font-size: 16px;">　　<strong>子网掩码的计算：</strong></span><span style="font-size: 16px;"><br></span></p>
<p><span style="font-size: 16px;">　　对于无须再划分成子网的IP地址来说，其子网掩码非常简单，即按照其定义即可写出：如某B类IP地址为 10.12.3.0，无须再分割子网，则该IP地址的子网掩码255.255.0.0。如果它是一个C类地址，则其子网掩码为 255.255.255.0。其它类推，不再详述。下面我们关键要介绍的是一个IP地址，还需要将其高位主机位再作为划分出的子网网络号，剩下的是每个子网的主机号，这时该如何进行每个子网的掩码计算。</span></p>
<p><span style="font-size: 16px;">　　下面总结一下有关子网掩码和网络划分常见的面试考题：</span></p>
<p><strong>　　<span style="font-size: 16px;">1）利用子网数来计算</span></strong></p>
<p><span style="font-size: 16px;">　　在求子网掩码之前必须先搞清楚要划分的子网数目，以及每个子网内的所需主机数</span><span style="font-size: 16px;">目。</span></p>
<p><span style="font-size: 16px;">　　(1) 将子网数目转化为二进制来表示;</span></p>
<p><span style="font-size: 16px;">　　如欲将B类IP地址168.195.0.0划分成27个子网：27=11011；</span></p>
<p><span style="font-size: 16px;">　　(2) 取得该二进制的位数，为N；</span></p>
<p><span style="font-size: 16px;">　　该二进制为五位数，N = 5</span></p>
<p><span style="font-size: 16px;">　　(3) 取得该IP地址的类子网掩码，将其主机地址部分的的前N位置1即得出该IP地址</span><span style="font-size: 16px;">划分子网的子网掩码。</span></p>
<p><span style="font-size: 16px;">　　将B类地址的子网掩码255.255.0.0的主机地址前5位置 1，得到 255.255.248.0</span></p>
<p><span style="font-size: 16px;">　　<strong>2）利用主机数来计算</strong></span></p>
<p><span style="font-size: 16px;">　　如欲将B类IP地址168.195.0.0划分成若干子网，每个子网内有主机700台：</span></p>
<p><span style="font-size: 16px;">　　(1) 将主机数目转化为二进制来表示；</span></p>
<p><span style="font-size: 16px;">　　700=1010111100；</span></p>
<p><span style="font-size: 16px;">　　(2) 如果主机数小于或等于254（注意去掉保留的两个IP地址），则取得该主机的二</span><span style="font-size: 16px;">进制位数，为N，这里肯定 N&lt;8。如果大于254，则 N&gt;8，这就是说主机地址将占</span><span style="font-size: 16px;">据不止8位；</span></p>
<p><span style="font-size: 16px;">　　该二进制为十位数，N=10；</span></p>
<p><span style="font-size: 16px;">　　(3) 使用255.255.255.255来将该类IP地址的主机地址位数全部置1，然后从后向前的</span><span style="font-size: 16px;">将N位全部置为 0，即为子网掩码值。</span></p>
<p><span style="font-size: 16px;">　　将该B类地址的子网掩码255.255.0.0的主机地址全部置1，得到255.255.255.255，然后再从后向前将后 10位置0,即为：11111111.11111111.11111100.00000000，即255.255.252.0。这就是该欲划分成主机为700台的B类IP地址 168.195.0.0的子网掩码。</span></p>
<p><span style="font-size: 16px;">　　<strong>3）还有一种题型，要你根据每个网络的主机数量进行子网地址的规划和</strong></span><strong><span style="font-size: 16px;">计算子网掩码。这也可按上述原则进行计算。</span></strong></p>
<p><span style="font-size: 16px;">　　比如一个子网有10台主机，那么对于</span><span style="font-size: 16px;">这个子网需要的IP地址是：</span></p>
<p><span style="font-size: 16px;">　　10＋1＋1＋1＝13</span></p>
<p><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>注意：加的第一个1是指这个网络连接时所需的网关地址，接着的两个1分别是指网</strong></span></span><span style="color: #ff0000;"><strong><span style="font-size: 16px;">络地址和广播地址。</span></strong></span></p>
<p><span style="font-size: 16px;">　　因为13小于16（16等于2的4次方），所以主机位为4位。而</span><span style="font-size: 16px;">256－16＝240，</span><span style="font-size: 16px;">所以该子网掩码为255.255.255.240。</span></p>
<p><span style="font-size: 16px;">　　如果一个子网有14台主机，不少人常犯的错误是：依然分配具有16个地址空间的子</span><span style="font-size: 16px;">网，而忘记了给网关分配地址。这样就错误了，因为</span><span style="font-size: 16px; line-height: 1.5;">14＋1＋1＋1＝17</span><span style="font-size: 16px;">，</span><span style="font-size: 16px;">17大于16，所以我们只能分配具有32个地址（32等于2的5次方）空间的子网。这时</span><span style="font-size: 16px;">子网掩码为：255.255.255.224。</span></p>
<h3 class="First"><span style="font-size: 16px;">5. ARP/RARP协议</span></h3>
<p><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>地址解析协议，即ARP（Address Resolution Protocol），是根据IP地址获取物理地址的一个TCP/IP协议。</strong></span>主机发送信息时将包含目标IP地址的ARP请求广播到网络上的所有主机，并接收返回消息，以此确定目标的物理地址；收到返回消息后将该IP地址和物理地址存入本机ARP缓存中并保留一定时间，下次请求时直接查询ARP缓存以节约资源。地址解析协议是建立在网络中各个主机互相信任的基础上的，网络上的主机可以自主发送ARP应答消息，其他主机收到应答报文时不会检测该报文的真实性就会将其记入本机ARP缓存；由此攻击者就可以向某一主机发送伪ARP应答报文，使其发送的信息无法到达预期的主机或到达错误的主机，这就构成了一个ARP欺骗。<span style="color: #ff0000;"><strong>ARP命令可用于查询本机ARP缓存中IP地址和MAC地址的对应关系、添加或删除静态对应关系等。</strong></span></span></p>
<p><span style="font-size: 16px;">　　ARP工作流程举例：</span></p>
<div class="para"><span style="font-size: 16px;">　　主机A的IP地址为192.168.1.1，MAC地址为0A-11-22-33-44-01；</span></div>
<div class="para"><span style="font-size: 16px;">　　主机B的IP地址为192.168.1.2，MAC地址为0A-11-22-33-44-02；</span></div>
<div class="para"><span style="font-size: 16px;">　　当主机A要与主机B通信时，地址解析协议可以将主机B的IP地址（192.168.1.2）解析成主机B的MAC地址，以下为工作流程：</span></div>
<div class="para"><span style="font-size: 16px;">　　（1）根据主机A上的路由表内容，IP确定用于访问主机B的转发IP地址是192.168.1.2。然后A主机在自己的本地ARP缓存中检查主机B的匹配MAC地址。</span></div>
<div class="para"><span style="font-size: 16px;">　　（2）如果主机A在ARP缓存中没有找到映射，它将询问192.168.1.2的硬件地址，从而将ARP请求帧广播到本地网络上的所有主机。源主机A的IP地址和MAC地址都包括在ARP请求中。本地网络上的每台主机都接收到ARP请求并且检查是否与自己的IP地址匹配。如果主机发现请求的IP地址与自己的IP地址不匹配，它将丢弃ARP请求。</span></div>
<div class="para"><span style="font-size: 16px;">　　（3）主机B确定ARP请求中的IP地址与自己的IP地址匹配，则将主机A的IP地址和MAC地址映射添加到本地ARP缓存中。</span></div>
<div class="para"><span style="font-size: 16px;">　　（4）主机B将包含其MAC地址的ARP回复消息直接发送回主机A。</span></div>
<div class="para"><span style="font-size: 16px;">　　（5）当主机A收到从主机B发来的ARP回复消息时，会用主机B的IP和MAC地址映射更新ARP缓存。本机缓存是有生存期的，生存期结束后，将再次重复上面的过程。主机B的MAC地址一旦确定，主机A就能向主机B发送IP通信了。</span></div>
<p><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>逆地址解析协议，即RARP，功能和ARP协议相对，其将局域网中某个主机的物理地址转换为IP地址</strong></span>，比如局域网中有一台主机只知道物理地址而不知道IP地址，那么可以通过RARP协议发出征求自身IP地址的广播请求，然后由RARP服务器负责回答。</span></p>
<p><span style="font-size: 16px;">　　RARP协议工作流程：</span></p>
<p><span style="font-size: 16px;">　　</span><span style="font-size: 16px;">（1）给主机发送一个本地的RARP广播，在此广播包中，声明自己的MAC地址并且请求任何收到此请求的RARP服务器分配一个IP地址；</span></p>
<p><span style="font-size: 16px;">　　</span><span style="font-size: 16px;">（2）本地网段上的RARP服务器收到此请求后，检查其RARP列表，查找该MAC地址对应的IP地址；</span></p>
<div class="para"><span style="font-size: 16px;">　　（3）如果存在，RARP服务器就给源主机发送一个响应数据包并将此IP地址提供给对方主机使用；</span></div>
<div class="para"><span style="font-size: 16px;">　　（4）如果不存在，RARP服务器对此不做任何的响应；</span></div>
<div class="para"><span style="font-size: 16px;">　　（5）源主机收到从RARP服务器的响应信息，就利用得到的IP地址进行通讯；如果一直没有收到RARP服务器的响应信息，表示初始化失败。</span></div>
<h3 class="First"><span style="font-size: 16px;">6. 路由选择协议</span></h3>
<div class="para">
<p><span style="font-size: 16px;">　　常见的路由选择协议有：RIP协议、OSPF协议。</span></p>
<p><span style="font-size: 16px;"><strong>　　RIP</strong><strong>协议</strong>&nbsp;：底层是贝尔曼福特算法，它选择路由的度量标准（metric)是跳数，最大跳数是15跳，如果大于15跳，它就会丢弃数据包。</span></p>



<span style="font-size: 16px;">
　　<strong>OSPF</strong><strong>协议</strong>&nbsp;：Open Shortest Path First开放式最短路径优先，底层是迪杰斯特拉算法，是链路状态路由选择协议，它选择路由的度量标准是带宽，延迟。</span></div>

<h3 class="First"><span style="font-size: 16px;">7. TCP/IP协议</span></h3>
<div class="para"><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>TCP/IP协议是Internet最基本的协议、Internet国际互联网络的基础，由网络层的IP协议和传输层的TCP协议组成。通俗而言：TCP负责发现传输的问题，一有问题就发出信号，要求重新传输，直到所有数据安全正确地传输到目的地。而IP是给因特网的每一台联网设备规定一个地址。</strong></span></span></div>
<div class="para"><span style="font-size: 16px;">　　IP层接收由更低层（网络接口层例如以太网设备驱动程序）发来的数据包，并把该数据包发送到更高层---TCP或UDP层；相反，IP层也把从TCP或UDP层接收来的数据包传送到更低层。IP数据包是不可靠的，因为IP并没有做任何事情来确认数据包是否按顺序发送的或者有没有被破坏，IP数据包中含有发送它的主机的地址（源地址）和接收它的主机的地址（目的地址）。</span></div>
<p>&nbsp;　　<span style="font-size: 16px;">TCP是面向连接的通信协议，通过三次握手建立连接，通讯完成时要拆除连接，由于TCP是面向连接的所以只能用于端到端的通讯。</span><span style="font-size: 16px;">TCP提供的是一种可靠的数据流服务，采用“带重传的肯定确认”技术来实现传输的可靠性。TCP还采用一种称为“滑动窗口”的方式进行流量控制，所谓窗口实际表示接收能力，用以限制发送方的发送速度。</span></p>
<p><span style="font-size: 16px;">　　<strong>TCP报文首部格式：</strong></span></p>
<p><span style="font-size: 16px;"><img style="display: block; margin-left: auto; margin-right: auto;" src="https://images2015.cnblogs.com/blog/764050/201509/764050-20150904110054856-961661137.png" alt="" width="528" height="389"></span></p>
<p><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>TCP协议的三次握手和四次挥手：</strong></span></span></p>
<p style="text-align: left;"><span style="font-size: 16px;"><img style="display: block; margin-left: auto; margin-right: auto;" src="https://images2015.cnblogs.com/blog/764050/201509/764050-20150904110008388-1768388886.gif" alt=""></span></p>
<p style="text-align: left;">&nbsp;</p>
<p style="text-align: left;"><span style="font-size: 16px;"><strong>　　注：seq</strong>:"sequance"序列号；<strong>ack</strong>:"acknowledge"确认号；<strong>SYN</strong>:"synchronize"请求同步标志；<strong>；ACK</strong>:"acknowledge"确认标志"<strong>；</strong><strong>FIN</strong>："Finally"结束标志。</span></p>
<p style="text-align: left;"><span style="font-size: 16px;">　　<strong>TCP连接建立过程：</strong>首先Client端发送连接请求报文，Server段接受连接后回复ACK报文，并为这次连接分配资源。Client端接收到ACK报文后也向Server段发生ACK报文，并分配资源，这样TCP连接就建立了。</span></p>
<p style="text-align: left;"><span style="font-size: 16px;">　　<strong>TCP连接断开过程：</strong>假设Client端发起中断连接请求，也就是发送FIN报文。Server端接到FIN报文后，意思是说"<span style="text-decoration: underline;">我Client端没有数据要发给你了</span>"，但是如果你还有数据没有发送完成，则不必急着关闭Socket，可以继续发送数据。所以你先发送ACK，"<span style="text-decoration: underline;">告诉Client端，你的请求我收到了，但是我还没准备好，请继续你等我的消息</span>"。这个时候Client端就进入FIN_WAIT状态，继续等待Server端的FIN报文。当Server端确定数据已发送完成，则向Client端发送FIN报文，"<span style="text-decoration: underline;">告诉Client端，好了，我这边数据发完了，准备好关闭连接了</span>"。Client端收到FIN报文后，"<span style="text-decoration: underline;">就知道可以关闭连接了，但是他还是不相信网络，怕Server端不知道要关闭，所以发送ACK后进入TIME_WAIT状态，如果Server端没有收到ACK则可以重传</span>。“，Server端收到ACK后，"<span style="text-decoration: underline;">就知道可以断开连接了</span>"。Client端等待了2MSL后依然没有收到回复，则证明<span style="text-decoration: underline;">Server端已正常关闭，那好，我Client端也可以关闭连接了</span>。Ok，TCP连接就这样关闭了！</span></p>
<p style="text-align: left;"><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>为什么要三次挥手？</strong></span></span></p>
<p style="text-align: left;"><span style="font-size: 16px;">　　在只有两次“握手”的情形下，假设Client想跟Server建立连接，但是却因为中途连接请求的数据报丢失了，故Client端不得不重新发送一遍；这个时候Server端仅收到一个连接请求，因此可以正常的建立连接。但是，有时候Client端重新发送请求不是因为数据报丢失了，而是有可能数据传输过程因为网络并发量很大在某结点被阻塞了，这种情形下Server端将先后收到2次请求，并持续等待两个Client请求向他发送数据...问题就在这里，Cient端实际上只有一次请求，而Server端却有2个响应，极端的情况可能由于Client端多次重新发送请求数据而导致Server端最后建立了N多个响应在等待，因而造成极大的资源浪费！所以，“三次握手”很有必要！</span></p>
<p style="text-align: left;"><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>为什么要四次挥手？</strong></span></span></p>
<p style="text-align: left;"><span style="font-size: 16px;">　　试想一下，假如现在你是客户端你想断开跟Server的所有连接该怎么做？第一步，你自己先停止向Server端发送数据，并等待Server的回复。但事情还没有完，虽然你自身不往Server发送数据了，但是因为你们之前已经建立好平等的连接了，所以此时他也有主动权向你发送数据；故Server端还得终止主动向你发送数据，并等待你的确认。其实，说白了就是保证双方的一个合约的完整执行！</span></p>
<p style="text-align: left;"><span style="font-size: 16px;">　　使用TCP的协议：FTP（文件传输协议）、Telnet（远程登录协议）、SMTP（简单邮件传输协议）、POP3（和SMTP相对，用于接收邮件）、HTTP协议等。</span></p><h3 class="First"><span style="font-size: 16px;">8. UDP协议</span><span style="font-size: 16px;">　</span></h3>
<div class="para"><span style="font-size: 16px;">　　<span style="color: #ff0000;"><strong>UDP用户数据报协议，是面向无连接的通讯协议，UDP数据包括目的端口号和源端口号信息，由于通讯不需要连接，所以可以实现广播发送。</strong></span></span><span style="color: #ff0000;"><strong><span style="font-size: 16px;">UDP通讯时不需要接收方确认，属于不可靠的传输，可能会出现丢包现象，实际应用中要求程序员编程验证。</span></strong></span></div>
<div class="para"><span style="font-size: 16px;">　　</span><span style="font-size: 16px;">UDP与TCP位于同一层，但它不管数据包的顺序、错误或重发。因此，UDP不被应用于那些使用虚电路的面向连接的服务，UDP主要用于那些面向查询---应答的服务，例如NFS。相对于FTP或Telnet，这些服务需要交换的信息量较小。</span></div>
<div class="para">
<div class="para"><span style="font-size: 16px;">　　每个UDP报文分UDP报头和UDP数据区两部分。报头由四个16位长（2字节）字段组成，分别说明该报文的源端口、目的端口、报文长度以及校验值。UDP报头由4个域组成，其中每个域各占用2个字节，具体如下：<br></span><span style="font-size: 16px;">　　（1）源端口号；</span></div>
<div class="para"><span style="font-size: 16px;">　　（2）目标端口号；</span></div>
<div class="para"><span style="font-size: 16px;">　　（3）数据报长度；</span></div>
<div class="para"><span style="font-size: 16px;">　　（4）校验值。</span></div>
<div class="para"><span style="font-size: 16px;">　　使用UDP协议包括：TFTP（简单文件传输协议）、SNMP（简单网络管理协议）、DNS（域名解析协议）</span><span style="font-size: 16px; line-height: 1.5;">、</span><span style="font-size: 16px; line-height: 24px;">NFS、</span><span style="font-size: 16px; line-height: 1.5;">BOOTP。</span></div>
<div class="para"><span style="font-size: 16px; line-height: 1.5;"><span style="font-size: 16px; line-height: 1.5;">　　</span></span><span style="color: #ff0000;"><span style="font-size: 16px;"><strong>TCP</strong><strong>&nbsp;</strong><strong>与</strong><strong>&nbsp;</strong><strong>UDP</strong><strong>&nbsp;</strong><strong>的区别：</strong></span><span style="font-size: 16px;">TCP是面向连接的，可靠的字节流服务；UDP是面向无连接的，不可靠的数据报服务。</span></span></div>
<h3 class="First"><span style="font-size: 16px;">9. DNS协议</span></h3>
<div class="para"><span style="font-size: 16px;">　　DNS是域名系统(DomainNameSystem)的缩写，该系统用于命名组织到域层次结构中的计算机和网络服务，<span style="color: #ff0000;"><strong>可以简单地理解为将URL转换为IP地址</strong></span>。域名是由圆点分开一串单词或缩写组成的，每一个域名都对应一个惟一的IP地址，在Internet上域名与IP地址之间是一一对应的，DNS就是进行域名解析的服务器。DNS命名用于Internet等TCP/IP网络中，通过用户友好的名称查找计算机和服务。</span></div>
<div class="para">
<h3 class="First"><span style="font-size: 16px;"><span style="font-size: 16px;">10. NAT协议</span></span></h3>
<p><span style="font-size: 16px;"><strong>　　</strong>NAT网络地址转换(Network Address Translation)属接入广域网(WAN)技术，是一种将私有（保留）地址转化为合法IP地址的转换技术，它被广泛应用于各种类型Internet接入方式和各种类型的网络中。原因很简单，NAT不仅完美地解决了lP地址不足的问题，而且还能够有效地避免来自网络外部的攻击，隐藏并保护网络内部的计算机。</span></p>
<h3 class="First"><span style="font-size: 16px;">11. DHCP协议</span></h3>
<p><span style="font-size: 16px;">　　DHCP动态主机设置协议（Dynamic Host Configuration Protocol）是一个局域网的网络协议，使用UDP协议工作，主要有两个用途：给内部网络或网络服务供应商自动分配IP地址，给用户或者内部网络管理员作为对所有计算机作中央管理的手段。</span></p>

<h3 class="First"><span style="font-size: 16px;">12. HTTP协议</span></h3>
<div class="para"><span style="font-size: 16px;"><span style="font-size: 16px;">　　超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。所有的WWW文件都必须遵守这个标准。</span></span>
<p><span style="font-size: 16px;"><strong>　　HTTP</strong><strong>&nbsp;</strong><strong>协议包括哪些请求？</strong></span></p>
<p><span style="font-size: 16px;">　　GET：请求读取由URL所标志的信息。</span></p>
<p><span style="font-size: 16px;">　　POST：给服务器添加信息（如注释）。</span></p>
<p><span style="font-size: 16px;">　　PUT：在给定的URL下存储一个文档。</span></p>
<p><span style="font-size: 16px;">　　DELETE：删除给定的URL所标志的资源。</span></p>
<p><span style="font-size: 16px;"><strong>　　HTTP</strong><strong>&nbsp;</strong><strong>中，</strong><strong>&nbsp;</strong><strong>POST</strong><strong>&nbsp;</strong><strong>与</strong><strong>&nbsp;</strong><strong>GET</strong><strong>&nbsp;</strong><strong>的区别</strong></span></p>
<p><span style="font-size: 16px;">　　1）Get是从服务器上获取数据，Post是向服务器传送数据。</span></p>
<p><span style="font-size: 16px;">　　2）Get是把参数数据队列加到提交表单的Action属性所指向的URL中，值和表单内各个字段一一对应，在URL中可以看到。</span></p>
<p><span style="font-size: 16px;">　　3）Get传送的数据量小，不能大于2KB；Post传送的数据量较大，一般被默认为不受限制。</span></p>
<p><span style="font-size: 16px;">　　4）根据HTTP规范，GET用于信息获取，而且应该是安全的和幂等的。</span></p>
<p><span style="font-size: 16px;">　　I. 所谓&nbsp;<strong>安全的</strong>&nbsp;意味着该操作用于获取信息而非修改信息。换句话说，GET请求一般不应产生副作用。就是说，它仅仅是获取资源信息，就像数据库查询一样，不会修改，增加数据，不会影响资源的状态。</span></p>
<p><span style="font-size: 16px;">　　II.&nbsp;<strong>幂等</strong>&nbsp;的意味着对同一URL的多个请求应该返回同样的结果。</span></p>

<h3 class="First"><span style="font-size: 16px;">13. 一个举例</span></h3>
<div class="para">
<p><span style="font-size: 16px;"><strong>　　在浏览器中输入</strong><strong>&nbsp;</strong><a href="http://www.baidu.com/" target="_blank"><strong>www.baidu.com</strong><strong><em>&nbsp;</em></strong></a>&nbsp;<strong>后执行的全部过程</strong></span></p>
<p><span style="font-size: 16px;">　　现在假设如果我们在客户端（客户端）浏览器中输入http://www.baidu.com,而baidu.com为要访问的服务器（服务器），下面详细分析客户端为了访问服务器而执行的一系列关于协议的操作：</span></p>
<p><span style="font-size: 16px;">　　1）客户端浏览器通过DNS解析到www.baidu.com的IP地址220.181.27.48，通过这个IP地址找到客户端到服务器的路径。客户端浏览器发起一个HTTP会话到220.161.27.48，然后通过TCP进行封装数据包，输入到网络层。</span></p>
<p><span style="font-size: 16px;">　　2）在客户端的传输层，把HTTP会话请求分成报文段，添加源和目的端口，如服务器使用80端口监听客户端的请求，客户端由系统随机选择一个端口如5000，与服务器进行交换，服务器把相应的请求返回给客户端的5000端口。然后使用IP层的IP地址查找目的端。</span></p>
<p><span style="font-size: 16px;">　　3）客户端的网络层不用关系应用层或者传输层的东西，主要做的是通过查找路由表确定如何到达服务器，期间可能经过多个路由器，这些都是由路由器来完成的工作，不作过多的描述，无非就是通过查找路由表决定通过那个路径到达服务器。</span></p>
<p><span style="font-size: 16px;">　　4）客户端的链路层，包通过链路层发送到路由器，通过邻居协议查找给定IP地址的MAC地址，然后发送ARP请求查找目的地址，如果得到回应后就可以使用ARP的请求应答交换的IP数据包现在就可以传输了，然后发送IP数据包到达服务器的地址。</span></p>




