---
linux ufw防火墙配置
---

[toc]

----

### 安装

`sudo apt-get install ufw`



### 使用

#### 启用

```shell
sudo ufw enable
sudo ufw default deny
```

作用：开启了防火墙并随系统启动同时关闭所有外部对本机的访问（本机访问外部正常）。

#### 关闭

`sudo ufw disable`

#### 状态

`sudo ufw status`



#### 开启/禁用相应端口或服务

```shell
sudo ufw allow 80 ##允许外部访问80端口
sudo ufw delete allow ##禁止外部访问80端口
sudo ufw allow from 192.168.1.1 ##允许此IP访问所有的本机端口
sudo ufw deny smtp   ##禁止外部访问smtp服务
sudo ufw delete allow smtp ##删除上面建立的某条规则
sudo ufw deny proto tcp from 10.0.0.0/8 to 192.168.0.1 port 22 ###禁止所有的TCP流量从10.0.0.0/8到192.168.0.1地址的22端口

####允许所有RFC198网格（局域网/无线局域网的）访问这个主机（/8,/16,/12是一种网格分级）：
sudo ufw allow from 10.0.0.0/8
sudo ufw allow from 172.16.0.0/12
sudo ufw allow from 192.168.0.0/16

##限制访问外网
sudo ufw default deny outgoing
###允许访问外网的某个IP
sudo ufw allow out proto tcp to 192.168.1.1 port https  ###允许访问所有端口可以不加port https
###允许访问域名解析用的DNS端口（53）
sudo ufw allow out to any port 53

###恢复允许访问所有外网IP
sudo ufw default allow outgoing
```



#### 推荐设置

```shell
sudo apt-get install ufw
sudo ufw enable
sudo ufw default deny
```

这样设置更安全，如果有特殊需要，可以使用sudo ufw allow 开启相应服务

[UFW中文指南](https://wiki.ubuntu.com.cn/Ufw%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97)



