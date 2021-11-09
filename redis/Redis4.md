---
title: Redis6入门学习(四)
---

[toc]



# Redis持久化



## RDB(Redis DataBase)

在指定的**时间间隔**内将内存中的数据集**快照**写入磁盘， 也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里



## 备份是如何执行的

Redis会单独创建**（fork**）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。

RDB的缺点是**最后一次持久化后的数据可能丢失。**



## Fork

- Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程
- 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了“写时复制技术”
- 一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。



## RDB持久化流程

![rdb1](http://qiliu.luxiaobai.cn/img/rdb1.png)

### dump.rdb文件

在redis.conf中配置文件名称，默认为dump.rdb



### 配置位置

rdb文件的保存路径，也可以修改。默认为Redis启动时命令行所在的目录下

```shell
dir "/myredis/"
```

![rdb2](http://qiliu.luxiaobai.cn/img/rdb2.png)



**如何触发RDB快照；保持策略**

**配置文件中默认的快照配置**

![rdb4](http://qiliu.luxiaobai.cn/img/rdb4.png)





### **命令save VS bgsave**

**save** ：save时只管保存，其它不管，全部阻塞。手动保存。不建议。

**bgsave**：Redis会在后台异步进行快照操作， 快照同时还可以响应客户端请求。

可以通过lastsave 命令获取最后一次成功执行快照的时间

**flushall命令**

执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义

**###SNAPSHOTTING快照###**

**Save**

格式：save 秒钟 写操作次数

RDB是整个内存的压缩过的Snapshot，RDB的数据结构，可以配置复合的快照触发条件，

默认是1分钟内改了1万次，或5分钟内改了10次，或15分钟内改了1次。

禁用

不设置save指令，或者给save传入空字符串

**stop-writes-on-bgsave-error**

![rdb3](http://qiliu.luxiaobai.cn/img/rdb3.png)

当Redis无法写入磁盘的话，直接关掉Redis的写操作。推荐yes.

**rdbcompression 压缩文件**

![rdb5](http://qiliu.luxiaobai.cn/img/rdb5.png)

对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。

如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。推荐yes.



**rdbchecksum 检查完整性**

![rdb6](http://qiliu.luxiaobai.cn/img/rdb6.png)

在存储快照后，还可以让redis使用CRC64算法来进行数据校验，

但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能

推荐yes.



### rdb的备份

先通过config get dir  查询rdb文件的目录

将*.rdb的文件拷贝到别的地方

**rdb的恢复**

- 关闭Redis
- 先把备份的文件拷贝到工作目录下 cp dump2.rdb dump.rdb
- 启动Redis, 备份数据会直接加载

**优势**

-  适合大规模的数据恢复
-  对数据完整性和一致性要求不高更适合使用
-  节省磁盘空间
-  恢复速度快

![rdb7](http://qiliu.luxiaobai.cn/img/rdb7.png)

**劣势**

- Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑
- 虽然Redis在fork时使用了写时拷贝技术,但是如果数据庞大时还是比较消耗性能。
- 在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

**如何停止**

动态停止RDB：redis-cli config set save ""#save后给空值，表示禁用保存策略

**总结**

![rdb8](http://qiliu.luxiaobai.cn/img/rdb8.png)



## **Redis持久化之AOF**

**AOF（Append Only File）**

以日志的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(读操作不记录)， 只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

### **AOF持久化流程**

1. 客户端的请求写命令会被append追加到AOF缓冲区内；
2. AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；
3. AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量；
4. Redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的；

<img src="http://qiliu.luxiaobai.cn/img/rdb9.png" alt="rdb9" style="zoom: 67%;" />

### **AOF默认不开启**

可以在redis.conf中配置文件名称，默认为 appendonly.aof

AOF文件的保存路径，同RDB的路径一致。

![rdb10](http://qiliu.luxiaobai.cn/img/rdb10.png)



### **AOF和RDB同时开启，redis听谁的？**

AOF和RDB同时开启，系统默认取AOF的数据（数据不会存在丢失）

### **AOF启动/修复/恢复**

- AOF的备份机制和性能虽然和RDB不同, 但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载。
- 正常恢复

- - 修改默认的appendonly no，改为yes
  - 将有数据的aof文件复制一份保存到对应目录(查看目录：config get dir)
  - 恢复：重启redis然后重新加载

- 异常恢复

- - 修改默认的appendonly no，改为yes
  - 如遇到AOF文件损坏，通过/usr/local/bin/redis-check-aof --fix appendonly.aof进行恢复
  - 备份被写坏的AOF文件
  - 恢复：重启redis，然后重新加载

### **AOF同步频率设置**

```shell
appendfsync always
```

始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好



```shell
appendfsync everysec
```

每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。



```shell
appendfsync no
```

redis不主动进行同步,把不同时机交给操作系统



## Rewrite压缩

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制, 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof



### **重写原理，如何实现重写**

AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，**redis4.0版本后的重写，是指上就是把rdb 的快照，以二级制的形式附在新的aof头部，作为已有的历史数据，替换掉原来的流水账操作。**

**no-appendfsync-on-rewrite：**

- 如果 no-appendfsync-on-rewrite=yes ,不写入aof文件只写入缓存，用户请求不会阻塞，但是在这段时间如果宕机会丢失这段时间的缓存数据。（降低数据安全性，提高性能）
- 如果 no-appendfsync-on-rewrite=no,  还是会把数据往磁盘里刷，但是遇到重写操作，可能会发生阻塞。（数据安全，但是性能降低）



### **触发机制，何时重写**

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。

**auto-aof-rewrite-percentage**：设置重写的基准值，文件达到100%时开始重写（文件是原来重写后文件的2倍时触发）

**auto-aof-rewrite-min-size**：设置重写的基准值，最小文件64MB。达到这个值开始重写。

例如：文件达到70MB开始重写，降到50MB，下次什么时候开始重写？100MB

系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size,

如果Redis的**AOF当前大小>= base_size +base_size\*100% (默认)且当前大小>=64mb(默认)**的情况下，Redis会对AOF进行重写。



### **重写流程**

1. bgrewriteaof触发重写，判断是否当前有bgsave或bgrewriteaof在运行，如果有，则等待该命令结束后再继续执行。
2. 主进程fork出子进程执行重写操作，保证主进程不会阻塞。
3. 子进程遍历redis内存中数据到临时文件，客户端的写请求同时写入aof_buf缓冲区和aof_rewrite_buf重写缓冲区保证原AOF文件完整以及新AOF文件生成期间的新的数据修改动作不会丢失。
4. 1).子进程写完新的AOF文件后，向主进程发信号，父进程更新统计信息。2).主进程把aof_rewrite_buf中的数据写入到新的AOF文件。
5. 使用新的AOF文件覆盖旧的AOF文件，完成AOF重写。

![rdb11](http://qiliu.luxiaobai.cn/img/rdb11.png)



### 优势

![rdb12](http://qiliu.luxiaobai.cn/img/rdb12.png)

- 备份机制更稳健，丢失数据概率更低。
- 可读的日志文本，通过操作AOF稳健，可以处理误操作。



### 劣势

- 比起RDB占用更多的磁盘空间。
- 恢复备份速度要慢。
- 每次读写都同步的话，有一定的性能压力。
- 存在个别Bug，造成恢复不能。

![rdb13](http://qiliu.luxiaobai.cn/img/rdb13.png)





### **RDB和AOF用哪个好**

官方推荐两个都启用。

如果对数据不敏感，可以选单独用RDB。

不建议单独用 AOF，因为可能会出现Bug。

如果只是做纯内存缓存，可以都不用。

#### **官网建议**

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储
- AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾.
- Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大
- 只做缓存：如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.
- 同时开启两种持久化方式
- 在这种情况下,当redis重启的时候会优先载入AOF文件来恢复原始的数据, 因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.
- RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件。那要不要只使用AOF呢？
- 建议不要，因为RDB更适合用于备份数据库(AOF在不断变化不好备份)， 快速重启，而且不会有AOF可能潜在的bug，留着作为一个万一的手段。
- 性能建议





> 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。 如果使用AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价,一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。





# Redis主从复制

主机数据更新后根据配置和策略， 自动同步到备机的**master/slaver**机制，**Master以写为主，Slave以读为主**

- 读写分离，性能扩展
- 容灾快速恢复

![redishost](http://qiliu.luxiaobai.cn/img/redishost.png)

==**主从复制只能一个主多个从**==

**一主三从配置**

1. 创建myredis文件夹

2. 复制redis.conf配置文件到文件夹

3. 配置一主三从,插件三个配置文件

4. 1. redis6379.conf
   2. redis6380.conf
   3. redis6381

5. 在三个配置文件写入内容

```shell
include /home/lushengyang/myredis/redis.conf
pidfile /run/redis_6379.pid
port 6379
dbfilename dump6379.rdb
```

==slave-priority 10==

设置从机的优先级，值越小，优先级越高，用于选举主机时使用。默认100

```shell
mkdir myredis
cd myredis/
cp /etc/redis.conf ./
vi redis6379.conf
include /home/lushengyang/myredis/redis.conf
pidfile /run/redis_6379.pid
port 6379
dbfilename dump6379.rdb
cp redis6379.conf redis6380.conf  ##修改内容

```



启动redis服务器

```shell
redis-server redis6379.conf 
redis-server redis6380.conf 
redis-server redis6381.conf 
```



查看三个redis主从服务情况

```shell
redis-cli -p 6379
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:ba96bc2f5e2bd9cb50834f9d54872b9fda8a887b
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```



### 配从不配主

```shell
在6380和6381上执行:
slaveof 127.0.0.1 6379 
```

![redishost2](http://qiliu.luxiaobai.cn/img/redishost2.png)

```shell
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6380> get k1
"v1"
127.0.0.1:6381> get k1
"v1"
127.0.0.1:6382> get k1
"v1"
127.0.0.1:6382> set k2 v2
(error) READONLY You can't write against a read only replica.
```



## **一主三从**

slave1、slave2是从头开始复制还是从切入点开始复制?比如从k4进来，那之前的k1,k2,k3是否也可以复制？

**从机宕机后重新启动默认为主机,需要重新配置主机IP和端口,从头开始复制**

从机是否可以写？set可否？**不可以**

主机shutdown后情况如何？从机是上位还是原地待命？**原地待命,主机重新上线,还是作为主机,**

主机又回来了后，主机新增记录，从机还能否顺利复制？ **能**

其中一台从机down后情况如何？依照原有它能跟上大部队吗？**能**



## **薪火相传**

上一个Slave可以是下一个slave的Master，Slave同样可以接收其他 slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力,去中心化降低风险。

用 **slaveof  <ip><port>**

**中途变更转向**:会清除之前的数据，重新建立拷贝最新的

风险是一旦某个slave宕机，后面的slave都没法备份

主机挂了，从机还是从机，无法写数据了



![redishost3](http://qiliu.luxiaobai.cn/img/redishost3.png)

![redishost4](http://qiliu.luxiaobai.cn/img/redishost4.png)



## **反客为主**

当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。

用 **slaveof  no one**  将从机变为主机。

## **复制原理**

- Slave启动成功连接到master后会发送一个sync命令
- Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
- 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
- 但是只要是重新连接master,一次完全同步（全量复制)将被自动执行

![redishost6](http://qiliu.luxiaobai.cn/img/redishost6.png)

## **哨兵模式(sentinel)**

**反客为主的自动版**，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

![redishost5](http://qiliu.luxiaobai.cn/img/redishost5.png)

```shell
vi sentinel.conf  //文件名不能错
sentinel monitor mymaster 127.0.0.1 6379 1 //其中mymaster为监控对象起的服务器名称， 1 为至少有多少个哨兵同意迁移的数量。
daemonize yes    //后台运行哨兵模式
sudo apt-get install redis-sentinel
redis-sentinel sentinel.conf
```

![redishost7](http://qiliu.luxiaobai.cn/img/redishost7.png)



**当主机挂掉，从机选举中产生新的主机**

(大概10秒左右可以看到哨兵窗口日志，切换了新的主机)哪个从机会被选举为主机呢？

根据优先级别：**slave-priority**

原主机重启后会变为从机。

![redishost8](/Users/lushengyang/Desktop/LSY/StudeyNotes/redis/Redis4.assets/redishost8.png)

### **复制延时**

由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。

### **故障恢复**

![redishost9](http://qiliu.luxiaobai.cn/img/redishost9.png)

优先级在redis.conf中默认：slave-priority 100，值越小优先级越高

偏移量是指获得原主机数据最全的

每个redis实例启动后都会随机生成一个40位的runid

主从复制

```shell
private static JedisSentinelPool jedisSentinelPool = null;
public static  Jedis getJedisFromSentinel(){
    if(jedisSentinelPool==null){
        Set<String> sentinelSet=new HashSet<>();
        sentinelSet.add("121.199.76.44:26379");

        JedisPoolConfig jedisPoolConfig =new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(10); //最大可用连接数
        jedisPoolConfig.setMaxIdle(5); //最大闲置连接数
        jedisPoolConfig.setMinIdle(5); //最小闲置连接数
        jedisPoolConfig.setBlockWhenExhausted(true); //连接耗尽是否等待
        jedisPoolConfig.setMaxWaitMillis(2000); //等待时间
        jedisPoolConfig.setTestOnBorrow(true); //取连接的时候进行一下测试 ping pong

        jedisSentinelPool=new JedisSentinelPool("mymaster",sentinelSet,jedisPoolConfig);
        return jedisSentinelPool.getResource();
    }else{
        return jedisSentinelPool.getResource();
    }
}
```

