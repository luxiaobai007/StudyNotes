---
Zookeeper从入门到精通
---

[toc]

-----

# Zookeeper概述

Zookeeper 是一个开源的分布式的，为分布式框架提供协调服务的 Apache 项目。

<img src="http://qiliu.luxiaobai.cn/img/image-20220329174706743.png" alt="image-20220329174706743" style="zoom:67%;" />

## 工作机制

> Zookeeper从设计模式角度来理解：是一个基于==观察者模式==设计的分布式服务管理框架，它**负责存储和管理**大家都关心的数据，然后接受观察者的注 册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

<img src="http://qiliu.luxiaobai.cn/img/image-20220329175142941.png" alt="image-20220329175142941" style="zoom:50%;" />



## 特点

<img src="http://qiliu.luxiaobai.cn/img/image-20220329175802736.png" alt="image-20220329175802736" style="zoom:50%;" />

1）Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。 

2）集群中只要有**半数以上**节点存活，Zookeeper集群就能正常服务。所 以Zookeeper适合安装奇数台服务器。 

3）全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。 

4）更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行。 

5）数据更新原子性，一次数据更新要么成功，要么失败。 

6）实时性，在一定时间范围内，Client能读到最新数据。



## 数据结构

ZooKeeper 数据模型的结构与 Unix 文件系统很类似，整体上可以看作是一棵树，每个节点称做一个 ZNode。每一个 ZNode 默认能够存储 1MB 的数据，每个 ZNode 都可以通过其路径唯一标识。

<img src="http://qiliu.luxiaobai.cn/img/image-20220329175940991.png" alt="image-20220329175940991" style="zoom:50%;" />



## 应用场景

提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等。

### 统一命名服务

在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。例如：IP不容易记住，而域名容易记住。

<img src="http://qiliu.luxiaobai.cn/img/image-20220329180127173.png" alt="image-20220329180127173" style="zoom:50%;" />

### 统一配置管理

1. 分布式环境下,配置文件同步非常常见
   1. 一般要求一个集群中,所有节点的配置信息是一致的,比如Kafka集群
   2. 对配置文件修改后,希望能快速同步到各个节点上
2. 配置管理可交由Zookeeper实现
   1. 可将配置信息写入ZooKeeper上的一个Znode
   2. 各个客户端服务器监听这个Znode
   3. 一旦Znode中的数据被修改,Zookeeper将通知各个客户端服务器

<img src="http://qiliu.luxiaobai.cn/img/image-20220330081949248.png" alt="image-20220330081949248" style="zoom:50%;" />

### 统一集群管理

1. 分布式环境中,实时掌握每个节点的状态是必要的
   1. 可根据节点状态做出一些调整
2. Zookeeper可以实现实时监控节点状态变化
   1. 可将节点信息写入Zookeeper上的一个ZNode
   2. 监听这个ZNode可获取它的实时状态变化

<img src="http://qiliu.luxiaobai.cn/img/image-20220330082523353.png" alt="image-20220330082523353" style="zoom:50%;" />

### 服务器动态上下线

客户端能实时洞察到服务器上下线的变化

<img src="http://qiliu.luxiaobai.cn/img/image-20220330082724570.png" alt="image-20220330082724570" style="zoom:50%;" />

### 软负载均衡

在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求

<img src="http://qiliu.luxiaobai.cn/img/image-20220330082935308.png" alt="image-20220330082935308" style="zoom:50%;" />



## 安装部署

[下载](https://archive.apache.org/dist/zookeeper/zookeeper-3.5.7/apache-zookeeper-3.5.7-bin.tar.gz)

```shell
mv apache-zookeeper-3.5.7-bin/ zookeeper-3.5.7
cd zookeeper-3.5.7/conf
mv zoo_sample.cfg zoo.cfg
vim zoo.cfg   dataDir=/opt/module/zookeeper-3.5.7/zkData
 mkdir zkData
```



### 操作Zookeeper

```shell
##启动
bin/zkServer.sh start
###查看进程是否启动
jps
###查看状态
 bin/zkServer.sh status
 ###启动客户端
 bin/zkCli.sh
 ###退出客户端
 quit
#### 停止
bin/zkServer.sh stop
```



## 配置参数解读

==zoo.cfg==

- **tickTime=2000**:**通信心跳时间，Zookeeper服务器与客户端心跳时间，单位毫秒**

<img src="http://qiliu.luxiaobai.cn/img/image-20220330084519860.png" alt="image-20220330084519860" style="zoom:50%;" />

- **initLimit = 10:LF初始通信时限** Leader和Follower初始连接时能容忍的最多心跳数（tickTime的数量）

  <img src="http://qiliu.luxiaobai.cn/img/image-20220330084556021.png" alt="image-20220330084556021" style="zoom:50%;" />

- **syncLimit = 5：LF同步通信时限** Leader和Follower之间通信时间如果超过syncLimit * tickTime，Leader认为Follwer死亡,从服务器列表中删除Follwer

- **dataDir: 保存Zookeeper中的数据** 注意：默认的tmp目录，容易被Linux系统定期删除，所以一般不用默认的tmp目录

- clientPort = 2181: 客户端连接端口,通常不做修改



# Zookeeper集群操作

##  集群安装

```shell
 tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz
 mv apache-zookeeper-3.5.7-bin/ zookeeper-3.5.7
 ###配置服务器编号
 mkdir zookeeper-3.5.7/zkData
 vim zookeeper-3.5.7/zkData/myid ##在文件中添加与 server 对应的编号（注意：上下不要有空行，左右不要有空格）
## 拷贝配置好的 zookeeper 到其他机器上
xsync zookeeper-3.5.7

```

### 配置zoo.cfg文件

```shell
mv zoo_sample.cfg zoo.cfg
vim zoo.cfg
#修改数据存储路径配置
dataDir=/opt/module/zookeeper-3.5.7/zkData
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888

###同步 zoo.cfg 配置文件
xsync zoo.cfg
```

配置参数解读`server.A=B:C:D`

- **A** 是一个数字，表示这个是第几号服务器；

  集群模式下配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面有一个数据就是 A 的值，Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是哪个 server。 

- **B** 是这个服务器的地址；

- **C** 是这个服务器 Follower 与集群中的 Leader 服务器交换信息的端口；

- **D** 是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。





## 选举机制(面试重点)

### Zookeeper选举机制------第一次启动

![image-20220408144321567](http://qiliu.luxiaobai.cn/img/image-20220408144321567.png)

**SID**：**服务器ID**。用来唯一标识一台ZooKeeper集群中的机器，每台机器不能重复，和**myid一致**。

**ZXID**：**事务ID。**ZXID是一个事务ID，用来标识一次服务器状态的变更。在某一时刻，集群中的每台机器的ZXID值不一定完全一致，这和ZooKeeper服务器对于客户端“更新请求”的处理逻辑有关。

**Epoch**：每个Leader任期的代号。没有Leader时同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加



（1）服务器1启 动，发起一次选举。服务器1投自己一票。此时服务器1票数一票，不够半数以上（3票），选举无法完成，服务器1状态保持为LOOKING； 

（2）服务器2启动，再发起一次选举。服务器1和2分别投自己一票并交换选票信息：此时**服务器1发现服务器2的myid比自己目前投票推举的（服务器1） 大**，更改选票为推举服务器2。此时服务器1票数0票，服务器2票数2票，没有半数以上结果，选举无法完成，服务器1，2状态保持LOOKING

（3）服务器3启动，发起一次选举。此时服务器1和2都会更改选票为服务器3。此次投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数，服务器3当选Leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING；

（4）服务器4启动，发起一次选举。此时服务器1，2，3已经不是LOOKING状态，不会更改选票信息。交换选票信息结果：服务器3为3票，服务器4为 1票。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为FOLLOWING； 

（5）服务器5启动，同4一样当小弟。



### Zookeeper选举机制------非第一次启动

<img src="http://qiliu.luxiaobai.cn/img/image-20220330090829951.png" alt="image-20220330090829951" style="zoom:50%;" />

1. 当Zookeeper集群中的一台服务器出现以下两种情况之一时,就会开始进入Leader选举:

   - 服务器初始化启动
   - 服务器运行期间无法和Leader保持连接

2. 而当一台机器进入Leader选举流程时,当前集群也可能处于以下两种状态:

   - 集群中本来就已经存在一个Leader。对于第一种已经存在Leader的情况，机器试图去选举Leader时，会被告知当前服务器的Leader信息，对于该机器来说，仅仅需要和Leader机器建立连接，并进行状态同步即可。

   - **集群中确实不存在Leader**

     假设ZooKeeper由5台服务器组成，SID分别为1、2、3、4、5，ZXID分别为8、8、8、7、7，并且此时SID为3的服务器是Leader。某一时刻，3和5服务器出现故障，因此开始进行Leader选举。

     ​											（EPOCH，ZXID，SID ）（EPOCH，ZXID，SID ）（EPOCH，ZXID，SID ）

     SID为1、2、4的机器投票情况： （1，8，1） 				（1，8，2） 							（1，7，4） 

==**选举Leader规则：** ①EPOCH大的直接胜出 ②EPOCH相同，事务id大的胜出 ③事务id相同，服务器id大的胜出==





## ZK集群启动停止脚步

`vim /home/luxiaobai/bin/zk.sh`

```shell
#!/bin/bash
case $1 in
"start"){
for i in hadoop102 hadoop103 hadoop104
do
echo ---------- zookeeper $i 启动 ------------
ssh $i "/home/luxiaobai/zookeeper-3.5.7/bin/zkServer.sh 
start"
done
};;
"stop"){
for i in hadoop102 hadoop103 hadoop104
do
echo ---------- zookeeper $i 停止 ------------
ssh $i "/home/luxiaobai/zookeeper-3.5.7/bin/zkServer.sh 
stop"
done
};;
"status"){
for i in hadoop102 hadoop103 hadoop104
do
echo ---------- zookeeper $i 状态 ------------
ssh $i "/home/luxiaobai/zookeeper-3.5.7/bin/zkServer.sh 
status"
done
};;
esac
```

```shell
###增加脚本执行权限
 chmod u+x zk.sh
 ####Zookeeper 集群启动脚本
 zk.sh start
 ##Zookeeper 集群停止脚本
  zk.sh stop
```



## 客户端命令行操作

- ls path:使用 ls 命令来查看当前 znode 的子节点 [可监听] 
  - -w 监听子节点变化
  - -s 附加次级信息
- create: 普通创建
  - -s 含有序列
  - -e 临时(重启或者超时消失)
- get path: 获得节点的值[可监听]
  - -w 监听节点内容变化
  - -s 附加次级信息
- set: 设置节点的具体值
- stat: 查看节点状态
- delete: 删除节点
- deleteall: 递归删除节点



```shell
[zk: localhost:2181(CONNECTED) 2] ls -s /
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x17
cversion = 7
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 9

```

- ==czxid：创建节点的事务 zxid==

每次修改 ZooKeeper 状态都会产生一个 ZooKeeper 事务 ID。事务 ID 是 ZooKeeper 中所有修改总的次序。每次修改都有唯一的 zxid，如果 zxid1 小于 zxid2，那么 zxid1 在 zxid2 之前发生。

- ctime：znode 被创建的毫秒数（从 1970 年开始）
- mzxid：znode 最后更新的事务 zxid
- mtime：znode 最后修改的毫秒数（从 1970 年开始）
- pZxid：znode 最后更新的子节点 zxid
- cversion：znode 子节点变化号，znode 子节点修改次数
- ==dataversion：znode 数据变化号==
- aclVersion：znode 访问控制列表的变化号
- ephemeralOwner：如果是临时节点，这个是 znode 拥有者的 session id。如果不是临时节点则是 0。 
- ==dataLength：znode 的数据长度==
- ==numChildren：znode 子节点数量==



## 节点类型(持久/短暂/有序号/无序号)

![image-20220330093439527](http://qiliu.luxiaobai.cn/img/image-20220330093439527.png)



（1）持久化目录节点:客户端与Zookeeper断开连接后，该节点依旧存在

（2）持久化顺序编号目录节点:客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号

（3）临时目录节点:客户端与Zookeeper断开连接后，该节点被删除

（4）临时顺序编号目录节点:客户端与 Zookeeper 断开连接后 ， 该 节 点 被 删 除 ， 只 是Zookeeper给该节点名称进行顺序编号。



```shell
##分别创建2个普通节点（永久节点 + 不带序号）
create /luxiaobai "diaochan"
create /luxiaobai/shuguo "liubei"
##获得节点的值
get -s /luxiaobai
get -s /luxiaobai/shuguo

###创建带序号的节点（永久节点 + 带序号）
#创建一个普通的根节点/luxiaobai/weiguo
create /luxiaobai/weiguo "caocao"

#创建带序号的节点(永久节点+带序号)
[zk: localhost:2181(CONNECTED) 4] create -s /luxiaobai/weiguo/zhangliao "zhangliao"
Created /luxiaobai/weiguo/zhangliao0000000000
[zk: localhost:2181(CONNECTED) 5] create -s /luxiaobai/weiguo/zhangliao "zhangliao"
Created /luxiaobai/weiguo/zhangliao0000000001
[zk: localhost:2181(CONNECTED) 6] create -s /luxiaobai/weiguo/xuchu "xuchu"
Created /luxiaobai/weiguo/xuchu0000000002

##如果原来没有序号节点，序号从 0 开始依次递增。如果原节点下已有 2 个节点，则再排序时从 2 开始，以此类推。

##创建短暂节点(短暂节点+不带序号 or 带序号)

#创建短暂的不带序号的节点
[zk: localhost:2181(CONNECTED) 7] create -e /luxiaobai/wuguo "zhouyu"
Created /luxiaobai/wuguo


#创建短暂的带序号的节点
[zk: localhost:2181(CONNECTED) 8] create -e -s /luxiaobai/wuguo "zhouyu"
Created /luxiaobai/wuguo0000000004

##在当前客户端是能查看到的
[zk: localhost:2181(CONNECTED) 9] ls /luxiaobai
[shuguo, weiguo, wuguo, wuguo0000000004]

###退出 重启客户端 ,短暂节点已经删除
quit
ls /luxiaobai
[shuguo, weiguo]
 
##修改节点数据值
set /luxiaobai/weiguo "simayi"
```







## 监听器原理

>客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、节点删除、子目录节点增加删除）时，ZooKeeper 会通知客户端。监听机制保证 ZooKeeper 保存的任何的数据的任何改变都能快速的响应到监听了该节点的应用程序。



### 监听器原理详解

1. 首先要有一个main()线程
2. 在main线程中创建Zookeeper客户端,这时就会创建两个线程,一个负责网络连接通信(connet),一个负责监听(listener)
3. 通过connect线程将注册的监听事件发送给Zookeeper
4. 在Zookeeper的注册监听器列表中将注册的监听事件添加到列表中
5. Zookeeper监听到有数据或路径变化,就会将这个消息发送到listener线程
6. listener线程内部调用了process()方法

![image-20220408163343561](http://qiliu.luxiaobai.cn/img/image-20220408163343561.png)

#### 常见的监听

- 监听节点数据的变化

  get path[watch]

- 监听子节点增减的变化

  ls path[watch]



### 节点的值变化监听

```shell
##在1主机上注册监听/luxiaobai 节点数据变化
get -w /luxiaobai

###在2主机上修改/luxiaobai节点的数据
set /luxiaobai "xisi"

###观察1主机收到数据变化的监听
WATCHER::
WatchedEvent     state:SyncConnected       type:NodeDataChanged
path:/luxiaobai
```

> 注意：在2再多次修改/luxiaobai的值，1上不会再收到监听。因为注册一次，只能监听一次。想再次监听，需要再次注册。



### 节点的子节点变化监听(路径变化)

```shell
###在1主机上注册监听/luxiaobai节点的子节点变化
ls -w /luxiaobai

###在2主机/luxiaobai节点上创建子节点
create /luxiaobai/jin "simayi"

###观察1主机收到子节点变化的监听
WATCHER::
WatchedEvent     state:SyncConnected     type:NodeChildrenChanged
path:/luxiaobai
```

>注意：节点的路径变化，也是注册一次，生效一次。想多次生效，就需要多次注册。



## 节点删除与查看

```shell
###删除节点
delete /luxiaobai/jin

##递归删除节点
deleteall /luxiaobai/shuguo

##查看节点状态
stat /luxiaobai
```





## 客户端API操作

```java
private static final String CONNECT_STRING = "127.0.0.1:2181";

    private static int sessionTimeout = 2000;

    private ZooKeeper zkClient = null;

    @Before
    public void init() throws Exception {
        zkClient = new ZooKeeper(CONNECT_STRING, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                // 收到事件通知后的回调函数（用户的业务逻辑）
                System.out.println(watchedEvent.getType() + "--" + watchedEvent.getPath());

                // 再次启动监听
                try {
                    List<String> children = zkClient.getChildren("/", true);
                    for (String child : children) {
                        System.out.println(child);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
    }


    /**
     * 创建子节点
     * @throws Exception
     */
    @Test
    public void create() throws Exception {
        /**
         * 参数1：要创建的节点路径
         * 参数2：节点数据
         * 参数3：节点权限
         * 参数4：节点的类型
         */
        String nodeCreated = zkClient.create("/testChilder", "shuaige".getBytes(StandardCharsets.UTF_8), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }


    /**
     * 获取子节点并监听节点变化
     * @throws Exception
     */
    @Test
    public void getChildren() throws Exception {
        List<String> children = zkClient.getChildren("/", true);
        for (String child : children) {
            System.out.println("child = " + child);
        }

        //延时阻塞
        Thread.sleep(Long.MAX_VALUE);
    }

    @Test
    public void exists() throws Exception{
        Stat stat = zkClient.exists("/lxb", false);
        System.out.println(stat == null ? "not exits" : "exist");
    }
```





## 客户端向服务端写数据流程

==写流程之写入请求直接发送给Leader节点==

![image-20220408174424881](http://qiliu.luxiaobai.cn/img/image-20220408174424881.png)

==写流程之写入请求发送给follower节点==

![image-20220408174646946](http://qiliu.luxiaobai.cn/img/image-20220408174646946.png)



# 服务器动态上下线监听案例

## 需求

某分布式系统中,主节点可以有多台,可以动态上下线,任意一台客户端都能实时感知到主节点服务器的上下线



## 分析

### 服务器动态上下线

![image-20220409113514362](http://qiliu.luxiaobai.cn/img/image-20220409113514362.png)



## 具体实现

1. 先在集群上创建/servers节点

   ```shell
   [zk: localhost:2181(CONNECTED) 0] create /servers "servers"
   Created /servers
   ```

2. 服务端向Zookeeper注册代码

   ```java
   //Server
   
   public class DistributeServer {
       private static final String CONNECT_STRING = "127.0.0.1:2181";
       private static int sessionTimeout = 2000;
       private ZooKeeper zk = null;
       private String parentNode = "/servers";
   
       /**
        * 创建zk的客户端连接
        * @throws Exception
        */
       public void getConnect() throws Exception {
            zk = new ZooKeeper(CONNECT_STRING, sessionTimeout, new Watcher() {
               @Override
               public void process(WatchedEvent watchedEvent) {
   
               }
           });
       }
   
       /**
        * 注册服务器
        * @param hostName
        * @throws Exception
        */
       public void registerServer(String hostName) throws Exception {
           zk.create(parentNode + "/server",hostName.getBytes(StandardCharsets.UTF_8), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
           System.out.println(hostName + " is online " + create);
       }
   
       /**
        * 业务功能
        * @param hostName
        * @throws Exception
        */
       public void business(String hostName) throws Exception{
           System.out.println(hostName + " is Working....");
           Thread.sleep(Long.MAX_VALUE);
       }
   
       public static void main(String[] args) throws Exception {
           // 1 获取zk连接
           DistributeServer server = new DistributeServer();
           server.getConnect();
   
           // 2 利用zk连接注册服务器信息
           server.registerServer(args[0]);
   
           // 3 启动业务功能
           server.business(args[0]);
   
       }
   }
   
   //Client
   
   public class DistributeClient {
       private static final String CONNECT_STRING = "127.0.0.1:2181";
       private static int sessiontTimeout = 2000;
       private ZooKeeper zk = null;
       private String parentNode = "/servers";
   
       public void getConnect() throws Exception {
           zk = new ZooKeeper(CONNECT_STRING, sessiontTimeout, new Watcher() {
               @Override
               public void process(WatchedEvent watchedEvent) {
                   /**
                    * 再次启动监听
                    */
                   try {
                       getServerList();
                   } catch (Exception e) {
                       e.printStackTrace();
                   }
               }
           });
       }
   
       /**
        * 获取服务器列表信息
        * @throws Exception
        */
       public void getServerList() throws Exception {
           // 获取服务器子节点信息，并且对父节点进行监听
           List<String> children = zk.getChildren(parentNode, true);
           // 存储服务器信息列表
           ArrayList<String> servers = new ArrayList<>();
           // 遍历所有节点，获取节点中的主机名称信息
           for (String child : children) {
               byte[] data = zk.getData(parentNode + "/" + child, false, null);
               servers.add(new String(data));
           }
   
           //打印服务器列表信息
           System.out.println(servers);
       }
   
       /**
        * 业务功能
        * @throws Exception
        */
       public void business() throws Exception{
           System.out.println("client is working...");
           Thread.sleep(Long.MAX_VALUE);
       }
   
       public static void main(String[] args) throws Exception {
           //获取zk连接
           DistributeClient client = new DistributeClient();
           client.getConnect();
           //获取servers的子节点信息，从中获取服务器信息列表
           client.getServerList();
           //业务进程启动
           client.business();
       }
   
   }
   ```

   





# Zookeeper分布式锁案例

==什么叫做分布式锁呢?==

>比如说"进程 1"在使用该资源的时候，会先去获得锁，"进程 1"获得锁以后会对该资源保持独占，这样其他进程就无法访问该资源，"进程 1"用完该资源以后就将锁释放掉，让其他进程来获得锁，那么通过这个锁机制，就能保证了分布式系统中多个进程能够有序的访问该临界资源。那么把这个分布式环境下的这个锁叫作分布式锁。



## 分布式锁案例分析

![image-20220409162632743](http://qiliu.luxiaobai.cn/img/image-20220409162632743.png)



## 原生Zookeeper实现分布式锁案例

### 分布式锁实现

```java
public class DistributedLock {
    private static final String CONNECTSTRING = "127.0.0.1:2181";
    private static int sessionTimeout = 2000;
    private ZooKeeper zk = null;
    private String rootNode = "locks";
    private String subNode = "seq-";
    // 当前client等待的子节点
    private String waitPath;

    // Zookeeper连接
    private CountDownLatch connectLatch = new CountDownLatch(1);

    // Zookeeper节点等待
    private CountDownLatch waitLatch = new CountDownLatch(1);

    //当前client创建的子节点
    private String currentNode;

    public DistributedLock() throws Exception {
        zk = new ZooKeeper(CONNECTSTRING, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                // 连接建立时， 打开latch， 唤醒wait在该latch上的线程
                if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                    connectLatch.countDown();
                }


                //发生了waitPath的删除事件
                if (watchedEvent.getType() == Event.EventType.NodeDeleted && watchedEvent.getPath().equals(waitPath)) {
                    waitLatch.countDown();
                }
            }
        });

        //等待连接建立
        connectLatch.await();

        //获取根节点状态
        Stat stat = zk.exists("/" + rootNode, false);

        //如果根节点不存在，则创建根节点，根节点类型为永久节点
        if (stat == null) {
            System.out.println("根节点不存在");

            zk.create("/" + rootNode, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }

    }

    /**
     * 加深方法
     */
    public void zkLock() {

        try {
            currentNode = zk.create("/" + rootNode + "/" + subNode, null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);

            //wait 一下会， 让结果更清晰一些
            Thread.sleep(10);

            List<String> childrenNodes = zk.getChildren("/" + rootNode, false);

            //列表中只有一个子节点，那肯定就是currentNode，说明client获得锁
            if (childrenNodes.size() == 1) {
                return;
            } else {
                //对根节点下的所有临时顺序节点进行从小到大排序
                Collections.sort(childrenNodes);

                // 当前节点名称
                String thisNode = currentNode.substring(("/" + rootNode + "/").length());
                //获取当前节点的位置
                int index = childrenNodes.indexOf(thisNode);
                if (index == -1) {
                    System.out.println("数据异常");
                } else if (index == 0) {
                    // index ==0,说明thisNode在列表中最小，当前client获得锁
                    return;
                } else {
                    //获得排名比currentNode前1位的节点
                    this.waitPath = "/" + rootNode + "/" + childrenNodes.get(index - 1);

                    //在waitPath上注册监听器，当waitPath被删除时，zookeeper会回调监听器的process方法
                    zk.getData(waitPath, true, new Stat());
                    //进入等待锁状态
                    waitLatch.await();
                    return;
                }
            }
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 解锁方法
     */
    public void zkUnlock() {
        try {
            zk.delete(this.currentNode, -1);
        } catch (InterruptedException | KeeperException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public static void main(String[] args) throws Exception {
        // 创建分布式锁1
        final DistributedLock lock1 = new DistributedLock();
        //创建分布式锁2
        final DistributedLock lock2 = new DistributedLock();

        new Thread(new Runnable() {
            @Override
            public void run() {
                //获取锁对象
                try{
                    lock1.zkLock();
                    System.out.println("线程1获取锁");
                    Thread.sleep(5000);

                    lock1.zkUnlock();
                    System.out.println("线程1释放锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                //获取锁对象
                try{
                    lock2.zkLock();
                    System.out.println("线程2获取锁");
                    Thread.sleep(5000);

                    lock2.zkUnlock();
                    System.out.println("线程2释放锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
```





## Curator框架实现分布式锁案例

### 原生的Java API开发存在的问题

1. 会话连接是异步的,需要自己处理.如使用`CountDownLatch`
2. Watch需要重复注册,不然就不能生效
3. 开发的复杂性还是比较高的
4. 不支持多节点删除和创建.需要自己去递归





### Curator是一个专门解决分布式锁的框架,解决了原生Java API开发分布式遇到的问题

[官方文档](https://curator.apache.org/index.html)



### Curator案例实操

```java
public class CuratorLockTest {
    private String rootNode = "/locks";

    private static final String CONNECT_STRING = "127.0.0.1:2181";
    private int connectTimeout = 2000;
    private int sessionTimeout = 2000;

    private void test(){
        //创建分布式锁1
        final InterProcessMutex lock1 = new InterProcessMutex(getCuratorFramework(), rootNode);
        //创建分布式锁2
        final InterProcessMutex lock2 = new InterProcessMutex(getCuratorFramework(), rootNode);

        new Thread(new Runnable() {
            @Override
            public void run() {
                try{
                    lock1.acquire();
                    System.out.println("线程1获取锁");
                    //测试锁重入
                    lock1.acquire();
                    System.out.println("线程1再次获取锁");
                    Thread.sleep(5000);
                    lock1.release();
                    System.out.println("线程1释放锁");
                    lock1.release();
                    System.out.println("线程1再次释放锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try{
                    lock2.acquire();
                    System.out.println("线程2获取锁");
                    //测试锁重入
                    lock2.acquire();
                    System.out.println("线程2再次获取锁");
                    Thread.sleep(5000);
                    lock2.release();
                    System.out.println("线程2释放锁");
                    lock2.release();
                    System.out.println("线程2再次释放锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

    }

    public CuratorFramework getCuratorFramework(){
        //重试策略，初始时间3秒，重试3次
        RetryPolicy policy = new ExponentialBackoffRetry(3000, 3);

        //通过工厂创建Curator
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(CONNECT_STRING)
                .connectionTimeoutMs(connectTimeout)
                .sessionTimeoutMs(sessionTimeout)
                .retryPolicy(policy)
                .build();
        //开启连接
        client.start();
        System.out.println("zookeeper 初始号wanc...");
        return client;
    }

    public static void main(String[] args) {
        new CuratorLockTest().test();
    }
}
```































































































































































































































































































































