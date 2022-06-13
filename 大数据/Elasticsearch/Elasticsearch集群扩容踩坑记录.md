---
ES集群扩容构建踩坑总结
---

[toc]

----



# 需求

> 在原有单一节点（node-1）中进行构建双节点集群





## 配置

node-1的`elasticsearch.yml`的配置信息如下

```shell
cluster.name: elasticsearch
node.name: node-1
node.master: true
node.data: true
network.host: 192.168.3.147
http.port: 9200
transport.tcp.port: 9300   ###在构建集群通信时必须有这个配置，否则无法建立通信
discovery.seed_hosts: ["192.168.3.147:9300","192.168.3.211:9300"]
cluster.initial_master_nodes: ["node-1"]
http.cors.enabled: true
http.cors.allow-origin: "*"
indices.breaker.fielddata.limit: 60%
indices.fielddata.cache.size: 40%
path.repo: ["/usr/local/elasticsearch/backups/es_backup"]
#indices.requests.cache.size: 5%
#indices.memory.index_buffer_size: 70%
#index.refresh_interval: 30s   ###此项设置，建议使用API动态设置
xpack.security.enabled: false
search.max_open_scroll_context: 1000000
cluster.routing.allocation.disk.threshold_enabled: false
#cluster.routing.allocation.disk.watermark.low: 90%
#cluster.routing.allocation.disk.watermark.high: 95%
#cluster.routing.allocation.disk.watermark.flood_stage: 98%
```

### 参数说明

- `cluster.name`: 集群名称
- `node.master`: 主节点 （默认true) 主要用来创建或者删除索引，以及决定哪些分片分配给相关的节点.
- `node.data`:  数据节点，主要用来存储索引数据的节点，即对文档增删改查、聚合等操作。
- `node.ingest`: 预处理节点（一般情况是不会去配置这个节点的）在索引数据之前可以先对数据做预处理操作，所有节点其实默认都是支持 Ingest 操作的，也可以专门将某个节点配置为 Ingest 节点。
- `http.port`: http请求端口
- `transport.tcp.port`:集群中节点通信的端口
- `discovery.seed_hosts`: 提供集群中其他的节点列表进行发现
- `cluster.initial_master_nodes`: 第一次启动elasticsearch集群时，集群引导步骤会确定符合主节点资格的节点集。
- `indices.breaker.fielddata.limit`: 断路器配置，用来防止操作时造成`OutOfMemoryError`，指定可以使用多少内存限制。[详见此](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/circuit-breaker.html)
- `path.repo`: 快照存放目录，还需要用API进行创建仓库`responsitry`,具体可见另一篇文档，有对快照备份的详细操作，==个人亲测有效==
- `cluster.routing.allocation.disk.threshold_enabled`: 磁盘分配设置，用来修改磁盘使用率。



增加节点node-2,配置如下

```
cluster.name: elasticsearch
node.name: node-2
node.master: false
node.data: true
network.host: 192.168.3.211
http.port: 9200
transport.tcp.port: 9300
discovery.seed_hosts: ["192.168.3.147:9300","192.168.3.211:9300"]
cluster.initial_master_nodes: ["node-1"]
http.cors.enabled: true
http.cors.allow-origin: "*"
xpack.security.enabled: false
search.max_open_scroll_context: 1000000
cluster.routing.allocation.disk.threshold_enabled: false
```

本机测试及其他服务器上测试中如上配置是可以稳定的进行节点扩容。



## Data node’s cluster uuid diffrent from master node’s cluster uuid

出现这个问题的原因主要是： `cluster.name`不同导致的`cluster.uuid`不同。产生的原因是`node-1`有与其他节点构成一个集群的部署，`cluster.name： elasticsearch`. 之后`node-1`又与其他节点构成一个集群，`cluster.name: elasticsearch_prod`. `node-1`因为前后`cluster.name`不一致造成的。还可能产生的原因是`cluster.name`后面有空格.

解决该问题的最好的方式就是：采用其默认的`cluster.name: elasticsearch`



## Elasticsearch: adding a second node to the cluster - [node-1] master not discovered yet: have discovered [{node-1}

出现上述问题可参考如下方式去排查

- 防火墙是否禁用
- `transport.tcp.port: 9300`这个配置是否有
- `network.host: 127.0.0.1` 改为 `192.168.3.211`
- `discovery.seed_hosts` 集群节点列表是否配置





### Elasticsearch: Max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

该问题属于常见问题，elastsearch允许的虚拟内存不足

```shell
sudo vim /etc/sysctl.conf
vm.max_map_count = 262144
sudo sysctl -w vm.max_map_count=262144
```



## 集群搭建完成，分片未分配



### 问题1

> 集群通信建立，但是分片未进行分配

![](http://qiliu.luxiaobai.cn/img/image-20220111094238373.png)



> 双节点集群搭建完成，由于之前的分片未恢复而导致分片无法进行再平衡设置时，可以参考官网的集群级分片和路由设置

**`cluster.routing.allocation.enable`**

（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/8.0/settings.html#dynamic-cluster-setting)）为特定类型的分片启用或禁用分配：

- `all` - （默认）允许为所有类型的分片分配分片。
- `primaries` - 只允许对主分片进行分片分配。
- `new_primaries` - 仅允许为新索引的主分片分配分片。
- `none` - 不允许对任何索引进行任何类型的分片分配。

**`cluster.routing.allocation.allow_rebalance`**

（[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）指定何时允许分片重新平衡：

- `always` - 始终允许重新平衡。
- `indices_primaries_active` - 仅当分配了集群中的所有主节点时。
- `indices_all_active` - （默认）仅当集群中的所有分片（主分片和副本）都已分配时。****

该设置不影响重启节点时本地主分片的恢复。具有未分配主分片副本的重新启动节点将立即恢复该主分片，假设其分配 id 与集群状态中的活动分配 id 之一匹配。

```
put /_cluster/settings
{
    "transient": {
        "cluster.routing.allocation.enable": "all",  //为所有类型的分片分配分配
        "cluster.routing.allocation.allow_rebalance": "always", //始终允许重新平衡
        // "cluster.routing.rebalance.enable": "all"  //为所有类型的分配进行分片进行再平衡设置
    }
}
```

`transient`: 暂时性的配置，ES服务重启后就失效了

`persistent`: 永久性的。

通过如上API设置，可以实现分片的再平衡设置。





### 问题2

> 分片未恢复
>
> CLUSTER_RECOVERED: 完全集群导致分片未分配

![image-20220112090600489](http://qiliu.luxiaobai.cn/img/image-20220112090600489.png)



通过**`GET _cluster/allocation/explain`** 对分片未分配原因进行分析

![image-20220112090645951](http://qiliu.luxiaobai.cn/img/image-20220112090645951.png)

报错原因是`checksum failed (hardware problem?) : expected=cdd45b03 actual=4ff129f6 (resource=BufferedChecksumIndexInput(MMapIndexInput(path=\"/usr/local/elasticsearch/data/nodes/0/indices/yKvhEnU9StmlPSRTpfWd1Q/46/index/_3g_Lucene50_0.tip\")))"`硬件问题 磁盘损坏，文档丢失，磁盘空间不足等等情况都可以能导致该错误的发生。

可以通过`Luence-core-8.20.1.jar`包对分片的segements运行`CheckIndex`程序。如下命令会**对分段进行验证校验和，还会确保索引实际上可以被读取并且使其数据结构彼此一致（对于这个使其数据结构彼此一致的说法个人是保持怀疑态度的，因为我进行如下命令也没发现有进行修复）**。这个程序运行时间可能会有点久，这个主要根据分片大小决定。31G分片执行校验在*400~500s*这个范围。

`java -cp /usr/share/elasticsearch/lib/lucene-core-4.10.4.jar -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex $datadir/nodes/0/indices/$index/$shard/index/`

这程序执行完之后。会显示该分片是否损坏。结果如下。**虽然提示的结果显示segments文档损坏，通过附加`-exorcise`参数可以对segemetns进行文档的修改，但是会丢失对应的文档。个人建议谨慎操作。再进行该项操作时可以看该目录下是否有`corrupted_*`这个文件，这个文件是对该分片的标识文件，可以将该文件进行移除，然后重启ES服务**。==个人亲测有效==。

如果要进行`-exoricise`参数对分段进行文档修复（这是在允许文档数据丢失情况下）之前最好做好备份。

![image-20220112153607533](http://qiliu.luxiaobai.cn/img/image-20220112153607533.png)

在对`corrupted_*`损坏文件进行移除后，集群恢复如下

![image-20220113092607521](http://qiliu.luxiaobai.cn/img/image-20220113092607521.png)

发现还有2个节点未恢复，根据**`GET _cluster/allocation/explain`**在进行分片未分配原因分析

<img src="http://qiliu.luxiaobai.cn/img/image-20220113092850419.png" alt="image-20220113092850419" style="zoom: 67%;" />

通过上述分析，没有报错原因。则可以尝试进行分片的重新路由

`cannot allocate because all found copies of the shard are either stale or corrupt`这个问题是毫无参考价值的，基本上分片只要未分配都是给你这个提示。



## 重新路由

重新路由命令可以手动的将未分配的分片进行分配到节点上。主要有3个参数

- **move**: 将已启动的分片从一个节点移动到另一个节点 
- **cancel**: 取消分片（或恢复）的分配 它还接受 allow_primary 标志以明确指定允许取消对主分片的分配。这可用于强制从主分片重新同步现有副本，方法是取消它们并允许它们通过标准重新分配过程重新初始化
- **allocate**: 将未分配的分片分配给节点。接受索引名称和分片编号的索引和分片，以及将分片分配到的节点。它还接受 allow_primary 标志来明确指定允许明确分配主分片（可能导致数据丢失）`allocate_stale_primary`对主分片进行操作。`accept_data_loss`：允许数据丢失，所以谨慎操作，最好能对索引进行备份。**Elasticserch7.0以后官方都推荐使用快照的形式对索引进行数据备份操作。**

```http
POST /_cluster/reroute
{
    "commands": [
        {
            "allocate_stale_primary": {
                "index": "sedb",
                "shard": 0,
                "node": "node-1",
                "accept_data_loss": true
            }
        }
    ]
}

```

==在进行分片重新路由时，必须确保分片目录下没有corrupted_*这个损坏文件的标识，否则是不会进行分片强制分配的。==



>个人认为：对于分片数据恢复或者说未分配的解决思路是：可以先对分片未分配原因进行分析，ES的这个API GET _cluster/allocation/explain 是非常不错的工具。步骤如下：
>
>1、检查分片文件的损坏文档情况 采用Lucence 库 Luence-core-8.20.1.jar 对segments进行校验。
>
>2、分片中的存在的损坏文件进行移除，重启ES服务
>
>3、对未分配的分片进行重新路由的方式，手动分配。
>
>4、重新索引（这种方式没有测试过对未分配的分片是否有效）





## 附

### Lucene索引文件介绍

`cd elasticsearch/data/node/0/indices/xxxx/$shard/index`某个目录下，存放分片的多个segments。如下所示

![image-20220112145358230](http://qiliu.luxiaobai.cn/img/image-20220112145358230.png)

|        Name         | Extension |                         Description                          |
| :-----------------: | :-------: | :----------------------------------------------------------: |
|     Term Index      |   .tip    |                   词典索引（需加载进内存）                   |
|   Term Dictionary   |   .tim    |                          倒排表指针                          |
|     Frequencies     |   .doc    |              包含Term和频率的文档列表（倒排表）              |
|       Fields        |   .fnm    |                       Field数据元信息                        |
|     Field Index     |   .fdx    |                 文档位置索引（需加载进内存）                 |
|     Field Data      |   .fdt    |                            文档值                            |
| Per-Document Values | .dvd .dvm | .dvm为DocValues元信息<br />.dvd为DocValue值<br />（默认情况下elasticsearch开启该功能用于快速排序、聚合等操作） |



### 多节点集群搭建的elasticsearch.yml配置

```shell
cluster.name: elasticsearch
node.name: node-3   ###依次修改
node.master: true   ###当节点数>=3时 改为true 实现高可用。
node.data: true
network.host: 192.168.3.xxx
bootstrap.memory_lock: true   ###禁用SWAP，防止内存与磁盘进行交互，造成ES性能问题
http.port: 9200
transport.tcp.port: 9300
discovery.seed_hosts: ["192.168.3.147:9300","192.168.3.211:9300","192.168.3.xxx"]  ###所有节点加入该列表
cluster.initial_master_nodes: ["node-1","node-2","node-3"]   ##node.master:true 的节点都加入
discovery.zen.minimum_master_nodes：2    ###节点数>=3时，增加该配置防止脑裂问题
http.cors.enabled: true
http.cors.allow-origin: "*"
xpack.security.enabled: false
search.max_open_scroll_context: 1000000
```





[1]: https://www.elastic.co/guide/en/elasticsearch/reference/8.0/modules-cluster.html#cluster-shard-allocation-settings	"集群级分片分配和路由设置"
[2]: https://cloud.tencent.com/developer/news/362991	"Elasticsearch搜索引擎性能调优"
[3]: https://www.cnblogs.com/technologykai/articles/11940806.html?ivk_sa=1024320u	"Elasticsearch亿级数据检索案例与原理"
[4]: https://mincong.io/cn/elasticsearch-corrupted-index/	"修复elasticsearch中损坏的索引"
[5]: https://segmentfault.com/a/1190000004504225	"如何防止脑裂问题"

