---
RabbitMQ
---

[toc]



# 消息队列

## MQ

>**MQ(message queue)**，从字面意思上看，本质是个队列，FIFO 先入先出，只不过队列中存放的内容是message 而已，还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，MQ 是一种非常常
>
>见的上下游==“逻辑解耦+物理解耦”==的消息通信服务。使用了 MQ 之后，消息发送上游只需要依赖 MQ，不
>
>用依赖其他服务。

- 流量消峰

  如果订单系统最多能处理一万次订单，这个处理能力应付正常时段的下单时绰绰有余，正常时段我们下单一秒后就能返回结果。但是在高峰期，如果有两万次下单操作系统是处理不了的，只能限制订单超过一万后不允许用户下单。使用消息队列做缓冲，我们可以取消这个限制，把一秒内下的订单分散成一段时间来处理，这时有些用户可能在下单十几秒后才能收到下单成功的操作，但是比不能下单的体验要好。

- 应用解耦

  以电商应用为例，应用中有订单系统、库存系统、物流系统、支付系统。用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障，都会造成下单操作异常。当转变成基于消息队列的方式后，系统间调用的问题会减少很多，比如物流系统因为发生故障，需要几分钟来修复。在这几分钟的时间里，物流系统要处理的内存被缓存在消息队列中，用户的下单操作可以正常完成。当物流系统恢复后，继续处理订单信息即可，中单用户感受不到物流系统的故障，提升系统的可用性。

  ![image-20211220224430945](http://qiliu.luxiaobai.cn/img/image-20211220224430945.png)

- 异步处理

有些服务间调用是异步的，例如 A 调用 B，B 需要花费很长时间执行，但是 A 需要知道 B 什么时候可以执行完，以前一般有两种方式，A 过一段时间去调用 B 的查询 api 查询。或者 A 提供一个 callback api， B 执行完之后调用 api 通知 A 服务。这两种方式都不是很优雅，使用消息总线，可以很方便解决这个问题，A 调用 B 服务后，只需要监听 B 处理完成的消息，当 B 处理完成后，会发送一条消息给 MQ，MQ 会将此消息转发给 A 服务。这样 A 服务既不用循环调用 B 的查询 api，也不用提供 callback api。同样 B 服务也不用做这些操作。A 服务还能及时的得到异步处理成功的消息。

![image-20211220224711266](http://qiliu.luxiaobai.cn/img/image-20211220224711266.png)



## MQ的分类

### ActiveMQ

#### 优点：

​	单机吞吐量万级，时效性 ms 级，可用性高，基于主从架构实现高可用性，消息可靠性较低的概率丢失数据

#### 缺点:

​	官方社区现在对 ActiveMQ 5.x **维护越来越少，高吞吐量场景较少使用**。

### Kafka

大数据的杀手锏，谈到大数据领域内的消息传输，则绕不开 Kafka，这款为**大数据而生**的消息中间件，以其**百万级** **TPS** 的吞吐量名声大噪，迅速成为大数据领域的宠儿，在数据采集、传输、存储的过程中发挥着举足轻重的作用。

#### 优点:

​	性能卓越，单机写入 TPS 约在百万条/秒，最大的优点，就是吞**吐量高**。时效性 ms 级可用性非常高，kafka 是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用,消费者采用 Pull 方式获取消息, 消息有序, 通过控制能够保证所有消息被消费且仅被消费一次;有优秀的第三方Kafka Web 管理界面 Kafka-Manager；在日志领域比较成熟，被多家公司和多个开源项目使用；功能支持：功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及**日志采集**被大规模使用

#### 缺点：

​	Kafka 单机超过 64 个队列/分区，Load 会发生明显的飙高现象，队列越多，load 越高，发送消息响应时间变长，使用短轮询方式，实时性取决于轮询间隔时间，消费失败不支持重试；支持消息顺序，但是一台代理宕机后，就会产生消息乱序，**社区更新较慢**； 



### RocketMQ

RocketMQ 出自阿里巴巴的开源产品，用 Java 语言实现，在设计时参考了 Kafka，并做出了自己的一些改进。被阿里巴巴广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理，binglog 分发等场景。

优点:

​	**单机吞吐量十万级**,可用性非常高，分布式架构,**消息可以做到** **0** **丢失****,**MQ 功能较为完善，还是分布式的，扩展性好,**支持** **10** **亿级别的消息堆积**，不会因为堆积导致性能下降,源码是 java 我们可以自己阅读源码，定制自己公司的 MQ

缺点：

​	**支持的客户端语言不多**，目前是 java 及 c++，其中 c++不成熟；社区活跃度一般,没有在 MQ

核心中去实现 JMS 等接口,有些系统要迁移需要修改大量代码

### RabbitMQ

优点:

​	由于 erlang 语言的**高并发特性**，性能较好；**吞吐量到万级**，MQ 功能比较完备,健壮、稳定、易用、跨平台、**支持多种语言** 如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持 AJAX 文档齐全；开源提供的管理界面非常棒，用起来很好用,**社区活跃度高**；更新频率相当高https://www.rabbitmq.com/news.html

缺点：商业版需要收费,学习成本较高





## MQ的选择

### Kafka

Kafka 主要特点是基于 Pull 的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输，适合产生**大量数据**的互联网服务的数据收集业务。**大型公司**建议可以选用，如果有**日志采集**功能，肯定是首选 kafka 了。

### RocketMQ

天生为**金融互联网**领域而生，对于可靠性要求很高的场景，尤其是电商里面的订单扣款，以及业务削峰，在大量交易涌入时，后端可能无法及时处理的情况。RoketMQ 在稳定性上可能更值得信赖，这些业务场景在阿里双 11 已经经历了多次考验，如果你的业务有上述并发场景，建议可以选择 RocketMQ。



### RabbitMQ

结合 erlang 语言本身的并发优势，性能好**时效性微秒级**，**社区活跃度也比较高**，管理界面用起来十分

方便，如果你的**数据量没有那么大**，中小型公司优先选择功能比较完备的 RabbitMQ。



# RabbitMQ

## 简介

>RabbitMQ 是一个消息中间件：它接受并转发消息。你可以把它当做一个快递站点，当你要发送一个包裹时，你把你的包裹放到快递站，快递员最终会把你的快递送到收件人那里，按照这种逻辑 RabbitMQ 是一个快递站，一个快递员帮你传递快件。RabbitMQ 与快递站的主要区别在于，它不处理快件而是**接收，存储和转发消息数据**。



## 四大核心概念

### 生产者

产生数据发送消息的程序是生产者

###  交换机

交换机是 RabbitMQ 非常重要的一个部件，**一方面它接收来自生产者的消息，另一方面它将消息推送到队列中**。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定

### 队列

队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式



### 消费者

消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。



## RabbitMQ核心部分

### 工作原理

![image-20211220230659373](http://qiliu.luxiaobai.cn/img/image-20211220230659373.png)



**Broker**：接收和分发消息的应用，RabbitMQ Server 就是 Message Broker

**Virtual host**：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个RabbitMQ server 提供的服务时，可以划分出

多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等

**Connection**：publisher／consumer 和 broker 之间的 TCP 连接

**Channel**：如果每一次访问 RabbitMQ 都建立一个Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。**Channel 作为轻量级的Connection **极大减少了操作系统建立 **TCP connection** **的开销**

**Exchange**：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发

消息到 queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout 

(multicast)

**Queue**：消息最终被送到这里等待 consumer 取走

**Binding**：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据



## 安装

https://www.rabbitmq.com/download.html

![image-20211220232053264](http://qiliu.luxiaobai.cn/img/image-20211220232053264.png)

```shell
###安装文件
rpm -ivh erlang-21.3-1.el7.x86_64.rpm
yum install socat -y
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm

####添加开启启动RabbitMQ服务
chkconfig rabbitmq-server on

###启动服务
/sbin/service rabbitmq-server start

###查看服务状态
/sbin/service rabbitmq-server status

###停止服务
/sbin/service rabbitmq-server stop

##开启web管理插件 默认账号密码 guest
rabbit-plugins enable rabbitmq_management
##首次访问可能会有权限问题

###添加新用户 创建账户
rabbitmqctl add_user admin 123

###设置用户角色
rabbitmqctl set_user_tags admin administrator

##设置用户权限 set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"  //用户 user_admin 具有/vhost1 这个 virtual host 中所有资源的配置、写、读权限


##当前用户和角色
rabbitmqctl list_users


##重置命令
##关闭应用命令
rabbitmqctl stop_app

##清除命令
rabbitmqctl reset

##重新启动命令
rabbitmqctl start_app
```





## 工作队列

主要思想是避免立即执行资源密集型任务，而不得不等待它完成。把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。

### 轮训分发消息

![image-20220306172536655](http://qiliu.luxiaobai.cn/img/image-20220306172536655.png)

通过程序执行发现生产者总共发送 6 个消息，消费者 1 和消费者 2 消费者3 分别分得两个消息，并且是按照有序的一个接收一次消息



### 消费应答

为了保证消息在发送过程中不丢失，rabbitmq 引入消息应答机制，消息应答就是:**消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。**



#### 自动应答

这种模式需要在**高吞吐量和数据传输安全性方面做权衡**,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失了,当然另一方面这种模式消费者那边可以传递过载的消息，**没有对传递的消息数量进行限制**，当然这样有可能使得消费者这边由于**接收太多还来不及处理的消息，导致这些消息的积压**，最终使得内存耗尽，最终这些消费者线程被操作系统杀死，**所以这种模式仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用**。



#### 手动应答

##### 应答方法

1. `Channel.basicAck(用于肯定确认)`: Rabbit MQ已知道该消息并且成功的处理消息,可以将其丢弃
2. `Channel.basicNack(用于否定确认)`
3. `Channel.basicReject(用于否定确认)`: 与`Channel.basicNack`相比少一个参数,不处理该消息了直接拒绝,可以将其丢弃



#### Multiple的解释

**手动应答的好处是可以批量应答并且减少网络拥堵**

![image-20220306173444873](http://qiliu.luxiaobai.cn/img/image-20220306173444873.png)

> **multiple**的true和false
>
> - true: 代表批量应答channel上未应答的消息, 比如说channel上有传送tag的消息,5,6,7,8 当前tag是8 那么此时5-8的这些还未应答的消息都会被确认收到消息应答
> - false: false 同上面相比只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答

![image-20220306173742512](http://qiliu.luxiaobai.cn/img/image-20220306173742512.png)





#### 消息自动重新入队

消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认,RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。



![image-20220306221247110](http://qiliu.luxiaobai.cn/img/image-20220306221247110.png)





# RabbitMQ持久化



## 队列如何实现持久化

创建的队列都是非持久化的，rabbitmq 如果重启的化，该队列就会被删除掉，如果要队列实现持久化 需要在声明队列的时候把 `durable` 参数设置为持久化

```java
//让消息队列持久化
boolean durable=true
channel.queueDeclare(ACK_QUEUE_NAME,durable,false,false,null);
```

> 但是需要注意的就是如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新创建一个持久化的队列，不然就会出现错误



### 持久化与非持久化队列的UI显示

但是需要注意的就是如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新

创建一个持久化的队列，不然就会出现错误

![image-20220307154232884](http://qiliu.luxiaobai.cn/img/image-20220307154232884.png)





## 消息实现持久化

要想让消息实现持久化需要在消息生产者修改代码，MessageProperties.PERSISTENT_TEXT_PLAIN 添加这个属性。

```java
channel.basicPublish("", TASK_QUEUE_NAME, null, message.getBytes("UTF-8"));
//当durable为true的时候
channel.basicPublish("", TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes("UTF-8"));
```





## 不公平分发

通过设置`channel.basicQos(1)`解决,某些消费者处理时间缓慢,某些处理时间迅速的问题,

如果这个任务我还没有处理完或者我还没有应答你，你先别分配给我，我目前只能处理一个任务，然后 rabbitmq 就会把该任务分配给没有那么忙的那个空闲消费者，当然如果所有的消费者都没有完成手上任务，队列还在不停的添加新任务，队列有可能就会遇到队列被撑满的情况，这个时候就只能添加新的 worker 或者改变其他存储任务的策略。



## 预取值

就存在一个未确认的消息缓冲区，因此希望开发人员能**限制此缓冲区的大小，以避免缓冲区里面无限制的未确认消息问题**。这个时候就可以通过使用 basic.qos 方法设置“预取计数”值来完成的。**该值定义通道上允许的未确认消息的最大数量**。一旦数量达到配置的数量，RabbitMQ 将停止在通道上传递更多消息，除非至少有一个未处理的消息被确认.

假设在通道上有未确认的消息 5、6、7，8，并且通道的预取计数设置为 4，此时 RabbitMQ 将不会在该通道上再传递任何消息，除非至少有一个未应答的消息被 ack。比方说 tag=6 这个消息刚刚被确认 ACK，RabbitMQ 将会感知这个情况到并再发送一条消息。消息应答和 QoS 预取值对用户吞吐量有重大影响。通常，增加预取将提高向消费者传递消息的速度。**虽然自动应答传输消息速率是最佳的，但是，在这种情况下已传递但尚未处理的消息的数量也会增加，从而增加了消费者的** **RAM** **消耗**(随机存取存储器)应该小心使用具有无限预处理的自动确认模式或手动确认模式，消费者消费了大量的消息如果没有确认的话，会导致消费者连接节点的内存消耗变大，所以找到合适的预取值是一个反复试验的过程，不同的负载该值取值也不同 100 到 300 范围内的值通常可提供最佳的吞吐量，并且不会给消费者带来太大的风险。预取值为 1 是最保守的。当然这将使吞吐量变得很低，特别是消费者连接延迟很严重的情况下，特别是在消费者连接等待时间较长的环境中。对于大多数应用来说，稍微高一点的值将是最佳的。

![image-20220307201607849](http://qiliu.luxiaobai.cn/img/image-20220307201607849.png)







# 发布确认

>生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，**所有在该信道上面发布的消息都将会被指派一个唯一的 ID**(从 1 开始)，一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。confirm 模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果 RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息，生产者应用程序同样可以在回调方法中处理该 nack 消息。

消息队列对消息进行持久化,然后告诉生产者确认发布.

## 发布确认的策略

### 开启发布确认的方法

发布确认默认是没有开启的,要开启需要调用方法`confirmSelect`,每当要使用发布确认,都需要在`channel`上调用该方法

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
```





### 单个确认发布

发布一个消息之后只有它被确认发布,后续的消息才能继续发布.`waitForConfirmsOrDie(long)`这个方法只有在消息被确认的时候才返回,如果在指定时间范围内这个消息没有被确认那么它将抛出异常

最大的缺点就是:**发布速度特别的慢**







### 批量确认发布

缺点就是:当发生故障导致发布出现问题时，不知道是哪个消息出现问题了，必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。





### 异步确认发布

利用回调函数来达到消息可靠性传递的，这个中间件也是通过函数回调来保证是否投递成功

![image-20220307230436849](http://qiliu.luxiaobai.cn/img/image-20220307230436849.png)



### 如何处理异步未确认消息

把未解决的消息放到一个基于内存的能被发布线程访问的队列,比如说用`ConcurrentLinkedQueue`这个队列在`confirm callbacks`与发布线程之间进行消息的传递.



- 单独发布消息:同步等待确认，简单，但吞吐量非常有限。
- 批量发布消息: 批量同步等待确认，简单，合理的吞吐量，一旦出现问题但很难推断出是那条消息出现了问题。
- 异步处理：最佳性能和资源使用，在出现错误的情况下可以很好地控制，但是实现起来稍微难些





# 交换机 Exchanges

RabbitMQ消息传递模型的核心思想是: **生产者生产的消息从不会直接发送到队列.** 

生产者只能将消息发送到交换机, 交换机一方面接收来自生产者的消息,另一方面将他们推入队列.

交换机必须确切知道如何处理收到的消息.



## Exchanges的类型

- 直接(direct)
- 主题(topic)
- 标题(headers)
- 扇出(fanout)



#### 无名exchange

`channel.basicPublish("","hello", null, message.getBytes());`

第一个参数是交换机的名称. **空字符串表示默认或无名称;**消息能路由发送到队列中其实是由`routingkey(bindingkey)`绑定key指定的,如果它存在的话.



### 临时队列

​	`String queueName = channel.queueDeclare().getQueue()`





### 绑定(bingdings)

bingding是exchange和queue之间的桥梁,如下X与Q1和Q2进行了绑定

![image-20220308222404984](http://qiliu.luxiaobai.cn/img/image-20220308222404984.png)

### Fanout

> Fanout是将接收到的所有消息广播到它知道的所有队列中.



#### Fanout实战

![image-20220314132744745](http://qiliu.luxiaobai.cn/img/image-20220314132744745.png)

Logs和临时队列的绑定关系:

![image-20220314132836667](http://qiliu.luxiaobai.cn/img/image-20220314132836667.png)

队列只对它绑定的交换机的消息感兴趣.绑定用参数:`routingKey`来表示也可称该参数为`bingding key`. `channle.queueBind(queueName, EXCHANGE, "routingKey")`



### Direct exchange

![image-20220314162304639](http://qiliu.luxiaobai.cn/img/image-20220314162304639.png)

![image-20220314162330402](http://qiliu.luxiaobai.cn/img/image-20220314162330402.png)

根据绑定参数routingKey,实现对交换机与队列的绑定,类似于路由分发的效果,从而能有选择性的接收消息

`channel.queueBind(queueName, EXCHANGE_NAME, "error");`



### Topic

发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求，它**必须是一个单词列表，以点号分隔开**。这些单词可以是任意单词，比说："stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit".这种类型的。当然这个单词列表最多不能超过 255 个字节。在这个规则列表中，其中有两个替换符是大家需要注意的

- ***(****星号**)可以代替一个单词
- **#(井号**)**可以替代零个或多个单词**

![image-20220328104402509](http://qiliu.luxiaobai.cn/img/image-20220328104402509.png)

- quick.orange.rabbit 		被队列 Q1Q2 接收到
- lazy.orange.elephant       被队列 Q1Q2 接收到
- quick.orange.fox               被队列 Q1 接收到
- lazy.brown.fox                   被队列 Q2 接收到
- lazy.pink.rabbit                  虽然满足两个绑定但只被队列 Q2 接收一次
- quick.brown.fox                 不匹配任何绑定不会被任何队列接收到会被丢弃
- quick.orange.male.rabbit  是四个单词不匹配任何绑定会被丢弃
- lazy.orange.male.rabbit     是四个单词但匹配 Q2

当队列绑定关系是下列这种情况时需要引起注意

**当一个队列绑定键是**#**,**那么这个队列将接收所有数据，就有点像 **fanout** **了**

**如果队列绑定键当中没有#和*出现，那么该队列绑定类型就是** **direct** **了**





# 死信队列

无法被消费的消息



##### 应用场景

> 为了保证订单业务的消息数据不丢失，需要使用到 RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中.还有比如说: 用户在商城下单成功并点击去支付后在指定时间未支付时自动失效



## 死信的来源

- 消息 TTL 过期
- 队列达到最大长度(队列满了，无法再添加数据到 mq 中)
- 消息被拒绝(basic.reject 或 basic.nack)并且 requeue=false.

![image-20220328111459998](http://qiliu.luxiaobai.cn/img/image-20220328111459998.png)





# 延迟队列

> 延迟队列,队列内部是有序的,延时队列中的元素是希望在指定时间到了以后或之前取出和处理.即用来存放需要在指定时间被处理的元素的队列



## 延迟队列使用场景

1.订单在十分钟之内未支付则自动取消

2.新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。

3.用户注册成功后，如果三天内没有登陆则进行短信提醒。

4.用户发起退款，如果三天内没有得到处理则通知相关运营人员。

5.预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议



## Rabbit中的TTL

TTL 是 RabbitMQ 中一个消息或者队列的属性，表明一条消息或者该队列中的所有

消息的最大存活时间，单位是毫秒.



### 消息设置TTL

针对每条消息设置TTL

```java
rabbitTemplate.convertAndSend("X", "XC", message, correlationData->{
	correlationData.getMessageProperties().setExpiration(ttlTime);
})
```



### 队列设置TTL

在创建队列的时候设置队列的“x-message-ttl”属性

```java
//声明队列的TTL
args.put("x-message-ttl", 5000);
return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
```



### 两者区别

> 设置了队列的 TTL 属性，那么一旦消息过期，就会被队列丢弃(如果配置了死信队列被丢到死信队列中)，而第二种方式，消息即使过期，也不一定会被马上丢弃，因为**消息是否过期是在即将投递到消费者之前判定的**，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间；另外，还需要注意的一点是，如果不设置 TTL，表示消息永远不会过期，如果将 TTL 设置为 0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。





## Rabbitmq插件实现延迟队列

[官网下载插件](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.8.0/rabbitmq_delayed_message_exchange-3.8.0.ez)**rabbitmq_delayed_message_exchange-3.8.0.ez**放置到Rabbit MQ的插件目录

```shell
cd /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8/plugins
#插件生效
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
##重新启动命令
rabbitmqctl start_app
```



>延时队列在需要延时处理的场景下非常有用，使用 RabbitMQ 来实现延时队列可以很好的利用RabbitMQ 的特性，如：消息可靠发送、消息可靠投递、死信队列来保障消息至少被消费一次以及未被正确处理的消息不会被丢弃。另外，通过 RabbitMQ 集群的特性，可以很好的解决单点故障问题，不会因为单个节点挂掉导致延时队列不可用或者消息丢失。当然，延时队列还有很多其它选择，比如利用 Java 的 DelayQueue，利用 Redis 的 zset，利用 Quartz或者利用 kafka 的时间轮，这些方式各有特点,看需要适用的场景





## 发布确认高级

>在生产环境中由于一些不明原因，导致 rabbitmq 重启，在 RabbitMQ 重启期间生产者消息投递失败，导致消息丢失，需要手动处理和恢复。于是，我们开始思考，如何才能进行 RabbitMQ 的消息可靠投递呢？特别是在这样比较极端的情况，RabbitMQ 集群不可用的时候，无法投递的消息该如何处理呢



### 确认机制方案

![image-20220329082342896](http://qiliu.luxiaobai.cn/img/image-20220329082342896.png)



### 代码架构图

![image-20220329082409201](http://qiliu.luxiaobai.cn/img/image-20220329082409201.png)

有了 mandatory 参数和回退消息，我们获得了对无法投递消息的感知能力，有机会在生产者的消息无法被投递时发现并处理。



## 备份交换机

如何处理那些被退回的消息,即不可路由消息呢?可以为队列设置死信交换机来存储那些处理失败的消息，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。



备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当为某一个交换机声明一个对应的备份交换机时，就是为它创建一个备胎，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。还可以建立一个报警队列，用独立的消费者来进行监测和报警。

### 代码架构图

![image-20220329093806542](http://qiliu.luxiaobai.cn/img/image-20220329093806542.png)



mandatory 参数与备份交换机可以一起使用的时候，如果两者同时开启，消息究竟何去何从？谁优先级高，即**备份交换机优先级高**





# RabbitMQ其他知识



## 幂等性

>用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用



### 消息重复消费

消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者，消费者在给 MQ 返回 ack 时网络中断，故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息



### 解决思路

MQ 消费者的幂等性的解决一般使用**全局 ID** 或者写个**唯一标识**比如时间戳 或者 **UUID** 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，每次消费消息时用该 id 先判断该消息是否已消费过。



### 消费端的幂等性保障

在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性，这就意味着我们的消息永远不会被消费多次，即使我们收到了一样的消息。业界主流的幂等性有两种操作:==a.唯一 ID+指纹码机制,利用数据库主键去重, b.利用 redis 的原子性去实现==



### 唯一ID+指纹码机制

指纹码:**一些规则或者时间戳加别的服务给到的唯一信息码,它并不一定是系统生成的，基本都是由业务规则拼接而来，但是一定要保证唯一性**，然后就利用查询语句进行判断这个 id 是否存在数据库中,优势就是实现简单就一个拼接，然后查询判断是否重复；劣势就是在高并发时，如果是单个数据库就会有写入性能瓶颈当然也可以采用分库分表提升性能，但也不是我们最推荐的方式。



### Redis原子性

利用redis执行setnx命令,天然具有幂等性.从而实现不重复消费.



## 优先级队列



### 添加方式

a .控制台页面添加

![image-20220329103954088](http://qiliu.luxiaobai.cn/img/image-20220329103954088.png)



b .队列中代码添加优先级

```java
Map<String,Object> params = new HashMap();
params.put("x-max-priority", 10);
channel.queueDeclare("hello",true, false,false,params);
```

 c.消息中代码添加优先级

```java
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
```

>要让队列实现优先级需要做的事情有如下事情:队列需要设置为优先级队列，消息需要设置消息的优先级，消费者需要等待消息已经发送到队列中才去消费因为，这样才有机会对消息进行排序



## 惰性队列

>惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了

默认情况下，当生产者将消息发送到 RabbitMQ 的时候，队列中的消息会尽可能的存储在内存之中，这样可以更加快速的将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份。当 RabbitMQ 需要释放内存的时候，会将内存中的消息换页至磁盘中，这个操作会耗费较长的时间，也会阻塞队列的操作，进而无法接收新的消息。虽然 RabbitMQ 的开发者们一直在升级相关的算法，但是效果始终不太理想，尤其是在消息量特别大的时候。



### 两种模式

队列具备两种模式：**default** 和 **lazy**。默认的为 **default** 模式，在 3.6.0 之前的版本无需做任何变更。lazy模式即为惰性队列的模式，可以通过调用 **channel.queueDeclare** 方法的时候在参数中设置，也可以通过Policy 的方式设置，如果一个队列同时使用这两种方式设置的话，那么 Policy 的方式具备更高的优先级。

```java
Map<String,Object> args = new HashMap<String, Object>();
args.put("x-queue-mode","lazy");
channel.queueDeclare("myqueue",false,false,false,args);
```



### 内存开销对比

![image-20220329111904123](http://qiliu.luxiaobai.cn/img/image-20220329111904123.png)

在发送 1 百万条消息，每条消息大概占 1KB 的情况下，普通队列占用内存是 1.2GB，而惰性队列仅仅占用 1.5MB



# Rabbit MQ集群

## 搭建步骤

```shell
#修改3台机器的主机名称
vim /etc/hostname
##配置各个节点的hosts文件,让各个节点都能互相识别对方
vim /etc/hosts
10.211.55.74 node1
10.211.55.75 node2
10.211.55.76 node3
#以确保各个节点的 cookie 文件使用的是同一个值
#在node1上至下远程操作命令
scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/.erlang.cookie
scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/.erlang.cookie
##启动 RabbitMQ 服务,顺带启动 Erlang 虚拟机和 RbbitMQ 应用服务(在三台节点上分别执行以下命令)
rabbitmq-server -detached
#在节点 2 执行
rabbitmqctl stop_app
(rabbitmqctl stop 会将 Erlang 虚拟机关闭，rabbitmqctl stop_app 只关闭 RabbitMQ 服务)
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node1
rabbitmqctl start_app(只启动应用服务)

##在节点 3 执行
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node2
rabbitmqctl start_app

##集群状态
rabbitmqctl cluster_status
#需要重新设置用户
#创建账号
rabbitmqctl add_user admin 123
#设置用户角色
rabbitmqctl set_user_tags admin administrator
#设置用户权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
##解除集群节点(node2 和 node3 机器分别执行)
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
rabbitmqctl cluster_status
rabbitmqctl forget_cluster_node rabbit@node2(node1 机器上执行)
```



## 镜像队列

类似于备份节点.引入镜像队列(Mirror Queue)的机制，可以将队列镜像到集群中的其他 Broker 节点之上，如果集群中的一个节点失效了，队列能自动地切换到镜像中的另一个节点上以保证服务的可用性





## Haproxy + Keepalive实现高可用负载均衡

<img src="http://qiliu.luxiaobai.cn/img/image-20220329112748568.png" alt="image-20220329112748568" style="zoom:50%;" />



### Haproxy实现负载均衡

HAProxy 提供高可用性、负载均衡及基于 TCPHTTP 应用的代理，支持虚拟主机，它是免费、快速并且可靠的一种解决方案，包括 Twitter,Reddit,StackOverflow,GitHub 在内的多家知名互联网公司在使用。HAProxy 实现了一种事件驱动、单一进程模型，此模型支持非常大的井发连接数。

[nginx,lvs,haproxy之间的区别](http://www.ha97.com/5646.html)

#### 搭建步骤

```shell
##下载haproxy(在node1和node2)
yum -y install haproxy
##修改node1和node2的haproxy.cfg,将iP改为当前机器IP
vim /etc/haproxy/haproxy.cfg
##在两台节点启动 haproxy
haproxy -f /etc/haproxy/haproxy.cfg
ps -ef | grep haproxy
##访问地址
http://121.199.76.44:8888/stats
```



### **Keepalived 实现双机(主备)热备**

> 如果前面配置的 HAProxy 主机突然宕机或者网卡失效，那么虽然 RbbitMQ 集群没有任何故障但是对于外界的客户端来说所有的连接都会被断开结果将是灾难性的为了确保负载均衡服务的可靠性同样显得十分重要，这里就要引入 Keepalived 它能够通过自身健康检查、资源接管功能做高可用(双机热备)，实现故障转移.



#### 搭建步骤

```shell
####1.下载 keepalived
yum -y install keepalived
###2.节点 node1 配置文件
vim /etc/keepalived/keepalived.conf

###把资料里面的 keepalived.conf 修改之后替换
###3.节点 node2 配置文件需要修改 global_defs 的 router_id,如:nodeB其次要修改 vrrp_instance_VI 中 state 为"BACKUP"；最后要将 priority 设置为小于 100 的值
###4.添加 haproxy_chk.sh (为了防止 HAProxy 服务挂掉之后 Keepalived 还在正常工作而没有切换到 Backup 上，所以这里需要编写一个脚本来检测 HAProxy 务的状态,当 HAProxy 服务挂掉之后该脚本会自动重启HAProxy 的服务，如果不成功则关闭 Keepalived 服务，这样便可以切换到 Backup 继续工作)
vim /etc/keepalived/haproxy_chk.sh(可以直接上传文件)

###修改权限 chmod 777 /etc/keepalived/haproxy_chk.sh
###5.启动 keepalive 命令(node1 和 node2 启动)
systemctl start keepalived
###6.观察 Keepalived 的日志
tail -f /var/log/messages -n 200

###7.观察最新添加的 vip
ip add show

###8.node1 模拟 keepalived 关闭状态
systemctl stop keepalived 

###9.使用 vip 地址来访问 rabbitmq 集群
```





## Federation Exchange

#### 搭建步骤

```shell
##1.需要保证每台节点单独运行
##2.在每台机器上开启 federation 相关插件
rabbitmq-plugins enable rabbitmq_federation
rabbitmq-plugins enable rabbitmq_federation_management

##3.原理图(先运行 consumer 在 node2 创建 fed_exchange)
##4.在 downstream(node2)配置 upstream(node1)
##5.添加 policy
##6.成功的前提
```

<img src="http://qiliu.luxiaobai.cn/img/image-20220329114759468.png" alt="image-20220329114759468" style="zoom:50%;" />

<img src="http://qiliu.luxiaobai.cn/img/image-20220329114850181.png" alt="image-20220329114850181" style="zoom:50%;" />

<img src="http://qiliu.luxiaobai.cn/img/image-20220329132728719.png" alt="image-20220329132728719" style="zoom:50%;" />

<img src="http://qiliu.luxiaobai.cn/img/image-20220329132817380.png" alt="image-20220329132817380" style="zoom:67%;" />



### FederationQueue

联邦队列可以在多个 Broker 节点(或者集群)之间为单个队列提供均衡负载的功能。一个联邦队列可以连接一个或者多个上游队列(upstream queue)，并从这些上游队列中获取消息以满足本地消费者消费消息的需求。



#### 搭建步骤

**原理图**

![image-20220329132953599](http://qiliu.luxiaobai.cn/img/image-20220329132953599.png)

添加policy

![image-20220329133030563](http://qiliu.luxiaobai.cn/img/image-20220329133030563.png)



## Shovel

Federation 具备的数据转发功能类似，Shovel 够可靠、持续地从一个 Broker 中的队列(作为源端，即source)拉取数据并转发至另一个 Broker 中的交换器(作为目的端，即 destination)。作为源端的队列和作为目的端的交换器可以同时位于同一个 Broker，也可以位于不同的 Broker 上。Shovel 可以翻译为"铲子"，是一种比较形象的比喻，这个"铲子"可以将消息从一方"铲子"另一方。Shovel 行为就像优秀的客户端应用程序能够负责连接源和目的地、负责消息的读写及负责连接失败问题的处理。

### 搭建步骤

```shell
##1.开启插件(需要的机器都开启)
rabbitmq-plugins enable rabbitmq_shovel
rabbitmq-plugins enable rabbitmq_shovel_management

##2.原理图(在源头发送的消息直接回进入到目的地队列)

##3.添加 shovel 源和目的地
```

<img src="http://qiliu.luxiaobai.cn/img/image-20220329133244487.png" alt="image-20220329133244487" style="zoom:67%;" />



<img src="http://qiliu.luxiaobai.cn/img/image-20220329133311231.png" alt="image-20220329133311231" style="zoom:67%;" />



























