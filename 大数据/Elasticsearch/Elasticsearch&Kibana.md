[toc]



# Elasticsearch

## 简介

Elasticsearch 是一个分布式、高扩展、高实时的搜索与[数据分析](https://baike.baidu.com/item/数据分析/6577123)引擎。它能很方便的使大量数据具有搜索、分析和探索的能力。充分利用Elasticsearch的水平伸缩性，能使数据在生产环境变得更有价值。

> **Elasticsearch**是与名为**Logstash**的数据收集和日志解析引擎以及名为**Kibana**的分析和可视化平台一起开发的。这三个产品被设计成一个集成解决方案，称为“Elastic Stack”（以前称为“ELK stack”）。

### 优点

- 横向可扩展：只需要增加一台服务器，做一点配置，启动一下Elasticsearch进程就可以并入集群
- 分片机制提供更好的分布性：同一个索引分成多个分片（sharding)，这点类似于HDFS的块机制；分而治之的方式可提升处理效率
- 高可用：提供复制（replica）机制，一个分片可以设置多个复制，使得某台服务器在宕机的情况下，集群依旧可以照常运行，并会把服务器宕机丢失数据信息复制恢复到其他可用节点上。
- 使用简单：只需一条命令就可以下载文件，然后很快就能搭建一个站内搜索引擎。

### 实现原理

1. 首先用户将数据提交到Elasticsearch 数据库中
2. 再通过分词控制器去将对应的语句分词，将其权重和分词结果一并存入数据
3. 当用户搜索数据时候，再根据权重将结果排名，打分，再将返回结果呈现给用户。



## 安装

### Windows

 [1](https://www.cnblogs.com/hualess/p/11540477.html)

1. 下载[elasticsearch-7.15.1-windows-x86_64.zip](https://artifacts.elastic.co/downloads/kibana/kibana-7.15.1-windows-x86_64.zip)

2. 解压进入elasticsearch/bin目录`cd elasticsearch-7.15.1/bin`双击`elasticsearch.bat`即可启动。

3. 在打开浏览器，输入localhost:9200即可查看是否安装成功

   ![image-20211029093846148](http://qiliu.luxiaobai.cn/img/image-20211029093846148.png)

4. 安装head插件(ES最初的管理工具，在浏览器中显示ES集群，索引等信息),在浏览器输入http://localhost:9100

   ```shell
   git clone git://github.com/mobz/elasticsearch-head.git
   cd elasticsearch-head
   npm install
   npm run start
   ```

5. head插件要访问ES，需要修改ES使用的参数,重启启动ES

   ```shell
   cd cd elasticsearch-7.15.1/conf
   vim elasticsearch.yml
   
   ##增加以下两行参数，就可以访问es
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   ```



### 后台运行步骤

```shell
cd elasticsearch\bin
elasticserach-service install			//安装elasticsearch-service
elasticsearch-service manager			//启动服务
```



## ES配置

主要有三个配置文件：

- `elasticsearch.yml`ES的配置，[more](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)
- `jvm.options` ES JVM配置，[more](https://www.elastic.co/guide/en/elasticsearch/reference/current/jvm-options.html#jvm-options)
- `log4j2.properties` ES 日志配置， [more](https://www.elastic.co/guide/en/elasticsearch/reference/current/logging.html#logging)



### JVM配置

默认情况下，弹性搜索会根据节点的角色和总内存自动设置JVM堆大小。建议大多数生产环境中的默认尺寸

JVM参数设置在ES_HOME/conf/jvm.options（推荐）中进行修改或者ES_JAVA_HOME环境变量来修改

[官网介绍了如何设置堆大小](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)

> 默认情况，ES 告诉 JVM 使用一个最小和最大都为 1GB 的堆。在生产环境，这个配置就比较重要了，确保 ES 有足够堆空间可用。
>
> ES 使用 `Xms(minimum heap size)` 和 `Xmx(maxmimum heap size)` 设置堆大小。你应该将这两个值设为同样的大小。
>
> **`Xms` 和 `Xmx` 不能大于你物理机内存的 50%。**



### [SLM自动备份](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-snapshot-lifecycle-management.html)

#### 配置快照存储库

在`elasticsearch.yml`配置存储库位置添加该参数`path.repo: ["D:\\eleasticSearchs\\snapshot_repo"]`

```http
PUT /_snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "D:\\eleasticSearchs\\snapshot_repo\\my_backup"
  }
}

#验证集群各个节点是否可以使用这个仓库
POST /_snapshot/my_backup/_verify

#保存集群下所有索引快照
#格式: _snapshot/仓库名称/快照名称
PUT _snapshot/my_backup/snapshot_1


##保存集群下指定索引快照
PUT _snapshot/my_backup/snapshot_2?wait_for_completion=true
{
    "indices": "txt_number"
}

#获取指定索引快照状态
GET _snapshot/my_backup/snapshot_2/_status
```

>**不要害怕配置频繁拍摄快照的策略。快照是增量的，有可能很多快照依赖于过去的段并可以有效地利用存储。所以在仓库下的文件不要手动删除**



#### snapshot还原

只需要在集群的快照ID后加上`_restore`即可

```http
POST _snapshot/my_backup/snapshot_2/_restore
```

如果集群中已有快照的索引那就会报索引已存在的错误

**如果在不替换现有数据的前提下，恢复老数据来验证内容，或者做其他处理。可以从快照里恢复单个索引并提供一个替换的名称：** 

```http
POST _snapshot/my_backup/snapshot_2/_restore
{
  "indices": "txt_number",
  "ignore_unavailable": true,
  "include_global_state": false,
  "rename_pattern": "txt_number",
  "rename_replacement": "restore_txt_number2",
  "include_aliases": false
}
```

==数据还原时，需要等待==默认情况下每个节点的节流恢复速度默认为40mb/s,每个系欸但的快照速率限制，默认为40mb/s



**其他**

```http
##删除快照
DELETE _snapshot/my_backup/snapshot_2

##删除多个
DELETE _snapshot/my_backup/snapshot_1,snapshot_2

##过期快照清理
POST _snapshot/my_backup/_cleanup
```





## 基本概念

**index(索引）**：相当于mysql中的数据库

**type(类型）**：相当于mysql中的一张表

**document(文档）**:相当于mysql中的一行（一条记录）

**field(域）**: 相当于mysql中的一列（一个字段）

**节点** ：一个服务器，由一个名字来标识

**集群** ：一个或多个节点组织在一起

**分片** ：将一份数据划分为多小份的能力，允许水平分割和扩展容量。多个分片可以响应请求，提高性能和吞吐量。

**副本** ：复制数据，一个节点出问题时，其余节点可以顶上。

>Elasticsearch 是利用分片将数据分发到集群内各处的。分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。
>当你的集群规模扩大或者缩小时， Elasticsearch 会自动的在各节点中迁移分片，使得数据仍然均匀分布在集群里。
>
>一个分片可以是 主 分片或者 副本 分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。
>
>一个副本分片只是一个主分片的拷贝。 副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。

创建一个索引，主分片为5，副分片为1

```shell
PUT job
{
	"settings":{
		"index":{
			"number_of_shards":5,
			"number_of_replicas":1
		}
	}
}

{
	"acknowledged":true
	"shards_acknowledged":true
}
```



> 分片是Elasticsearch在集群周围分发数据的单位。 Elasticsearch在重新平衡数据时 （例如 发生故障后） 移动分片的速度 取决于分片的大小和数量以及网络和磁盘性能。
>
> 提示：避免有非常大的分片，因为大的分片可能会对集群从故障中恢复的能力产生负面影响。 对于多大的分片没有固定的限制，但是分片大小为50GB通常被界定为适用于各种用例的限制。

#### 基本字段类型

- 字符串： text(分词), keyword(不分词), StringField(不分词文本)，TextFiled(要分词文本)，text默认为全文文本，keyword默认为非全文文本
- 数字： long, integer, short, double, float
- 日期： date
- 逻辑： Boolean

#### 复制字段类型

- 对象类型： object
- 数组类型： array
- 地理位置： geo_point, geo_shape

#### 默认映射

|            JSON type             | Field type |
| :------------------------------: | :--------: |
|      Boolean: true or false      | "boolean"  |
|        Whole number: 123         |   "long"   |
|      Floating point: 123.45      |  "double"  |
| String, valid date: "2014-09-15" |   "date"   |
|        String: "foo bar"         |  "string"  |



#### 映射规则

|     属性名      |                             解释                             |
| :-------------: | :----------------------------------------------------------: |
|      type       | 字段的类型：基本数据类型，integer,long,date,boolean,keyword,text… |
|     enable      |     是否启用：默认为true。 false：不能索引、不能搜索过滤     |
|      boost      | 权重提升倍数：用于查询时加权计算最终的得分,比如标题中搜索到的内容比简介中搜索到的内容跟重要，那么可以提升 |
|      index      | 索引模式：analyzed (索引并分词,text默认模式), not_analyzed (索引不分词，keyword默认模式)，no（不索引） |
|    analyzer     | 索引分词器：索引创建时使用的分词器，如ik_smart,ik_max_word,standard |
| search_analyzer |    搜索分词器：搜索该字段的值时，传入的查询内容的分词器。    |
|     fields      | 多字段索引：当对该字段需要使用多种索引模式时使用。如：城市搜索New York"city": |





## 倒排索引

>ElasticSearch引擎把文档数据写入到倒排索引（Inverted Index）的数据结构中，倒排索引建立的是分词（Term）和文档（Document）之间的映射关系，在倒排索引中，数据是面向词（Term）而不是面向文档的
>
>一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表





### 组成部分

#### **单词词典（Term Dictionary）**

搜索引擎通常索引单位是==单词==，单词是**由文档集合中出现过的所有单词构成的字符串集合**，同时用来记载某个单词对应的倒排列表在倒排文件中的位置信息。

==结构组成：哈希+链表+B树（或者B+树）==



主体部分是**哈希表**，每个哈希表项保存一个指针，指针指向冲突链表，在冲突链表里，相同哈希值的单词形成链表结构。

查询某个词时，利用哈希函数获取对应的哈希值，根据哈希值找到对应哈希表中保存的指针,找到对应的冲突链表。在冲突链表中找到对应的倒排列表。



树形结构主要根据层级查找，中间节点用于指出一定顺序范围的词典项目存储在哪个子树中，起到根据词典项比较大小进行导航的作用，最底层的叶子节点存储单词的地址信息，根据这个地址就可以提取出单词字符串



#### **倒排列表（Posting List）**

倒排列表**记载了出现过某个单词的所有文档的文档列表**及**单词在该文档中出现的位置信息**及**频率**做==关键性算分==，每条记录称为一个倒排项**Posting**。根据倒排列表，即可获知哪些文档包含某个单词。

倒排索引项包含如下信息：

1. 文档ID：用于获取原始信息
2. 单词频率（TF，Term Frequency): 记录该单词在该文档中出现的次数，用于后续相关性算分
3. 位置(Positing): 记录单词在文档中的分词位置（多个），用于词语搜索(Phrase Query)
4. 偏移(Offset): 记录单词在文档的开始和结束位置，用于高亮显示



#### **倒排文件（Inverted File）**

所有单词的倒排列表往往顺序地存储在磁盘的某个文件里，这个文件即被称之为倒排文件，==倒排文件==是**存储倒排索引的物理文件**





### 示例

| 文档ID |                  文档内容                  |
| :----: | :----------------------------------------: |
|   1    |          谷歌地图之父跳槽Facebook          |
|   2    |          谷歌地图之父加盟Facebook          |
|   3    |   谷歌地图创始人拉斯离开谷歌加盟Facebook   |
|   4    | 谷歌地图之父跳槽Facebook与Wave项目取消有关 |
|   5    |    谷歌地图之父拉斯加盟社交网站Facebook    |

| 单词ID | 单词 | 文档频率 |              倒排列表（DocID;TF;$<POS>$)               |
| :----: | ---- | :------: | :----------------------------------------------------: |
|   1    | 谷歌 |    5     | (1;1;<1>), (2;1;<1>), (3;2<1;6>), (4;1;<1>), (5;1;<1>) |
|   2    | 地图 |    5     | (1;1;<2>), (2;1;<2>), (3;1;<2>), (4;1;<2>), (5;1;<2>)  |
|   3    | 之父 |    4     |       (1;1;<3>), (2;1<3>), (4;1;<3>), (5;1;<3>)        |
|   4    | 跳槽 |    2     |                  (1;1;<4>),(4;1;<4>)                   |

>**单词ID**：记录每个单词的单词编号；
>**单词**：对应的单词；
>**文档频率**：代表文档集合中有多少个文档包含某个单词
>**倒排列表**：包含单词ID及其他必要信息
>**DocId**：单词出现的文档id
>**TF**：单词在某个文档中出现的次数
>**POS**：单词在文档中出现的位置



#### 倒排索引不可变的好处

- 不需要锁，提升并发能力，避免锁的问题
- 数据不变，一直保存在os cache中，只要cache内存足够
- filter cache一直驻留在内存，因为数据不变
- 可以压缩，节省cpu和IO开销





## 分词

> 分词是将文本转换成一系列单词（Term or Token）的过程，也可以叫文本分析，在ES里面称为Analysis



### 分词器

分词器是ES中专门处理分词的组件，英文为Analyzer，它的组成如下：

**Character Filters**：针对原始文本进行处理，比如去除html标签
**Tokenizer**：将原始文本按照一定规则切分为单词
**Token Filters**：针对Tokenizer处理的单词进行再加工，比如转小写、删除或增新等处理

[预定义分词器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html)

- Standard Analyzer
  - 默认分词器
  - 按词切分，支持多语言
  - 小写处理

- Simple Analyzer
  - 按照非字母切分
  - 小写处理

- Whitespace Analyzer
  - 空白字符作为分隔符

- Stop Analyzer
  - 相比Simple Analyzer多了去除请用词处理
  - 停用词指语气助词等修饰性词语，如the, an, 的， 这等
- Keyword Analyzer
  - 不分词，直接将输入作为一个单词输出
- Pattern Analyzer
  - 通过正则表达式自定义分隔符
  - 默认是\W+，即非字词的符号作为分隔符
- Language Analyzer
  - 提供了30+种常见语言的分词器



### 中文分词器插件es-ik安装

- 点击[ES-ik](https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.15.1)下载压缩包，解压到ES\plugin\ik目录下
- 重启ES即可





## 版本不一致问题

1、ES 7.x默认不再支持指定索引类型，默认索引类似是_doc，如果想改变，则需要配置`include_type_name:true`。同时这一字段将在8.x舍弃

```err
PUT /shakespeare?include_type_name=true
{
  "mappings": {
    "doc" : {
      "properties":{
        "speaker":{
          "type": "keyword"
        },
        "play_name":{
          "type": "keyword"
        },
        "line_id":{
          "type": "integer"
        },
        "speech_number":{
          "type": "integer"
        }
      }
    }
  }
}
```



2、数据导入,导入之前需创建对应的index。

```shell
    curl -H "Content-Type: application/x-ndjson" -XPOST "127.0.0.1:9200/shakespeare/doc/_bulk?pretty" --data-binary @D:\TestData\elasticsearch\shakespeare_6.0.json
    curl -H "Content-Type: application/x-ndjson" -XPOST "127.0.0.1:9200/bank/account/_bulk?pretty" --data-binary @D:\TestData\elasticsearch\accounts.json
	curl -H "Content-Type: application/x-ndjson" -XPOST "127.0.0.1:9200/_bulk?pretty" --data-binary @D:\TestData\elasticsearch\logs.jsonl
```



3、搜索时，若提示未启用安全功能，可以在`Elasticsearch/conf/elasticsearch.yml`文件中增加如下参数，重启ES示例即可

```
#禁用安全选项
xpack.security.enabled: false
```



## 聚合（aggregations)

### 桶（bucket)

> 桶的作用，是按照某种方式对数据进行分组，每一组数据在ES中称为一个`桶`，例如我们根据国籍对人划分，可以得到`中国桶`、`英国桶`，`日本桶`……或者我们按照年龄段对人进行划分：0$~$10,10$~$20,20$~$30,30~40等。

划分桶的方式：

- **Date Histogram Aggregation**：根据==日期阶梯==分组，例如给定阶梯为周，会自动每周分为一组
- **Histogram Aggregation**：根据==数值阶梯==分组，与日期类似
- **Terms Aggregation**：根据==词条内容==分组，词条内容完全匹配的为一组
- **Range Aggregation**：==数值==和==日期的范围==分组，指定开始和结束，然后按段分组
- ..............



### 度量（metrics)

> 分组完成以后，一般会对组中的数据进行聚合运算，例如求平均值、最大、最小、求和等，这些在ES中称为`度量`

常用的度量聚合方式

- **Avg Aggregation**：求平均值
- **Max Aggregation**：求最大值
- **Min Aggregation**：求最小值
- **Percentiles Aggregation**：求百分比
- **Stats Aggregation**：同时返回avg、max、min、sum、count等
- **Sum Aggregation**：求和
- **Top hits Aggregation**：求前几
- **Value Count Aggregation**：求总数
- ................





## 批量操作



1. bulk：ES本地支持的批量导入方式，如果是txt文本需要进行数据格式转换为json文件
2. logstash： ELK生态中的另一个产品，可以将数据文本 转换为ES的数据源
3. SpringData-ES的Java方式



### mget

这个命令的话直接上代码比较容易理解的

```http
PUT example/_mapping
{
  "properties": {
    "id": {"type": "long"},
    "name": {"type": "text"},
    "counter": {"type": "integer"},
    "tags": {"type": "text"}
  }
}

POST example/_doc/1
{
    "id": 1,
    "name": "admin",
    "counter": 1,
    "tags": ["gray"]
}
POST example/_doc/2
{
    "id": 2,
    "name": "zhangsan",
    "counter": 1,
    "tags": ["black"]
}
POST example/_doc/3
{
    "id": 3,
    "name": "lisi",
    "counter": 1,
    "tags": ["purple"]
}

PUT example_test/_mapping
{
  "properties": {
    "id": {"type": "long"},
    "name": {"type": "text"},
    "counter": {"type": "integer"},
    "tags": {"type": "text"}
  }
}

POST example_test/_doc/1
{
    "id": 1,
    "name": "test-admin",
    "counter": 1,
    "tags": ["gray"]
}
POST example_test/_doc/2
{
    "id": 2,
    "name": "test-zhangsan",
    "counter": 1,
    "tags": ["black"]
}
POST example_test/_doc/3
{
    "id": 3,
    "name": "test-lisi",
    "counter": 1,
    "tags": ["purple"]
}
GET example/_search
GET _mget
{
    "docs": [
        {
            "_index": "example",
            "_id": "1"
        },
        {
            "_index": "example",
            "_id": "2"
        },
        {
            "_index": "example_test",
            "_id": "1"
        },
        {
            "_index": "example_test",
            "_id": "2"
        }
    ]
}
GET example/_mget
{
    "docs": [
        {
            "_id": "1"
        },
        {
            "_id": "2"
        },
        {
            "_id": "3"
        }
    ]
}
```



### Bulk processtor

> BulkProcessor提供了一个简单的接口来实现批量提交请求（多种请求，如IndexRequest，DeleteRequest），且可根据请求数量、大小或固定频率进行flush提交。flush方式可选同步或异步

bulk request 会加载到内存里，如果太大的话，性能反而会下降，因此需要反复尝试一个最佳的 bulk size。一般从 1000~5000 条数据开始，尝试逐渐增加。另外，如果看大小的话，最好是在 5~15 MB 之间

#### 官方示例

```java
public BulkProcessor bulkProcessor() throws UnknownHostException {
        Settings settings = Settings.builder().put("cluster.name", "elasticsearch").build();
        //客户端连接的采用端口为9300，否则获取不到对应的节点
        Client client = new PreBuiltTransportClient(settings)
                .addTransportAddress(new TransportAddress(InetAddress.getByName("localhost"), Integer.parseInt("9300")));

        BulkProcessor bulkProcessor = BulkProcessor.builder(
                        client,//添加搜索客户端
                        new BulkProcessor.Listener() {
                            @Override
                            public void beforeBulk(long executionId,
                                                   BulkRequest request) {
                                //批量执行之前被调用

                            }

                            @Override
                            public void afterBulk(long executionId,
                                                  BulkRequest request,
                                                  BulkResponse response) {
                                //批量执行之后被调用 检查是否有一些失败的请求

                            }

                            @Override
                            public void afterBulk(long executionId,
                                                  BulkRequest request,
                                                  Throwable failure) {//
                                System.out.println("error>>> " + failure);
                            }
                        })
                .setBulkActions(10000)//10000个请求执行一次批量请求
                .setBulkSize(new ByteSizeValue(1, ByteSizeUnit.GB))//每1G冲洗一次批量
                .setFlushInterval(TimeValue.timeValueSeconds(5))//无论请求数量多少，每5秒冲洗一次批量
                .setConcurrentRequests(1)//设置并发请求的数量,值 0 表示仅允许执行单个请求,值 1 表示允许在累积新的批量请求时执行 1 个并发请求。
                .setBackoffPolicy(
                        /**
                         *
                         * 设置一个自定义的退订策略，最初将等待 100m，成倍增加，重述多达三次。
                         * 每当一个或多个批量项目请求失败时，都会尝试重试，这表明可用于处理请求的计算资源太少。
                         * 要禁用后退，通过。EsRejectedExecutionExceptionBackoffPolicy.noBackoff()
                         */
                        BackoffPolicy.exponentialBackoff(TimeValue.timeValueMillis(100), 3))
                .build();

        return bulkProcessor;
    }
```









# kibana

## 简介

Kibana 是一款开源的**数据分析**和**可视化平台**，它是 Elastic Stack 成员之一，设计用于和 Elasticsearch 协作。您可以使用 Kibana 对 Elasticsearch 索引中的数据进行搜索、查看、交互操作。您可以很方便的利用图表、表格及地图对数据进行多元化的分析和呈现。

Kibana 可以使大数据通俗易懂。它很简单，基于浏览器的界面便于您快速创建和分享动态数据仪表板来追踪 Elasticsearch 的实时数据变化。



## 安装

1. 下载[Kibana](https://www.elastic.co/cn/downloads/kibana)压缩包
2. `cd kibana-7.15.1-windows-x86_64\kibana-7.15.1-windows-x86_64\bin`执行`.\kibana`（双击kibana.bat）即可启动
3. 在kibana.yml即可修改配置属性

[kibana.yml 配置文件相关说明介绍](https://www.elastic.co/guide/cn/kibana/current/settings.html)



# Elasticsearch集群搭建

## 优点

1. **高可用性**：通过设计减少系统不能提供服务的时间。假设系统一直能够提供服务，那么该系统的可用性是 100%。如果系统在某个时刻宕掉了，比如某个网站在某个时间挂掉了，那么它临时是不可用的。所以，为了保证 Elasticsearch 的高可用性，就应该尽量减少 Elasticsearch 的不可用时间。

   - **分片（shard)**：分片是对数据切分成了多个部分。索引存储时，就是被分片存储的，默认情况下ES会把一个索引分成5个分片。分片就是数据的容器，数据保存在分片内，分片又被分配到集群内的各个节点里。当集群规模扩大或者缩小时，Elasticsearch会自动的在各个节点中迁移分片，使得数据仍然均匀分布在集群里，即一份 数据被分成了多分并保存在不同的主机上。

   > 如果一台主机挂掉了，那么这个分片里面的数据不就无法访问了？别的主机都是存储的其他的分片。其实是可以访问的，因为其他主机存储了这个分片的备份，叫做副本

   - **副本（Replica）**：就是对原分片的复制，和原分片的内容是一样的，Elasticsearch 默认会生成一份副本，所以相当于是五个原分片和五个分片副本，相当于一份数据存了两份，并分了十个分片，当然副本的数量也是可以自定义的。这时我们只需要将某个分片的副本存在另外一台主机上，这样当某台主机宕机了，我们依然还可以从另外一台主机的副本中找到对应的数据

     > 一般来说，Elasticsearch 会尽量把一个索引的不同分片存储在不同的主机上，分片的副本也尽可能存在不同的主机上，这样可以提高容错率，从而提高高可用性。

2. **健康状态**：

   - **绿色（green）**：这代表所有的主分片和副本分片都已分配。你的集群是 100% 可用的
   - **黄色（yellow**）：黄色。所有的主分片已经分片了，但至少还有一个副本是缺失的。不会有数据丢失，所以搜索结果依然是完整的。不过，高可用性在某种程度上被弱化。如果更多的分片消失，就会丢数据了。所以可把 yellow 想象成一个需要及时调查的警告
   - **红色（red）**：至少一个主分片以及它的全部副本都在缺失中。这意味着你在缺少数据：搜索只能返回部分数据，而分配到这个分片上的写入请求会返回一个异常

   > 如果你只有一台主机的话，其实索引的健康状况也是 yellow，因为一台主机，集群没有其他的主机可以防止副本，所以说，这就是一个不健康的状态，因此集群也是十分有必要

3. **存储空间**



## 集群

三节点集群：P~0~、P~1~、P~2~是主分片，R~0~、R~1~、R~2~都是副分片，具有MASTER的节点为主节点

![image-20211102101758006](http://qiliu.luxiaobai.cn/img/image-20211102101758006.png)

### 节点类型

- 主节点: 即 Master 节点。主要职责是和集群操作相关的内容，如==创建或删除索引，跟踪哪些节点是群集的一部分，并决定哪些分片分配给相关的节点==。稳定的主节点对集群的健康是非常重要的。**默认情况下任何一个集群中的节点都有可能被选为主节点**。索引数据和搜索查询等操作会占用大量的cpu，内存，io资源，为了确保一个集群的稳定，分离主节点和数据节点是一个比较好的选择。**虽然主节点也可以协调节点，路由搜索和从客户端新增数据到数据节点，但最好不要使用这些专用的主节点。一个重要的原则是，尽可能做尽量少的工作。** `node.master=true,node.data=false`
- 数据节点：即Data节点。数据节点主要是存储索引数据的节点，==主要对文档进行增删改查操作，聚合操作==等。数据节点对 CPU、内存、IO 要求较高，在优化的时候需要监控数据节点的状态，当资源不够的时候，需要在集群中添加新的节点。`node.master=false,node.data=true`
- 负载均衡节点：Client节点，也称作客户端节点。当一个节点既不配置为主节点，也不配置为数据节点时，该节点==只能处理路由请求，处理搜索，分发索引操作==等，从本质上来说该客户节点表现为**智能负载平衡器**。独立的客户端节点在一个比较大的集群中是非常有用的，他协调主节点和数据节点，客户端节点加入集群可以得到集群的状态，根据集群的状态可以直接路由请求。`node.master=false, node.data=false`
- 预处理节点：Ingest节点，在索引数据之前可以先对数据做预处理操作，所有节点其实默认都是支持 Ingest 操作的，也可以专门将某个节点配置为 Ingest 节点。

> 一个节点其实可以对应不同的类型，如一个节点可以同时成为主节点和数据节点和预处理节点，但如果一个节点既不是主节点也不是数据节点，那么它就是负载均衡节点。具体的类型可以通过具体的配置文件来设置。



> 在一个生产集群中我们可以对这些节点的职责进行划分，建议==集群中设置3台以上的节点作为master节点==，这些节点**只负责成为主节点，维护整个集群的状态**。再==根据数据量设置一批data节点==，这些节点**只负责存储数据，后期提供建立索引和查询索引的服务**，这样的话如果用户请求比较频繁，这些节点的压力也会比较大，所以在集群中建议再==设置一批client节点(node.master: false node.data: false)==，这些节点**只负责处理用户请求，实现请求转发，负载均衡等功能**



### 性能更优？

- 每个Node更少的Shard,每个Shard资源跟充沛,性能更高
- 扩容极限：6个shard（3 primary，3 replica），最多扩容到6台机器，每个shard可以占用单台服务器的所有资源，性能最好
- 超出扩容极限，动态修改replica数量，9个shard（3primary，6 replica），扩容到9台机器，比3台机器时，拥有3倍的读吞吐量
  

### 





## 搭建集群

分别进行`es7.15.1-1`、`es7.15.1-2`、`es7.15.1-3`进行集群配置。

```yaml

# ================= Elasticsearch Configuration ===================
# 配置es的集群名称, es会自动发现在同一网段下的es,如果在同一网段下有多个集群,就可以用这个属性来区分不同的集群｡
cluster.name: elasticsearch
# 节点名称
node.name: node-1
# 指定该节点是否有资格被选举成为node
node.master: true
# 指定该节点是否存储索引数据,默认为true｡
node.data: true
# 指定该节点是否为预处理节点
node.ingest: false
# 设置绑定的ip地址还有其它节点和该节点交互的ip地址,本机ip
network.host: 127.0.0.1
# 指定http端口,你使用head､kopf等相关插件使用的端口
http.port: 9200
# 设置节点间交互的tcp端口,默认是9300｡
transport.tcp.port: 9300
#用于启动当前节点时，发现其他节点的初始列表
discovery.seed_hosts: ["127.0.0.1:9300","127.0.0.1:9301", "127.0.0.1:9302"]
#设置集群中master节点的初始列表,可以通过这些节点来自动发现新加入集群的节点｡
cluster.initial_master_nodes: ["node-1"]
#因为下两台elasticsearch的port端口会设置成9301 和 9302 所以写入两台#elasticsearch地址的完整路径
#discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300","127.0.0.1:9301","127.0.0.1:9302"]


#增加新的参数，这样head插件可以访问es
http.cors.enabled: true
http.cors.allow-origin: "*"


#禁用安全选项
xpack.security.enabled: false


# ================= Elasticsearch Configuration =================== 
# 配置es的集群名称, es会自动发现在同一网段下的es,如果在同一网段下有多个集群,就可以用这个属性来区分不同的集群｡ 
cluster.name: elasticsearch 
# 节点名称 
node.name: node-2 
# 指定该节点是否有资格被选举成为node 
node.master: true 
# 指定该节点是否存储索引数据,默认为true｡ 
node.data: true
# 指定该节点是否为预处理节点
node.ingest: false
# 设置绑定的ip地址还有其它节点和该节点交互的ip地址,本机ip
network.host: 127.0.0.1
# 指定http端口,你使用head､kopf等相关插件使用的端口 
http.port: 9201 
# 设置节点间交互的tcp端口,默认是9300｡ 
transport.tcp.port: 9301 
#用于启动当前节点时，发现其他节点的初始列表
discovery.seed_hosts: ["127.0.0.1:9300","127.0.0.1:9301", "127.0.0.1:9302"]

#设置集群中master节点的初始列表,可以通过这些节点来自动发现新加入集群的节点｡
#因为下一台elasticsearch的port端口会设置成9301  所以写入两台#elasticsearch地址的完整路径 
#discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300","127.0.0.1:9301","127.0.0.1:9302"]



#增加新的参数，这样head插件可以访问es
http.cors.enabled: true
http.cors.allow-origin: "*"


#禁用安全选项
xpack.security.enabled: false

# ================= Elasticsearch Configuration ===================
# 配置es的集群名称, es会自动发现在同一网段下的es,如果在同一网段下有多个集群,就可以用这个属性来区分不同的集群｡ 
cluster.name: elasticsearch
# 节点名称 
node.name: node-3
# 指定该节点是否有资格被选举成为node 
node.master: true 
# 指定该节点是否存储索引数据,默认为true｡ 
node.data: true 
# 指定该节点是否为预处理节点
node.ingest: false
# 设置绑定的ip地址还有其它节点和该节点交互的ip地址,本机ip 
network.host: 127.0.0.1 
# 指定http端口,你使用head､kopf等相关插件使用的端口
http.port: 9202
# 设置节点间交互的tcp端口,默认是9300｡ 
transport.tcp.port: 9302
#用于启动当前节点时，发现其他节点的初始列表
discovery.seed_hosts: ["127.0.0.1:9300","127.0.0.1:9301", "127.0.0.1:9302"]


#设置集群中master节点的初始列表,可以通过这些节点来自动发现新加入集群的节点｡
#因为下一台elasticsearch的port端口会设置成9301  所以写入两台#elasticsearch地址的完整路径
#discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300","127.0.0.1:9301","127.0.0.1:9302"] 


#增加新的参数，这样head插件可以访问es
http.cors.enabled: true
http.cors.allow-origin: "*"


#禁用安全选项
xpack.security.enabled: false
```

> discovery.zen.minimum_master_nodes:  用来决定多个个符合主节点条件的节点可以形成法定数量。
>
> 在每个节点上一定要正确配置此项设置，并在集群动态扩展时也正确地更新它，这一点至关重要。系统无法检测到用户是否错误配置了此项设置，而且在实践当中，在添加或删除节点后很容易忘记调整此项设置。因此，Zen Discovery 试图通过在每次主节点选举过程中等待几秒来防止出现这种错误配置，并且对其他的超时机制通常也十分保守。这意味着，如果选举的主节点失败，在选择替代节点之前，集群至少在几秒钟内是不可用的。如果集群无法选举出一个主节点，则有时会很难了解是什么原因
>
> 
>
> [127.0.0.1:9200/_cat/nodes](http://127.0.0.1:9200/_cat/nodes) 可以查看集群是否部署完毕



## 脑裂问题

当出现网络问题，一个节点和其他节点无法连接

- Node2和Node3会重新选举Master
- Node1自己还是作为Master，组成一个集群，同时更新Cluster state
- 导致2个Master节点，维护不同的cluster state。当网络恢复时，无法选择正确恢复

### 如何避免？

1. ES7.0之前，限定一个选举条件，设置quorum(仲裁)， 只有在Master eligishble 节点数大于quorum时，才能进行选举
   - `quorum=（master节点数/2) + 1`
   - 当3个master eligible时，设置discovery.zen.minimum_master_nodes为2，既避免脑裂
2. ES7.0之后，无需此配置
   - 移除`minimum_master_nodes`参数，让Elasticsearch自己选择可以形成仲裁的节点
   - 典型的主节点选举现在只需要很短的时间就可以完成。集群的伸缩变得更安全、更容易、并且可能造成丢失数据的系统配置选项更少了
   - 节点更清楚的记录它们的状态，有助于判断为什么它们不能加入集群或为什么无法选举出主节点



## 分片数设定

- 主分片数过小：例如创建1个primary shard 的index
  - 如果该索引增长很快，集群无法通过增加节点实现对这个索引的数据扩展
- 主分片数设置过大：导致单个shard容量很小，引发一个节点上过多分片，影响性能
- 副本分片设置过多，会降低集群整体写入性能



### 文档到分片的映射算法

- 确保文档能均匀分布在所有分片上，充分利用硬件资源，避免部分机器空闲，部分机器繁忙
- 潜在算法
  - 随机/Round Robin。当查询文档1，分片数很多，需要多次查询才可能查到文档1
  - 维护文档到分片的映射关系，当文档数据量很大的时候，维护成本高
  - 实时计算，通过文档1，自动算出，需要去那个分片上获取文档



### 文档到分片的路由算法

`shard = hash(_routing)%number_of_primary_shards(主分片数量)`

- hash算法确保文档均匀分散到分片中
- 默认的`_routing`值是文档id
- 可以自行限定`_ronting`数值，例如相同国家的商品，都分配到指定的shard
- 设置Index settings 后，Primary数，不能随意修改的根本原因



## 写流程

新建、索引和删除请求都是写操作，必须在主分片上面完成之后才能被复制到相关的副本分片

![image-20211102172346323](http://qiliu.luxiaobai.cn/img/image-20211102172346323.png)

1. 客户端向Node1发送新建、索引或者删除请求
2. 节点使用文档的_id确定文档属于分片0，请求会被转发到Node3，因为分片0的主分片目前被分配在Node3上
3. Node3在主分片上面执行请求。如果成功，则将请求并行转发到Node1和Node2的副本分片上。一旦所有的副本分片都报告成功，Node3将向Node1节点（协调节点）报告成功，协调节点向客户端报告成功。



## 读流程

从主分片或者从其它任意副本分片检索文档

![image-20211102173127918](http://qiliu.luxiaobai.cn/img/image-20211102173127918.png)

从主分片或者副本分片检索文档的步骤顺序：
1. 客户端向 Node1 发送获取请求。
2. 节点使用文档的 _id 来确定文档属于分片 0 。分片 0 的副本分片存在于所有的三个节点上 在这种情况下，它将请求转发到Node2。
3. Node2 将文档返回给 Node 1 ，然后将文档返回给客户端。
在处理读取请求时，协调结点在每次请求的时候都会通过轮询所有的副本分片来达到负载均衡。在文档被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报告文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的



## 更新流程

![image-20211102173609558](http://qiliu.luxiaobai.cn/img/image-20211102173609558.png)



1. 客户端向 Node 1 发送更新请求。
2. 它将请求转发到主分片所在的 Node 3 。
3. Node 3 从主分片检索文档，修改 _source 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试步骤 3 ，超过 retry_on_conflict 次后放弃。
4. 如果 Node 3 成功地更新文档，它将新版本的文档并行转发到 Node 1 和 Node 2 上的副本分片，重新建立索引。一旦所有副本分片都返回成功， Node 3 向协调节点也返回成功，协调节点向客户端返回成功

>当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。 如果 Elasticsearch 仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档





# 参考资源

[windows环境下elasticsearch安装教程](https://www.cnblogs.com/hualess/p/11540477.html)

[ES:倒排索引、分词详解](https://blog.csdn.net/jiaojiao521765146514/article/details/83750548)

[什么是倒排索引](https://www.cnblogs.com/zlslch/p/6440114.html)

[ES快速上手](https://www.cnblogs.com/aaanthony/p/7380662.html)

**官方测试数据**：[logs.jsonl.gz](https://download.elastic.co/demos/kibana/gettingstarted/logs.jsonl.gz) [accounts.zip](https://download.elastic.co/demos/kibana/gettingstarted/accounts.zip) [shakespeare_6.0.json](https://download.elastic.co/demos/kibana/gettingstarted/shakespeare_6.0.json) 点击即可下载

[elasticsearch7.X专业术语详解](https://blog.csdn.net/qq_34168515/article/details/108315484)

[使用kibana操作ES教程](https://www.cnblogs.com/strict/p/12642146.html)

[ES操作之批量写-BulkProcessor原理浅析](https://www.jianshu.com/p/4ddf2db5c290)

[优化ElasticSearch之合理分配索引分片](https://blog.csdn.net/weixin_33973609/article/details/89064752?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link)

[手把手教你搭建一个Elasticsearch](https://www.cnblogs.com/tianyiliang/p/10291305.html)

[Elasticsearch集群协调迎来新时代](https://www.elastic.co/cn/blog/a-new-era-for-cluster-coordination-in-elasticsearch)

[ElasticSearch分片是什么?投票机制是怎样的？脑裂问题如何解决](https://blog.csdn.net/weixin_45031111/article/details/103204605)

[Elasticsearch7-分布式及分布式搜索机制](https://www.cnblogs.com/xzkzzz/p/12119387.html)

[Elasticsearch Snapshot恢复数据分片显示未分片](https://cloud.tencent.com/developer/article/1659221)

[Elasticsearch集群和索引健康状态及常见错误说明](https://www.cnblogs.com/kevingrace/p/10671063.html)

[Elasticsearch7之snapshot使用](https://rstyro.github.io/blog/2020/09/28/Elasticsearch7%E4%B9%8Bsnapshot%E4%BD%BF%E7%94%A8/)

[ElasticSearch导入txt文本或者json文本](https://www.cnblogs.com/ttzsqwq/p/11077574.html)

[将普通文本文件导入Elasticsearch](https://www.pianshen.com/article/5399591779/)

[ELK之logstash使用-----logstash将txt文本的接送数据导入到ES中](https://blog.csdn.net/weixin_44993313/article/details/106340355)

[elasticsearch使用BulkProcesssor导入txt大文件](https://qyi.io/archives/771.html)

[Elasticsearch Bulk Processor](https://www.elastic.co/guide/en/elasticsearch/client/java-api/2.3/java-docs-bulk-processor.html)

[Elasticsearch-BulkProcessor浅析](https://blog.csdn.net/baichoufei90/article/details/97117025)

