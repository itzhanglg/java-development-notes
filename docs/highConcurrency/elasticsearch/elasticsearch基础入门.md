## 一.Elasticsearch概述

### 1.Elasticsearch介绍

Elaticsearch简称为ES,是一个开源的可 扩展的分布式的**全文检索引擎**，它可以近乎实时的存储、检索数据。ES使用Java开发并使用Lucene作为其核心来实现索引和搜索的功能，但是它通过简单的**RestfulAPI和javaAPI**来隐藏Lucene的复杂性，从而让全文搜索变得简单。官网：[https://www.elastic.co/cn/products/elasticsearch](https://www.elastic.co/cn/products/elasticsearch)

![image-20201218234821155](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201218234821155.png)

Elasticsearch**功能**：

- 分布式的搜索引擎：Elasticsearch自动将海量数据分散到多台服务器上去存储和检索；
- 全文检索：提供模糊搜索等自动度很高的查询方式，并进行相关性排名，高亮等功能；
- 数据分析引擎（分组聚合）：对数据进行聚合处理；
- 对海量数据进行近实时的处理：Elasticsearch可以采用大量的服务器去存储和检索数据，可以实现秒级别的数据搜索和分析。

Elasticsearch**特点**：Elasticsearch的特点是它提供了一个极速的搜索体验。这源于它的高速（**speed**）。相比较其它的一些大数据引擎，Elasticsearch可以实现秒级的搜索，速度非常有优势。Elasticsearch的cluster是一种分布式的部署，极易扩展(**scale** )这样很容易使它处理PB级的数据库容量。最重要的是Elasticsearch是它搜索的结果可以按照分数进行排序，它能提供我们最相关的搜索结果（**relevance**) 。

- 安装方便：没有其他依赖，下载后安装非常方便；只用修改几个参数就可以搭建起来一个集群；
- JSON：输入/输出格式为 JSON，意味着不需要定义 Schema，快捷方便；
- RESTful：基本所有操作 ( 索引、查询、甚至是配置 ) 都可以通过 HTTP 接口进行；
- 分布式：节点对外表现对等（每个节点都可以用来做入口） 加入节点自动负载均衡；
- 多租户：可根据不同的用途分索引，可以同时操作多个索引；
- 支持超大数据： 可以扩展到 PB 级的结构化和非结构化数据 海量数据的近实时处理。

Elasticsearch**使用场景**：

- 搜索类场景：比如说电商网站、招聘网站、新闻资讯类网站、各种app内的搜索；
- 日志分析类场景：ELK组合（Elasticsearch/Logstash/Kibana），可以完成日志收集，日志存储，日志分析查
    询界面基本功能；
- 数据预警平台及数据分析场景：数据分析常见的比如分析电商平台销售量top 10的品牌，分析博客系统、头条网站top 10关注度、评论数、访问量的内容等等。
- 商业BI(Business Intelligence)系统：Elasticsearch执行数据分析和挖掘，Kibana做数据可视化。

### 2.主流全文搜索方案对比

Lucene、Solr、Elasticsearch是目前主流的全文搜索方案，基于**倒排索引机制**完成快速全文搜索。[https://db-engines.com/en/ranking/search+engine](https://db-engines.com/en/ranking/search+engine)

![image-20201218234544576](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201218234544576.png)

- Lucene是Apache基金会维护的一套完全使用Java编写的信息搜索工具包，Lucene只是一个框架，我们需要在Java程序中集成它再使用。
- Solr是一个有HTTP接口的基于Lucene的查询服务器，是一个搜索引擎系统，封装了很多Lucene细节，Solr可以直接利用HTTP GET/POST请求去查询，维护修改索引。
- Elasticsearch也是一个建立在全文搜索引擎 Apache Lucene基础上的搜索引擎。采用的策略是分布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索。

Solr和Elasticsearch都是基于Lucene实现的。但Solr和Elasticsearch之间也是有区别的：

- Solr利用Zookpper进行分布式管理，而Elasticsearch自身带有分布式协调管理功能；
- Solr比Elasticsearch实现更加全面，Solr官方提供的功能更多，而Elasticsearch本身更注重于核心功能， 高级功能多由第三方插件提供；
- Solr在传统的搜索应用中表现好于Elasticsearch，而Elasticsearch在实时搜索应用方面比Solr表现好。

### 3.Elasticsearch版本

#### 3.1 版本介绍

Elasticsearch 主流版本为5.x , 6.x及7.x版本。7.x 更新的内容如下：

- **集群连接变化**：TransportClient被废弃。es7的java代码，只能使用restclient。对于java编程，建议采用 `High-level-rest-client` 的方式操作ES集群。
- ES**数据存储结构变化**：简化了Type 默认使用 `_doc`。api请求方式也发送变化，如获得某索引的某ID的文档：`GET index/_doc/id` 其中index和id为具体的值。
- ES程序包默认打包jdk：以至于7.x版本的程序包大小突然增大了200MB+, 对比6.x发现，包大了200MB+， 正是JDK的大小。
- **默认配置变化**：默认节点名称为主机名，默认分片数改为1，不再是5。
- Lucene升级为lucene 8 查询相关性速度优化：Weak-AND算法（取TOP N结果集，估算命中记录数）。
- **间隔查询**(Intervals queries)： intervals query 允许用户精确控制查询词在文档中出现的先后关系，实现了对terms顺序、terms之间的距离以及它们之间的包含关系的灵活控制。
- 引入新的集群协调子系统：移除 `minimum_master_nodes` 参数，让 Elasticsearch 自己选择可以形成仲裁的节点。
- 7.0将不会再有OOM的情况：JVM引入了新的 `circuit breaker`（熔断）机制，当查询或聚合的数据量超出单机处理的最大内存限制时会被截断。设置 `indices.breaker.fielddata.limit`  的默认值已从JVM堆大小的60％降低到40％。
- **分片搜索空闲时跳过refresh**：以前版本的数据插入，每一秒都会有refresh动作，这使得es能成为一个近实时的搜索引擎。但是当没有查询需求的时候，该动作会使得es的资源得到较大的浪费。

#### 3.2 软件兼容

[Elasticsearch与操作系统](https://www.elastic.co/cn/support/matrix#matrix_os)

![image-20201218235552153](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201218235552153.png)

[Elasticsearch与JVM](https://www.elastic.co/cn/support/matrix#matrix_jvm)

![image-20201218235441354](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201218235441354.png)

## 二.环境安装与配置

Elasticsearch是一个分布式全文搜索引擎，支持单节点模式(Single-Node Mode)和集群模式(Cluster Mode)部署。

### 1.Elasticsearch安装与配置

Elasticsearch下载地址：[https://www.elastic.co/cn/downloads/past-releases#elasticsearch](https://www.elastic.co/cn/downloads/past-releases#elasticsearch)

关闭虚拟机的防火墙：

> systemctl stop firewalld.service    #停止firewall
>
> systemctl disable firewalld.service    #禁止firewall开机启动
>
> firewall-cmd --state    #查看防火墙

JDK环境可选择性安装，环境准备好后下面为安装Elasticsearch的步骤：

1.上传 elasticsearch-7.3.0-linux-x86_64.tar.gz 包并解压：

> tar -zxvf elasticsearch-7.3.0-linux-x86_64.tar.gz -C /opt/servers
>
> mv elasticsearch-7.3.0/ elasticsearch

2.配置elasticsearch

```shell
[root@linux100 servers]# vim /opt/servers/elasticsearch/config/elasticsearch.yml
# 单机安装
node.name: node-1
# 修改网络和端口
network.host: 192.168.91.100
http.port: 9200
# 单机只保留一个node
cluster.initial_master_nodes: ["node-1"]
```

3.按需要修改jvm.options内存设置

> 根据实际情况修改占用内存，默认都是1G，单机1G内存，启动会占用700m+然后在安装kibana后，基本上无法运行了，运行了一会就挂了报内存不足。 内存设置超出物理内存，也会无法启动，启动报错。

```shell
[root@linux100 servers]# vim /opt/servers/elasticsearch/config/jvm.options
# 默认都是1G
-Xms1g
-Xmx1g
```

4.添加es用户，es默认root用户无法启动，需要改为其他用户

```shell
# 添加用户estest
[root@linux100 servers]# useradd estest
# 设置用户estest密码
[root@linux100 servers]# passwd estest
# 改变es目录拥有者账号
[root@linux100 servers]# chown -R estest /opt/servers/elasticsearch/
drwxr-xr-x.  9 estest root   154 7月  25 2019 elasticsearch
```

5.修改/etc/sysctl.conf

```shell
[root@linux100 servers]# vim /etc/sysctl.conf
vm.max_map_count=655360
# 执行sysctl -p 让其生效
[root@linux100 servers]# sysctl -p
vm.max_map_count = 655360
```

6.修改/etc/security/limits.conf

```shell
[root@linux100 servers]# vim /etc/security/limits.conf
# 末尾添加硬连接与软连接
* soft nofile 65536
* hard nofile 65536
* soft nproc 4096
* hard nproc 4096
```

7.启动es

```shell
# 切换刚刚新建的用户
[root@linux100 servers]# su estest
# 启动
[estest@linux100 servers]$ /opt/servers/elasticsearch/bin/elasticsearch
```

8.配置完成，浏览器访问测试。 `ip:9200`

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214225119737.png" alt="image-20201214225119737" style="zoom:60%;" />

### 2.Kibana安装与配置

Kibana是一个基于Node.js的Elasticsearch索引库数据统计工具，可以利用Elasticsearch的聚合功能，生成各种图表，如柱形图，线状图，饼图等。

Kibana下载地址：[https://www.elastic.co/cn/downloads/past-releases#kibana](https://www.elastic.co/cn/downloads/past-releases#kibana)

Kibana与OS：[https://www.elastic.co/cn/support/matrix#matrix_os](https://www.elastic.co/cn/support/matrix#matrix_os)

1.上传 kibana-7.3.0-linux-x86_64.tar.gz 并解压：

```shell
tar -zxvf kibana-7.3.0-linux-x86_64.tar.gz -C /opt/servers/
mv kibana-7.3.0-linux-x86_64/ kibana
```

2.修改 kibana目录拥有者账号和访问权限

```shell
[root@linux100 servers]# chown -R estest /opt/servers/kibana/
[root@linux100 servers]# chmod -R 777 /opt/servers/kibana/
drwxrwxrwx. 14 estest root   271 12月 14 22:09 kibana
```

3.修改配置文件

```shell
vim /opt/servers/kibana/config/kibana.yml

# 设置端口
server.port: 5601
# 设置访问ip，监听所有host
server.host: "0.0.0.0"
# 修改elasticsearch服务器ip
elasticsearch.hosts: ["http://192.168.91.100:9200"]
```

4.切换用户，再启动

```shell
[root@linux100 servers]# su estest
# 启动
[estest@linux100 servers]$ /opt/servers/kibana/bin/kibana

# 也可以使用root账户启动
[root@linux100 analysis-ik]# /opt/servers/kibana/bin/kibana --allow-root
```

5.访问 `ip:5601` ，快捷键： `ctrl+enter` 提交请求， `ctrl+i` 自动缩进。

![image-20201214234210901](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201214234210901.png)

### 3.安装IK分词器

IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包。新版本的IKAnalyzer3.0则发展为面向Java的公用分词组件，独立于Lucene项目，同时提供了对Lucene的默认优化实现。

- 采用了特有的“正向迭代最细粒度切分算法“，具有60万字/秒的高速处理能力。
- 采用了多子处理器分析模式，支持：**英文字母**（IP地址、Email、URL）、**数字**（日期，常用中文数量词，罗马数字，科学计数法），**中文词汇**（姓名、地名处理）等分词处理。
- 支持个人词条的优化的词典存储，更小的内存占用。
- 支持**用户词典扩展定义**。
- 针对Lucene全文检索优化的查询分析器IKQueryParser；采用歧义分析算法优化查询关键字的搜索排列组合，能极大的提高Lucene检索的命中率。

#### 3.1 IK分词器安装与配置

下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.3.0

安装方式一：**下载插件并安装**

1.在elasticsearch的bin目录下执行以下命令,es插件管理器会自动帮我们安装，然后等待安装完成：

```shell
/opt/servers/elasticsearch/bin/elasticsearch-plugin install
https://github.com/medcl/elasticsearch-analysisik/releases/download/v7.3.0/elasticsearch-analysis-ik-7.3.0.zip

# 下载完成后会提示 Continue with installation? 输入 y 即可完成安装
```

2.重启Elasticsearch和Kibana

```shell
# 启动elasticsearch
[root@linux100 servers]# su estest
[estest@linux100 servers]$ /opt/servers/elasticsearch/bin/elasticsearch
# 启动kibana
[root@linux100 analysis-ik]# /opt/servers/kibana/bin/kibana --allow-root
```

安装方式二：**上传安装包安装**

1.在elasticsearch安装目录的plugins目录下新建 analysis-ik 目录：

```shell
#新建analysis-ik文件夹
mkdir analysis-ik
#切换至 analysis-ik文件夹下
cd analysis-ik
#上传 elasticsearch-analysis-ik-7.3.0.zip 压缩包并解压
unzip elasticsearch-analysis-ik-7.3.3.zip
#解压完成后删除zip
rm -rf elasticsearch-analysis-ik-7.3.0.zip
```

2.重启Elasticsearch和Kibana

```shell
# 启动elasticsearch
[root@linux100 servers]# su estest
[estest@linux100 servers]$ /opt/servers/elasticsearch/bin/elasticsearch
# 启动kibana
[root@linux100 analysis-ik]# /opt/servers/kibana/bin/kibana --allow-root
```

安装完成：**测试案例**

IK分词器有两种分词模式：ik_max_word（最细粒度的拆分）和ik_smart（最粗粒度的拆分）模式。

ik_max_word模式：

```json
// ik_max_word 请求
POST _analyze
{
  "analyzer": "ik_max_word",
  "text": "南京市长江大桥"
}
// 响应结果
{
  "tokens" : [
    {
      "token" : "南京市",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "南京",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "市长",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "长江大桥",
      "start_offset" : 3,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "长江",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "大桥",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 5
    }
  ]
}
```

ik_smart模式：

```json
// ik_smart 请求
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "南京市长江大桥"
}
// 响应结果
{
  "tokens" : [
    {
      "token" : "南京市",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "长江大桥",
      "start_offset" : 3,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 1
    }
  ]
}
```

![image-20201215001452387](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201215001452387.png)

#### 3.2 扩展词典和停用词典使用

**扩展词**：就是不想让哪些词被分开，让他们分成一个词。比如上面的江大桥。

**停用词**：有些词在文本中出现的频率非常高。但对本文的语义产生不了多大的影响。停用词经常被过滤掉，不会被进行索引。在检索的过程中，如果用户的查询词中含有停用词，系统会自动过滤掉。

进入到 `config/analysis-ik/`(插件命令安装方式) 或 `plugins/analysis-ik/config` (安装包安装方式) 目录
下, 新增自定义词典：

```shell
# 此时下面是采用 安装包安装方式
[root@linux100 config]# cd /opt/servers/elasticsearch/plugins/analysis-ik/config

[root@linux100 config]# vim lagou_ext_dict.dic
# 配置 江大桥

[root@linux100 config]# vim lagou_stop_dict.dic
# 配置 市长

# 将我们自定义的扩展词典和停用词典文件添加到IKAnalyzer.cfg.xml配置中
[root@linux100 config]# vim IKAnalyzer.cfg.xml
```

配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict">lagou_ext_dict.dic</entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords">lagou_stop_dict.dic</entry>
        <!--用户可以在这里配置远程扩展字典 -->
        <!-- <entry key="remote_ext_dict">words_location</entry> -->
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

重启Elasticsearch后，进行测试：

```json
// 请求
POST _analyze
{
  "analyzer": "ik_max_word",
  "text": "南京市长江大桥"
}
// 响应结果：有 江大桥，没有 市长
{
  "tokens" : [
    {
      "token" : "南京市",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "南京",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "长江大桥",
      "start_offset" : 3,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "长江",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "江大桥",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "大桥",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 5
    }
  ]
}
```

#### 3.3 同义词典使用

同义词：很多相同意思的词。比如“番茄”和“西红柿”，“馒头”和“馍”等。同义词查询：在搜索的时候，我们输入的可能是“番茄”，但是应该把含有“西红柿”的数据一起查询出来。

注意：扩展词和停用词是在**索引**的时候使用，而同义词是**检索**时候使用。

Elasticsearch 自带一个名为 synonym 的同义词 filter。为了能让 IK 和 synonym 同时工作，我们需要定义新的 analyzer，用 IK 做 tokenizer，synonym 做 filter。

**配置IK同义词**

1.在 `/opt/servers/elasticsearch/plugins/analysis-ik/config` 下创建 synonym.txt 文件，输入一些同义词并存为 utf-8 格式。例如：

```
English,英语
Math,数学
```

2.创建索引时，使用同义词配置，示例模板如下：

```json
PUT /索引名称
{
    "settings": {
        "analysis": {
            "filter": {
                "word_sync": {
                    "type": "synonym",
                    "synonyms_path": "analysis-ik/synonym.txt"
                }
            },
            "analyzer": {
                "ik_sync_max_word": {
                    "filter": [
                        "word_sync"
                    ],
                    "type": "custom",
                    "tokenizer": "ik_max_word"
                },
                "ik_sync_smart": {
                    "filter": [
                        "word_sync"
                    ],
                    "type": "custom",
                    "tokenizer": "ik_smart"
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "字段名": {
                "type": "字段类型",
                "analyzer": "ik_sync_smart",
                "search_analyzer": "ik_sync_smart"
            }
        }
    }
}
```

以上配置定义了ik_sync_max_word和ik_sync_smart这两个新的 analyzer，对应 IK 的 ik_max_word 和 ik_smart两种分词策略。ik_sync_max_word和 ik_sync_smart都会使用 synonym filter 实现同义词转换。

3.案例

创建索引：

```json
PUT /fishleap-es-synonym
{
    "settings": {
        "analysis": {
            "filter": {
                "word_sync": {
                    "type": "synonym",
                    "synonyms_path": "analysis-ik/synonym.txt"
                }
            },
            "analyzer": {
                "ik_sync_max_word": {
                    "filter": [
                        "word_sync"
                    ],
                    "type": "custom",
                    "tokenizer": "ik_max_word"
                },
                "ik_sync_smart": {
                    "filter": [
                        "word_sync"
                    ],
                    "type": "custom",
                    "tokenizer": "ik_smart"
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "name": {
                "type": "text",
                "analyzer": "ik_sync_max_word",
                "search_analyzer": "ik_sync_max_word"
            }
        }
    }
}
```

添加数据：

```json
POST /fishleap-es-synonym/_doc/1
{
	"name":"我爱英语也爱数学"
}
```

使用同义词"English"或者“Math”进行搜索：

```json
POST /fishleap-es-synonym/_doc/_search
{
    "query": {
        "match": {
        	"name": "English"
        }
    }
}
```

## 三.Elasticsearch入门使用

### 1.核心概念

Elasticsearch是基于Lucene的全文检索引擎，本质也是存储和检索数据。具体有哪些数据类型可看官方：[https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)

- **索引**(index)：类似的数据放在一个索引，非类似的数据放不同索引， 一个索引也可以理解成一个**关系型数据**
    **库**。
- **类型**(type)：代表document属于index中的哪个类别（type）也有一种说法一种type就像是**数据库的表**，比如dept表，user表。ES 5.x中一个index可以有多种type。ES 6.x中一个index只能有一种type。ES 7.x以后要逐渐移除type这个概念。
- **映射**(mapping)：mapping定义了每个字段的类型等信息。相当于关系型数据库中的**表结构**。常用数据类型：text、keyword、number、array、range、boolean、date、geo_point、ip、nested、object。

![image-20201219002201590](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219002201590.png)

两个官方API文档地址：

Rest风格API官方文档地址：[https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)

![image-20201219002854616](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219002854616.png)

客户端API：[https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)

![image-20201219002949283](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219002949283.png)

### 2.索引操作

Elasticsearch采用Rest风格API，因此其API就是一次http请求，你**可以用任何工具发起http请求**。

创建索引（PUT）：

```json
PUT /索引名称
{
    // 索引库设置，定义索引库的各种属性，比如分片数、副本数等
    "settings": {
    	"属性名": "属性值"
    }
}
```

判断索引是否存在（HEAD）：

```json
HEAD /索引名称
```

查看索引相关属性信息（GET）：

```json
// 查看单个索引
GET /索引名称

// 批量查看索引
GET /索引名称1,索引名称2,索引名称3,...

// 查看所有索引（相关属性信息）
GET _all

// 查看所有索引（集群健康信息）
GET /_cat/indices?v

// 绿色：索引的所有分片都正常分配。
// 黄色：至少有一个副本没有得到正确的分配。
// 红色：至少有一个主分片没有得到正确的分配。
```

打开/关闭索引（POST）：

```json
// 打开索引
POST /索引名称/_open
// 关闭索引
POST /索引名称/_close
```

删除索引库（DELETE）：

```json
DELETE /索引名称1,索引名称2,索引名称3...
```

### 3.映射操作

#### 3.1 映射基本操作

索引创建之后，等于有了关系型数据库中的database。Elasticsearch7.x索引type类型默认为`_doc`。我们设置字段的约束信息（表结构信息），叫做**字段映射**（mapping）。

字段的约束包括但不限于：字段的数据类型、是否要存储、是否要索引、分词器。映射属性链接：[https://www.elastic.co/guide/en/elasticsearch/reference/7.3/mapping-params.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/mapping-params.html)

![image-20201219160903995](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219160903995.png)

创建映射字段（约束关系）（PUT）：

```json
PUT /索引库名/_mapping
{
    "properties": {
        // 任意填写，下面指定许多属性
        "字段名": {
            "type": "类型",    //类型，可以是text、long、short、date、integer、object等
            "index": true，    //是否索引，默认为true
            "store": true，    //是否存储，默认为false
            "analyzer": "分词器"    //指定分词器
        },
        "字段名": {
            "type": "类型",    //类型，可以是text、long、short、date、integer、object等
            "index": true，    //是否索引，默认为true
            "store": true，    //是否存储，默认为false
            "analyzer": "分词器"    //指定分词器
        }
        //...
    }
}
```

查看映射关系（GET）：

```json
// 查看单个索引映射关系
GET /索引名称/_mapping

// 查看所有索引映射关系
GET _mapping
GET _all/_mapping
```

修改索引映射关系（PUT）：修改映射增加字段，其它更改只能删除索引，重新建立映射。

```json
PUT /索引库名/_mapping 
{
	"properties": {
        "字段名": {
            "type": "类型",
            "index": true,
            "store": true, 
            "analyzer": "分词器"
        }
    }
}
```

#### 3.2 一次性创建索引和映射

在创建索引库的同时，直接制定索引库中的索引，基本语法：

```json
put /索引库名称
{
    "settings": {
        "索引库属性名": "索引库属性值"
    },
    "mappings": {
        "properties": {
            "字段名": {
                "映射属性名": "映射属性值"
            }
        }
    }
}
```

示例：

```json
PUT /lagou-employee-index
{
    "settings": {},
    "mappings": {
        "properties": {
            "name": {
                "type": "text",
                "analyzer": "ik_max_word"
            }
        }
    }
}
```

#### 3.3 映射属性详解

**type属性**信息地址：[https://www.elastic.co/guide/en/elasticsearch/reference/7.3/mapping-types.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/mapping-types.html)

![image-20201219162701240](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219162701240.png)

type属性值详解：

```json
String类型：又分两种。
- text：可分词，不可参与聚合；
- keyword：不可分词，数据会作为完整字段进行匹配，可以参与聚合。

Numerical：数值类型，分两类。
- 基本数据类型：long、interger、short、byte、double、float、half_float；
- 浮点数的高精度类型：scaled_float。需要指定一个精度因子，比如10或100。elasticsearch会把真实值乘
以这个因子后存储，取出时再原。

Date：日期类型。elasticsearch可以对日期格式化为字符串存储，但是建议我们存储为毫秒值，存储为long，节
省空间。

Array：数组类型。
- 进行匹配时，任意一个元素满足，都认为满足；
- 排序时，如果升序则用数组中的最小值来排序，如果降序则用数组中的最大值来排序。

Object：对象
```

index属性值：影响字段的索引情况。默认值为true，所有字段都会被索引。

```json
- true：字段会被索引，则可以用来进行搜索。默认值就是true
- false：字段不会被索引，不能用来搜索
```

store属性值：是否将数据进行独立存储。默认值为false。

```json
默认情况下其他提取出来的字段都是从_source 里面提取出来的。只要设置store:true即可，获取独立存储的字段
要比从_source中解析快得多，但是也会占用更多的空间。
```

analyzer属性值：指定分词器。一般我们处理中文会选择ik分词器 `ik_max_word` 或 `ik_smart` 。

### 4.文档操作

文档，即索引库中的数据，会根据规则创建索引，将来用于搜索。可以类比做数据库中的一行数据。

#### 4.1 新增文档

新增文档：

```json
// 手动指定id，若id不存在，则创建；如果存在，则全量替换，替换文档的json串内容
POST /索引名称/_doc/{id}
{
    "field":"value"
}    

// 自动生成id
POST /索引名称/_doc
{
	"field":"value"
}

// 强制创建，如果id 存在就会报错
PUT /索引名称/_doc/{id}?op_type=create {}
// 或者
PUT /索引名称/_doc/{id}/_create {}
```

#### 4.2 查看文档

查看单个文档：

```json
GET /索引名称/_doc/{id}

// 响应结果：文档元数据
{
  "_index" : "elasticsearch_test",    //document所属index
  "_type" : "_doc",    //document所属type，7.x默认type为_doc
  "_id" : "1",    //document的唯一标识，与index和type组合在一起可以唯一标识一个document
  "_version" : 1,    //document的版本号，确保变更不会导致数据丢失。修改数据需要指定version号
  "_seq_no" : 0,    //严格递增的顺序号，每个文档一个
  "_primary_term" : 1,    //任何类型的写操作，都会生成一个_seq_no
  "found" : true,    //true/false，是否查找到文档
  "_source" : {    //存储原始文档数据
    "price" : 35.6,
    "name" : "spring cloud实战"
  }
}
```

查看所有文档：

```json
POST /索引名称/_search
{
    "query":{
        "match_all": {
        }
    }
}
```

`_source`定制返回结果：使用 `_source`进行定制返回 `_source`中的字段。

```json
//多个字段之间使用逗号分隔
GET /索引名称/_doc/{id}?_source=field,field
// 示例
GET /elasticsearch_test/_doc/1?_source=price,name
```

#### 4.3 更新文档

全部更新：是直接把之前的老数据，标记为删除状态，然后再添加一条更新的（使用PUT或者POST）。

```json
// 请求方式为PUT，必须指定id
// 若id存在，则更新数据（updated）；若不存在，则新增数据（created）。
PUT /索引名称/_doc/{id}
{
    "field":"value"
}
```

局部更新：只是修改某个字段（使用POST）。

```json
POST /索引名称/_update/{id}
{
    "doc":{
    	"field":"value"
    }
}
```

#### 4.4 删除文档

```json
// 根据id进行删除
DELETE /索引名称/_doc/{id}

// 根据查询条件进行删除
POST /索引名称/_delete_by_query
{
    "query": {
        "match": {
        	"字段名": "搜索关键字"
        }
    }
}

// 删除所有文档
POST /索引名称/_delete_by_query
{
    "query": {
    	"match_all": {}
    }
}
```

## 四.SpringBoot整合Elasticsearch应用

### 1.应用配置

Maven工程引入依赖：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.3.0</version>
        <exclusions>
            <exclusion>
                <groupId>org.elasticsearch</groupId>
                <artifactId>elasticsearch</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.3.0</version>
    </dependency>
</dependencies>
```

application.yml

```yaml
lagouelasticsearch:
  elasticsearch:
    hostlist: 192.168.91.100:9200 #多个结点中间用逗号分隔
```

elasticsearch配置文件：

```java
/**
 * @author zlg
 */
@Configuration
public class ESConfig {

    @Value("${lagouelasticsearch.elasticsearch.hostlist}")
    private String hostlist;
    
    @Bean
    public RestHighLevelClient restHighLevelClient(){
        //解析hostlist配置信息
        String[] split = hostlist.split(",");
        //创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for(int i=0;i<split.length;i++){
            String item = split[i];
            httpHostArray[i] = new HttpHost(item.split(":")[0],
                    Integer.parseInt(item.split(":")[1]), "http");
        }
        //创建RestHighLevelClient客户端
        return new RestHighLevelClient(RestClient.builder(httpHostArray));
    }
    
    //项目主要使用RestHighLevelClient，对于低级的客户端暂时不用
    @Bean
    public RestClient restClient(){
        //解析hostlist配置信息
        String[] split = hostlist.split(",");
        //创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for(int i=0;i<split.length;i++){
            String item = split[i];
            httpHostArray[i] = new HttpHost(item.split(":")[0],
                    Integer.parseInt(item.split(":")[1]), "http");
        }
        return RestClient.builder(httpHostArray).build();
    }
    
}
```

主启动类：

```java
/**
 * @author zlg
 */
@SpringBootApplication
public class ESApplication {
    public static void main(String[] args) {
        SpringApplication.run(ESApplication.class, args);
    }
}
```

### 2.索引库操作

```json
PUT /elasticsearch_test
{
    "settings": {},
    "mappings": {
        "properties": {
            "description": {
                "type": "text",
                "analyzer": "ik_max_word"
            },
            "name": {
                "type": "keyword"
            },
            "pic": {
                "type": "text",
                "index": false
            },
            "studymodel": {
                "type": "keyword"
            }
        }
    }
}
```

创建索引库:

```java
/* 
 * 创建索引库
 */
@Autowired
RestHighLevelClient client;

// 创建一个索引创建请求对象
CreateIndexRequest createIndexRequest  = new CreateIndexRequest("elasticsearch_test");
//设置映射
/*XContentBuilder builder  = XContentFactory.jsonBuilder()
    .startObject()
    .field("properties")
    .startObject()
.field("description").startObject().field("type","text").field("analyzer","ik_max_word").endObject()
    .field("name").startObject().field("type","keyword").endObject()
.field("pic").startObject().field("type","text").field("index","false").endObject()
    .field("studymodel").startObject().field("type","keyword").endObject()
    .endObject()
    .endObject();
createIndexRequest.mapping("doc",builder);*/

createIndexRequest.mapping("doc","{\n" +
                           "        \"properties\": {\n" +
                           "          \"description\": {\n" +
                           "            \"type\": \"text\",\n" +
                           "            \"analyzer\": \"ik_max_word\"\n" +
                           "          },\n" +
                           "          \"name\": {\n" +
                           "            \"type\": \"keyword\"\n" +
                           "          },\n" +
                           "          \"pic\": {\n" +
                           "            \"type\": \"text\",\n" +
                           "            \"index\": false\n" +
                           "          },\n" +
                           "          \"studymodel\": {\n" +
                           "            \"type\": \"keyword\"\n" +
                           "          }\n" +
                           "        }\n" +
                           "      }", XContentType.JSON);
// 操作索引的客户端
IndicesClient indicesClient  = client.indices();
CreateIndexResponse createIndexResponse = indicesClient.create(createIndexRequest, RequestOptions.DEFAULT);
// 得到响应
boolean  acknowledged = createIndexResponse.isAcknowledged();
System.out.println(acknowledged);
```

删除索引库：

```java
/*
 * 删除索引库
 */
// 构建 删除索引库的请求对象
DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("elasticsearch_test");
IndicesClient indicesClient = client.indices();
AcknowledgedResponse deleteResponse = indicesClient.delete(deleteIndexRequest,RequestOptions.DEFAULT);
// 得到响应
boolean acknowledge = deleteResponse.isAcknowledged();
System.out.println(acknowledge);
```

### 3.文档操作

```json
POST /elasticsearch_test/_doc/1
{
    "name": "spring cloud实战",
    "description": "本课程主要从四个章节进行讲解： 1.微服务架构入门 2.spring cloud 基础入门 3.实战Spring Boot 4.注册中心eureka。",
    "studymodel":"201001",
    "timestamp": "2020-08-22 20:09:18",
    "price": 5.6
}
```

添加文档：

```java
// 准备索取请求对象
//IndexRequest indexRequest  = new IndexRequest("elasticsearch_test","doc");
IndexRequest indexRequest  = new IndexRequest("elasticsearch_test");

indexRequest.id("1");
// 文档内容  准备json数据
Map<String,Object> jsonMap = new HashMap<>();
jsonMap.put("name","spring cloud实战3");
jsonMap.put("description","本课程主要从四个章节进行讲解3： 1.微服务架构入门 2.spring cloud 基础入门 3.实战Spring Boot 4.注册中心eureka。");
jsonMap.put("studymodel","3101001");
jsonMap.put("timestamp","2020-07-22 20:09:18");
jsonMap.put("price",35.6);
indexRequest.source(jsonMap);
// 执行请求
IndexResponse indexResponse = client.index(indexRequest,RequestOptions.DEFAULT);
DocWriteResponse.Result  result = indexResponse.getResult();
System.out.println(result);
```

查询文档：

```java
// 查询请求对象
GetRequest getRequest  = new GetRequest("elasticsearch_test","1");
GetResponse getResponse  = client.get(getRequest,RequestOptions.DEFAULT);

// 得到文档内容
Map<String,Object> sourceMap = getResponse.getSourceAsMap();
System.out.println(sourceMap);
```

### 4.搜索操作

```json
GET /elasticsearch_test/_search
{
    "query":{
        "match_all":{}
    }
}
```

搜索操作：

```java
// 搜索请求对象
SearchRequest searchRequest = new SearchRequest("elasticsearch_test");
//searchRequest.searchType(SearchType.QUERY_THEN_FETCH);
// 搜索源构建对象
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
// 设置搜索方法
// searchSourceBuilder.query(QueryBuilders.matchAllQuery());//全部匹配
searchSourceBuilder.query(QueryBuilders.termQuery("description","spring"));//term匹配
// 设置_source返回结果中字段
searchSourceBuilder.fetchSource(new String[]{"name","price","timestamp"},new String[]{});

// 分页和排序
pageAndSort(searchSourceBuilder);

// 请求对象设置 搜索源对象
searchRequest.source(searchSourceBuilder);
// 使用client  执行搜索
SearchResponse searchResponse = client.search(searchRequest,RequestOptions.DEFAULT);
// 解析搜索结果
getSearchHits(searchResponse);
```

分页和排序：

```java
private void pageAndSort(final SearchSourceBuilder searchSourceBuilder) {
    // 设置分页参数
    int  page = 1;
    int  size = 2;
    // 计算出 from
    int  form = (page-1)*size;
    searchSourceBuilder.from(form);
    searchSourceBuilder.size(size);
    // 设置price 降序
    searchSourceBuilder.sort("price",SortOrder.DESC);
}
```

解析搜索结果：

```java
private void getSearchHits(final SearchResponse searchResponse) {
    // 搜索结果
    SearchHits  hits = searchResponse.getHits();
    // 匹配到的总记录数
    TotalHits  totalHits  = hits.getTotalHits();
    System.out.println("查询到的总记录数:"+totalHits.value);
    // 得到的匹配度高的文档
    SearchHit[] searchHits = hits.getHits();
    for (SearchHit  hit : searchHits){
        String  id = hit.getId();
        // 源文档的内容
        Map<String,Object>  sourceMap = hit.getSourceAsMap();
        String  name  = (String)sourceMap.get("name");
        String  timestamp  = (String)sourceMap.get("timestamp");
        String  description  = (String)sourceMap.get("description");
        Double   price  = (Double)sourceMap.get("price");
        System.out.println(name);
        System.out.println(timestamp);
        System.out.println(description);
        System.out.println(price);
    }
}
```

## 五.Elasticsearch之搜索实战

MySQL中的数据批量导入到ES中, 然后搜索职位信息进行展示。

1. 在MySQL数据中建立lagou_db数据库, 将position.sql中的数据导入到mysql 数据中。
2. 选择一种合理的方式(可以选择自己熟悉的插件等)，将mysql中的数据导入到ES中。
3. 使用SpringBoot访问ES，使用positionName 字段检索职位信息，如果检索到的职位信息不够5条，则需要启用poitionAdvantage 查找美女多、员工福利好的企业职位信息进行补充够5条。

提示：搜索一次查询不能满足要求时发起第二次搜索请求。搜索一次满足需求不需要发送第二次搜索请求。

### 1.环境准备

新建SpringBoot工程，然后引入依赖：

```xml
<properties>
    <elasticsearch.version>7.3.0</elasticsearch.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>${elasticsearch.version}</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.elasticsearch/elasticsearch -->
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>${elasticsearch.version}</version>
    </dependency>

    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.3.0</version>
        <exclusions>
            <exclusion>
                <groupId>org.elasticsearch</groupId>
                <artifactId>elasticsearch</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.3.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- HttpClient -->
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
        <version>4.5.3</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.58</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.9</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <!--devtools热部署-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
        <scope>true</scope>
    </dependency>
</dependencies>
```

application.yml

```yaml
spring:
  devtools:
    restart:
      enabled: true  #设置开启热部署
      additional-paths: src/main/java #重启目录
      exclude: WEB-INF/**
    freemarker:
      cache: false    #页面不加载缓存，修改即时生效
  elasticsearch:
    rest:
      uris: 192.168.91.100:9200
      
server:
  port: 8080

logging:
  level:
    root: info
    com.xdclass.search: debug
```

### 2.配置文件

数据库工具类：

```java
public class DBHelper {
    public static final String url = "jdbc:mysql://localhost:3306/lagou_position?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai";
    public static final String name = "com.mysql.cj.jdbc.Driver";
    public static final String user = "root";
    public static final String password = "root";
    private  static Connection  connection = null;

    public  static   Connection  getConn(){
        try {
            Class.forName(name);
            connection = DriverManager.getConnection(url,user,password);
        } catch (Exception e){
            e.printStackTrace();
        }
        return  connection;
    }
}
```

Elasticsearch配置类：

```java
@Configuration
public class EsConfig {
    
    @Value("${spring.elasticsearch.rest.uris}")
    private  String  hostlist;
    
    @Bean
    public RestHighLevelClient client() {
        //解析hostlist配置信息
        String[] split = hostlist.split(",");
        //创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for(int i=0;i<split.length;i++){
            String item = split[i];
            System.out.println(item);
            httpHostArray[i] = new HttpHost(item.split(":")[0], Integer.parseInt(item.split(":")[1]), "http");
        }
        //创建RestHighLevelClient客户端
        return new RestHighLevelClient(RestClient.builder(httpHostArray));
    }
}
```

### 3.控制类

```java
@Controller
public class PositionController {
    
    @Autowired
    private PositionService service;

    // 测试页面
    @GetMapping({"/","/index"})
    public String indexPage(){
        return "index";
    }
    
    // 根据关键字分页查询
    @GetMapping("/search/{field}/{keyword}/{pageNo}/{pageSize}")
    @ResponseBody
    public List<Map<String,Object>> searchPosition(@PathVariable("field") String field,@PathVariable("keyword") String keyword,@PathVariable("pageNo")int pageNo,@PathVariable("pageSize")int pageSize) throws IOException{
        List<Map<String,Object>> list = service.searchPos(field, keyword, pageNo, pageSize);
        return  list;
    }
    
    // mysql数据库数据导入es
    @RequestMapping("/importAll")
    @ResponseBody
    public String importAll(){
        try {
            service.importAll();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return  "success";
    }
}
```

### 4.服务类

服务接口：

```java
public interface PositionService {
    /*分页查询*/
    public List<Map<String,Object>> searchPos(String field,String keyword,int pageNo,int pageSize) throws  IOException;
    
    /*导入数据*/
    void importAll() throws IOException;
}
```

#### 4.1 分页搜索

多词查询：[https://www.elastic.co/guide/cn/elasticsearch/guide/current/match-multi-word.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/match-multi-word.html)

```java
@Service
public class PositionServiceImpl  implements PositionService {
    
    private  static  final Logger  logger = LogManager.getLogger(PositionServiceImpl.class);
    
    @Autowired
    private RestHighLevelClient  client;
    private  static  final  String  POSITION_INDEX = "position";

    // 搜索职位，分页查询
    @Override
    public List<Map<String, Object>> searchPos(String field, String keyword, int pageNo, int pageSize) throws IOException {
        if (pageNo <= 1){
            pageNo = 1;
        }
        //条件搜索
        SearchRequest  searchRequest = new SearchRequest(POSITION_INDEX);

        SearchSourceBuilder  searchSourceBuilder = new SearchSourceBuilder();
        //分页 index = (当前页-1)*一页显示条数
        searchSourceBuilder.from((pageNo-1)*pageSize);
        searchSourceBuilder.size(pageSize);
        QueryBuilder builder = QueryBuilders.matchQuery(field, keyword);
        searchSourceBuilder.query(builder);
        searchSourceBuilder.timeout(new TimeValue(60,TimeUnit.SECONDS));
        // 执行搜索
        searchRequest.source(searchSourceBuilder);
        SearchResponse  searchResponse = client.search(searchRequest,RequestOptions.DEFAULT);
        ArrayList<Map<String,Object>>  list = new ArrayList<>();
        SearchHit[]  hits = searchResponse.getHits().getHits();
        for (SearchHit hit:hits){
            list.add(hit.getSourceAsMap());
        }
        return   list;
    }

	// 将mysql数据批量写入es
    @Override
    public void importAll() throws IOException {
         writeMySQLDataToES("position");
    }
}
```

#### 4.2 MySQL数据批量写入ES

MySQL数据如下：

![image-20201219223853442](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219223853442.png)

将MySQL数据批量写入ES中，写入后ES中效果：

![image-20201219223402090](https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219223402090.png)

```java
private void writeMySQLDataToES(String tableName){
    // 获取BulkProcessor对象
    BulkProcessor  bulkProcessor  = getBulkProcessor(client);
    
    // 连接数据库
    Connection  connection = null;
    PreparedStatement  ps = null;
    ResultSet  rs = null;
    try {
        connection = DBHelper.getConn();
        logger.info("start handle data :" + tableName);
        String  sql = "select * from " + tableName;
        ps = connection.prepareStatement(sql,ResultSet.TYPE_FORWARD_ONLY,ResultSet.CONCUR_READ_ONLY);
        // 根据自己需要设置 fetchSize
        ps.setFetchSize(20);
        rs = ps.executeQuery();
        ResultSetMetaData  colData = rs.getMetaData();
        ArrayList<HashMap<String,String>> dataList = new ArrayList<>();
        // bulkProcessor 添加的数据支持的方式并不多，查看其api发现其支持map键值对的方式
        HashMap<String,String>  map = null;
        int  count = 0;
        // c 就是列的名字   v 就是列对应的值
        String  c = null;
        String  v = null;
        while(rs.next()){
            count ++;
            map = new HashMap<String,String>(128);
            for (int i=1;i< colData.getColumnCount();i++){
                c = colData.getColumnName(i);
                v = rs.getString(c);
                map.put(c,v);
            }
            dataList.add(map);

            // 每1万条 写一次 不足的批次的数据 最后一次提交处理
            if (count % 10000 == 0){
                logger.info("mysql handle data  number:"+count);
                // 将数据添加到 bulkProcessor
                for (HashMap<String,String> hashMap2 : dataList){
                    bulkProcessor.add(new IndexRequest(POSITION_INDEX).source(hashMap2));
                }
                // 每提交一次，清空map和 dataList
                map.clear();
                dataList.clear();
            }
        }
        // 处理未提交的数据
        for (HashMap<String,String> hashMap2 : dataList){
            bulkProcessor.add(new IndexRequest(POSITION_INDEX).source(hashMap2));
        }
        logger.info("-------------------------- Finally insert number total : " + count);
        //将数据刷新到es, 注意这一步执行后并不会立即生效，取决于bulkProcessor设置的刷新时间
        bulkProcessor.flush();

    } catch (SQLException e) {
        e.printStackTrace();
    }finally {
        try {
            rs.close();
            ps.close();
            connection.close();
            boolean  terinaFlag = bulkProcessor.awaitClose(150L,TimeUnit.SECONDS);
            logger.info(terinaFlag);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

使用Bulk Processor 官网地址：[Using Bulk Processor](https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.3/java-docs-bulk-processor.html)

```java
private BulkProcessor getBulkProcessor(RestHighLevelClient client) {

    BulkProcessor bulkProcessor = null;
    try {
        BulkProcessor.Listener listener = new BulkProcessor.Listener() {
            @Override
            public void beforeBulk(long executionId, BulkRequest request) {
                logger.info("Try to insert data number : "
                            + request.numberOfActions());
            }

            @Override
            public void afterBulk(long executionId, BulkRequest request,
                                  BulkResponse response) {
                logger.info("************** Success insert data number : "
                            + request.numberOfActions() + " , id: " + executionId);
            }

            @Override
            public void afterBulk(long executionId, BulkRequest request, Throwable failure) {
                logger.error("Bulk is unsuccess : " + failure + ", executionId: " + executionId);
            }
        };

        BiConsumer<BulkRequest, ActionListener<BulkResponse>> bulkConsumer = (request, bulkListener) -> client
            .bulkAsync(request, RequestOptions.DEFAULT, bulkListener);

        BulkProcessor.Builder builder = BulkProcessor.builder(bulkConsumer, listener);
        builder.setBulkActions(5000);
        builder.setBulkSize(new ByteSizeValue(100L, ByteSizeUnit.MB));
        builder.setConcurrentRequests(10);
        builder.setFlushInterval(TimeValue.timeValueSeconds(100L));
        builder.setBackoffPolicy(BackoffPolicy.constantBackoff(TimeValue.timeValueSeconds(1L), 3));
        // 注意点：让参数设置生效
        bulkProcessor = builder.build();

    } catch (Exception e) {
        e.printStackTrace();
        try {
            bulkProcessor.awaitClose(100L, TimeUnit.SECONDS);
        } catch (Exception e1) {
            logger.error(e1.getMessage());
        }
    }
    return bulkProcessor;
}
```

#### 4.3 SQL脚本

```sql
/*
SQLyog Ultimate v12.09 (64 bit)
MySQL - 5.7.31 : Database - lagou_position
*********************************************************************
*/
/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`lagou_position` /*!40100 DEFAULT CHARACTER SET latin1 */;

USE `lagou_position`;

/*Table structure for table `position` */

DROP TABLE IF EXISTS `position`;

CREATE TABLE `position` (
  `companyName` varchar(300) DEFAULT NULL,
  `id` double DEFAULT NULL,
  `positionAdvantage` varchar(300) DEFAULT NULL,
  `companyId` double DEFAULT NULL,
  `positionName` varchar(240) DEFAULT NULL,
  `salary` varchar(120) DEFAULT NULL,
  `salaryMin` double DEFAULT NULL,
  `salaryMax` double DEFAULT NULL,
  `salaryMonth` double DEFAULT NULL,
  `education` varchar(60) DEFAULT NULL,
  `workYear` varchar(60) DEFAULT NULL,
  `jobNature` varchar(120) DEFAULT NULL,
  `chargeField` blob,
  `createTime` datetime DEFAULT NULL,
  `email` varchar(300) DEFAULT NULL,
  `publishTime` varchar(150) DEFAULT NULL,
  `isEnable` double DEFAULT NULL,
  `isIndex` double DEFAULT NULL,
  `city` varchar(150) DEFAULT NULL,
  `orderby` double DEFAULT NULL,
  `isAdvice` double DEFAULT NULL,
  `showorder` double DEFAULT NULL,
  `publishUserId` double DEFAULT NULL,
  `workAddress` varchar(300) DEFAULT NULL,
  `generateTime` datetime DEFAULT NULL,
  `bornTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `isReward` double DEFAULT NULL,
  `rewardMoney` varchar(60) DEFAULT NULL,
  `isExpired` double DEFAULT NULL,
  `positionDetailPV` double DEFAULT NULL,
  `offlineTime` datetime DEFAULT NULL,
  `positionDetailPV_cnbeta` double DEFAULT NULL,
  `adviceTime` datetime DEFAULT NULL,
  `comeFrom` varchar(150) DEFAULT NULL,
  `receivedResumeCount` double DEFAULT NULL,
  `refuseResumeCount` double DEFAULT NULL,
  `markCanInterviewCount` double DEFAULT NULL,
  `haveNoticeInterCount` double DEFAULT NULL,
  `isForbidden` double DEFAULT NULL,
  `reason` varchar(768) DEFAULT NULL,
  `verifyTime` datetime DEFAULT NULL,
  `adWord` double DEFAULT NULL,
  `adRankAndTime` varchar(120) DEFAULT NULL,
  `adTimes` double DEFAULT NULL,
  `adStartTime` datetime DEFAULT NULL,
  `adEndTime` datetime DEFAULT NULL,
  `adBeforeDetailPV` double DEFAULT NULL,
  `adAfterDetailPV` double DEFAULT NULL,
  `adBeforeReceivedCount` double DEFAULT NULL,
  `adAfterReceivedCount` double DEFAULT NULL,
  `adjustScore` double DEFAULT NULL,
  `weightStartTime` datetime DEFAULT NULL,
  `weightEndTime` datetime DEFAULT NULL,
  `isForward` bit(1) DEFAULT NULL,
  `forwardEmail` varchar(300) DEFAULT NULL,
  `isSchoolJob` bit(1) DEFAULT NULL,
  `type` tinyint(4) DEFAULT NULL,
  `prolong_offline_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `position` */

insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('百度',1,'lianhfisnhdh',141823,'java','15k-25k',15,25,0,'本科','3-5年','全职',NULL,'2018-12-06 17:17:24','',NULL,1,0,'青岛',NULL,0,0,100013344,NULL,'2018-11-22 14:58:49','2020-09-06 11:52:45',0,NULL,1,0,'2018-12-06 17:21:26',0,NULL,'andorid',0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,'2020-07-27 14:55:59','2020-09-30 00:00:58','\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('阿里',3,'Web测试工程师1',141956,'Web测试工程师1','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2017-08-24 07:14:11','anan1@lagou.com','0',0,0,'青岛',65,0,1373869361621,100013340,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,828,'2018-05-02 16:00:05',0,NULL,'andorid',0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('腾讯',4,'美女多222',147,'Web测试工程师','10k-12k',10,12,0,'本科','4','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','2',0,0,'北京',65,0,1373870620501,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,752,'2017-10-28 00:00:05',0,NULL,'lagouOpen',0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'','alan@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('360',5,'手机测试工程师111',147,'手机测试工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','2',0,0,'北京',65,0,1373875824893,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,788,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('小米',6,'前端工程师（页面制作）',147,'前端工程师（页面制作）','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','6',0,0,'北京',65,0,1373876343375,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,723,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('京东',7,'PHP软件工程师',147,'PHP软件工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','0',0,0,'北京',65,0,1373877071767,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,634,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('亚马逊',8,'PHP软件工程师',147,'PHP软件工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','2',0,0,'北京',65,0,1373877366356,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,774,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('口袋购物',9,'IPhone手机开发工程师',147,'IPhone手机开发工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','0',0,0,'北京',65,0,1373877881539,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,650,'2017-01-16 09:41:33',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('当当网',10,'产品经理',147,'产品经理','2k-20k',2,20,0,'本科','3-5年','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','6',0,0,'北京',65,0,1373878138496,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,299,'2017-01-16 09:41:34',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('聚美优品',11,'扁平管理',147,'C++高级软件工程师（界面开发）','2k-20k',2,20,0,'本科','不限','全职',NULL,'2016-07-22 10:24:18','fengshao@lagou.com','0',0,0,'北京',100,0,1373878590050,100011815,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,34,'2017-01-16 09:41:34',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('美团',12,'产品运营',147,'产品运营','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2019-01-29 11:18:13','fengshao@lagou.com','7',0,0,'北京',65,0,1373878652026,100011815,'北京','2015-08-18 12:05:24','2019-01-30 12:02:33',0,NULL,1,327,'2017-01-16 09:41:34',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('拉手网',13,'制度和环境比较人性化',5,'Flash开发工程师','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:24','fengshao@lagou.com','0',0,0,'上海',100,0,1373878742151,2,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,40,'2018-10-20 00:00:19',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('明星衣橱',14,'扁平管理',5,'PHP开发工程师','2k-20k',2,20,0,'大专','3-5年','全职',NULL,'2020-06-15 18:06:24','fengshao@lagou.com','1',0,0,'上海',100,0,1373878926196,2,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,26,'2018-10-20 00:00:19',1,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('美丽说',15,'帅哥美女多',4,'社区运营经理','2k-20k',2,20,0,'大专','1-3年','全职',NULL,'2020-06-15 18:06:08','6666@qq.com','3',0,0,'上海',65,0,1373878937637,1,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,234,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('去哪儿',16,'扁平管理',5,'产品经理','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2020-06-15 18:06:24','fengshao@lagou.com','18',0,0,'上海',100,0,1373879131552,2,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,63,'2018-10-20 00:00:20',2,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('艺龙',17,'员工福利好',4,'UI交互设计师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2020-06-15 18:06:09','6666@qq.com','8',0,0,'上海',65,0,1373879215143,1,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,243,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('豆瓣',18,'员工福利好',4,'视觉设计师','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:09','6666@qq.com','8',0,0,'上海',65,0,1373879318142,1,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,243,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('知乎',19,'员工福利好',5,'JAVA开发工程师','2k-20k',2,20,0,'大专','1-3年','全职',NULL,'2020-06-15 18:06:24','fengshao@lagou.com','12',0,0,'上海',100,0,1373879768887,2,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,6,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('Nice',20,'员工福利好',4,'数据编辑','2k-20k',2,20,0,'大专','不限','全职',NULL,'2020-06-15 18:06:09','6666@qq.com','3',0,0,'上海',65,0,1373879543712,1,'上海','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,214,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('脉脉',21,'帅哥多',5,'测试工程师（移动端）','2k-20k',2,20,0,'大专','1-3年','全职',NULL,'2020-06-15 18:06:25','fengshao@lagou.com','1',0,0,'上海',100,0,1373879757316,2,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,5,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('果壳',22,'员工福利好',4,'网站运营专员','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2020-06-15 18:06:10','6666@qq.com','4',0,0,'上海',65,0,1373879661503,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,315,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('陌陌',23,'美女多、员工福利好',5,'IOS 开发工程师','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:25','fengshao@lagou.com','2',0,0,'上海',100,0,1373879747360,2,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,3,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('锤子手机',24,'节日礼物',5,'Android开发工程师','2k-20k',2,20,0,'本科','1-3年','全职',NULL,'2020-06-15 18:06:25','fengshao@lagou.com','3',0,0,'上海',100,0,1373879738700,2,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,8,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('凤凰网',25,'员工福利好',4,'BD','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:10','6666@qq.com','0',0,0,'上海',65,0,1373879769461,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,360,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('豌豆荚',26,'美女多、员工福利好',5,'PHP开发工程师（游戏网站）','2k-20k',2,20,0,'大专','1-3年','全职',NULL,'2020-06-15 18:06:25','fengshao@lagou.com','1',0,0,'上海',100,0,1373880113172,2,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,6,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,1,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('友盟',27,'员工福利好',6,'产品经理','2k-20k',2,20,0,'不限','不限','全职',NULL,'2020-06-15 18:06:11','6666@qq.com','6',0,0,'上海',55,0,1373881057811,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,210,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('新浪',28,'员工福利好',8,'编辑助理','2k-20k',2,20,0,'不限','不限','全职',NULL,'2020-06-15 18:06:11','6666@qq.com','3',0,0,'上海',15,0,1373882286658,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,360,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('网易',29,'员工福利好',8,'美工、网页设计','2k-20k',2,20,0,'不限','不限','全职',NULL,'2020-06-15 18:06:11','6666@qq.com','1',0,0,'上海',15,0,1373882357697,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,329,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('今日头条',30,'职位年薪：15-30万',7,'js前端高级工程师','12k-25k',12,25,0,'本科','3-5年','全职',NULL,'2016-07-22 10:24:18','test@testtest01.com','1',1,0,'北京',50,0,1373882639044,183,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,379,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','fengshao@lagou.com','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('极客公园',31,'薪资构成：基本薪资 + 奖金/提成',7,'UI设计师','2k-20k',2,20,0,'不限','不限','全职',NULL,'2016-07-22 10:24:18','test@testtest01.com','25',1,0,'北京',50,0,1373883039599,183,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,311,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('微软',32,'通信交通：有补助',7,'android程序员','12k-15k',12,15,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','test@testtest01.com','3',1,0,'北京',50,0,1373883142193,183,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,270,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('36氪',33,'居住福利：住房补贴',7,'IOS程序员','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2016-07-22 10:24:18','test@testtest01.com','13',1,0,'北京',50,0,1373883247413,183,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,301,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,'2019-04-11 00:00:00','2019-04-18 23:59:59','\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('搜房网',34,'国际企业',9,'技术开发工程师','7k-10k',7,10,0,'不限','不限','全职',NULL,'2020-06-15 18:06:12','6666@qq.com','1',0,0,'北京',40,0,1373884037734,1,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,337,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('央视网',35,'员工福利好',9,'Flash 资深设计师','10k-15k',10,15,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','0',0,0,'北京',40,0,1373884229000,1,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,371,'2018-10-20 00:00:20',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('汽车之家',36,'员工福利好',9,'数码分析师','7k-10k',7,10,0,'本科','不限','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','3',0,0,'北京',40,0,1373884393050,1,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,290,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('印象笔记',37,'员工福利好',9,'数据顾问','7k-10k',7,10,0,'本科','不限','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','7',0,0,'北京',40,0,1373884954342,1,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,379,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('金山',38,'如果你只想跟聪明的人一起工作，就加入我们',10,'软件工程师','7k-7k',7,7,0,'本科','不限','全职',NULL,'2016-07-22 10:24:18','anming@hsgh.com','1',1,0,'北京',65,0,1373885055935,193,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,292,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('滴滴',39,'优秀者，可解决北京市户口',11,'手机测试工程师','2k-5k',2,5,0,'大专','1-3年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','9',1,0,'北京',40,0,1373886219984,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,348,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('猎豹移',40,'优秀者，可解决北京市户口',11,'美术UI','5k-10k',5,10,0,'本科','3-5年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','1',1,0,'北京',40,0,1373886377328,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,345,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('联想',41,'员工福利好',12,'系统工程师','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','1',0,0,'上海',15,0,1373890215550,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,315,'2018-10-20 00:00:21',2,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,42,'员工福利好',12,'前端工程师','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','3',0,0,'上海',15,0,1373890347725,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,545,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values ('APUS',43,'员工福利好',12,'Android开发工程师','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:13','6666@qq.com','6',0,0,'上海',15,0,1373890444399,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,296,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,44,'员工福利好',12,'IOS开发工程师','2k-20k',2,20,0,'不限','1-3年','全职',NULL,'2020-06-15 18:06:14','6666@qq.com','7',0,0,'上海',15,0,1373890590732,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,327,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,45,'优秀者，可解决北京市户口',11,'ios海外推广','5k-10k',5,10,0,'本科','1-3年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','8',1,0,'北京',40,0,1373890665844,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,332,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,46,'优秀者，可解决北京市户口',11,'美术特效','5k-10k',5,10,0,'不限','3-5年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','1',1,0,'北京',40,0,1373890778792,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,326,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,47,'员工福利好',12,'数据库工程师','2k-20k',2,20,0,'不限','不限','全职',NULL,'2020-06-15 18:06:14','6666@qq.com','0',0,0,'上海',15,0,1373890917741,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,190,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,48,'优秀者，可解决北京市户口',11,'测试实习','2k-5k',2,5,0,'本科','应届毕业生','实习',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','13',1,0,'北京',40,0,1373890841711,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,327,'2018-10-20 00:00:21',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,49,'员工福利好',12,'后端工程师','2k-20k',2,20,0,'本科','不限','全职',NULL,'2020-06-15 18:06:14','6666@qq.com','0',0,0,'上海',15,0,1373890902610,1,'上海','2013-07-20 11:01:35','2013-07-20 11:01:35',0,NULL,1,162,'2018-10-20 00:00:22',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,50,'优秀者，可解决北京市户口',11,'.NET开发工程师','5k-10k',5,10,0,'不限','1-3年','全职',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','5',1,0,'北京',40,0,1373890960056,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,329,'2018-10-20 00:00:22',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
insert  into `position`(`companyName`,`id`,`positionAdvantage`,`companyId`,`positionName`,`salary`,`salaryMin`,`salaryMax`,`salaryMonth`,`education`,`workYear`,`jobNature`,`chargeField`,`createTime`,`email`,`publishTime`,`isEnable`,`isIndex`,`city`,`orderby`,`isAdvice`,`showorder`,`publishUserId`,`workAddress`,`generateTime`,`bornTime`,`isReward`,`rewardMoney`,`isExpired`,`positionDetailPV`,`offlineTime`,`positionDetailPV_cnbeta`,`adviceTime`,`comeFrom`,`receivedResumeCount`,`refuseResumeCount`,`markCanInterviewCount`,`haveNoticeInterCount`,`isForbidden`,`reason`,`verifyTime`,`adWord`,`adRankAndTime`,`adTimes`,`adStartTime`,`adEndTime`,`adBeforeDetailPV`,`adAfterDetailPV`,`adBeforeReceivedCount`,`adAfterReceivedCount`,`adjustScore`,`weightStartTime`,`weightEndTime`,`isForward`,`forwardEmail`,`isSchoolJob`,`type`,`prolong_offline_time`) values (NULL,51,'优秀者，可解决北京市户口',11,'美术实习','2k-5k',2,5,0,'硕士','1年以下','实习',NULL,'2016-07-22 10:24:18','lixiao@a-onesoft.com','1',1,0,'北京',40,0,1373891110617,197,'北京','2015-08-18 12:05:24','2015-08-18 12:05:24',0,NULL,1,267,'2018-10-20 00:00:22',0,NULL,NULL,0,0,0,0,0,NULL,NULL,0,NULL,0,NULL,NULL,0,0,0,0,0,NULL,NULL,'\0','','\0',0,NULL);
commit;
/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```

### 5.实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Position {
    //主键
    private String id;

    //公司名称
    private String companyName;

    //职位名称
    private String positionName;

    //职位诱惑
    private String  positionAdvantage;

    //薪资
    private String salary;

    //薪资下限
    private int salaryMin;

    //薪资上限
    private int salaryMax;

    //学历
    private String education;

    //工作年限
    private String workYear;

    //发布时间
    private String publishTime;

    //工作城市
    private String city;

    //工作地点
    private String workAddress;

    //创建时间
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;

    //工作模式
    private String jobNature;
}
```

### 6.前端页面

```html
<!DOCTYPE html>
<html xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <link rel="stylesheet" href="layout_8a42639.css">
    <link rel="stylesheet"  href="main.html_aio_2_29de31f.css">
    <link rel="stylesheet"  href="main.html_aio_16da111.css">
    <link rel="stylesheet"  href="widgets_03f0f0e.css">

    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
</head>
<body>
<div id="top_bannerC">
    <a rel="nofollow" href="https://zhuanti.lagou.com/qiuzhaozhc.html"
       style="background: url(https://www.lgstatic.com/i/image/M00/3C/48/CgqCHl8nkiOAOihGAAC5a9EQ2rU358.PNG) center center no-repeat"
       target="_blank" data-lg-tj-id="bd00" data-lg-tj-no="idnull" data-lg-tj-cid="11127" class=""></a>
</div>

<div id="lg_header" >
    <!-- 页面主体START -->
    <div id="content-container">

        <div class="search-wrapper" style="height: 180px;">
            <div id="searchBar" class="search-bar" style="padding-top: 30px;">
                <div class="tab-wrapper" style="display: block;">
                    <a id="tab_pos" class="active" rel="nofollow" href="javascript:;">职位 ( <span>500+</span> ) </a>
                    <a id="tab_comp" class="disabled" rel="nofollow" href="javascript:;">公司 ( <span>0</span> ) </a>
                </div>
                <div class="input-wrapper" data-lg-tj-track-code="search_search" data-lg-tj-track-type="1">
                    <div class="keyword-wrapper">
                        <span role="status" aria-live="polite" class="ui-helper-hidden-accessible"></span><input
                            type="text" id="keyword" v-model="keyword" autocomplete="off" maxlength="64" placeholder="搜索职位、公司或地点"
                            value="java" class="ui-autocomplete-input"   @keyup.enter="searchPosition">
                    </div>
                    <input type="button" id="submit"  value="搜索" @click="searchPosition">
                </div>
            </div>
        </div>

        <!-- 搜索输入框模块 -->
        <div id="main_container">
            <!-- 左侧 -->
            <div class="content_left">
                <div class="s_position_list " id="s_position_list">
                    <ul class="item_con_list" style="display: block;">
                        <li class="con_list_item default_list"  v-for="element in results">
                            <span class="top_icon direct_recruitment"></span>
                            <div class="list_item_top">
                                <div class="position">
                                    <div class="p_top">
                                        <h3 style="max-width: 180px;">{{element.positionName}}</h3>
                                        <span class="add">[<em>{{element.workAddress}}</em>]</span>
                                        <span class="format-time"> {{element.createTime}} 发布</span>
                                    </div>
                                    <div class="p_bot">
                                        <div class="li_b_l">
                                            <span class="money">{{element.salary}}</span>
                                            经验 {{element.workYear}} 年 / {{element.education}}
                                        </div>
                                    </div>
                                </div>
                                <div class="company">
                                    <div class="company_name">
                                        <a href="#" target="_blank">{{element.companyName}}</a>
                                        <i class="company_mark"><span>该企业已经上传营业执照并通过资质验证审核</span></i>
                                    </div>
                                    <div class="industry">
                                        福利 【{{element.positionAdvantage}}】
                                    </div>
                                </div>
                                <div class="com_logo">
                                    <img src="//www.lgstatic.com/thumbnail_120x120/i/image2/M01/79/70/CgotOV1aS4qAWK6WAAAM4NTpXws809.png"
                                         alt="拉勾渠道" width="60" height="60">
                                </div>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</div>
</div>
</body>
<script>
    new Vue({
        el: "#lg_header",
        data: {
            keyword: "",  //搜索关键字
            results: []  //查询结果
        },
        methods: {
            searchPosition(){
                var keyword = this.keyword;
                let field = "positionName";
                axios.get('http://localhost:8080/search/' + field + '/' + keyword + '/1/5').then(response =>{
                    this.results = response.data;
                    //如果返回结果不够5条，根据职位优势推荐查询数据补全5条
                    if (this.results.length < 5){
                        field = "positionAdvantage";
                        let fieldValue = "美女多,员工福利好";
                        let num = 5 - this.results.length;
                        axios.get('http://localhost:8080/search/' + field +'/'+ fieldValue + '/1/' + num).then(response =>{
                            response.data.forEach(item => {
                                this.results.push(item);
                            })
                        });
                    }
                });
            }
        }
    });
</script>

</html>
```

### 7.效果图

<img src="https://gitee.com/itzlg/mypictures/raw/master/img/image-20201219222642525.png" alt="image-20201219222642525" />

















