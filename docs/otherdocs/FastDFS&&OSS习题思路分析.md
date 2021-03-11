## 习题一

业务场景：用户上传的头像图片大小不一、手机和PC等设备显示尺寸也存在差异，因此需要能根据http请求指定的尺寸参数对图片进行动态压缩，以达到屏幕适配的作用。

功能需求：搭建FastDFS图片服务器，通过http请求访问服务器中图片时，显示动态缩略图。具体要求如下：

- 在Linux系统中安装FastDFS服务器
- 可以使用FastDFS自带的工具将文件上传到FastDFS
- 通过http访问某个图片时，FastDFS通过GraphicsMagick工具生成缩略图，将动态缩略图响应输出

作业提示：

- GraphicsMagick选择1.3.18版本，安装完成之后测试是否支持png、jpg，以及验证gm是否可用
- 安装LuaJIT-2.0.4、lua-nginx-module-0.8.10和ngx_devel_kit-0.2.18
- 安装Nginx选择nginx-1.4.2版本，安装完成后测试lua环境是否支持，比如输出hello world
- 在nginx中配置lua脚本，在lua脚本中使用gm命令完成图片压缩

实现思路、环境介绍。提供Linux下的安装步骤文档

### 软件环境

1.环境介绍

| 环境                      | 版本 |
| ------------------------- | ---- |
| 虚拟机&VMware Workstation | 10.0 |
| 服务器&CentOS             | 7.8  |
| 远程连接&Xshell           | 6    |
| 远程文件传输&WinSCP、Xftp | 6    |

2.Linux下各个软件的版本

| 软件名称         | 版本   |
| ---------------- | ------ |
| fastdfs5.11      | 5.11   |
| GraphicsMagick   | 1.3.18 |
| LuaJIT           | 2.0.4  |
| lua-nginx-module | 0.8.10 |
| ngx_devel_kit    | 0.2.18 |
| nginx            | 1.4.2  |

3.知识回顾

FastDFS是一个开源的 轻量级 分布式文件系统。它解决了大数据量存储和负载均衡问题。适合以中小[4k-500M]文件为载体的在线服务。

- 对等结构，不存在单点问题（对等结构）
- 文件**分组存储**不分块存储，上传的文件和OS文件系统中的文件一一对应（分组存储）
- 文件ID有Storage生成后返回给Client，Tracker不需要存储文件的索引信息（轻量级）

FastDFS由客户端(Client)、 跟踪服务器(Tracker Server)和存储服务器(Storage Server)构成。

![image-20200914175212093](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200914175212093.png)

- Client作为业务请求的发起方
- Tracker Server作用是负载均衡和调度
- Storage Server作用是文件存储

文件上传原理：

![image-20200914175310244](https://gitee.com/itzlg/mypictures/raw/master/img/image-20200914175310244.png)

- 1.选择tracker server和group
- 2.选择storage server
- 3.选择storage path
- 4.生成文件名
- 5.返回文件id

业务场景：

- nginx 头像缩略图：上传图片时直接处理成缩略图；先存储图片，使用时实时计算
- 避免文件重复上传 ：上传成功后计算文件对应的MD5然后存入MySQL

### 实现步骤

#### 1.安装fastdfs

1.安装编译环境

> yum install git gcc gcc-c++ make automake vim wget libevent -y

2.安装libfastcommon 基础库

```
mkdir /root/fastdfs

cd /root/fastdfs
git clone https://github.com/happyfish100/libfastcommon.git --depth 1

cd libfastcommon/
./make.sh && ./make.sh install
```

3.安装FastDFS

```
cd /root/fastdfs
wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz

tar -zxvf V5.11.tar.gz
cd fastdfs-5.11
./make.sh && ./make.sh install
```

4.配置文件准备并修改

```
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf 
cp /root/fastdfs/fastdfs-5.11/conf/http.conf /etc/fdfs
cp /root/fastdfs/fastdfs-5.11/conf/mime.types /etc/fdfs
```

编辑tracker.conf

```
vim /etc/fdfs/tracker.conf

#需要修改的内容如下
port=22122 
base_path=/home/fastdfs
```

编辑storage.conf

```
vim /etc/fdfs/storage.conf

#需要修改的内容如下
port=23000 
base_path=/home/fastdfs # 数据和日志文件存储根目录
store_path0=/home/fastdfs # 第一个存储目录
tracker_server=192.168.91.108:22122

# http访问文件的端口(默认8888,看情况修改,和nginx中保持一致)
http.server_port=8888
```

5.启动tracker和storage服务端

```
mkdir /home/fastdfs -p

/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart

查看所有运行的端口
netstat -ntlp
```

6.使用fastdfs自带的client进行测试上传

```
vim /etc/fdfs/client.conf

#需要修改的内容如下
base_path=/home/fastdfs
#tracker服务器IP和端口
tracker_server=192.168.91.108:22122

#保存后测试,返回ID表示成功 如：group1/M00/00/00/xxx.png
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /root/fastdfs/aa.jpg

#返回file_id
group1/M00/00/00/wKjTiF7h5EWASb5aAACGZa9JdFo611.jpg
```

#### 2.安装GraphicsMagick

1.下载安装包并解压：

>wget http://ftp.icm.edu.pl/pub/unix/graphics/GraphicsMagick/1.3/GraphicsMagick-1.3.18.tar.gz
>
>tar -zxvf GraphicsMagick-1.3.18.tar.gz

2.安装插件：

```
#如果png,jpg 都是no 需要添加支持
yum install -y libjpeg-devel libjpeg
yum install -y libpng-devel libpng
yum install -y giflib-devel giflib

#freetype 支持
yum -y install libjpeg libjpeg-devel libpng libpng-devel giflib giflib-devel freetype freetype-devel
```

3.配置、编译并安装：

```
cd GraphicsMagick-1.3.18

./configure -enable-shared -enable-lzw -without-perl -with-modules
或者
./configure --prefix=/root/GraphicsMagick-1.3.18 --with-quantum-depth=8  --enable-shared --enable-static

#若configure提示“configure: error: libltdl is required for modules build”，安装相应依赖
yum install libtool-ltdl libtool-ltdl-devel

make
make install
```

4.查看gm版本信息：

```
gm version

#若出现如下表示支持png、jpg
JPEG                     yes
PNG                      yes
```

5.测试裁剪图片：上传aa.jpg图片后

```
gm convert aa.jpg bb.jpg
gm convert aa.jpg -resize 100x100 cc.jpg
```

#### 3.安装LuaJIT(lua即时编译器)

1.下载并解压

```
wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz  
tar -zxvf   LuaJIT-2.0.4.tar.gz
```

2.编译并安装

```
cd  LuaJIT-2.0.4
make && sudo make install

#出现如下信息安装成功
==== Successfully installed LuaJIT 2.0.4 to /usr/local ====

#它的安装前缀是/usr/local,我们更新动态库
ln -s /usr/local/lib/libluajit-5.1.so.2 /usr/lib/libluajit-5.1.so.2

#或者手动添加
vim /etc/ld.so.conf
#添加如下信息
echo “/usr/local/lib” >> /etc/ld.so.conf
ldconfig

#查看ld.so.conf文件
cat /etc/ld.so.conf
include ld.so.conf.d/*.conf
echo “/usr/local/lib” >> /etc/ld.so.conf
ldconfig
```

3.配置环境变量

```
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.0
```

4.使用lua shell测试

```
[root@atzlg4 LuaJIT-2.0.4]# lua
Lua 5.1.4  Copyright (C) 1994-2008 Lua.org, PUC-Rio
> print("Hello World")
Hello World
```

#### 4.下载解压其它模块包

1.下载和解压 lua-nginx-module-0.8.10.tar.gz（lua nginx模块）

```
wget -O lua-nginx-module-0.8.10.tar.gz https://github.com/chaoslawful/lua-nginx-module/archive/v0.8.10.tar.gz

tar -zxvf lua-nginx-module-0.8.10.tar.gz
```

2.下载和解压 ngx_devel_kit-0.2.18.tar.gz（nginx开发工具包）

```
wget -O ngx_devel_kit-0.2.18.tar.gz https://github.com/simpl/ngx_devel_kit/archive/v0.2.18.tar.gz

tar -zxvf ngx_devel_kit-0.2.18.tar.gz
```

#### 5.安装nginx 

1.下载并解压nginx-1.4.2.tar.gz

```
wget http://nginx.org/download/nginx-1.4.2.tar.gz
tar -zxvf  nginx-1.4.2.tar.gz
```

2.配置相应所需模块fastdfs-nginx-module-1.20、lua-nginx-module-0.8.10、ngx_devel_kit-0.2.18

```
cd nginx-1.4.2

./configure  --add-module=/root/fastdfs/fastdfs-nginx-module-1.20/src --add-module=/root/fastdfs/lua-nginx-module-0.8.10 --add-module=/root/fastdfs/ngx_devel_kit-0.2.18
```

3.编译并安装

```
yum -y install pcre-devel openssl openssl-devel

#编译安装
make && make install

#查看模块是否安装上
/usr/local/nginx/sbin/nginx -V
```

4.安装问题

nginx lua 查看nginx 版本时报错找不到libluajit-5.1.so.2：

> [root@atzlg4 nginx-1.4.2]# /usr/local/nginx/sbin/nginx -V
> /usr/local/nginx/sbin/nginx: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory

查找安装的libluajit-5.1.so.2：

> find / -name libluajit-5.1.so.2

由于编译时没有生成动态链接库，只能手动链接：

```
vim /etc/ld.so.conf.d/libc.conf

#添加如下信息，加入自己编译安装后的lib目录
/usr/local/lib
/usr/local/luajit/lib

#再次运行
ldconfig
```

再次查看nginx版本信息：

```
/usr/local/nginx/sbin/nginx -V

[root@atzlg4 ld.so.conf.d]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.4.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
configure arguments: --add-module=/root/fastdfs/fastdfs-nginx-module-1.20/src --add-module=/root/fastdfs/lua-nginx-module-0.8.10 --add-module=/root/fastdfs/ngx_devel_kit-0.2.18
```

#### 6.编辑nginx配置文件

> vim /usr/local/nginx/conf/nginx.conf

文件：

```
server {
        listen       8888;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        #location ~/group[0-9]/ {
        #    ngx_fastdfs_module;
        #}
        location / {
            root   /home/fastdfs/data/00/00;
            #index  index.html index.htm;
        }
        location /test {
            default_type text/html;
            content_by_lua '
            ngx.say("hello world")
            ngx.log(ngx.ERR,"err err")
            ';
        }
        # fastdfs 缩略图生成
        location ~/group[0-9]/M00 {
            alias /home/fastdfs/data;

            set $image_root "/home/fastdfs/data";
            if ($uri ~ "/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/([a-zA-Z0-9]+)/(.*)") {
                set $image_dir "$image_root/$3/$4/";
                set $image_name "$5";
                set $file "$image_dir$image_name";
            }

            if (!-f $file) {
                # 关闭lua代码缓存，方便调试lua脚本
                #lua_code_cache off;
                content_by_lua_file  /usr/local/nginx/conf/img_crop.lua;
            }
            ngx_fastdfs_module;
        }
}
```

#### 7.编写lua脚本

> cd /usr/local/nginx/conf/img_crop.lua
>
> vi img_crop.lua

文件：

```
-- 写入文件
local function writefile(filename, info)
    local wfile=io.open(filename, "w") --写入文件(w覆盖)
    assert(wfile)  --打开时验证是否出错		
    wfile:write(info)  --写入传入的内容
    wfile:close()  --调用结束后记得关闭
end

-- 检测路径是否目录
local function is_dir(sPath)
    if type(sPath) ~= "string" then return false end

    local response = os.execute( "cd " .. sPath )
    if response == 0 then
        return true
    end
    return false
end

-- 检测文件是否存在
local file_exists = function(name)
    local f=io.open(name,"r")
    if f~=nil then io.close(f) return true else return false end
end

local area = nil
local originalUri = ngx.var.uri;
local originalFile = ngx.var.file;
local index = string.find(ngx.var.uri, "([0-9]+)x([0-9]+)");  
if index then 
    originalUri = string.sub(ngx.var.uri, 0, index-2);  
    area = string.sub(ngx.var.uri, index);  
    index = string.find(area, "([.])");  
    area = string.sub(area, 0, index-1);  

    local index = string.find(originalFile, "([0-9]+)x([0-9]+)");  
    originalFile = string.sub(originalFile, 0, index-2)
end

-- check original file
if not file_exists(originalFile) then
    local fileid = string.sub(originalUri, 2);
    -- main
    local fastdfs = require('restyfastdfs')
    local fdfs = fastdfs:new()
    fdfs:set_tracker("192.168.211.136", 22122)
    fdfs:set_timeout(1000)
    fdfs:set_tracker_keepalive(0, 100)
    fdfs:set_storage_keepalive(0, 100)
    local data = fdfs:do_download(fileid)
    if data then
       -- check image dir
        if not is_dir(ngx.var.image_dir) then
            os.execute("mkdir -p " .. ngx.var.image_dir)
        end
        writefile(originalFile, data)
    end
end

-- 创建缩略图
local image_sizes = {"80x80", "800x600", "40x40", "60x60"};  
function table.contains(table, element)  
    for _, value in pairs(table) do  
        if value == element then
            return true  
        end  
    end  
    return false  
end 

if table.contains(image_sizes, area) then  
    local command = "gm convert " .. originalFile  .. " -thumbnail " .. area .. " -background gray -gravity center -extent " .. area .. " " .. ngx.var.file;  
    os.execute(command);  
end;

if file_exists(ngx.var.file) then
    --ngx.req.set_uri(ngx.var.uri, true);  
    ngx.exec(ngx.var.uri)
else
    ngx.exit(404)
end
```

#### 8.启动nginx测试

改变目录的权限：

> chmod -R 777 /home/fastdfs/data/

启动nginx 测试：

```
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -s reload

#测试下载
#关闭防火墙
systemctl stop firewalld

#测试lua脚本
http://192.168.211.136:8888/test
# 输出
hello world

#上传文件
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /home/ee.jpg
group1/M00/00/00/wKhbbF9fVcCAaLaOAABVrds44e4028.jpg

#web访问图片（图片尺寸在lua脚本中做了限制）
#local image_sizes = {"80x80", "800x600", "40x40", "60x60"}; 
http://192.168.91.108:8888/group1/M00/00/00/wKhbbF9fVcCAaLaOAABVrds44e4028.jpg
http://192.168.91.108:8888/group1/M00/00/00/wKhbbF9fVcCAaLaOAABVrds44e4028.jpg_800x600.jpg

#web访问（需要在原始路径访问图片之后才能访问到，否则会报404）
http://192.168.91.108:8888/wKhbbF9fVcCAaLaOAABVrds44e4028.jpg
http://192.168.91.108:8888/wKhbbF9fVcCAaLaOAABVrds44e4028.jpg_800x600.jpg
```





## 习题二

使用SpringBoot和OSS实现图片的上传、下载和删除功能，  具体要求如下：

1.可以使用postman  发送上传请求 /oss/upload ，实现图片上传到OSS对应的Bucket中

- 类型检查：必须为 jpg、png 或者 jpeg类型，其它图片类型返回错误提示信息
- 大小检查：必须小于5M，大于5M时返回错误提示信息
- 图片名称：生成的文件名，必须保持唯一

2.可以使用postman 发送下载请求 /oss/download，实现图片下载

- 可以根据图片名进行文件的下载

3.可以使用postman 发送删除请求/oss/delete，实现图片删除

- 可以根据图片名进行文件的删除

### 知识回顾

阿里云对象存储服务（Object Storage Service，简称OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。相比自建服务器存储上可靠性高、提供多层安全防护、零成本运维和提供多种数据处理能力。

- 存储空间（Bucket）
- 对象/文件（Object）
- Region（地域）
- Endpoint（访问域名）
- AccessKey（访问密钥）
- Service（虚拟存储空间）

OSS功能：基本功能、OSS防盗链、自定义域名绑定、访问日志记录。

三种权限控制方式：ACL、RAM Policy、Bucket Policy。

### 实现思路

#### 1.引入依赖

```xml
<!--spring boot的支持-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.0.RELEASE</version>
</parent>

<dependencies>
    <!--springboot 测试支持-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.aliyun.oss</groupId>
        <artifactId>aliyun-sdk-oss</artifactId>
        <version>2.8.3</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.7</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.4</version>
    </dependency>
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>2.9.9</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

</dependencies>
```

#### 2.配置文件

application.properties

```properties
server.port=8999
spring.servlet.multipart.max-file-size=1024000
```

aliyun.properties

```properties
aliyun.endpoint=http://oss-cn-shanghai.aliyuncs.com
aliyun.accessKeyId=xxxxxx
aliyun.accessKeySecret=xxxxxx
aliyun.bucketName=fishleap-imgs
aliyun.urlPrefix=https://fishleap-imgs.oss-cn-shanghai.aliyuncs.com/
```

#### 3.配置类

```java
@Configuration
@PropertySource("classpath:aliyun.properties")
@ConfigurationProperties(prefix = "aliyun")
@Data
public class AliyunConfig {
    private String endpoint;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;
    private String urlPrefix;
    // 生成OSSClient
    @Bean
    public OSSClient  ossClient(){
        return   new OSSClient(endpoint,accessKeyId,accessKeySecret);
    }
}
```

#### 4.响应bean

```java
@Data
public class UpLoadResult {
    // 文件唯一标识
    private String uid;
    // 文件名
    private String name;
    // 状态有：uploading done error removed
    private String status;
    // 服务端响应内容，如：'{"status": "success"}'
    private String response;
}
```

#### 5.服务类

```java
@Service
public class FileUpLoadService {

    @Autowired
    private AliyunConfig aliyunConfig;
    @Autowired
    private OSSClient  ossClient;

    // 允许上传的格式  .pdf 是测试文件大小时使用
    private static final String[] IMAGE_TYPE = new String[]{".bmp", ".jpg",
            ".jpeg", ".gif", ".png"};

    /**
     * 文件上传
     * @param multipartFile 表单文件
     * @return 响应类
     */
    public UpLoadResult upload(MultipartFile multipartFile){
        // 校验图片格式
        boolean  isLegal = false;
        for (String type:IMAGE_TYPE){
            if(StringUtils.endsWithIgnoreCase(multipartFile.getOriginalFilename(),type)){
                isLegal = true;
                break;
            }
        }
        //封装Result对象，并且将文件的byte数组放置到result对象中
        UpLoadResult upLoadResult = new UpLoadResult();
        if (!isLegal){
            upLoadResult.setStatus("error 文件类型不符合 请重新选择");
            return  upLoadResult;
        }
        System.out.println("file_size:"+multipartFile.getSize());
        //校验文件大小
        if(multipartFile.getSize()>1024*1024*5){
            upLoadResult.setStatus("error 文件超过5m 请重新选择");
            return  upLoadResult;
        }
        //文件新路径
        String fileName = multipartFile.getOriginalFilename();
        String filePath = getFilePath(fileName);
        //上传阿里云
        try {
            ossClient.putObject(aliyunConfig.getBucketName(),filePath,new ByteArrayInputStream(multipartFile.getBytes()));
        } catch (IOException e) {
            e.printStackTrace();
            // 上传失败
            upLoadResult.setStatus("error");
            return  upLoadResult;
        }
        upLoadResult.setStatus("done");
        upLoadResult.setName(aliyunConfig.getUrlPrefix()+filePath);
        upLoadResult.setUid(filePath);
        return  upLoadResult;
    }

    /**
     *   下载文件
     * @param os
     * @param fileName 文件名称
     * @throws IOException
     */
    public void exportOssFile(OutputStream os, String fileName) throws IOException {
        // ossObject包含文件所在的存储空间名称、文件名称、文件元信息以及一个输入流。
        OSSObject ossObject = ossClient.getObject(aliyunConfig.getBucketName(), objectName);
        // 读取文件内容。
        BufferedInputStream in = new BufferedInputStream(ossObject.getObjectContent());
        BufferedOutputStream out = new BufferedOutputStream(os);
        byte[] buffer = new byte[1024];
        int lenght = 0;
        while ((lenght = in.read(buffer)) != -1) {
            out.write(buffer, 0, lenght);
        }
        if (out != null) {
            out.flush();
            out.close();
        }
        if (in != null) {
            in.close();
        }
    }

    /**
     * 删除文件
     * @param filePath 文件名称
     * @return 响应类
     */
    public UpLoadResult  deleteFile(String filePath) {
        UpLoadResult upLoadResult = new UpLoadResult();

        try {
            if (!ossClient.doesObjectExist(aliyunConfig.getBucketName(),filePath)){
                upLoadResult.setStatus("删除的文件不存在");
                return  upLoadResult;
            }
            // 根据BucketName,文件路径删除文件
            ossClient.deleteObject(aliyunConfig.getBucketName(),filePath);
            upLoadResult.setStatus("删除文件成功");
        } catch (Exception e) {
            e.printStackTrace();
            upLoadResult.setStatus("删除文件失败");
        }
        return  upLoadResult;
    }

    /**
     * 查看文件列表
     * @return
     */
    public List<OSSObjectSummary> list() {
        // 设置最大个数。
        final int maxKeys = 200;
        // 列举文件。
        ObjectListing objectListing = ossClient.listObjects(new ListObjectsRequest(aliyunConfig.getBucketName()).withMaxKeys(maxKeys));
        List<OSSObjectSummary> sums = objectListing.getObjectSummaries();
        return sums;
    }

    // 生成不重复的文件路径和文件名
    private String getFilePath(String sourceFileName) {
        DateTime dateTime = new DateTime();
        return "images/" + dateTime.toString("yyyy")
                + "/" + dateTime.toString("MM") + "/"
                + dateTime.toString("dd") + "/" + UUID.randomUUID().toString() + "." +
                StringUtils.substringAfterLast(sourceFileName, ".");
    }
}
```

#### 6.控制类

```java
@RestController
@RequestMapping("/oss")
public class UploadController {

    @Autowired
    private FileUpLoadService fileUpLoadService;

    /**
     * 文件上传
     * @param multipartFile
     * @return
     */
    @PostMapping("/upload")
    public UpLoadResult upload(@RequestParam("file") MultipartFile  multipartFile){
        return  fileUpLoadService.upload(multipartFile);
    }

    /**
     *  根据文件名进行下载
     * @param objectName
     * @param response
     * @throws IOException
     */
    @RequestMapping("/download")
    public void download(@RequestParam("fileName") String objectName, HttpServletResponse response) throws IOException {
        //通知浏览器以附件形式下载
        response.setHeader("Content-Disposition",
                "attachment;filename=" + new String(objectName.getBytes(), "ISO-8859-1"));
        fileUpLoadService.exportOssFile(response.getOutputStream(),objectName);
    }

    /**
     * 根据文件名删除
     * @param filePath
     * @return
     */
    @PostMapping("/delete")
    public UpLoadResult delete(@RequestParam("fileName") String filePath){
        return  fileUpLoadService.deleteFile(filePath);
    }

    /**
     *  查看桶内所有文件
     * @return
     * @throws Exception
     */
    @RequestMapping("/list")
    public List<OSSObjectSummary> list() throws Exception {
        return fileUpLoadService.list();
    }
}
```

#### 7.启动类

```java
@SpringBootApplication
public class ApplicationBoot {
    public static void main(String[] args) {
        SpringApplication.run(ApplicationBoot.class,args);
    }
}
```

#### 8.使用postman进行测试

- 文件上传：http://localhost:8999/oss/upload （post）、参数（file：上传文件）
- 获取文件列表：http://localhost:8999/oss/list （get）
- 文件下载：http://localhost:8999/oss/download?fileName=aa.jpg （get）
- 文件删除：http://localhost:8999/oss/delete?fileName=aa.jpg（post）









