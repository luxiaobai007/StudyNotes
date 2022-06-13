---
Elasticsearch索引优化技术预言
---

> ES集群性能优化是多方面的。总的来说有以下三个方面:
>
> - 代码层面，对检索、写入文档操作的优化，API、逻辑
> - ES配置，集群管理的重要配置。
> - 硬件层面，节点所在服务器的性能，以及如果条件允许情况下的多节点部署





## 代码设计

### 多索引

> 搜索 1 个有着 50 个分片的索引与搜索 50 个每个都有 1 个分片的索引完全等价：搜索请求均命中 50 个分片

[官网详细介绍](https://www.elastic.co/guide/cn/elasticsearch/guide/current/multiple-indices.html)

此方法可以在不停服务的情况下，增加容量。主要解决将数据迁移（reindex）到更大的索引中

- 创建一个新的索引来存储新的数据。
- 同时搜索两个索引来获取新数据和旧数据

使用这种方式，可以解决**原索引容量太大，分片数的容量达到30G以上，而又不好执行重新索引的方式重新设计索引库问题**，可以通过==索引别名进行多索引查询，将查询方式改为多索引查询，将新增文档加入到新索引中，实现容量的新扩容==。

个人亲测有效

```http
PUT http://localhost:9200/tweets_1/_alias/tweets_search
PUT http://127.0.0.1:9200/tweets_1/_alias/tweets_index
PUT http://127.0.0.1:9200/tweets_2
POST http://127.0.0.1:9200/_aliases
{
    "actions": [
        {
            "add": {
                "index": "tweets_2",
                "alias": "tweets_search"
            }
        },
        {
            "remove": {
                "index": "tweets_1",
                "alias": "tweets_index"
            }
        },
        {
            "add": {
                "index": "tweets_2",
                "alias": "tweets_index"
            }
        }
    ]
}
```

## ES配置

### 内存分配

设置Java堆大小 `-Xmx`和`-Xms`。官方建议是不要超过32G。[详细信息](https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html) 建议如果机器性能为64G，则可以配置`-Xmx31G -Xms31G`

当机器内存小于 64G 时，遵循通用的原则，50% 给 ES，50% 留给 lucene。

当机器内存大于 64G 时，遵循以下原则：

- 如果主要的使用场景是全文检索，那么建议给 ES Heap 分配 4~32G 的内存即可；其它内存留给操作系统，供 lucene 使用（segments cache），以提供更快的查询性能。
- 如果主要的使用场景是聚合或排序，并且大多数是 numerics，dates，geo_points 以及 not_analyzed 的字符类型，建议分配给 ES Heap 分配 4~32G 的内存即可，其它内存留给操作系统，供 lucene 使用，提供快速的基于文档的聚类、排序性能。
- 如果使用场景是聚合或排序，并且都是基于 analyzed 字符数据，这时需要更多的 heap size，建议机器上运行多 ES 实例，每个实例保持不超过 50% 的 ES heap 设置（但不超过 32 G，堆内存设置 32 G 以下时，JVM 使用对象指标压缩技巧节省空间），50% 以上留给 lucene。

一般而言配置为31G即可



### 禁用SWAP

禁止 swap，一旦允许内存与磁盘的交换，会引起致命的性能问题。可以通过在 elasticsearch.yml 中 `bootstrap.memory_lock: true`，以保持 JVM 锁定内存，保证 ES 的性能



### 路由优化

计算某个文档存放到那个分片中`shard = hash(routing) % number_of_primary_shards`

routing默认值是文档的ID，也可以采用自定义值。

##### 不带routing查询

查询的时候因为不知道要查询的数据具体在哪个分片上，所以整个过程分为2个步骤：

- 分发：请求到达协调节点后，协调节点将查询请求分发到每个分片上。
- 聚合：协调节点搜集到每个分片上查询结果，再将查询的结果进行排序，之后给用户返回结果。



##### 带routing查询

查询的时候，可以直接根据 `routing `信息定位到某个分配查询，不需要查询所有的分配，经过协调节点排序。

向上面自定义的用户查询，如果 `routing `设置为 `userid `的话，就可以直接查询出数据来，效率提升很多。



### Filter VS Query

尽可能使用过滤器上下文（Filter）替代查询上下文（Query）

- **Query**：此文档与此查询子句的匹配程度如何？
- **Filter**：此文档和查询子句匹配吗？

Elasticsearch 针对 Filter 查询只需要回答「是」或者「否」，不需要像 Query 查询一样计算相关性分数，同时Filter结果可以缓存。



### Cache的设置及使用

- `QueryCache`: ES查询的时候，使用filter查询会使用query cache, 如果业务场景中的过滤查询比较多，建议将querycache设置大一些，以提高查询速度。

`indices.queries.cache.size： 10%`（默认），可设置成百分比，也可设置成具体值，如256mb。

当然也可以禁用查询缓存（默认是开启）， 通过`index.queries.cache.enabled：false`设置。

- `FieldDataCache`: 在聚类或排序时，field data cache会使用频繁，因此，设置字段数据缓存的大小，在聚类或排序场景较多的情形下很有必要，可通过indices.fielddata.cache.size：30% 或具体值10GB来设置。但是如果场景或数据变更比较频繁，设置cache并不是好的做法，因为缓存加载的开销也是特别大的。
- `ShardRequestCache`: 查询请求发起后，每个分片会将结果返回给协调节点(Coordinating Node), 由协调节点将结果整合。 如果有需求，可以设置开启; 通过设置`index.requests.cache.enable: true`来开启。 不过，shard request cache只缓存hits.total, aggregations, suggestions类型的数据，并不会缓存hits的内容。也可以通过设置`indices.requests.cache.size: 1%`（默认）来控制缓存空间大小。



### GC设置

推荐使用`G1 GC`,老版本的ES推荐默认设置： `Concurrent-Mark and Sweep(CMS)`

```shell
vim jvm.options
##将这几行
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly
###改为如下
-XX:+UseG1GC
-XX:MaxGCPauseMillis=50
```

` -XX:MaxGCPauseMillis`是控制预期的最高GC时长，默认值为200ms，如果线上业务特性对于GC停顿非常敏感，可以适当设置低一些。但是 这个值如果设置过小，可能会带来比较高的cpu消耗。

G1对于集群正常运作的情况下减轻G1停顿对服务时延的影响还是很有效的，但是如果是GC导致集群卡死，那么很有可能换G1也无法根本上解决问题。 通常都是集群的数据模型或者Query需要优化



### ES集群开启慢查询配置定位慢查询

```shell
PUT  /_template/{TEMPLATE_NAME}
{
 
  "template":"{INDEX_PATTERN}",
  "settings" : {
    "index.indexing.slowlog.level": "INFO",
    "index.indexing.slowlog.threshold.index.warn": "10s",
    "index.indexing.slowlog.threshold.index.info": "5s",
    "index.indexing.slowlog.threshold.index.debug": "2s",
    "index.indexing.slowlog.threshold.index.trace": "500ms",
    "index.indexing.slowlog.source": "1000",
    "index.search.slowlog.level": "INFO",
    "index.search.slowlog.threshold.query.warn": "10s",
    "index.search.slowlog.threshold.query.info": "5s",
    "index.search.slowlog.threshold.query.debug": "2s",
    "index.search.slowlog.threshold.query.trace": "500ms",
    "index.search.slowlog.threshold.fetch.warn": "1s",
    "index.search.slowlog.threshold.fetch.info": "800ms",
    "index.search.slowlog.threshold.fetch.debug": "500ms",
    "index.search.slowlog.threshold.fetch.trace": "200ms"
  },
  "version"  : 1
}
 
PUT {INDEX_PAATERN}/_settings
{
    "index.indexing.slowlog.level": "INFO",
    "index.indexing.slowlog.threshold.index.warn": "10s",
    "index.indexing.slowlog.threshold.index.info": "5s",
    "index.indexing.slowlog.threshold.index.debug": "2s",
    "index.indexing.slowlog.threshold.index.trace": "500ms",
    "index.indexing.slowlog.source": "1000",
    "index.search.slowlog.level": "INFO",
    "index.search.slowlog.threshold.query.warn": "10s",
    "index.search.slowlog.threshold.query.info": "5s",
    "index.search.slowlog.threshold.query.debug": "2s",
    "index.search.slowlog.threshold.query.trace": "500ms",
    "index.search.slowlog.threshold.fetch.warn": "1s",
    "index.search.slowlog.threshold.fetch.info": "800ms",
    "index.search.slowlog.threshold.fetch.debug": "500ms",
    "index.search.slowlog.threshold.fetch.trace": "200ms"
}
```





### 数据结构优化

#### 设置合理的副本数量

默认设置1，除非是数据非常重要的才设置2个以上(比如像银行、金融那种)，副本数在一定程度上可以提高集群的可用性，以及搜索性能。



#### 合理的配置使用 index 属性

合理的配置使用 index 属性，`analyzed `和 `not_analyzed`，根据业务需求来控制字段是否分词或不分词。只有 `groupby `需求的字段，配置时就设置成 `not_analyzed`，以提高查询或聚类的效率。



#### 减少不需要的字段

**Elasticsearch** 用于业务搜索服务，一些不需要用于搜索的字段最好不存到 ES 中，这样即节省空间，同时在相同的数据量下，也能提高搜索性能。

避免使用动态值作字段，动态递增的 mapping，会导致集群崩溃；同样，也需要控制字段的数量，业务中不使用的字段，就不要索引。控制索引的字段数量、mapping 深度、索引字段的类型，对于 ES 的性能优化是重中之重。

ES字段数、mapping深度的一些默认设置：

```shell
index.mapping.nested_objects.limit: 10000
index.mapping.total_fields.limit: 1000
index.mapping.depth.limit: 20
```



#### Nested Object vs Parent/Child

尽量避免使用 `nested `或` parent/child` 的字段，能不用就不用；`nested query` 慢，`parent/child query` 更慢，比` nested query` 慢上百倍；因此能在 `mapping `设计阶段搞定的（大宽表设计或采用比较 `smart `的数据结构），就不要用父子关系的 `mapping`。

如果一定要使用 `nested fields`，保证 `nested fields` 字段不能过多，目前 ES 默认限制是 **`50`**。因为针对 1 个 `document`，每一个 `nested field`，都会生成一个独立的 document，这将使 doc 数量剧增，影响查询效率，尤其是 JOIN 的效率。

```
index.mapping.nested_fields.limit: 50
```

| 对比 | Nested Object                        | Parent/Child                                       |
| :--- | :----------------------------------- | :------------------------------------------------- |
| 优点 | 文档存储在一起，因此读取性高         | 父子文档可以独立更新，互不影响                     |
| 缺点 | 更新父文档或子文档时需要更新整个文档 | 为了维护 join 关系，需要占用部分内存，读取性能较差 |
| 场景 | 子文档偶尔更新，查询频繁             | 子文档更新频繁                                     |







## 硬件

尽量使用固态硬盘（SSD）

**使用 RAID0 是提高硬盘速度的有效途径**，没有必要使用镜像或其它 RAID 变体，因为 Elasticsearch 在自身层面通过副本，已经提供了备份的功能，所以不需要利用磁盘的备份功能，同时如果使用磁盘备份功能的话，对写入速度有较大的影响。

**避免使用网络附加存储（NAS）**

