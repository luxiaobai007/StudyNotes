---
title: Redis6入门学习(一)
---

[toc]

# **NoSQL数据库**

## **NoSQL数据库**

### **概述**

NoSQL(NoSQL = Not Only SQL )，意即“不仅仅是SQL”，泛指非关系型的数据库。NoSQL 不依赖业务逻辑方式存储，而以简单的key-value模式存储。因此大大的增加了数据库的扩展能力。

- 不遵循SQL标准。
- 不支持ACID。(事务的4个特征:原子性、一致性、隔离性和持久性)
- 远超于SQL的性能。

#### **适用场景**

- 对数据高并发的读写
- 海量数据的读写
- 对数据高可扩展性的

#### **不适用的场景**

- 需要事务支持
- 基于sql的结构化查询存储，处理复杂的关系,需要即席查询。

> （用不着sql的和用了sql也不行的情况，请考虑用NoSql）

#### **Memcache**

- 很早出现的NoSql数据库
- 数据都在内存中，一般不持久化
- 支持简单的key-value模式，支持类型单一
- 一般是作为缓存数据库辅助持久化的数据库

#### **Redis**

- 几乎覆盖了Memcached的绝大部分功能
- 数据都在内存中，支持持久化，主要用作备份恢复
- 除了支持简单的key-value模式，还支持多种数据结构的存储，比如 list、set、hash、zset等。
- 一般是作为缓存数据库辅助持久化的数据库

#### **MongoDB**

- 高性能、开源、模式自由(schema  free)的文档型数据库
- 数据都在内存中， 如果内存不足，把不常用的数据保存到硬盘
- 虽然是key-value模式，但是对value（尤其是json）提供了丰富的查询功能
- 支持二进制数据及大型对象
- 可以根据数据的特点替代RDBMS ，成为独立的数据库。或者配合RDBMS，存储特定的数据



### **行式存储数据库**

![行式存储](http://qiliu.luxiaobai.cn/img/redisrow.png)



### 列式存储数据库

![列式数据库](http://qiliu.luxiaobai.cn/img/rediscol.png)



### 图关系型数据库

主要应用：社会关系，公共交通网络，地图及网络拓谱(n*(n-1)/2)

![图关系型](http://qiliu.luxiaobai.cn/img/redisimg.png)

### **Cassandra[kəˈsændrə]**

Apache Cassandra是一款免费的开源NoSQL数据库，其设计目的在于管理由大量商用服务器构建起来的庞大集群上的海量数据集(数据量通常达到PB级别)。在众多显著特性当中，Cassandra最为卓越的长处是对写入及读取操作进行规模调整，而且其不强调主集群的设计思路能够以相对直观的方式简化各集群的创建与扩展流程。

### **Hbase**

HBase是Hadoop项目中的数据库。它用于需要对大量的数据进行随机、实时的读写操作的场景中。

HBase的目标就是处理数据量非常庞大的表，可以用普通的计算机处理超过10亿行数据，还可处理有数百万列元素的数据表。



# Redis安装

## **介绍**

- Redis是一个开源的key-value存储系统。
- 和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。
- 这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。
- 在此基础上，Redis支持各种不同方式的排序。
- 与memcached一样，为了保证效率，数据都是缓存在内存中。
- 区别的是Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。
- 并且在此基础上实现了master-slave(主从)同步

## **适用场景**

### **配合关系型数据库做高速缓存**

- 高频次，热门访问的数据，降低数据库IO
- 分布式架构，做session共享



### **多样的数据结构存储持久化数据**

![redis1](http://qiliu.luxiaobai.cn/img/redis1.png)



## **Ubuntu安装Redis**

```shell
#安装
sudo apt update
sudo apt install redis-server

#启动
redis-server

#查看redis是否启动
redis-cli
root@iZbp13941xpzjmefjge9chZ:~# redis-cli
127.0.0.1:6379> auth "lxb991209"
127.0.0.1:6379> ping
PONG
```



### **Ubuntu默认目录 /usr/bin**

```shell
redis-benchmark:性能测试工具，可以在自己本机运行，看看自己本机性能如何
redis-check-aof：修复有问题的AOF文件，rdb和aof后面讲 
redis-check-dump：修复有问题的dump.rdb文件
redis-sentinel：Redis集群使用
redis-server：Redis服务器启动命令
redis-cli：客户端，操作入口
```



#### **前台启动(不推荐)**

```shell
redis-server
```



#### 后台启动(推荐)

```shell
cp -r redis.conf /etc/redis.conf
redis-server /etc/redis.conf
```



### 相关知识

```shell
默认端口6379
默认16个数据库，类似数组下标从0开始，初始默认使用0号库
使用命令 select   <dbid>来切换数据库。如: select 8
统一密码管理，所有库同样密码。
dbsize查看当前数据库的key的数量
flushdb清空当前库
flushall通杀全部库
```

Redis是**单线程+多路IO复用技术**

多路复用是指**使用一个线程来检查多个文件描述符（Socket）的就绪状态**，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）

**串行  vs  多线程+锁（memcached） vs  单线程+多路IO复用(Redis)**

（

与Memcache三点不同: 

- 支持多数据类型
- 支持持久化
- 单线程+多路IO复用

）



# 五大数据类型

## Redis键(Key)

```shell
keys *                ==查看当前库所有key==    (匹配：keys *1)
exists key            判断某个key是否存在
type key              查看你的key是什么类型
del key               删除指定的key数据
unlink key            根据value选择非阻塞删除 仅将keys从keyspace     元数据中删除，真正的删除会在后续异步操作。
expire key 10         10秒钟：为给定的key设置过期时间
ttl key               查看还有多少秒过期，-1表示永不过期，-2表示已过期
 
select                命令切换数据库
dbsize               查看当前数据库的key的数量
flushdb              清空当前库
flushall             通杀全部库
```



## **Redis字符串(String)**

- String是Redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。
- String类型是二进制安全的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象。
- String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M



### **常用命令**

```shell
set <key><value>      添加键值对
    *NX：当数据库中key不存在时，可以将key-value添加数据库
    *XX：当数据库中key存在时，可以将key-value添加数据库，与NX参数互斥
    *EX：key的超时秒数
    *PX：key的超时毫秒数，与EX互斥

get     <key>             查询对应键值
append  <key><value>      将给定的<value> 追加到原值的末尾
strlen  <key>             获得值的长度
setnx  <key><value>      只有在 key 不存在时    设置 key 的值
 
incr  <key>             将 key 中储存的数字值增1,只能对数字值操作，如果为空，新增值为1
decr  <key>             将 key 中储存的数字值减1,只能对数字值操作，如果为空，新增值为-1
incrby / decrby  <key> <步长>   将 key 中储存的数字值增减。自定义步长。        
mset  <key1><value1><key2><value2>  .....    同时设置一个或多个 key-value对  
mget  <key1><key2><key3> .....               同时获取一个或多个 value  
msetnx <key1><value1><key2><value2>  .....   同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
原子性，有一个失败则都失败
 
getrange  <key><起始位置><结束位置>             获得值的范围，类似java中的substring，前包，后包
setrange  <key><起始位置><value>               用 <value>  覆写<key>所储存的字符串值，从<起始位置>开始(索引从0开始)。
 
setex  <key><过期时间><value>                  设置键值的同时，设置过期时间，单位秒。
getset <key><value>                           以新换旧，设置了新值同时获得旧值。

```

>**incr key :对存储在指定key的数值执行原子的加1操作**
>
>所谓原子操作是指不会被线程调度机制打断的操作；
>
>这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。
>
>（1）在单线程中， 能够在单条指令中完成的操作都可以认为是"原子操作"，因为中断只能发生于指令之间。
>
>（2）在多线程中，不能被其它进程（线程）打断的操作就叫原子操作。
>
>Redis单命令的原子性主要得益于Redis的单线程



### **数据结构**

String的数据结构为简单动态字符串(Simple Dynamic String,缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配.

![redisstring](http://qiliu.luxiaobai.cn/img/redisstring.png)

内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。**当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间**。需要注意的是字符串最大长度为512M。



## Redis列表(List)

### **简介**

单键多值

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。**它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。**

![redislist](http://qiliu.luxiaobai.cn/img/redislist.png)

### **常用命令**

```shell
lpush/rpush  <key><value1><value2><value3> .... 从左边/右边插入一个或多个值。
lpop/rpop  <key>            从左边/右边吐出一个值。值在键在，值光键亡。
rpoplpush  <key1><key2>     从<key1>列表右边吐出一个值，插到<key2>列表左边。
lrange <key><start><stop>     按照索引下标获得元素(从左到右)
lrange mylist 0 -1             0左边第一个，-1右边第一个，（0-1表示获取所有）
lindex <key><index>          按照索引下标获得元素(从左到右)
llen <key>                    获得列表长度
linsert <key>  before/after <value><newvalue>     在<value>的后面插入<newvalue>插入值
lrem <key><n><value>                         从左边删除n个value(从左到右)
lset<key><index><value>                    将列表key下标为index的值替换成value
```



### 数据结构

List的数据结构为==**快速链表quickList**。==

首先在**列表元素较少的情况下会使用一块连续的内存存储**，这个结构是**ziplist**，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当**数据量比较多的时候才会改成quicklist**。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

![redislist1](http://qiliu.luxiaobai.cn/img/redislist1.png)

Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。



## Redis集合(Set)

### 简介

Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于**set是可以自动排重**的，set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表，所以添加，删除，查找的**复杂度都是O(1)**。

一个算法，随着数据的增加，执行时间的长短，如果是O(1)，数据增加，查找数据的时间不变

### 常用命令

```shell
sadd <key><value1><value2> .....   将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
smembers <key>                     取出该集合的所有值。
sismember <key><value>             判断集合<key>是否为含有该<value>值，有1，没有0
scard<key>                         返回该集合的元素个数。
srem <key><value1><value2> ....    删除集合中的某个元素。
spop <key>                         随机从该集合中吐出一个值。
srandmember <key><n>               随机从该集合中取出n个值。不会从集合中删除 。
smove <source><destination>value        把集合中一个值从一个集合移动到另一个集合
sinter <key1><key2>                        返回两个集合的交集元素。
sunion <key1><key2>                        返回两个集合的并集元素。
sdiff <key1><key2>                返回两个集合的差集元素(key1中的，不包含key2中的)
```



### 数据结构

Set数据结构是dict字典，字典是用哈希表实现的。Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。



## Redis哈希(hash)

### 简介

Redis hash 是一个键值对集合。

Redis hash是一个string类型的**field**和**value**的映射表，hash特别适合用于存储对象。

类似Java里面的Map<String,Object>

用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，如果用普通的key/value结构来存储

![redishash](http://qiliu.luxiaobai.cn/img/redishash.png)



![redishash2](http://qiliu.luxiaobai.cn/img/redishash2.png)



通过 **key(用户ID) + field(属性标签)** 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题



### 常用命令

```shell
hset <key><field><value>         给<key>集合中的<field>键赋值<value>
hget <key1><field>                从<key1>集合<field>取出 value
hmset <key1><field1><value1><field2><value2>...         批量设置hash的值
hexists<key1><field>                        查看哈希表 key 中，给定域 field 是否存在。
hkeys <key>                                列出该hash集合的所有field
hvals <key>                                列出该hash集合的所有value
hincrby <key><field><increment>           为哈希表 key 中的域 field 的值加上增量 1   -1
hsetnx <key><field><value>                将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .
```



### **数据结构**

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。



## Redis有序集合(Zset)

### 简介

Redis有序集合zset与普通集合set非常相似，是一个**没有重复元素**的字符串集合。

不同之处是有序集合的每个成员都关联了一个**评分（score**）,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复了 。

因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。



### **常用命令**

```shell
zadd  <key><score1><value1><score2><value2>…        将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
zrange <key><start><stop>  [WITHSCORES]            返回有序集 key 中，下标在<start><stop>之间的元素  带WITHSCORES，可以让分数一起和值返回到结果集。
zrangebyscore key minmax [withscores] [limit offset count]       返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。
zrevrangebyscore key maxmin [withscores] [limit offset count]    同上，改为从大到小排列。
zincrby <key><increment><value>                    为元素的score加上增量
zrem  <key><value>                                 删除该集合下，指定值的元素
zcount <key><min><max>                             统计该集合，分数区间内的元素个数
zrank <key><value>                                   返回该值在集合中的排名，从0开始。
```



### **数据结构**

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构

（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。



## 跳跃表

### 简介

有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

### **实例**

对比有序链表和跳跃表，从链表中查询出51

（1） 有序链表

![redis2](http://qiliu.luxiaobai.cn/img/redis2.png)

要查找值为51的元素，需要从第一个元素开始依次查找、比较才能找到。共需要6次比较。

（2） 跳跃表

![redis3](http://qiliu.luxiaobai.cn/img/redis3.png)

从第2层开始，1节点比51节点小，向后比较。

21节点比51节点小，继续向后比较，后面就是NULL了，所以从21节点向下到第1层

在第1层，41节点比51节点小，继续向后，61节点比51节点大，所以从41向下

在第0层，51节点为要查找的节点，节点被找到，共查找4次。

 

从此可以看出跳跃表比有序链表效率要高


