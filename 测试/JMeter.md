# **下载**

1、官网下载：https://jmeter/apache.org/   ----apache-jmeter-5.3.zip

## **目录讲解**

```shell
bin:核心可执行文件，包含配置
    jmeter.bat:windows启动文件
    jmeter:mac或者linux启动文件
    jmeter-server:mac或者Linux分布式压测使用的启动文件
    jmeter-server.bat:Windows分布式压测使用的启动文件
    jmeter.properties:核心配置文件
 extras:插件扩展的包
 lib:核心的依赖包  
```

  

### brew安装JMeter

```shell
brew search jmeter ##搜索是否存在JMeter安装包
brew install jmeter
jmeter  ###打开jmeter软件
```



## **GUI菜单栏主要组件**

**添加->threads->线程组（控制总体并发）**

```shell
线程数：虚拟用户数。一个虚拟用户占用一个进程或线程
准备时长（Ramp-Up Period(in seconds)）：全部线程启动的时长，比如100个线程，20秒，则表示20秒内100个线程都启动
循环次数：每个线程发送的次数，例如值为5，100个线程，则会发送500次请求，可以勾选永远循环
```

![jmeter](http://qiliu.luxiaobai.cn/img/jmeter.png)

**线程组->添加Sampler(采样器)->Http(一个线程组下面可以增加几个Sampler）**

```shell
名称：采样器名称
注释：对这个采样器的描述
wbe服务器：
    默认协议是http
    默认端口是80
    服务器名称或ip：请求的目标服务器名称或ip地址
路径：服务器URL

```

![jmeter2](http://qiliu.luxiaobai.cn/img/jmeter2.png)



## **查看测试结果**

新增汇总报告：线程组->添加->监听器->汇总报告（Aggregate Report）

```shell
lable：sampler的名称
Samples(样本)：一共发出多少请求，例如10个用户，循环10次，则是100
Average(平均值)：平均响应时间
Median(中位数)：中位数，也就是50%用户的响应时间

90% Line：90% 用户的响应不会超过该时间 (90% of the samples took no more than this time.The remaining samples at least as long this)
95% Line：95%用户的响应不会超过该时间
99% Line：99%用户的响应不会超过该时间
min(最小值)：最小响应时间
max(最大值)：最大响应时间

Error%(异常)：错误的请求的数量/请求的总数
Throughput(吞吐量)：吞吐量--默认情况下表示每秒完成的请求数（Request per Second）可类比为qps、tps KB、Sec：每秒接收数据量
```

![jmeter3](http://qiliu.luxiaobai.cn/img/jmeter3.png)