---
Eureka服务注册与发现
---

[toc]

----

# Eureka

## 服务治理

> 在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。



## 服务注册

>Eureka采用了CS的设计架构，Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。
>在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息 比如 服务地址通讯地址等以别名方式注册到注册中心上。另一方（消费者|服务提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何rpc远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址))



### Eureka系统架构 VS Dubbo架构

![image-20220420151734145](http://qiliu.luxiaobai.cn/img/image-20220420151734145.png)



## Eureka两组件

### Eureka Server

Eureka Server提供服务注册服务
各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。



### Eureka Client

EurekaClient通过注册中心进行访问
是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）



![image-20220420215950374](http://qiliu.luxiaobai.cn/img/image-20220420215950374.png)



==微服务RPC远程服务调用最核心的是什么==

即高可用,搭建Eureka注册中心集群,实现负载均衡+故障容错



### 集群构建

本机配置===》修改映射配置

```shell
sudo vim /etc/hosts
#####SpringCloud######
127.0.0.1	eureka7001.com
127.0.0.1	eureka7002.com
127.0.0.1	eureka7003.com 
```



## actuator微服务信息完善

### 主机名称:服务名称修改

![image-20220421145544951](http://qiliu.luxiaobai.cn/img/image-20220421145544951.png)

```application.yml
  instance:
      instance-id: payment8001
      prefer-ip-address: true ##访问路径可以显示IP地址
```



## 服务发现Discovery

对于注册进eureka里面的微服务,可以通过服务发现来获得该服务的信息

```java
//在主main方法上增加 @EnableDiscoveryClient

@Resource
private DiscoveryClient discoveryClient;

@GetMapping(value = "/payment/discovery")
    public Object discovery() {
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info("****element:" + element);
        }

        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info("****instance:" + instance.getServiceId() + ", Host:" + instance.getHost() + ", Port:" + instance.getPort() + ",Uri:" + instance.getUri());
        }
        return this.discoveryClient;
    }
```





## Eureka自我保护

### 故障现象

> 保护模式主要用于一组客户端和Eureka Server之间 存在网络分区场景下的保护.一旦进入保护模式
>
> Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。

如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式：

```eureka
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. 
RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE 

```



### 导致原因

> 某时刻某一个微服务不可用了,Eureka不会立刻清理,依旧会对该微服务的信息进行保存
>
> 属于CAP里面的AP分支

#### 为什么会产生Eureka自我保护机制？

为了防止EurekaClient可以正常运行，但是 与 EurekaServer网络不通情况下，EurekaServer不会立刻将EurekaClient服务剔除

#### 什么是自我保护模式？

默认情况下，**如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）**。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。


在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。
它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话讲解：好死不如赖活着

综上**，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。**



### 怎么禁止自我保护

出厂默认,自我保护机制是开启的

server7001配置

```yaml
  server:
    #关闭自我保护机制，保证不可用服务被及时剔除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

![image-20220422105638442](http://qiliu.luxiaobai.cn/img/image-20220422105638442.png)



client8001配置

```yaml
  instance:
    instance-id: payment8001
      ##访问路径可以显示IP地址
    prefer-ip-address: true
      #Eureka客户端向服务端发送的心跳的时间间隔，单位为秒（默认是30秒）
    lease-renewal-interval-in-seconds: 1
    ##Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒（默认90秒），超时将剔除
    lease-expiration-dureation-in-seconds: 2
```

















































































































































































































































































































