---

title: Redis6入门学习(五)
---

[toc]

# Redis集群

## 集群

Redis 集群实现了对Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。

Redis 集群通过分区（partition）来提供一定程度的可用性（availability）： 即使集群中有一部分节点失效或者无法进行通讯， 集群也可以继续处理命令请求。



### 删除持久化数据

将rdb,aof文件都删除掉



## 制作6个实例 **6379,6380,6381,6389,6390,6391**

### 配置基本信息

```shell
开启daemonize yes
Pid文件名字
指定端口
Log文件名字
Dump.rdb名字
Appendonly 关掉或者换名字

redis cluster 配置修改
cluster-enabled yes    打开集群模式
cluster-config-file nodes-6379.conf  设定节点配置文件名
cluster-node-timeout 15000   设定节点失联时间，超过该时间（毫秒），集群自动进行主从切换。
```

```shell
include /home/lushengyang/myredis/redis.conf
pidfile "/run/redis_6379.pid"
port 6379
dbfilename "dump6379.rdb"
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000

//替换命令
:%s/6379/6380
```



### 启动redis服务

![rediss1](http://qiliu.luxiaobai.cn/img/rediss1.png)



### 将六个节点合成一个集群

组合之前，请**确保所有redis实例启动**后，nodes-xxxx.conf文件都生成正常

<img src="http://qiliu.luxiaobai.cn/img/rediss2.png" alt="rediss2" style="zoom:67%;" />



### 合体

此处不要用127.0.0.1， 请用真实IP地址

--replicas 1 采用最简单的方式配置集群，一台主机，一台从机，正好三组。

```shell
cd /usr/bin
redis-cli --cluster create --cluster-replicas 1 121.199.76.44:6379 121.199.76.44:6380 121.199.76.44:6381 121.199.76.44:6389 121.199.76.44:6390 121.199.76.44:6391

#注意一定不要用127.0.0.1这种本地的局域ip，要用公网ip
ruby redis-trib.rb  create --replicas 1 121.199.76.44:6379 121.199.76.44:6380 121.199.76.44:6381 121.199.76.44:6389 121.199.76.44:6390 121.199.76.44:6391

```

<img src="http://qiliu.luxiaobai.cn/img/rediss3.png" alt="rediss3" style="zoom:50%;" />



### 登录方式

#### 普通登录方式

```shell
redis-cli -p 6379
```

**-c 采用集群策略连接，设置数据会自动切换到相应的写主机**

```shell
redis-cli -c -p 6379
```



**cluster nodes命令查看集群信息**

```shell
127.0.0.1:6379> cluster nodes
83efced12fa56cba8e99875dcace068aae6e9edf 172.16.229.80:6380@16380 master - 0 1628573989000 2 connected 5461-10922
81648ae1400ccd7ecc3013a067ecda1887e18339 172.16.229.80:6389@16389 slave 83efced12fa56cba8e99875dcace068aae6e9edf 0 1628573988097 2 connected
8d66ff91896f261a16ee90bae06f1fb04ae6c4e7 172.16.229.80:6381@16381 master - 0 1628573990104 3 connected 10923-16383
37c5f14250c4138630e002ec32585e5749b56733 172.16.229.80:6390@16390 slave 8d66ff91896f261a16ee90bae06f1fb04ae6c4e7 0 1628573989100 3 connected
80c883e07d10310e3d41a525e5c4462191e13f2e 172.16.229.80:6391@16391 slave 0a669d2b580e7def8354ce24faa735864d3012ef 0 1628573991108 1 connected
0a669d2b580e7def8354ce24faa735864d3012ef 172.16.229.80:6379@16379 myself,master - 0 1628573989000 1 connected 0-5460
```



**redis	cluster如何分配这六个节点?**

一个集群至少要有三个主节点。

选项 **--cluster-replicas 1** 表示我们希望为集群中的每个主节点创建一个从节点。

分配原则尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个IP地址上。



#### 什么是slots

[OK] All nodes agree about slots configuration.

\>>> Check for open slots...

\>>> Check slots coverage...

**[OK] All 16384 slots covered.**

一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个，

**集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键 key 的 CRC16 校验和 。**

**集群中的每个节点负责处理一部分插槽**。 举个例子， 如果一个集群可以有主节点， 其中：

节点 A 负责处理 0 号至 5460 号插槽。

节点 B 负责处理 5461 号至 10922 号插槽。

节点 C 负责处理 10923 号至 16383 号插槽。

```shell
127.0.0.1:6379> set k1 v1
-> Redirected to slot [12706] located at 172.16.229.80:6381
OK
172.16.229.80:6381> set k2 v2
-> Redirected to slot [449] located at 172.16.229.80:6379
OK
```



#### **在集群中录入值**

在redis-cli每次录入、查询键值，redis都会计算出该key应该送往的插槽，如果不是该客户端对应服务器的插槽，redis会报错，并告知应前往的redis实例地址和端口。

redis-cli客户端提供了 –c 参数实现自动重定向。

如 redis-cli  -c –p 6379 登入后，再录入、查询键值对可以自动重定向。

不在一个slot下的键值，是不能使用mget,mset等多键操作。

可以通过**{}来定义组**的概念，从而使**key中{}内相同内容的键值**对放到一个slot中去。

```shell
172.16.229.80:6379> mset name luch age 20 address china
(error) CROSSSLOT Keys in request don't hash to the same slot

172.16.229.80:6379> mset name{user} luch age{user} 20 address{user} china
-> Redirected to slot [5474] located at 172.16.229.80:6380
OK
```



**查询集群中的值**

```shell
172.16.229.80:6380> cluster keyslot user
(integer) 5474
172.16.229.80:6380> cluster countkeysinslot 5474
(integer) 3
172.16.229.80:6380> cluster getkeysinslot 5474 10
1) "address{user}"
2) "age{user}"
3) "name{user}"
```



**故障恢复**

如果主节点下线？从节点能否自动升为主节点？注意：15秒超时

![rediss4](http://qiliu.luxiaobai.cn/img/rediss4.png)

主节点恢复后，主从关系会如何？主节点回来变成从机。

![rediss5](http://qiliu.luxiaobai.cn/img/rediss5.png)

如果所有某一段插槽的主从节点都宕掉，redis服务是否还能继续?

如果某一段插槽的主从都挂掉，而cluster-require-full-coverage 为yes ，那么 ，整个集群都挂掉

如果某一段插槽的主从都挂掉，而cluster-require-full-coverage 为no ，那么，该插槽数据全都不能使用，也无法存储。

redis.conf中的参数  cluster-require-full-coverage





# **集群的Jedis开发**

即使连接的不是主机，集群会自动切换主机存储。主机写，从机读。

**无中心化主从集群**。无论从哪台主机写的数据，其他主机上都能读到数据。

```shell
public class JedisClusterTest {
    public static void main(String[] args) {
        Set<HostAndPort> set = new HashSet<>();
        set.add(new HostAndPort("121.199.76.44", 6379));
//        HostAndPort hostAndPort = new HostAndPort("121.199.76.44",6379);
        JedisCluster jedisCluster = new JedisCluster(set);
        jedisCluster.set("k1","v1");
        System.out.println(jedisCluster.get("k1"));
        jedisCluster.close();
    }
}
```

### **Redis集群提供了以下好处**

- 实现扩容
- 分摊压力
- 无中心配置相对简单

### **Redis 集群的不足**

多键操作是不被支持的

多键的Redis事务是不被支持的。lua脚本不被支持

由于集群方案出现较晚，很多公司已经采用了其他的集群方案，而代理或者客户端分片的方案想要迁移至redis cluster，需要整体迁移而不是逐步过渡，复杂度较大。





# **Redis应用问题解决**

## **缓存穿透**

key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会压到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

![rediscache1](http://qiliu.luxiaobai.cn/img/rediscache1.png)

### **解决方案**

一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。

**解决方案：**

（1） **对空值缓存**：如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟

（2） **设置可访问的名单**（白名单）：

使用bitmaps类型定义一个可以访问的名单，名单id作为bitmaps的偏移量，每次访问和bitmap里面的id进行比较，如果访问id不在bitmaps里面，进行拦截，不允许访问。

（3） **采用布隆过滤器**：(布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量(位图)和一系列随机映射函数（哈希函数）。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。)

将所有可能存在的数据哈希到一个足够大的bitmaps中，一个一定不存在的数据会被 这个bitmaps拦截掉，从而避免了对底层存储系统的查询压力。

（4） **进行实时监控**：当发现Redis的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务



## **缓存击穿**

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

![rediscache2](http://qiliu.luxiaobai.cn/img/rediscache2.png)

![rediscache3](http://qiliu.luxiaobai.cn/img/rediscache3.png)

### **解决方案**

key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题。

**解决问题：**

（1）预先设置热门数据：在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长

（2）实时调整：现场监控哪些数据热门，实时调整key的过期时长

（3）使用锁：

- 1. 就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db。	
  2. 先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key
  3. 当操作返回成功时，再进行load db的操作，并回设缓存,最后删除mutex key；
  4. 当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再重试整个get缓存的方法。

<img src="http://qiliu.luxiaobai.cn/img/rediscache4.png" alt="rediscache4" style="zoom:50%;" />



## **缓存雪崩**

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

缓存雪崩与缓存击穿的区别在于这里针对很多key缓存，前者则是某一个key

 

**正常访问**

<img src="http://qiliu.luxiaobai.cn/img/rediscache6.png" alt="rediscache6" style="zoom:67%;" />

**缓存失效瞬间**

![rediscache7](http://qiliu.luxiaobai.cn/img/rediscache7.png)

### **解决方案**

缓存失效时的雪崩效应对底层系统的冲击非常可怕！

**解决方案：**

（1） **构建多级缓存架构**：nginx缓存 + redis缓存 +其他缓存（ehcache等）

（2） **使用锁或队列：**

用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况

（3） **设置过期标志更新缓存**：

记录缓存数据是否过期（设置提前量），如果过期会触发通知另外的线程在后台去更新实际key的缓存。

（4） **将缓存失效时间分散开**：

比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。





## **分布式锁**

随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的Java API并不能提供分布式锁的能力。为了解决这个问题就需要**一种跨JVM的互斥机制来控制共享资源的访问**，这就是分布式锁要解决的问题！

分布式锁主流的实现方案：

\1. 基于数据库实现分布式锁

\2. 基于缓存（Redis等）

\3. 基于Zookeeper

每一种分布式锁解决方案都有各自的优缺点：

\1. 性能：redis最高

\2. 可靠性：zookeeper最高

### **使用redis实现分布式锁**

redis:命令

\# set sku:1:info “OK” NX PX 10000

```shell
//上锁的同时,设置过期时间
set users 10 nx ex 12
EX second ：设置键的过期时间为 second 秒。 SET key value EX second 效果等同于 SETEX key second value 。
PX millisecond ：设置键的过期时间为 millisecond 毫秒。 SET key value PX millisecond 效果等同于 PSETEX key millisecond value 。
NX ：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value 。
XX ：只在键已经存在时，才对键进行设置操作。

1、使用setnx上锁,通过del释放锁
2、锁一直没有释放,设置锁的过期时间,自动释放
    setnx users 10
    expire users 20
```

<img src="http://qiliu.luxiaobai.cn/img/rediscache8.png" alt="rediscache8" style="zoom:67%;" />

1. 多个客户端同时获取锁（setnx）
2. 获取成功，执行业务逻辑{从db获取数据，放入缓存}，执行完成释放锁（del）
3. 其他客户端等待重试



### **优化之设置锁的过期时间**

设置过期时间有两种方式:

\1. 首先想到通过expire设置过期时间（缺乏原子性：如果在setnx和expire之间出现异常，锁也无法释放）

\2. 在set时指定过期时间（推荐）

<img src="http://qiliu.luxiaobai.cn/img/rediscache9.png" alt="rediscache9" style="zoom:67%;" />

### **优化之UUID防误删**

<img src="http://qiliu.luxiaobai.cn/img/rediscache10.png" alt="rediscache10" style="zoom:67%;" />

<img src="http://qiliu.luxiaobai.cn/img/rediscache11.png" alt="rediscache11" style="zoom: 50%;" />

问题：删除操作缺乏原子性。

场景：

1. index1执行删除时，查询到的lock值确实和uuid相等

```shell
uuid=v1
set(lock,uuid)；
```

```java
if(uuid.equals(redisTemplate.opsForValue().get("lock"))){}
```



2. index1执行删除前，lock刚好过期时间已到，被redis自动释放

在redis中没有了lock，没有了锁。

```shell
this.redisTemplate.delete("lock");
```

3. index2获取了lock

index2线程获取到了cpu的资源，开始执行方法

```shell
uuid=v2
set(lock,uuid);
```

4. index1执行删除，此时会把index2的lock删除

index1 因为已经在方法中了，所以不需要重新上锁。index1有执行的权限。index1已经比较完成了，这个时候，开始执行

```java
this.redisTemplate.delete("lock");
```

删除的index2的锁！





## **优化之LUA脚本保证删除的原子性**

```java
@GetMapping("testLockLua")
public void testLockLua() {
    //1 声明一个uuid ,将做为一个value 放入我们的key所对应的值中
    String uuid = UUID.randomUUID().toString();
    //2 定义一个锁：lua 脚本可以使用同一把锁，来实现删除！
    String skuId = "25"; // 访问skuId 为25号的商品 100008348542
    String locKey = "lock:" + skuId; // 锁住的是每个商品的数据

    // 3 获取锁
    Boolean lock = redisTemplate.opsForValue().setIfAbsent(locKey, uuid, 3, TimeUnit.SECONDS);

    // 第一种： lock 与过期时间中间不写任何的代码。
    // redisTemplate.expire("lock",10, TimeUnit.SECONDS);//设置过期时间
    // 如果true
    if (lock) {
        // 执行的业务逻辑开始
        // 获取缓存中的num 数据
        Object value = redisTemplate.opsForValue().get("num");
        // 如果是空直接返回
        if (StringUtils.isEmpty(value)) {
            return;
        }
        // 不是空 如果说在这出现了异常！ 那么delete 就删除失败！ 也就是说锁永远存在！
        int num = Integer.parseInt(value + "");
        // 使num 每次+1 放入缓存
        redisTemplate.opsForValue().set("num", String.valueOf(++num));
        /*使用lua脚本来锁*/
        // 定义lua 脚本
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        // 使用redis执行lua执行
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
        redisScript.setScriptText(script);
        // 设置一下返回值类型 为Long
        // 因为删除判断的时候，返回的0,给其封装为数据类型。如果不封装那么默认返回String 类型，
        // 那么返回字符串与0 会有发生错误。
        redisScript.setResultType(Long.class);
        // 第一个要是script 脚本 ，第二个需要判断的key，第三个就是key所对应的值。
        redisTemplate.execute(redisScript, Arrays.asList(locKey), uuid);
    } else {
        // 其他线程等待
        try {
            // 睡眠
            Thread.sleep(1000);
            // 睡醒了之后，调用方法。
            testLockLua();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```





## **Lua脚本详解**

![lua3](http://qiliu.luxiaobai.cn/img/lua3.png)

项目中正确使用：

```shelll
1. 定义key，key应该是为每个sku定义的，也就是每个sku有一把锁。
String locKey ="lock:"+skuId; // 锁住的是每个商品的数据
Boolean lock = redisTemplate.opsForValue().setIfAbsent(locKey, uuid,3,TimeUnit.SECONDS);
```

![lua4](http://qiliu.luxiaobai.cn/img/lua4.png)



### **总结**

**1、加锁**

```java
// 1. 从redis中获取锁,set k1 v1 px 20000 nx
String uuid = UUID.randomUUID().toString();
Boolean lock = this.redisTemplate.opsForValue()
      .setIfAbsent("lock", uuid, 2, TimeUnit.SECONDS);
```



**2、使用lua释放锁**

```java
// 2. 释放锁 del
String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
// 设置lua脚本返回的数据类型
DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
// 设置lua脚本返回类型为Long
redisScript.setResultType(Long.class);
redisScript.setScriptText(script);
redisTemplate.execute(redisScript, Arrays.asList("lock"),uuid);
```



**3、重试**

```java
Thread.sleep(500);
testLock();
```

- 为了确保分布式锁可用，我们至少要确保锁的实现同时**满足以下四个条件**：
- 互斥性。在任意时刻，只有一个客户端能持有锁。
- 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
- 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。
- 加锁和解锁必须具有原子性。





# Redis6.0新功能

## **ACL**

Redis ACL是Access Control List（访问控制列表）的缩写，该功能允许根据可以执行的命令和可以访问的键来限制某些连接。

在Redis 5版本之前，Redis 安全规则只有密码控制 还有通过rename 来调整高危命令比如 flushdb ， KEYS* ， shutdown 等。Redis 6 则提供ACL的功能对用户进行更细粒度的权限控制 ：

（1）接入权限:用户名和密码

（2）可以执行的命令

（3）可以操作的 KEY

参考官网：https://redis.io/topics/acl

### **命令**

使用acl list命令展现用户权限列表

![Acl](http://qiliu.luxiaobai.cn/img/Acl.png)

使用acl cat命令

（1）查看添加权限指令类别

```shell
127.0.0.1:6379> acl cat
 1) "keyspace"
 2) "read"
 3) "write"
 4) "set"
 5) "sortedset"
 6) "list"
 7) "hash"
 8) "string"
 9) "bitmap"
10) "hyperloglog"
11) "geo"
12) "stream"
13) "pubsub"
14) "admin"
15) "fast"
16) "slow"
17) "blocking"
18) "dangerous"
19) "connection"
20) "transaction"
21) "scripting"
```



加参数类型名可以查看类型下具体命令

```shell
127.0.0.1:6379> acl cat string
 1) "psetex"
 2) "setex"
 3) "setrange"
 4) "substr"
 5) "set"
 6) "incr"
 7) "strlen"
 8) "getdel"
 9) "get"
10) "getset"
11) "mset"
12) "setnx"
13) "decrby"
14) "getex"
15) "mget"
16) "stralgo"
17) "append"
18) "msetnx"
19) "decr"
20) "incrbyfloat"
21) "incrby"
22) "getrange"
```



使用all whoami命令查看当前用户

```shell
127.0.0.1:6379> acl whoami
"default"
```

**使用acl setuser 命令创建和编辑用户ACL**

ACL规则

有效ACL规则的列表。某些规则只是用于激活或删除标志，或对用户ACL执行给定更改的单个单词。其他规则是字符前缀，它们与命令或类别名称、键模式等连接在一起。

![image-20211018162210997](http://qiliu.luxiaobai.cn/img/image-20211018162210997.png)

通过命令创建新用户默认权限

```shell
127.0.0.1:6379> acl setuser user1
OK
127.0.0.1:6379> acl list
1) "user default on nopass ~* &* +@all"
2) "user user1 off &* -@all"
```

如果用户不存在，这将使用just created的默认属性来创建用户。如果用户已经存在，则上面的命令将不执行任何操作。



设置有用户名、密码、ACL权限、并启用的用户

```shell
127.0.0.1:6379> acl setuser user2 on >password ~cached:* +get
OK 
127.0.0.1:6379> acl list
1) "user default on nopass ~* &* +@all"
2) "user user1 off &* -@all"
3) "user user2 on #5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8 ~cached:* &* -@all +get"
```



切换用户,验证权限

```shell
127.0.0.1:6379> acl whoami
"default"
127.0.0.1:6379> auth user2 password
OK
127.0.0.1:6379> acl whoami
(error) NOPERM this user has no permissions to run the 'acl' command or its subcommand
127.0.0.1:6379> get foo
(error) NOPERM this user has no permissions to access one of the keys used as arguments
127.0.0.1:6379> get cached:1121
(nil)
127.0.0.1:6379> set cached:1121 1121
(error) NOPERM this user has no permissions to run the 'set' command or its subcommand
```



## IO多线程

Redis6终于支持多线程了，告别单线程了吗？

IO多线程其实指客户端交互部分的网络IO交互处理模块多线程，而非执行命令多线程。Redis6执行命令依然是单线程



### **原理架构**

Redis 6 加入多线程,但跟 Memcached 这种从 IO处理到数据访问多线程的实现模式有些差异。Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。之所以这么设计是不想因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP 等等的并发问题。整体的设计大体如下:

<img src="http://qiliu.luxiaobai.cn/img/acl4.png" alt="acl4" style="zoom: 67%;" />

另外，多线程IO默认也是不开启的，需要再配置文件中配置

```properties
io-threads-do-reads  yes
io-threads 4
```



**工具支持Cluster**

之前老版Redis想要搭集群需要单独安装ruby环境，Redis 5 将 redis-trib.rb 的功能集成到 redis-cli 。另外官方 redis-benchmark 工具开始支持 cluster 模式了，通过多线程的方式对多个分片进行压测。

```shell
root@iZbp13941xpzjmefjge9chZ:/var/lib/redis# redis-benchmark --help
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests>] [-k <boolean>]

 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 --user <username>  Used to send ACL style 'AUTH username pass'. Needs -a.
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 3)
 --dbnum <db>       SELECT the specified db number (default 0)
 --threads <num>    Enable multi-thread mode.
 --cluster          Enable cluster mode.
 --enable-tracking  Send CLIENT TRACKING on before starting benchmark.
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD,
                    random members and scores for ZADD.
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -q                 Quiet. Just show query/sec values
 --precision        Number of decimal places to display in latency output (default 0)
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
 --tls              Establish a secure TLS connection.
 --sni <host>       Server name indication for TLS.
 --cacert <file>    CA Certificate file to verify with.
 --cacertdir <dir>  Directory where trusted CA certificates are stored.
                    If neither cacert nor cacertdir are specified, the default
                    system-wide trusted root certs configuration will apply.
 --insecure         Allow insecure TLS connection by skipping cert validation.
 --cert <file>      Client certificate to authenticate with.
 --key <file>       Private key file to authenticate with.
 --tls-ciphers <list> Sets the list of prefered ciphers (TLSv1.2 and below)
                    in order of preference from highest to lowest separated by colon (":").
                    See the ciphers(1ssl) manpage for more information about the syntax of this string.
 --help             Output this help and exit.
 --version          Output version and exit.

Examples:

 Run the benchmark with the default configuration against 127.0.0.1:6379:
   $ redis-benchmark

 Use 20 parallel clients, for a total of 100k requests, against 192.168.1.1:
   $ redis-benchmark -h 192.168.1.1 -p 6379 -n 100000 -c 20

 Fill 127.0.0.1:6379 with about 1 million keys only using the SET test:
   $ redis-benchmark -t set -n 1000000 -r 100000000

 Benchmark 127.0.0.1:6379 for a few commands producing CSV output:
   $ redis-benchmark -t ping,set,get -n 100000 --csv

 Benchmark a specific command line:
   $ redis-benchmark -r 10000 -n 10000 eval 'return redis.call("ping")' 0

 Fill a list with 10000 random elements:
   $ redis-benchmark -r 10000 -n 10000 lpush mylist __rand_int__

 On user specified command lines __rand_int__ is replaced with a random integer
 with a range of values selected by the -r option.
```



>**Redis新功能**
>
>1、RESP3新的 Redis 通信协议：优化服务端与客户端之间通信
>
>2、Client side caching客户端缓存：基于 RESP3 协议实现的客户端缓存功能。为了进一步提升缓存的性能，将客户端经常访问的数据cache到客户端。减少TCP网络交互。
>
>3、Proxy集群代理模式：Proxy 功能，让 Cluster 拥有像单实例一样的接入方式，降低大家使用cluster的门槛。不过需要注意的是代理不改变 Cluster 的功能限制，不支持的命令还是不会支持，比如跨 slot 的多Key操作。
>
>4、Modules API
>
>Redis 6中模块API开发进展非常大，因为Redis Labs为了开发复杂的功能，从一开始就用上Redis模块。Redis可以变成一个框架，利用Modules来构建不同系统，而不需要从头开始写然后还要BSD许可。Redis一开始就是一个向编写各种系统开放的平台。

