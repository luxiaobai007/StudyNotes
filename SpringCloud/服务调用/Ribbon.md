---
Ribbon负载均衡服务调用
---

[toc]

----

# Ribbon概述

> Spring Cloud Ribbon是基于Netflix Ribbon实现的一套 客户端 负载均衡的工具 。 

Ribbon是Netflix发布的开源项目，主要功能是**提供 客户端的软件负载均衡算法和服务调用**。 Ribbon客户端组件提供一系列**完善的配置项如连接超时，重试**等。简单的说，就是**在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器**。我们很容易使用Ribbon实现自定义的负载均衡算法。 

[官网地址](https://github.com/Netflix/ribbon/wiki/Getting-Started)

*Ribbon目前也进入维护模式*,未来替换方案



## LB(负载均衡)

用于将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）

常见的负载均衡有软件Nginx，LVS，硬件 F5等。 



==Ribbon本地负载均衡客户端 VS Nginx服务端负载均衡区别==

 **Nginx是服务器负载均衡**，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即**负载均衡是由服务端实现的。** 

 **Ribbon本地负载均衡**，在调用微服务接口时候，**会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。**



### 集中式LB

> 即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；



### 进程内LB 

> 将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。 

Ribbon就属于进程内LB ，它只是一个类库， 集成于消费方进程 ，消费方通过它来获取到服务提供方的地址。 





# Ribbon负载均衡演示

## 架构说明

![image-20220426205548365](http://qiliu.luxiaobai.cn/img/image-20220426205548365.png)

Ribbon在工作时分成两步 :

1. 先选择 EurekaServer ,它优先选择在同一个区域内负载较少的server. 
2. 再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。 

其中Ribbon提供了多种策略：比如**轮询、随机和根据响应时间加权**。 

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId> 
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId> 
</dependency> 
```



## RestTemplate

[官网地址](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html )

### getForObject

返回对象为响应体中数据转化成的对象，基本上可以理解为Json

```java
@GetMapping("/consumer/payment/get/{id}")
public CommonResult<Payment> getPayment(@PathVariable("id")Long id){
  return restTemplate.getForObject(PAYMENT_SRV+"/payment/get" + id, CommonResult.class);
}
```



### getForEntity

返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等 

```java
@GetMapping("/consumer/payment/getForEntity/{id}")
public CommonResult<Payment> getPayment2(@PathVariable("id")Long id){
  ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_SRV+"/payment/get" + id, CommonResult.class);
  if(entity.getStatusCode().is2xxSuccessful()){
    return entity.getBody();
  }else{
    return new CommonResult(444,"fail");
  }
}
```





# Ribbon核心组件IRule

==IRule:根据特定算法中从服务列表中选取一个要访问的服务==

![image-20220426210755451](http://qiliu.luxiaobai.cn/img/image-20220426210755451.png)

- `com.netflix.loadbalancer.RoundRobinRule`: 轮询
- `com.netflix.loadbalancer.RandomRule`:随机
- `com.netflix.loadbalancer.RetryRule`:先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
- `WeightedResponseTimeRule`:对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- `BestAvailableRule`:会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- `AvailabilityFilteringRule`:先过滤掉故障实例，再选择并发较小的实例
- `ZoneAvoidanceRule`:默认规则,复合判断server所在区域的性能和server的可用性选择服务器



==注意配置规则==

官方文档明确给出了警告： 

这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下， 

否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了

![image-20220426212255739](http://qiliu.luxiaobai.cn/img/image-20220426212255739.png)



## Ribbon负载均衡算法

==负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标 ，每次服务重启动后rest接口计数从1开始。==

```
List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE"); 

如：  List [0] instances = 127.0.0.1:8002 

　　　List [1] instances = 127.0.0.1:8001 

8001+ 8002 组合成为集群，它们共计2台机器，集群总数为2， 按照轮询算法原理： 

当总请求数为1时： 1 % 2 =1 对应下标位置为1 ，则获得服务地址为127.0.0.1:8001 

当总请求数位2时： 2 % 2 =0 对应下标位置为0 ，则获得服务地址为127.0.0.1:8002 

当总请求数位3时： 3 % 2 =1 对应下标位置为1 ，则获得服务地址为127.0.0.1:8001 

当总请求数位4时： 4 % 2 =0 对应下标位置为0 ，则获得服务地址为127.0.0.1:8002 

如此类推...... 
```



















































































































































