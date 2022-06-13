---
Consul服务注册与发现
---

[toc]

----

# Consul简介

>Consul 是一套开源的**分布式服务发现和配置管理系统**，由 HashiCorp 公司 用 Go 语言开发 。 

它提供了微服务系统中的**服务治理、配置中心、控制总线**等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。 



## 优点

- 基于RAFT协议,比较简洁
- 支持健康检查
- 支持HTTP和DNS协议支持跨数据中心的WAN集群
- 提供图形界面
- 跨平台,支持Linux、Mac、Windows



## 特点

- **服务发现**：consul的客户端可以注册服务，例如`应用程序编程接口`或`mysql数据库`，其他客户端可以使用consul来发现给定服务的提供者。**使用DNS或HTTP**，应用程序可以很容易地找到它们所依赖的服务。
- **健康检查**：consul客户机可以提供任意数量的运行状况检查，要么与给定的服务关联（“web服务器是否返回200ok”），要么与本地节点（“内存利用率是否低于90%”). **支持多种方式,HTTP、TCP、Docker、Shell脚本定制化监控**
- **KV仓库**：应用程序可以将consul的**分层键/值存储用于任何目的**，包括动态配置、功能标记、协调、领导人选举等。简单的httpapi使其易于使用。
- **安全服务通信**：consul可以为服务生成和分发TLS证书，以建立相互TLS连接。[意图](https://www.consul.io/docs/connect/intentions)可用于定义允许哪些服务进行通信。服务分段可以很容易地管理，意图可以实时更改，而不是使用复杂的网络拓扑和静态防火墙规则。
- **多数据中心**：c**onsul支持多个现成的数据中心**。这意味着consul的用户不必担心构建额外的抽象层来扩展到多个区域。





## 下载安装

[官网网址](https://www.consul.io/downloads)

```shell
###Mac
brew tap hashicorp/tap
brew install hashicorp/tap/consul

##CentOS
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install consul

###Ubuntu
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install consul
```



## 相关操作

```shell
###查看consul的安装路径
whereis consul

###启动consul客户端 通过浏览器访问8500端口,即可以看到consul的主界面
consul agent -dev -ui -client=0.0.0.0

##以server方式来启动consul
consul agent -server -ui -bootstrap-expect=3 -data-dir=/tmp/consul -node=consul-1 -client=0.0.0.0 -bind=127.0.0.1 -datacenter=dc1

##查看当前consul中所有节点成员
consul members
```

