---
ES扩展学习
---

[toc]



# 基本概念

## 分片

将一份数据划分为多小份的能力，允许水平分割和扩展容量。多个分片可以响应请求，提高性能和吞吐量。

由于ES索引库的分片数量**在索引库创建时就已经确定，后续无法进行分片数量得修改**



> 至于为什么分片的数量在创建后就不能修改，主要原因如下

当索引一个文档的时候，文档会被存储到一个主分片中,而ES如何知道一个文档存放在那个分片中，主要是由这个公式决定的：**`shard = hash(routing) % number_of_primary_shards`**

- **routing**：是一个可变值，默认为文档的`_id`,也可以设置为一个自定义的值， routing 通过 hash 函数生成一个数字，然后这个数字再除以 `number_of_primary_shards` （主分片的数量）后得到 余数 。这个分布在 0 到 `number_of_primary_shards-1` 之间的余数，就是所寻求的文档所在分片的位置。

所以综上所述：主分片数量在确定好后，就永远不会改变了。否则数量变化，则之前路由的值都会无效，文档也在也找不到。



### 重新索引实现分片数量的修改

这种方式的本质就是重新创建新的索引，然后进行数据的迁移。示例操作如下：

```kibana
###创建索引studens1
PUT students1
{
"settings":{
"number_of_shards":3,
"number_of_replicas":1
},
"mappings":{
  "student":{
    "dynamic":"strict",
    "properties":{
    "id":{"type": "text", "store": true},
    "name":{"type": "text","store": true},
    "age":{"type": "integer","store": true},
    "times": {"type": "text", "index": false}
  }
  }
}
}


##增加数据
PUT students1/student/1
{ "id" : "1", "name" : "张三", "age" : 20 , "times" : "2020-02-01" }


GET students1/_search


##创建索引2进行重新索引
PUT students2 
{ 
  "settings":{ 
    "number_of_shards":4, 
    "number_of_replicas":1 
    
  },
  "mappings":{ 
      "properties":{ 
        "id":{
          "type": "text", 
          "store": true
        }, 
        "name":{
          "type": "text",
          "store": true
        },
        "age":{
          "type": "integer",
          "store": true
          
        }, 
        "times": {
          "type": "text", 
          "index": false
        } 
      } 
  } 
}


###将student1 的数据拷贝到student2
POST _reindex 
{ 
  "source": {
    "index": "students1"
    },
    "dest": { 
      "index": "students2" 
    } 
}


GET /students2/_settings
GET /students2/_search
```





## 副本

复制数据，一个节点出问题时，其余节点可以顶上

副本数量在索引库创建之后是可以进行动态修改的

```http
###修改副本数量
PUT students1/_settings
{
  "settings":{
    "number_of_replicas":3
  }
}
```



## 两者的作用

Elasticsearch 是利用分片将数据分发到集群内各节点的。**分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。**
当集群规模扩大或者缩小时， ES会自动的在各节点中迁移分片，使得数据仍然均匀分布在集群里。

一个分片可以是主分片或者副本分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量， 同时对文档的新建、索引和删除请求，必须在主分片上面完成之后才能复制到相关的副本分片上。

一个副本分片只是一个主分片的拷贝。 副本分片作为硬件故障时**保护数据不丢失的冗余备份，并为搜索和返回文档等读操作**提供服务。



# ES扩容分片及磁盘基本情况

ES集群扩容时， 会将之前的节点数据都备份到新节点中，自动实现数据的均衡分配



## 分片情况

以`txt_number2`索引为例,分片为3，副本为2.

```http
PUT txt_number2
{
    "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      "txt_content": {
        "type": "text"
      }
    }
  }
}
```

当前节点有2个`node-1`,`node-2` 节点情况如下

![image-20211229174009900](http://qiliu.luxiaobai.cn/img/image-20211229174009900.png)

![image-20211229173958359](http://qiliu.luxiaobai.cn/img/image-20211229173958359.png)

当前`txt_number2`的各shard的详细情况如下所示，`node-1`和`node-2`数据大小基本一致，同时可以看出==各节点的分片和副本都分布在不同节点，数据大小接近相等， 相同分片的副本不会放在同一个节点上==

![image-20211229174258208](http://qiliu.luxiaobai.cn/img/image-20211229174258208.png)



动态增加`node-3`节点后，各shard的详细情况如下所示。数据重新复制一份。

![image-20211229181007246](http://qiliu.luxiaobai.cn/img/image-20211229181007246.png)



### 集群分片分配策略、

某个shard分配在哪个节点上，是由Elasticsearch自动决定的。以下几种情况会触发分配动作

- 新索引生成
- 索引的删除
- 新增副本分片
- 节点增减引发的数据均衡





### 分片数量和数据的关系

分片的实质是一个Lucene的一个实例，单个Lucene实例最多包含2147483519（=Integer.MAX_VALUE-128）个Document,即分片存储的文档数多为20亿条，同时分片的最大容量限制为30G。

目录结构：

- 一个ES的`Index`由多个ES的`shard`构成
- 一个ES的`shard`底层对应一个`Lucene Index`
- 一个`Lucene Index` 底层是对应`Lucene segement`构成





## 磁盘情况

### 磁盘使用率

集群扩容后，节点的容量只受所在磁盘的容量限制。磁盘使用率也影响这分片的分布。如下三种情况：

- **磁盘使用率超过85%**： 会导致新的分片无法分配（**已测试**）
- **磁盘使用率超过90%** ： ES会尝试将对应节点中的分片迁移到其他磁盘使用率比较低的数据节点中（**已测试**）
- **磁盘使用率超过95%**： 系统会对ES集群中的每个索引强制设置`read_only_allow_delete`属性，此时索引将无法写入数据，只能读取和删除对应索引 （**已测试**）



> 本机测试：
>
> ​    磁盘D：容量 264G
>
> ​	磁盘C： 容量200G
>
> 85%：264*85%=224.4
>
> 90%： 264*90%=247.6
>
> 95%:	264*95%=250.8

​	 

#### 磁盘使用率超过85%

将磁盘D的磁盘容量使用率超过85%，同时在增加`node-5`节点。可以发现`node-5`中没有分片进行分配

![磁盘D](http://qiliu.luxiaobai.cn/img/image-20211230153039969.png)

![rong](http://qiliu.luxiaobai.cn/img/image-20211230105259176.png)





#### 磁盘使用率超过90%

将`磁盘D`使用率超过90%，如下所示

![image-20211230151757114](http://qiliu.luxiaobai.cn/img/image-20211230151757114.png)



通过增加节点`node-6`,部署到**C磁盘**中，复现了磁盘使用率超过90%的场景，实际的显示是将索引`txt_number`中`node-5`的`shards:2`转移到了新节点`node-6`中.网页的显示是，分片会进行重新分配，状态颜色为紫色。进行重新分配的磁盘容量如下所示：

![image-20211230153039969](http://qiliu.luxiaobai.cn/img/image-20211230153039969.png)



#### 磁盘使用率超过95%

将`磁盘D`使用率降低到95%以上,新增数据失败.索引的属性状态变为：`read-only-allow-delete`

![image-20211230150421073](http://qiliu.luxiaobai.cn/img/image-20211230150421073.png)

![image-20211230155936589](http://qiliu.luxiaobai.cn/img/image-20211230155936589.png)



### 更改磁盘使用率配置

#### 更改配置文件

配置文件`elasticsearch.yml`

```http
cluster.routing.allocation.disk.threshold_enabled: true
cluster.routing.allocation.disk.watermark.low: 90%
cluster.routing.allocation.disk.watermark.high: 95%
cluster.routing.allocation.disk.watermark.flood_stage: 98%
```



#### API动态更改

`transient`临时更改， `persistent`永久更改

```http
PUT /_cluster/settings
{
	"persistent":{
		"cluster.routing.allocation.disk.watermark.low": "90%",
    	"cluster.routing.allocation.disk.watermark.high": "95%",
    	"cluster.info.update.interval": "1m"
	}
}
```







# 近实时搜索

ES底层采用lucene这个库实现倒排索引功能。lucene使用segment(分段)来存储数据，用commit point 来记录所有segment的元数据。

> 一条记录要被搜索到，必须写入到segment中

ES使用translog来记录所有的操作==>`write-ahead-log`, 新增一条记录时，ES会把数据写到`translog`和`in-memory buffer(内存缓存区)`。

> **translog**是实时`fsync`的，也既写入es的数据，其对应的translog内容是实时写入磁盘，并且是以顺序append文件的方式，写磁盘性能很高。

新增的数据必须写入到segment后才能被搜索到，因此把数据写入到内存缓冲区之后并不能搜索到，如果希望立刻被搜索，需要手动调用**refresh**操作



- **refresh**：默认情况ES每隔一秒执行一次`refresh`，可以通过参数`index.refresh_interval`来修改刷新间隔



### 原因

因为内存缓存区的数据，被写入segment文件，且segment文件被写入虚拟文件系统后，才能打开这个新segment文件，以被检索到，因此在新segment文件形成之前，内存缓冲区里的数据是无法被检索到的。而refresh操作，默认1s执行一次，即 `insert`的doc，默认在1s后才能被检索到。



## Segment Elasticsearch原理

一个 Lucene Index 由许多独立的 segments 组成，而 segments 包含了文档中的词汇字典、词汇字典的倒排索引以及 Document 的字段数据（设置为Stored.YES的字段），所有的 segments 数据存储于 _.cfs的文件中

**ES 的一个 Shard （Lucene Index）中是由大量的 Segment 文件组成的，且每一次 fresh 都会产生一个新的 Segment 文件，这样一来 Segment 文件有大有小，相当碎片化。ES 内部则会开启一个线程将小的 Segment 合并（Merge）成大的 Segment，减少碎片化，降低文件打开数，提升 I/O 性能。**

**Segment 文件是不可变更的。当一个 Document 更新的时候，实际上是将旧的文档标记为删除，然后索引一个新的文档。在 Merge 的过程中会将旧的 Document 删除掉。具体到文件系统来说，文档 A 是写入到 .cfs 文件里的，删除文档 A 实际上是在.del文件里标记某个 document 已被删除，那么下次查询的时候则会跳过这个文档，是为逻辑删除。当归并（Merge）的时候，老的 segment 文件将会被删除，合并成新的 segment 文件，这个时候也就是物理删除了**







# ES性能优化

## 写优化

> 应用场景：每秒 300 万的写入速度，每条 500 字节左右。

针对这种对于搜索性能要求不高，但是对写入要求较高的场景，需要尽可能的选择恰当写优化策略。

可以考虑以下几个方面来提升写索引的性能：

- 加大 Translog Flush ，目的是降低 Iops、Writeblock。
- 增加 Index Refresh 间隔，目的是减少 Segment Merge 的次数。
- 调整 Bulk 线程池和队列。
- 优化节点间的任务分布。
- 优化 Lucene 层的索引建立，目的是降低 CPU 及 IO。



### 批量提交

ES 提供了 Bulk API 支持批量操作，当有大量的写任务时，可以使用 Bulk 来进行批量写入。

每次提交的数据量为多少时，能达到最优的性能，主要受到**文件大小、网络情况、数据类型、集群状态**等因素影响。 

**通用的策略如下：**Bulk 默认设置**批量提交的数据量不能超过 100M**。数据条数一般是根据文档的大小和服务器性能而定的，但是单次批处理的数据大小应从 5MB～15MB 逐渐增加，当性能没有提升时，把这个数据量作为最大值。





### **优化存储设备**

ES 是一种密集使用磁盘的应用，在段合并的时候会频繁操作磁盘，所以对磁盘要求较高，当磁盘速度提升之后，集群的整体性能会大幅度提高。

磁盘的选择，提供以下几点建议：

- **使用固态硬盘（Solid State Disk）替代机械硬盘。**SSD 与机械磁盘相比，具有高效的读写速度和稳定性。

- **使用 RAID 0。**RAID 0 条带化存储，可以提升磁盘读写效率。

- 在 ES 的服务器上挂载多块硬盘。使用多块硬盘同时进行读写操作提升效率，在配置文件 ES 中设置多个存储路径

  ```
  path.data:/path/to/data1,/path/to/data2
  ```

  *避免使用 NFS（Network File System）等远程存储设备，网络的延迟对性能的影响是很大的。*



### **合理使用合并**

Lucene 以段的形式存储数据。当有新的数据写入索引时，Lucene 就会自动创建一个新的段。

随着数据量的变化，段的数量会越来越多，消耗的多文件句柄数及 CPU 就越多，查询效率就会下降。

由于 Lucene 段合并的计算量庞大，会消耗大量的 I/O，所以 ES 默认采用较保守的策略，让后台定期进行段合并，如下所述：

- **索引写入效率下降：**当段合并的速度落后于索引写入的速度时，ES 会把索引的线程数量减少到 1。 这样可以避免出现堆积的段数量爆发，同时在日志中打印出“now throttling indexing”INFO 级别的“警告”信息。
- **提升段合并速度：**ES 默认对段合并的速度是 20m/s，如果使用了 SSD，可以通过以下的命令将这个合并的速度增加到 100m/s。

```http
PUT /_cluster/settings
{
	"persistent" : {
		"indices.store.throttle.max_bytes_per_sec" : "100mb"
	}
}
```



### 减少Refresh的次数,增加segments的刷新时间

Lucene 在新增数据时，采用了**延迟写入**的策略，默认情况下**索引的 refresh_interval 为 1 秒。**

Lucene 将待写入的数据先写到内存中，超过 1 秒（默认）时就会触发一次 Refresh，然后 Refresh 会把内存中的的数据刷新到操作系统的文件缓存系统中。

如果我们对搜索的实效性要求不高，可以将 Refresh 周期延长，例如 30 秒。

这样还可以有效地减少段刷新次数，但这同时意味着需要消耗更多的Heap内存

```
index.refresh_interval:30s
```



### **加大 Flush 设置**

Flush 的主要目的是把文件缓存系统中的段持久化到硬盘，当 Translog 的数据量达到 512MB 或者 30 分钟时，会触发一次 Flush。

`index.translog.flush_threshold_size` 参数的默认值是 **512MB**，

增加参数值意味着文件缓存系统中可能需要存储更多的数据，所以需要为操作系统的文件缓存系统留下足够的空间。



### **减少副本的数量**

ES 为了保证集群的可用性，提供了 Replicas（副本）支持，然而每个副本也会执行分析、索引及可能的合并过程，所以 Replicas 的数量会严重影响写索引的效率。

当写索引时，需要把写入的数据都同步到副本节点，副本节点越多，写索引的效率就越慢。

如果需要大批量进行写入操作，可以先禁止 Replica 复制，设置 index.number_of_replicas: 0 关闭副本。在写入完成后，Replica 修改回正常的状态



## 优化检索性能



- 关闭不需要字段的`doc values`
- 尽量使用keyword替代一些long或者int之类，term查询比range查询好
- 关闭不需要查询字段的_source功能
- 评分消耗资源，如果不需要可使用`filter`过滤来达到关闭评分功能，score则为0
- 分页 官方的建议是采用==时间点(Point in time) > scroller_after > scroller > from + size==
- 多使用写入前预处理操作
- 尽量规避使用脚本
- 对历史索引数据使用段合并
- 启用 eager global ordinals 提升高基数聚合性能
- 设置合理的分片数和副本数
  - 在许多情况下，拥有更多副本**有助于提高搜索性能**。但是**不代表副本越多越好**。增加副本的前提是考虑：**磁盘存储空间的容量上限和磁盘警戒水位线。本质还是以空间换时间。一般非高可用场景，基本一个副本足够**。













## 附集群状态接口命令

```shell
/_cat/allocation      #查看单节点的shard分配整体情况
/_cat/shards          #查看各shard的详细情况
/_cat/shards/{index}  #查看指定分片的详细情况
/_cat/master          #查看master节点信息
/_cat/nodes           #查看所有节点信息
/_cat/indices         #查看集群中所有index的详细信息
/_cat/indices/{index} #查看集群中指定index的详细信息
/_cat/segments        #查看各index的segment详细信息,包括segment名, 所属shard, 内存(磁盘)占用大小, 是否刷盘
/_cat/segments/{index}#查看指定index的segment详细信息
/_cat/count           #查看当前集群的doc数量
/_cat/count/{index}   #查看指定索引的doc数量
/_cat/recovery        #查看集群内每个shard的recovery过程.调整replica。
/_cat/recovery/{index}#查看指定索引shard的recovery过程
/_cat/health          #查看集群当前状态：红、黄、绿
/_cat/pending_tasks   #查看当前集群的pending task
/_cat/aliases         #查看集群中所有alias信息,路由配置等
/_cat/aliases/{alias} #查看指定索引的alias信息
/_cat/thread_pool     #查看集群各节点内部不同类型的threadpool的统计信息,
/_cat/plugins         #查看集群各个节点上的plugin信息
/_cat/fielddata       #查看当前集群各个节点的fielddata内存使用情况
/_cat/fielddata/{fields}     #查看指定field的内存使用情况,里面传field属性对应的值
/_cat/nodeattrs              #查看单节点的自定义属性
/_cat/repositories           #输出集群中注册快照存储库
/_cat/templates              #输出当前正在存在的模板信息
```





## 参考

[1]: https://its201.com/article/zwgdft/83478241	"谈Elasticsearch下分布式存储的数据分布"
[2]: https://cloud.tencent.com/developer/article/1361467	"Elasticsearch集群管理"
[3]: https://cloud.tencent.com/developer/article/1066351	"Elasticsearch究竟要设置多少分片数"
[4]: https://cloud.tencent.com/document/product/845/43615	"集群变配建议和原理介绍"
[5]: https://cloud.tencent.com/developer/article/1691544	"解决Elasticsearch分片未分配的问题"
[6]: https://cloud.tencent.com/developer/article/1122703	"为什么说Elasticsearch搜索时近实时的"
[7]: https://tech.ebayinc.com/engineering/elasticsearch-performance-tuning-practice-at-ebay/	"eBay 上的 Elasticsearch 性能调优实践"
[8]: https://cloud.tencent.com/developer/article/1436787	"超详细的Elasticsearch高性能优化实践"
[9]: https://cloud.tencent.com/developer/article/1696747	"腾讯云Elasticsearch集群规划及性能优化实践"

[10]: https://www.cnblogs.com/technologykai/articles/11940806.html?ivk_sa=1024320u	"Elasticsearch亿级数据检索案例与原理"
[11]: https://blog.csdn.net/qq_36254699/article/details/112549418	"亿级数据分页查询优化过程"

[12]: https://jishuin.proginn.com/p/763bfbd6aa17	"Elasticsearch 检索性能优化实战指南"

