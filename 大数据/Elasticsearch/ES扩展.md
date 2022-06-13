

# ES扩展

>- 集群扩展之后 磁盘分布情况
>
>- 集群扩展之后 分片情况
>- 数据迁移过程中搜索功能是否影响
>- 分片数量和数据的关系
> - 分片和副本作用
> - 分片、副本数修改



# 分片

由于ES索引库得分片数量在索引库创建时就已经确定，后续无法进行分片数量得修改



## 增加副本数量

```http
###修改副本数量
PUT students1/_settings
{
  "settings":{
    "number_of_replicas":3
  }
}

```



## 重新索引实现分片数量的修改

创建索引`student1`

```http
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

```

创建索引`student2` 作为重新索引

```http
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





# 查看集群状态接口命令

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



# 集群扩展之后 磁盘分布情况

## 磁盘使用率过高问题

- **磁盘使用率超过85%**： 会导致新的分片无法分配（**已测试**）
- **磁盘使用率超过90%** ： ES会尝试将对应节点中的分片迁移到其他磁盘使用率比较低的数据节点中（**已测试**）
- **磁盘使用率超过95%**： 系统会对ES集群中的每个索引强制设置`read_only_allow_delete`属性，此时索引将无法写入数据，只能读取和删除对应索引 （**已测试**）

`264*90%=237.6`

![image-20211230151757114](http://qiliu.luxiaobai.cn/img/image-20211230151757114.png)

通过增加节点`node-6`,部署到**C磁盘**中，复现了磁盘使用率超过90%的场景，实际的显示是将索引`txt_number`中`node-5`的`shards:2`转移到了新节点`node-6`中

![image-20211230153039969](http://qiliu.luxiaobai.cn/img/image-20211230153039969.png)







`264*95%=250.8`

![image-20211230153722967](http://qiliu.luxiaobai.cn/img/image-20211230153722967.png)

将磁盘使用率降低到95%以上,新增数据失败

![image-20211230150421073](http://qiliu.luxiaobai.cn/img/image-20211230150421073.png)

![image-20211230155936589](http://qiliu.luxiaobai.cn/img/image-20211230155936589.png)



### 更改磁盘使用率配置

#### 更改配置文件

`elasticsearch.yml`

```shell
cluster.routing.allocation.disk.threshold_enabled: true
cluster.routing.allocation.disk.watermark.low: 90%
cluster.routing.allocation.disk.watermark.high: 95%
cluster.routing.allocation.disk.watermark.flood_stage: 98%
```

#### 动态更改

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



集群扩容时，会将之前的节点数据都备份到新节点中，数据均衡分配

当前节点有2个`node-1`,`node-2` 节点情况如下

![image-20211229174009900](http://qiliu.luxiaobai.cn/img/image-20211229174009900.png)

![image-20211229173958359](http://qiliu.luxiaobai.cn/img/image-20211229173958359.png)



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



当前`txt_number2`的各shard的详细情况如下所示，`node-1`和`node-2`数据大小基本一致，同时可以看出==各节点的分片和副本都分布在不同节点，数据大小接近相等== 相同分片的副本不会放在同一个节点上

![image-20211229174258208](http://qiliu.luxiaobai.cn/img/image-20211229174258208.png)





<img src="http://qiliu.luxiaobai.cn/img/image-20211229175018573.png" alt="image-20211229175018573" style="zoom:67%;" />





如下所示为`txt_number2`各分段(segment)的相关情况

![image-20211229175508193](http://qiliu.luxiaobai.cn/img/image-20211229175508193.png)



动态增加`node-3`节点后，各shard的详细情况如下所示。数据重新复制一份。

![image-20211229181007246](http://qiliu.luxiaobai.cn/img/image-20211229181007246.png)



增加副本数量及节点`node-4`,情况如下所示

```http
PUT /txt_number2/_settings
{
    "number_of_replicas": 5
}
```



扩容4个节点，每个节点占据的数据量11.2G，其中`node-2`占据2个主分片，`node-2`占据1个主分片

![image-20211230091148435](http://qiliu.luxiaobai.cn/img/image-20211230091148435.png)

![image-20211229182230647](http://qiliu.luxiaobai.cn/img/image-20211229182230647.png)



![image-20211230091050775](http://qiliu.luxiaobai.cn/img/image-20211230091050775.png)

由于磁盘使用率超过了==85%==，增加节点导致没有进行分片复制

![image-20211230105259176](http://qiliu.luxiaobai.cn/img/image-20211230105259176.png)





# 集群扩展之后 分片情况

写索引是只能写在主分片上，然后同步到副本分片 。

写操作： 对文档的新建、索引和删除请求，必须在主分片上面完成之后才能复制到奥相关的副本分片上。

分片底层实现即一个`Lucene`索引， 而一个Elasticsearch索引是分片的集合。



## 分片与路由

当索引一个文档的时候，文档会被存储到一个主分片中。 Elasticsearch 如何知道一个文档应该存放到哪个分片中呢？当我创建文档时，它如何决定这个文档应当被存储在分片 1 还是分片 2 中呢？

实际上，这个过程是根据下面这个公式决定的**`shard = hash(routing) % number_of_primary_shards`**

routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。 routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到 余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置。

这就解释了为什么要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了

> ES自动管理和组织分片, 并在必要的时候对分片数据进行再平衡分配, 所以用户基本上不用担心分片的处理细节，一个分片默认最大文档数量是20亿.





## 集群级分片分配设置



`cluster.routing.allocation.enable`(动态)为特定类型的分片启用或禁用分配：

- `all` -（默认）允许为各种分片分配分片。
- `primaries` - 只允许为主分片分配分片。
- `new_primaries` - 仅允许为新索引的主分片分配分片。
- `none` - 任何索引都不允许进行任何类型的分片分配。



# ES集群扩容方式

| **变配方式** | **适用场景**                                         | **实现原理介绍**                                             | **特点说明**                                                 |
| ------------ | ---------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 增加磁盘容量 | 计算资源充足，磁盘存储空间不足                       | 直接调整，逐个对集群中节点上挂载的磁盘扩容容量（**对加密云盘将采取蓝绿模式**）。 | 平滑扩容不停服，流程时间短，每个节点预计30秒。               |
| 增加磁盘数量 | 计算资源充足，磁盘存储空间不足，或 IOPS 和吞吐量不足 | 可选择滚动模式或蓝绿模式。                                   | 滚动模式较快，蓝绿模式耗时长但对线上业务无影响。             |
| 添加更多节点 | 节点规格较高，但集群整体计算资源不足                 | 直接调整，向集群中加入更多同等规格的节点。                   | 平滑扩容不停服，流程时间短，不受节点数量影响，预计5 - 10分钟完成。 |
| 提升节点规格 | 节点规格较低，且集群整体计算资源不足                 | 可选择滚动模式或蓝绿模式。                                   | 滚动模式较快，蓝绿模式耗时长但对线上业务无影响。             |





## 蓝绿模式

不重启集群，为原集群添加相同数量的新节点（配置为需要的节点且拷贝原节点数据），完成配置后，无缝切换并剔除掉原节点，**变配完成后节点 IP 会发生变化**。

- **优点：**变配过程中业务不停服，且扩容非常平滑，适合对集群可用性较高的场景。
- **缺点：**因涉及数据拷贝，变配耗时和集群数据量成正比，变配时间从数分钟到数天或更长。







## 滚动模式

对集群中节点逐个滚动重启完成变配，期间系统服务不间断，但可能影响线上访问性能。

- **优点：**由于不需要迁移数据，扩容的时间不受集群数据规模影响，变配能够快速完成。适合于集群遇到性能瓶颈，期望快速完成扩容变配的场景。
- **缺点：**需要滚动重启集群的每一个数据节点，因此如果集群存在无副本情况下会有丢失数据的风险，且在变配过程中会不停的有节点离开集群和加入集群，会对集群的性能有一定的影响，因此不建议在业务高峰期间选择此种变配方案。

