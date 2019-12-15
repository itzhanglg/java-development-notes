### 一:网络编程概述

1.  Java提供的网络类库，可以实现无痛的网络连接，联网的底层细节被隐藏在 Java 的本机安装系统里，由 JVM 进行控制。并且 Java 实现了一个跨平台的网络库， **程序员面对的是一个统一的网络编程环境。**
2.  计算机网络: 把分布在不同地理区域的计算机与专门的外部设备用通信线路互连成一个规模大、功能强的网络系统，从而使众多的计算机可以方便地互相传递信息、共享硬件、软件、数据信息等资源.
3.  网络编程的目的: **直接或间接地通过网络协议与其它计算机实现数据交换，进行通讯。**
4.  主要问题: 
    1.   **如何准确地定位网络上一台或多台主机；定位主机上特定的应用.**
    2.   **找到主机后如何可靠高效地进行数据传输.**

### 二:通信要素概述

#### 1.如何实现网络中的主机互相通信

1.  **通信双方地址: IP 和 端口号**
2.  **网络通信协议: OSI参考模型 和 TCP/IP参考模型(协议)**

#### 2.网络通信协议
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124164205713.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)


#### 3.数据传输
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124164212660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

### 三:IP和端口号
#### 1.IP地址: InetAddress

1.  唯一的标识 Internet 上的计算机（通信实体）
2.  本地回环地址(hostAddress)：127.0.0.1 主机名(hostName)：localhost
3.  IP地址分类方式1：**IPV4 和 IPV6**
    1.  IPV4: 4个字节组成,4个0-255. 2011年初已经用尽.以点分十进制表示,如192.168.0.1
    2.  IPV6: 16个字节组成, 128位,写成8个无符号整数, 每个整数用四个十六进制位表示,数之间用冒号(:)分开,如3ffe:3201:1401:1280:c8ff:fe4d:db39:1984
4.  IP地址分类方式2： **公网地址( 万维网使用)和 私有地址( 局域网使用)。192.168.开头的就是私有址址，范围即为192.168.0.0--192.168.255.255，专门为组织机构内部使用**

#### 2.端口号

1.  标识正在计算机上运行的进程（程序）,不同的进程有不同的端口号
2.  被规定为一个 16 位的整数 0~65535
3.  端口分类:
    1.  **公认端口：0~1023**。被预先定义的服务通信占用（如：HTTP占用端口80，FTP占用端口21，Telnet占用端口23）
    2.  **注册端口：1024~49151**。分配给用户进程或应用程序。（如：Tomcat占用端口8080，MySQL占用端口3306，Oracle占用端口1521等）
    3.  **动态/ 私有端口：49152~65535**.

**IP地址与端口号的组合得出一个网络套接字：Socket。**

#### 3.InetAddress类

1.  Internet上的主机有两种方式表示地址：

    **域名(hostName)**：www.zhangligong.xyz
    **IP 地址(hostAddress)**：220.250.64.225

2.  InetAddress类主要表示IP地址，两个子类：Inet4Address、Inet6Address。

3.  域名容易记忆，当在连接网络时输入一个主机的域名后，**域名服务器(DNS)负责将域名转化成IP地址**，这样才能和主机建立连接。(先找本机hosts，是否有输入的域名地址，没有的话，再通过DNS服务器，找主机。) ------- **域名解析**

4.  InetAddress 类没有提供公共的构造器，而是提供了如下几个静态方法来获取InetAddress 实例

    `public static InetAddress getLocalHost()`
    `public static InetAddress getByName(String host)`

5.  InetAddress 提供了如下几个常用的方法

    **public String getHostAddress()** ：返回 IP 地址字符串（以文本表现形式）。
    **public String getHostName()** ：获取此 IP 地址的主机名
    **public boolean isReachable(int timeout)**： ：测试是否可以达到该地址

6.  示例

    ```java
    import java.net.InetAddress;
    import java.net.UnknownHostException;

    /**
     * 一、网络编程中有两个主要的问题：
     * 1.如何准确地定位网络上一台或多台主机；定位主机上的特定的应用
     * 2.找到主机后如何可靠高效地进行数据传输
     *
     * 二、网络编程中的两个要素：
     * 1.对应问题一：IP和端口号
     * 2.对应问题二：提供网络通信协议：TCP/IP参考模型（应用层、传输层、网络层、物理+数据链路层）
     *
     * 三、通信要素一：IP和端口号
     * 1. IP:唯一的标识 Internet 上的计算机（通信实体）
     * 2. 在Java中使用InetAddress类代表IP
     * 3. IP分类：IPv4 和 IPv6 ; 万维网 和 局域网
     * 4. 域名:   www.baidu.com   www.mi.com  www.sina.com  www.jd.com
     *            www.vip.com
     * 5. 本地回路地址：127.0.0.1 对应着：localhost
     * 6. 如何实例化InetAddress:两个方法：getByName(String host) 、 getLocalHost()
     *        两个常用方法：getHostName() / getHostAddress()
     * 7. 端口号：正在计算机上运行的进程。
     * 要求：不同的进程有不同的端口号
     * 范围：被规定为一个 16 位的整数 0~65535。
     * 8. 端口号与IP地址的组合得出一个网络套接字：Socket
     *
     * @author zlg
     * @create 2019-10-16 1:01
     */
    public class InetAdressTest {
        @Test
        public void ipTest(){
            try {
                //根据域名返回InetAddress对象
                InetAddress inet = InetAddress.getByName("zhangligong.com");
                System.out.println(inet);   // zhangligong.com/220.250.64.225

                //获取本地ip
                InetAddress lh = InetAddress.getByName("localhost");
                System.out.println(lh);  // localhost/127.0.0.1

                //获取本地ip
                InetAddress localHost = InetAddress.getLocalHost();
                System.out.println(localHost);  // IT-Xiaobai/169.254.34.219

                //获取IP名
                String hostName = localHost.getHostName();
                System.out.println(hostName);   // IT-Xiaobai
                //获取IP地址
                String hostAddress = localHost.getHostAddress();
                System.out.println(hostAddress);    // 169.254.34.219

            } catch (UnknownHostException e) {
                e.printStackTrace();
            }
        }
    }
    ```

### 四:通信协议

**计算机网络中实现通信必须有一些约定，即通信协议，对速率、传输代码、代码结构、传输控制步骤、出错控制等制定标准。**

#### 1.TCP/IP协议簇

1.  传输层协议中有两个非常重要的协议：**传输控制协议TCP(Transmission Control Protocol) 和 用户数据报协议UDP(User Datagram Protocol)**。
2.  **TCP/IP  以其两个主要协议：传输控制协议(TCP) 和网络互联协议(IP)**而得名，实际上是一组协议，包括多个具有不同功能且互为关联的协议。
3.  IP(Internet Protocol)协议是**网络层**的主要协议，支持网间互连的数据通信。
4.  TCP/IP协议模型从更实用的角度出发，形成了高效的四层体系结构，即 **物理链路层、IP 层、传输层和应用层。**

#### 2.TCP协议

1.  使用TCP协议前，须先**建立TCP连接，形成传输数据通道**
2.  传输前，采用“ **三次握手**”方式，**点对点通信，是可靠的**
3.  TCP协议进行通信的两个应用进程：**客户端、服务端**
4.  在连接中可**进行大数据量的传输**
5.  传输完毕，需**释放已建立的连接，效率低**

#### 3.UDP协议

1.  将数据、源、目的封装成数据包，**不需要建立连接**
2.  每个数据报的大小**限制在64K内**
3.  发送不管对方是否准备好，接收方收到也不确认，故是**不可靠的**
4.  可以广播发送
5.  发送数据结束时**无需释放资源，开销小，速度快**

#### 4.TCP三次握手
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124164236346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 5.TCP四次挥手
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124164246341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 6.Socket套接字

1.  **网络上具有唯一标识的IP地址和端口号组合在一起才能构成唯一能识别的标识符套接字。**

2.  通信的两端都要有Socket，是两台机器间通信的端点。网络通信其实就是Socket间的通信。

3.  Socket允许程序把网络连接当成一个流，数据在两个Socket间通过IO传输。

4.   一般主动发起通信的应用程序属客户端，等待通信请求的为服务端。

5.  Socket分类：

    1.  **流套接字（stream socket）：使用TCP提供可依赖的字节流服务**
    2.  **数据报套接字（datagram socket）：使用UDP提供“尽力而为”的数据报服务**

6.  常用API

  **Socket 类的常用构造器 ：**
   - `public Socket(InetAddress address,int port)`创建一个流套接字并将其连接到指定IP地址的指定端口号。
   - `public Socket(String host,int port)`创建一个流套接字并将其连接到指定主机上的指定端口号。

   **Socket 类的常用方法：**
  -    `public InputStream getInputStream()`返回此套接字的输入流。可以用于接收网络消息
  -    `public OutputStream getOutputStream()`返回此套接字的输出流。可以用于发送网络消息
  -    `public InetAddress getInetAddress()`此套接字连接到的远程 IP 地址；如果套接字是未连接的，则返回 null。
  -    `public InetAddress getLocalAddress()`获取套接字绑定的本地地址。 即本端的IP-地址
  -    `public int getPort()`此套接字连接到的远程端口号；如果尚未连接套接字，则返回0。
  -    `public int getLocalPort()`返回此套接字绑定到的本地端口。 如果尚未绑定套接字，则返回-1。即本端的端口号。
  -    `public void close()`关闭此套接字。套接字被关闭后，便不可在以后的网络连接中使用（即无法重新连接或重新绑定）。需要创建新的套接字对象。 关闭此套接字也将会关闭该套接字的InputStream 和OutputStream。
  -    `public void shutdownInput()`如果在套接字上调用 shutdownInput() 后从套接字输入流读取内容，则流将返回 EOF（文件结束符）。即不能在从此套接字的输入流中接收任何数据。
  -    `public void shutdownOutput()`禁用此套接字的输出流。对于 TCP 套接字，任何以前写入的数据都将被发送，并且后跟 TCP 的正常连接终止序列。 如果在套接字上调用 shutdownOutput() 后写入套接字输出流，则该流将抛出 IOException。 即不能通过此套接字的输出流发送任何数据。


### 五:TCP网络编程

#### 1.Socket的TCP编程

Java语言的基于套接字编程分为服务端编程和客户端编程，其通信模型如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124164710921.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poaXhpbmd3dQ==,size_16,color_FFFFFF,t_70)

#### 2.客服端Socket的工作过程

1.  创建 Socket ：根据指定服务端的 IP 地址或端口号构造 Socket 类对象。**创建的同时会自动向服务器方发起连接,若服务器端响应，则建立客户端到服务器的通信线路。若连接失败，会出现异常。**
2.  打开连接到 Socket  的输入/ 出流： 使用 **getInputStream()**方法获得输入流，使用**getOutputStream()**方法获得输出流，进行数据传输.
3.  按照一定的协议对 Socket 进行读/ 写操作：通过输入流读取服务器放入线路的信息（但不能读取自己放入线路的信息），通过输出流将信息写入线程。
4.  关闭 Socket：断开客户端到服务器的连接，释放线路

#### 3.服务器程序的工作过程

1.  调用 ServerSocket(int port) ：创建一个服务器端套接字，并绑定到指定端口上。用于监听客户端的请求。**服务器必须事先建立一个等待客户请求建立套接字的 连接的ServerSocket 对象**。
2.  调用 accept()：**监听连接请求，如果客户端请求连接，则接受连接，返回Socket通信套接字对象。**
3.  调用该Socket 类对象的 getOutputStream() 和 和 getInputStream ()：获取输出流和输入流，开始网络数据的发送和接收。
4.  关闭ServerSocket 和Socket 对象：客户端访问结束，关闭通信套接字。

#### 4.示例

```java
import java.io.*;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * 实现TCP的网络编程
 * 示例：从客户端发送文件给服务端，服务端保存到本地。并返回“发送成功”给客户端。
 * 并关闭相应的连接。
 *
 * @author zlg
 * @create 2019-10-20 15:34
 */
public class TCPTest {

    /**
     * 客户端
     * @throws IOException
     */
    @Test
    public void clientTest() throws IOException {
        //1.创建Socket对象，指明服务器端的ip和端口号
        Socket socket = new Socket(InetAddress.getByName("127.0.0.1"), 8989);
        //2.获取一个输出流，用于输出数据
        OutputStream os = socket.getOutputStream();
        //3.创建缓冲流,用于读取文件
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream("beauty.jpg"));
        //4.写出数据的操作,发送文件
        byte[] buffer = new byte[1024];
        int len;
        while ( (len = bis.read(buffer)) != -1){
            os.write(buffer,0,len);
        }
        //关闭数据的输出(IO流是阻塞的)
        //会在流末尾写入一个标记,对方才能读到-1,否则对方的读取方法会一致阻塞
        socket.shutdownOutput();

        //5.接收来自于服务器端的数据，并显示到控制台上
        InputStream is = socket.getInputStream();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] buf = new byte[30];
        int len2;
        while ( (len2 = is.read(buf)) != -1){
            baos.write(buf,0,len2);
        }
        System.out.println(baos.toString());

        //6.资源的关闭
        baos.close();
        bis.close();
        //关闭socket, is,os意味着也关闭了
        socket.close();
    }

    /**
     * 服务端
     * @throws IOException
     */
    @Test
    public void serverTest() throws IOException {

        //1.创建服务器端的ServerSocket，指明自己的端口号
        ServerSocket ss = new ServerSocket(8989);
        //2.调用accept()表示接收来自于客户端的socket
        //监听一个客服端的连接,该方法是个阻塞方法,若没有连接,会一直等待
        Socket socket = ss.accept();
        //3.获取输入流
        InputStream is = socket.getInputStream();
        //4.创建输出流,用于保存文件
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("beauty_dest.jpg"));
        //5.读取数据,保存文件到本地
        byte[] buffer = new byte[1024];
        int len;
        while ( (len = is.read(buffer)) != -1){
            bos.write(buffer,0,len);
        }
        System.out.println("照片已保存!");

        //6.服务器端给予客户端反馈
        OutputStream os = socket.getOutputStream();
        os.write("你好,美美,你发的照片已收到!".getBytes());

        //7.关闭资源
        bos.close();
        socket.close();
        //不再接受任何客服端通信
        ss.close();
    }

}
```

### 六:UDP网络编程

#### 1.UDP网络通信

1.  类 DatagramSocket 和 DatagramPacket 实现了基于 UDP 协议网络程序。
2.  UDP数据报通过数据报套接字 DatagramSocket 发送和接收，**系统不保证UDP数据报一定能够安全送到目的地，也不能确定什么时候可以抵达。**
3.  DatagramPacket 对象封装了UDP数据报，在数据报中包含了**发送端的IP地址和端口号以及接收端的IP地址和端口号。**
4.  UDP协议中每个数据报都给出了完整的地址信息，因此无须建立发送方和接收方的连接。如同发快递包裹一样。

#### 2.DatagramSocket类
**常用方法**
- `public DatagramSocket(int port)`创建数据报套接字并将其绑定到本地主机上的指定端口。套接字将被
  绑定到通配符地址，IP 地址由内核来选择。
- `public DatagramSocket(int port,InetAddress laddr)`创建数据报套接字，将其绑定到指定的本地地址。本地端口必须在 0 到 65535 之间（包括两者）。如果 IP 地址为 0.0.0.0，套接字将被绑定到通配符地址，IP 地址由内核选择。
- `public void close()`关闭此数据报套接字。
- `public void send(DatagramPacket p)`从此套接字发送数据报包。DatagramPacket 包含的信息指示：将要发送的数据、其长度、远程主机的IP 地址和远程主机的端口号。
- `public void receive(DatagramPacket p)`从此套接字接收数据报包。当此方法返回时，DatagramPacket的缓冲区填充了接收的数据。数据报包也包含发送方的 IP 地址和发送方机器上的端口号。 此方法在接收到数据报前一直阻塞。数据报包对象的 length 字段包含所接收信息的长度。如果信息比包的长度长，该信息将被截短。
- `public InetAddress getLocalAddress()`获取套接字绑定的本地地址。
- `public int getLocalPort()`返回此套接字绑定的本地主机上的端口号。
- `public InetAddress getInetAddress()`返回此套接字连接的地址。如果套接字未连接，则返回null。
- `public int getPort()`返回此套接字的端口。如果套接字未连接，则返回-1。


#### 3.DatagramPacket类
**常用方法**
- `public DatagramPacket(byte[] buf,int length)`构造 DatagramPacket，用来接收长度为 length 的数据包。 length 参数必须小于等于 buf.length。
- `public DatagramPacket(byte[] buf,int length,InetAddress address,int port)`构造数据报包，用来将长度为 length 的包发送到指定主机上的指定端口号。length参数必须小于等于buf.length。
- `public InetAddress getAddress()`返回某台机器的 IP 地址，此数据报将要发往该机器或者是从该机器接收到的。
- `public int getPort()`返回某台远程主机的端口号，此数据报将要发往该主机或者是从该主机接收到的。
- `public byte[] getData()`返回数据缓冲区。接收到的或将要发送的数据从缓冲区中的偏移量 offset 处开始，持续 length 长度。
- `public int getLength()`返回将要发送或接收到的数据的长度。


#### 4.流程

1.  DatagramSocket 与 DatagramPacket
2.  建立发送端,接受端
3.  建立数据包
4.  调用Socket的发送,接受方法
5.  关闭Socket

**发送端与接收端是两个独立的运行程序.**

#### 5.示例

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

/**
 * 实现UDP协议的网络编程
 *
 * @author zlg
 * @create 2019-10-20 17:10
 */
public class UDPTest {

    /**
     * 发送者
     */
    @Test
    public void senderTest() {
        DatagramSocket socket = null;
        try {
            //1.创建DatagramSocket对象
            socket = new DatagramSocket();

            //2.创建数据包,存储数据
            byte[] data = "你好,我使用UDP传输数据给你了".getBytes();
            DatagramPacket packet = new DatagramPacket(data, data.length, InetAddress.getLocalHost(), 8989);

            //3.发送数据包
            socket.send(packet);

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            socket.close();
        }
    }


    /**
     * 接收者
     */
    @Test
    public void receiverTest() {

        DatagramSocket socket = null;
        try {
            //1.创建DatagramSocket对象
            socket = new DatagramSocket(8989);

            //2.创建数据包,接受发送过来的数据,发送多少,存储多少
            byte[] buffer = new byte[1024];
            DatagramPacket packet = new DatagramPacket(buffer,0, buffer.length);

            //3.接受数据,保存在packet中
            socket.receive(packet);

            //4.打印输出接受到的数据
            System.out.println(new String(packet.getData(), 0, packet.getLength()));

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            socket.close();
        }
    }

}
```

### 七:URL网络编程

#### 1.URL类

1.  **URL(Uniform Resource Locator)：统一资源定位符，它表示 Internet 上 某一资源的地址。**

2.  通过 URL 我们可以访问 Internet 上的各种网络资源，比如最常见的 www，ftp站点。浏览器通过解析给定的 URL 可以在网络上查找相应的文件或其他资源。

3.  URL的基本结构由5部分组成：

    1.  **< 传输协议>://< 主机名>:< 端口号>/< 文件名># 片段名?参数列表**
    2.  片段名：即锚点，例如看小说，直接定位到章节
    3.  参数列表格式：参数名=参数值&参数名=参数值....

4.  URL类构造器:

    1.  `public URL (String spec)`：通过一个表示URL地址的字符串可以构造一个URL对象。例
        如：URL url = new URL ("http://www.zhangligong.xyz/");
    2.  `public URL(URL context, String spec)`：通过基 URL 和相对 URL 构造一个 URL 对象。
        例如：URL downloadUrl = new URL(url, “download.html")
    3.  `public URL(String protocol, String host, String file)`; 例如：new URL("http",
        "www.zhangligong.xyz", “download. html");
    4.  `public URL(String protocol, String host, int port, String file)`; 例如: URL gamelan = new
        URL("http", "www.zhangligong.xyz", 80, “download.html");

    URL类的构造器都声明抛出非运行时异常，必须要对这一异常进行处理，通常是用 try-catch 语句进行捕获。

5.  URL类常用方法

    1.  `public String getProtocol( )` 获取该URL的协议名
    2.  `public String getHost( )` 获取该URL的主机名
    3.  `public String getPort( )` 获取该URL的端口号
    4.  `public String getPath( )` **获取该URL的文件路径**
    5.  `public String getFile( )` 获取该URL的文件名
    6.  `public String getQuery( )` **获取该URL的查询名**

#### 2.URL类示例

```java
import java.net.URL;
	// 获取URL对象的相关属性信息
		try {
            URL url = new URL("http://localhost:8080/helloworld/index.html?username=jack");

//          1.获取该URL的协议名
            System.out.println(url.getProtocol());  // http
//          2.获取该URL的主机名
            System.out.println(url.getHost());    // localhost
//          3.获取该URL的端口号
            System.out.println(url.getPort());    // 8080
//          4.获取该URL的文件路径
            System.out.println(url.getPath());    // /helloworld/index.html
//          5.获取该URL的文件名
            System.out.println(url.getFile());    // /helloworld/index.html?username=jack
//          6.获取该URL的查询名
            System.out.println(url.getQuery());   // username=jack

        } catch (MalformedURLException e) {
            e.printStackTrace();
        }
```

#### 3.URLConnection类

1.  **URL的方法 openStream()：能从网络上读取数据**
2.  若希望输出数据，例如向服务器端的 CGI （公共网关接口-Common GatewayInterface-的简称，是用户浏览器和服务器端的应用程序进行连接的接口）程序发送一些数据，则必须先与URL建立连接，然后才能对其进行读写，此时需要使用URLConnection 。
3.  URLConnection：**表示到URL所引用的远程对象的连接。当与一个URL建立连接时，首先要在一个 URL 对象上通过方法 openConnection() 生成对应的 URLConnection对象。**如果连接过程失败，将产生IOException.
4.  通过URLConnection对象获取的**输入流和输出流**，即可以与现有的CGI程序进行交互。

#### 4.URLConnection类示例

```java
	// 获取url定位的资源,保存到本地
		//1.创建url对象
        URL url = new URL("http://localhost:8080/helloworld/index.html?username=jack");
        //2.创建Http协议的rulConnection对象
        HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
        //3.进行连接
        urlConnection.connect();
        //4.获取输入流
        InputStream is = urlConnection.getInputStream();
        //5.创建输出流
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("index_dest.html"));
        //6.数据处理,将数据保存到本地
        byte[] buffer = new byte[1024];
        int len;
        while ( (len = is.read(buffer)) != -1){
            bos.write(buffer,0,len);
        }
        System.out.println("资源下载完成!");
        //7.关闭资源
        bos.close();
        is.close();
        urlConnection.disconnect();
```

#### 5.URI,URL,URN区别

**URI，是uniform resource identifier，统一资源标识符**，用来唯一的标识一个资源。而**URL是uniform resource locator，统一资源定位符**，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。而**URN，uniform resource name，统一资源命名**，是通过名字来标识资源，比如mailto:java-net@java.sun.com。也就是说，**URI是以一种抽象的，高层次概念定义统一资源标识，而URL和URN则是具体的资源标识的方式。URL和URN都是一种URI。**

在Java的URI中，一个URI实例可以代表绝对的，也可以是相对的，只要它符合URI的语法规则。而**URL类**则不仅符合语义，还包含了定位该资源的信息，因此它**不能是相对的**

### 八:小结

1.  计算机具有唯一的**IP地址**，这样不同的主机可以互相区分
2.  **端口号**是对一个服务的访问场所，它用于区分同一物理计算机上的多个服务。
3.  **套接字**用于连接客户端和服务器，客户端和服务器之间的每个通信会话使用一个不同的套接字。TCP协议用于实现面向连接的会话。
4.  Java 用 **InetAddress 对象**表示 IP地址，该对象里有两个字段：主机名(String) 和 IP 地址(int)。
5.  **类 Socket 和 ServerSocket** 实现了基于TCP协议的客户端－服务器程序。Socket是客户端和服务器之间的一个连接，连接创建的细节被隐藏了。这个连接提供了一个**安全的数据传输通道**，这是因为 TCP 协议可以解决数据在传送过程中的丢失、损坏、重复、乱序以及网络拥挤等问题，它保证数据可靠的传送。
6.  **类 URL 和 URLConnection** 提供了最高级网络应用。URL 的网络资源的位置来同一表示Internet 上各种网络资源。通过URL对象可以创建当前应用程序和 URL 表示的网络资源之间的连接，这样当前程序就可以**读取网络资源数据**，或者把自己的数据传送到网络上去。




