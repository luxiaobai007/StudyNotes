---
title: Redis6入门学习(二)
---

[toc]

# Redis6配置文件

```shell
//配置文件路径
vim /etc/redis/redis.conf
```

## **Units单位**

配置大小单位,开头定义了一些基本的度量单位，只支持bytes，不支持bit大小写不敏感

![redisconf1](http://qiliu.luxiaobai.cn/img/redisconf1.png)

## **INCLUDES包含**

![redis6](http://qiliu.luxiaobai.cn/img/redis6.png)

类似jsp中的include，多实例的情况可以把公用的配置文件提取出来



## 网络相关配置

### bind

默认情况bind=127.0.0.1只能接受本机的访问请求,不写的情况下，无限制接受任何ip地址的访问

生产环境肯定要写你应用服务器的地址；服务器是需要远程访问的，所以需要将其注释掉

如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的响应

![rediscof](http://qiliu.luxiaobai.cn/img/rediscof.png)

保存配置,重启服务

```shell
redis-cli
shutdown
redis-server /etc/redis.conf
```



### **protected-mode**

protected-mode配置，默认是yes，即开启。设置外部网络连接redis服务，设置方式如下：

1、关闭protected-mode模式，此时外部网络可以直接访问

2、开启protected-mode保护模式，需配置bind ip或者设置访问密码



### **tcp-backlog**

设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列 + 已经完成三次握手队列。

在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。

注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值（128），所以需要确认增大/proc/sys/net/core/somaxconn和/proc/sys/net/ipv4/tcp_max_syn_backlog（128）两个值来达到想要的效果

![redisconf2](http://qiliu.luxiaobai.cn/img/redisconf2.png)



### **timeout**

一个空闲的客户端维持多少秒会关闭，0表示关闭该功能。即永不关闭。



### **tcp-keepalive**

对访问客户端的一种心跳检测，每个n秒检测一次。单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60

![redisconf3](http://qiliu.luxiaobai.cn/img/redisconf3.png)



## **GENERAL通用**

### **daemonize**

是否为后台进程，设置为yes 守护进程，后台启动



### **pidfile**

存放pid文件的位置，每个实例会产生一个不同的pid文件



### **loglevel**

指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为notice

四个级别根据使用阶段来选择，**生产环境选择notice 或者warning**



### **logfile**



### **databases16**

设定库的数量 默认16，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id





## **SECURITY安全**

**设置密码**

![redisconf4](http://qiliu.luxiaobai.cn/img/redisconf4.png)

访问密码的查看、设置和取消

在命令中设置密码，只是临时的。重启redis服务器，密码就还原了。

永久设置，需要再配置文件中进行设置。

```shell
root@iZbp13941xpzjmefjge9chZ:/etc# redis-cli
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass "123456"
OK
127.0.0.1:6379> config get requirepass
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456
OK
```





##  **LIMITS限制**

### **maxclients**

设置redis同时可以与多少个客户端进行连接。

默认情况下为10000个客户端。

如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应。

![redisconf5](http://qiliu.luxiaobai.cn/img/redisconf5.png)



### **maxmemory**

- 建议必须设置，否则，将内存占满，造成服务器宕机
- 设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过**maxmemory-policy**来指定。
- 如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。
- 但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。

![redisconf6](http://qiliu.luxiaobai.cn/img/redisconf6.png)



### **maxmemory-policy**

**volatile-lru**：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）

**allkeys-lru**：在所有集合key中，使用LRU算法移除key

**volatile-random**：在过期集合中移除随机的key，只对设置了过期时间的键

**allkeys-random**：在所有集合key中，移除随机的key

**volatile-ttl**：移除那些TTL值最小的key，即那些最近要过期的key

**noeviction**：不进行移除。针对写操作，只是返回错误信息

![redisconf7](http://qiliu.luxiaobai.cn/img/redisconf7.png)



### **maxmemory-samples**

- 设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个。
- 一般设置3到7的数字，数值越小样本越不准确，但性能消耗越小。

![redisconf8](http://qiliu.luxiaobai.cn/img/redisconf8.png)





# Redis6的发布和订阅

> Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。
>
> Redis 客户端可以订阅任意数量的频道。



## **Redis的发布和订阅**

**1、客户端可以订阅频道**

![redispub1](http://qiliu.luxiaobai.cn/img/redispub1.png)

**2、当给这个频道发布消息后，消息就会发送给订阅的客户端**

![redispub2](http://qiliu.luxiaobai.cn/img/redispub2.png)



## **发布订阅命令行实现**

1、 打开一个客户端订阅channel1

```shell
SUBSCRIBE channel1
```

2、打开另一个客户端，给channel1发布消息hello

```shell
127.0.0.1:6379> publish channel hello (integer) 1
```

返回的1是订阅者数量

3、打开第一个客户端可以看到发送的消息

![redispub4](http://qiliu.luxiaobai.cn/img/redispub4.png)

> 发布的消息没有持久化，如果在订阅的客户端收不到hello，只能收到订阅后发布的消息





# **Redis6新数据类型**

## **Bitmaps**

>现代计算机用二进制（位） 作为信息的基础单位， 1个字节等于8位， 例如“abc”字符串是由3个字节组成， 但实际在计算机存储时将其用二进制表示， “abc”分别对应的ASCII码分别是97、 98、 99， 对应的二进制分别是01100001、 01100010和01100011

![redisdata1](http://qiliu.luxiaobai.cn/img/redisdata1.png)

合理地使用操作位能够有效地提高内存使用率和开发效率。

Redis提供了Bitmaps这个“数据类型”可以实现对位的操作：

（1） Bitmaps本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的位进行操作。

（2） Bitmaps单独提供了一套命令， 所以在Redis中使用Bitmaps和使用字符串的方法不太相同。 可以把Bitmaps想象成一个以位为单位的数组， 数组的每个单元只能存储0和1， 数组的下标在Bitmaps中叫做偏移量。

![redisdata2](http://qiliu.luxiaobai.cn/img/redisdata2.png)

### **命令**

**setbit**

```shell
setbit <key><offset><value>                 设置Bitmaps中某个偏移量的值（0或1）
```

offset:偏移量从0开始



### 实例

每个独立用户是否访问过网站存放在Bitmaps中， 将访问的用户记做1， 没有访问的用户记做0， 用偏移量作为用户的id。

设置键的第offset个位的值（从0算起） ， 假设现在有20个用户，userid=1， 6， 11， 15， 19的用户对网站进行了访问， 那么当前Bitmaps初始化结果如图

![redisdata3](http://qiliu.luxiaobai.cn/img/redisdata3.png)

unique:users:20201106代表2020-11-06这天的独立访问用户的Bitmaps

```shell
127.0.0.1:6379> setbit unique:users:20201106 1 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201106 6 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201106 11 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201106 15 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201106 19 1
(integer) 0
```

> 很多应用的用户id以一个指定数字（例如10000） 开头， 直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费， 通常的做法是每次做setbit操作时将用户id减去这个指定数字。
>
> 在第一次初始化Bitmaps时， 假如偏移量非常大， 那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞。



## getbit

```shell
getbit <key> <offset>        获取Bitmaps中某个偏移量的值
```

获取键的第offset位的值(从0开始算



### 实例

获取 id=8的用户是否在2020-11-06这天访问过， 返回0说明没有访问过：

```shell
127.0.0.1:6379> getbit unique:users:20201106 8
(integer) 0
127.0.0.1:6379> getbit unique:users:20201106 1
(integer) 1
127.0.0.1:6379> getbit unique:users:20201106 100
(integer) 0
```

==因为100根本不存在,所以返回0==





## **bitcount**

统计字符串被设置为1的bit数。一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。start 和 end 参数的设置，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位，start、end 是指bit组的字节的下标数，二者皆包含。

```shell
bitcount<key>[start end]  统计字符串从start字节到end字节比特值为1的数量
```



### 实例

2022-11-06这天的独立访问用户数量

```shell
127.0.0.1:6379> bitcount unique:users:20201106
(integer) 5
```



start和end代表起始和结束字节数， 下面操作计算用户id在第1个字节到第3个字节之间的独立访问用户数， 对应的用户id是11， 15， 19。

```shell
127.0.0.1:6379> bitcount unique:users:20201106 1 3
(integer) 3
##1个字节8位
```

>**redis的setbit设置或清除的是bit位置，而bitcount计算的是byte位置。**



## **bitop**

```shell
bitop and(or/not/xor) <destkey> [key...]
```

bitop是一个复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在destkey中。

```shell
127.0.0.1:6379> setbit unique:users:20201104 1 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201104 2 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201104 5 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201104 9 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201103 0 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201103 1 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201103 4 1
(integer) 0
127.0.0.1:6379> setbit unique:users:20201103 9 1
(integer) 0
127.0.0.1:6379> bitop and unique:users:and:20201104_03 unique:users:20201103 unique:users:20201104
(integer) 2
```



## **Bitmaps与set对比**

假设网站有1亿用户， 每天独立访问的用户有5千万， 如果每天用集合类型和Bitmaps分别存储活跃用户可以得到表

<table>
	<tr align="center">
	    <th colspan="9">set和Bitmaps存储独立用户空间对比</th>
	</tr >
	<tr align="center">
	    <td>数据类型</td>
	    <td>一天</td>
	    <td>一个月</td>
    <td>一年</td>
	</tr>
	<tr align="center">
	    <td>集合类型</td>
	    <td>400MB</td>
    	<td>12GB</td>
    	<td>144GB</td>
	</tr>
	<tr align="center">
	    <td>Bitmaps</td>
	    <td>12.5MB</td>
    	<td>375MB</td>
    	<td>4.5GB</td>
	</tr>
</table>

很明显， 这种情况下使用Bitmaps能节省很多的内存空间， 尤其是随着时间推移节省的内存还是非常可观的

但Bitmaps并不是万金油， 假如该网站每天的独立访问用户很少， 例如只有10万（大量的僵尸用户） ， 那么两者的对比如下表所示， 很显然， 这时候使用Bitmaps就不太合适了， 因为基本上大部分位都是0

<table>
	<tr>
	    <th align="center" colspan="9">set和Bitmaps存储一天活跃用户对比（独立用户比较少)</th>
	</tr >
	<tr align="center">
	    <td>数据类型</td>
	    <td>每个userid占用空间</td>
	    <td>需要存储的用户量</td>
    	<td>全部内存量</td>
	</tr>
	<tr align="center">
	    <td>集合类型</td>
	    <td>64位</td>
    	<td>100000</td>
    	<td>64位*100000 = 800KB</td>
	</tr>
	<tr align="center">
	    <td>Bitmaps</td>
	    <td>1位</td>
    	<td>100000000</td>
    	<td>1位*100000000 = 12.5MB</td>
	</tr>
</table>



## **HyperLogLog**

>在工作当中，我们经常会遇到与统计相关的功能需求，比如统计网站PV（PageView页面访问量）,可以使用Redis的incr、incrby轻松实现。但像UV（UniqueVisitor，独立访客）、独立IP数、搜索记录数等需要去重和计数的问题如何解决？这种求集合中不重复元素个数的问题称为基数问题。解决基数问题有很多种方案：
>
>（1）数据存储在MySQL表中，使用distinct count计算不重复个数
>
>（2）使用Redis提供的hash、set、bitmaps等数据结构来处理
>
>以上的方案结果精确，但随着数据不断增加，导致占用空间越来越大，对于非常大的数据集是不切实际的。能否能够降低一定的精度来平衡存储空间？
>
>Redis推出了HyperLogLog. Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。什么是基数?比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。



### 命令

#### pfadd

```shell
pfadd <key><element>[element...]		添加指定元素到HyperLogLog中
```

将所有元素添加到指定HyperLogLog数据结构中。如果执行命令后HLL估计的近似基数发生变化，则返回1，否则返回0。



#### pfcount

```shell
pfcount <key>[key...]  计算HLL的近似基数，可以计算多个HLL

127.0.0.1:6379> pfadd hll1 "redis" "mysql"
(integer) 1
127.0.0.1:6379> pfadd hll1 "redis"
(integer) 0
127.0.0.1:6379> pfcount hll1
(integer) 2
127.0.0.1:6379> pfadd hll1 "mongodb"
(integer) 1
127.0.0.1:6379> pfadd h2 "redis"
(integer) 1
127.0.0.1:6379> pfadd h2 "java"
(integer) 1
127.0.0.1:6379> pfcount hll1 h2
(integer) 4
```



#### pfmerge

```shell
pfmerge <destkey><sourcekey>[sourcekey...] .  将一个或多个HLL合并后的结果存储在另一个HLL中 比如每月活跃用户可以使用每天的活跃用户来合并计算可得
127.0.0.1:6379> pfcount hll1 h2
(integer) 4
127.0.0.1:6379> pfmerge h3 hll1 h2
OK
127.0.0.1:6379> pfcount h3
(integer) 4
```



## **Geospatial**

> GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。



### 命令

**geoadd**

```shell
geoadd <key>< longitude><latitude><member> [longitude latitude member...]   添加地理位置（经度，纬度，名称）
```

两极无法直接添加，一般会下载城市数据，直接通过 Java 程序一次性导入。有效的经度从 -180 度到 180 度。有效的纬度从 -85.05112878 度到 85.05112878 度。当坐标位置超出指定范围时，该命令将会返回一个错误。已经添加的数据，是无法再次往里面添加的。



#### **geopos**

```shell
geodist<key><member1><member2>  [m|km|ft|mi ]  获取两个位置之间的直线距离

127.0.0.1:6379> geodist china:city beijing shanghai km
"1068.1535"
```

>单位：
>
>m 表示单位为米[默认值]。
>
>km 表示单位为千米。
>
>mi 表示单位为英里。
>
>ft 表示单位为英尺。
>
>如果用户没有显式地指定单位参数， 那么 GEODIST 默认使用米作为单位



#### **georadius**

```shell
georadius<key>< longitude><latitude>radius  m|km|ft|mi   以给定的经纬度为中心，找出某一半径内的元素
经度 纬度 距离 单位
127.0.0.1:6379> georadius china:city 110 30 1000 km
1) "chonggin"
2) "shenzhen"
```


