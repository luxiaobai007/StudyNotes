---
title: 全面了解Nginx如何配置
---

[toc]



# 介绍

## Nginx

Nginx是一个高性能的HTTP和反向代理Web服务器.同时也提供了IMAP/POP3/SMTP服务.占有内存少、并发能力强。



### **正向代理**

在客户端（浏览器）配置代理服务器，通过代理服务器进行互联网访问



### **反向代理**

客户端对代理是无感知的，不需要任何配置就可以访问。只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址。



### 负载均衡

并发请求时，将请求分发到多个服务器上，将负载分发到不同的服务器



### 动静分离

为加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。 



# Nginx配置文件

## 第一部分: 全局块

从配置文件开始到events块之间的内容，主要会设置一些影响Nginx服务器整体运行的配置指令，主要包括：配置运行Nginx服务器的用户（组）、运行生成的worker process数、进程PID存放路径、日志存放路径和类型以及配置文件的引入等。

```nginx
worker_processes 1; ##worker_processes值越大，可以支持的并发处理量也越多
```



## 第二部分: events

主要影响Nginx服务器与用户的网络连接，常用设置包括是否开启对多workprocess下的网络连接进行序列化，是否允许同时接收多个网络连接，选取那种事件驱动模型来处理连接请求，每个word process可以同时支持的最大连接数等。

```nginx
events {
	worker_connections 768; ##支持的最大连接数
	# multi_accept on;
}
```



## 第三部分:http块

http块包括：http块和server块

**http块： 配置指令包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单链接请求数上限等。** 

**server块：** 

1. 全局server块：本虚拟机主机的监听配置和本虚拟机主机的名称或IP配置
2. location块：一个server块可以配置多个location块。主要作用是基于Nginx服务器接收到的请求字符串（例如server_name/uri-string)，**对虚拟主机名称（也可以是IP别名）之外的字符串（例如前面的/uri-string)进行匹配，对特定的请求进行处理**。**地址定向、数据缓存和应答控制**等功能，以及**第三方模块的配置**也在这里进行。



# Nginx源码安装

```shell
##CentOS 安装
yum -y install gcc
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
wget http://nginx.org/download/nginx-1.9.9.tar.gz
tar -zxvf  nginx-1.9.9.tar.gz
cd nginx-1.9.9
./configure
make
make install
```

**注意**:如果连接不上，检查阿里云安全组是否开放端口，或者服务器防火墙是否开放端口

```shell
#开启
service firewalld start

#重启
service firewalld restart

#关闭
service firewalld stop

#查看防火墙规则
firewall-cmd --list-all

#查询端口是否开放
firewall-cmd --query-port=8080/tcp

#开放80端口
firewall-cmd --permanent --add-port=80/tcp

#移除端口
firewall-cmd --permanent --remove-port=8080/tcp

#重启防火墙（修改配置后要重启防火墙)
firewall-cmd --reload

##参数解释
firwall-cmd:是Linux提供的操作firewall的一个工具：
    --permanent：表示设置为持久
    --add-port：标识添加的端口
```







# Nginx功能模块配置



## 反向代理

### 请求转发配置

```nginx
server{
  listen 8080;
  server_name: 192.168.17.219;
  root /home/project/DIANBoard/build;
  location / {
    proxy_pass http://127.0.0.1:8888;
  }
}
```



使用Nginx反向代理，根据访问的路径跳转到不同端口的服务中，Nginx监听9001

访问http://127.0.0.1:9001/edu/ 直接跳转到127.0.0.1:8080

访问http://127.0.0.1:9001/vod/ 直接跳转到127.0.0.1:8081

```nginx
server {
  listen 9091;
  server_name localhost;
  //允许cros跨域访问
  add_header 'Access-Control-Allow-Origin' '*';
   #proxy_redirect default;
      #跟代理服务器连接的超时时间，必须留意这个time out时间不能超过75秒，当一台服务器当掉时，过10秒转发到另外一台服务器。
   proxy_connect_timeout 10;
  
  location ~ /edu/ {
    proxy_pass http://127.0.0.1:8080;
  }
  
   location ~ /vod/ {
    proxy_pass http://127.0.0.1:8081;
  }
}
```



#### location指令说明

用于匹配URL

```shell
location [ = | ~ | ~* | ^~] uri {
    
}
```

1. =：用于不含正则表达式的URI前，要求请求字符串与URI严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求
2. ~：用于表示URI包含正则表达式，并且区分大小写。
3. ~*：用于表示URI包含正则表达式，并且不区分大小写
4. *~：用于不含正则表达式的URI前，要求Nginx服务器找到标识URI和请求字符串匹配度最高的location后，立即使用此location处理请求，而不再使用location块中的正则URI和请求字符串做匹配。

**注意：如果URI包含正则表达式，则必须要有~ 或者 ~*标识。**



## **负载均衡**



```nginx
http{
  ...
    upstream myserver{
    	ip_hash;
    	server 121.199.76.44:8080;
    	server 121.199.76.44:8081
  }
  
  server{
    listen 7777;
    server_name localhost;
    root /myproject/build;
    location / {
      ....
      proxy_pass http://myserver;
      proxy_connection_timeout 10;
    }
  }
}
```

#### 1.**轮询**（默认）：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

#### 2.**weight**：weight代表权重默认为1，权重越高被分配的客户端越多。

```nginx
upstream myserver{
        server 121.199.76.44:8080 weight=1;
        server 121.199.76.44:8081 weight=1;
    }
```

#### 3.**ip_hash**：每个请求按访问IP的hash结果分配，这样每个请求固定访问一个后端服务器。可以解决session的问题。

```nginx
upstream myserver{
        ip_hash;
        server 121.199.76.44:8080;
        server 121.199.76.44:8081;
    }
```

#### 4.**fair（第三方）**：按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```nginx
upstream myserver{
  server 121.199.76.44;
  server 121.199.76.44;
  fair;
}
```





## 动静分离

将动态跟静态请求分开，不能理解成只是单纯的把动态页面和静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，可以理解成使用Nginx处理静态页面，Tomcat处理动态页面。

动静分离实现角度：

- 纯粹把静态文件独立成单独的域名，放在独立的服务器上
- 动态和静态文件混合在一起发布，通过Nginx来分开。

通过location指定不同的后缀名实现不同的请求转发。通过expires参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。

Expires：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用Expires来缓存），3d，表示3天之内访问这个URL，发送一个请求，比对服务器改文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304， 如果有修改，则直接从服务器重新下载，返回状态码200。

![截图](http://qiliu.luxiaobai.cn/img/%E6%88%AA%E5%9B%BE.png)



创建data目录，分别放data/a.html和image/1.jpeg

```nginx
location /www/ {
    root /data/;
    index index.html index.htm;
}

location /image/ {
    root /data/;
    autoindex on; #列出当前文件中的内容 
}
```





## Nginx限流

### **限制访问评率(正常流量)**

采用 ngx_http_limit_req_module模块来限制请求的访问频率，基于漏桶算法原理实现

使用 nginx limit_req_zone 和 limit_req 两个指令，限制单个IP的请求处理速率

```nginx
http{
    limit_req_zone $binary_remote_addr zone=serviceRateLimit:10m rate=10r/s  //每秒最多处理10个请求
}

server{
    location / {
        limit_req zone=servicelRateLimit;
        proxy_pass http://upstream_clusterl;
    }
}
```

**语法:limit_req_zone key zone rate**

- **key**:定义限流对象,binary_remote_addr是一种key,表示基于remote_addr(客户端IP)来做限流,binary_的目的是压缩内存占用量
- **zone**:定义共享内存区来存储访问信息， myRateLimit:10m 表示一个大小为10M，名字为myRateLimit的内存区域。1M能存储16000 IP地址的访问信息，10M可以存储16W IP地址访问信息。
- **rate**:用于设置最大访问速率，rate=10r/s 表示每秒最多处理10个请求。Nginx 实际上以毫秒为粒度来跟踪请求信息，因此 10r/s 实际上是限制：每100毫秒处理一个请求。这意味着，自上一个请求处理完后，若后续100毫秒内又有请求到达，将拒绝处理该请求



### **限制并发连接数(突发流量)**

按上面的配置在流量突然增大时，超出的请求将被拒绝，无法处理突发流量，那么在处理突发流量的时候，该怎么处理呢？Nginx提供了 burst 参数来解决突发流量的问题，并结合 nodelay 参数一起使用。burst 译为突发、爆发，表示在超过设定的处理速率后能额外处理的请求数。

```nginx
http{
    limit_req_zone $binary_remote_addr zone=serviceRateLimit:10m rate=10r/s  //每秒最多处理10个请求
}

server{
    location / {
        limit_req zone=servicelRateLimit burst=20 nodelay;
        proxy_pass http://upstream_clusterl;
    }
}
```

burst=20 nodelay表示这20个请求立马处理，不能延迟，相当于特事特办。不过，即使这20个突发请求立马处理结束，后续来了请求也不会立马处理。burst=20 相当于缓存队列中占了20个坑，即使请求被处理了，这20个位置这只能按 100ms一个来释放。这就达到了速率稳定，但突然流量也能正常处理的效果。



### **限制并发连接数**

ngx_http_limit_conn_module模块提供了对资源连接数进行限制的功能，使用 limit_conn_zone 和 limit_conn 两个指令

```nginx
http{
    limit_conn_zone $binary_remote_addr zone=perip:10m;
    limit_conn_zone $server_name zone=perserver:10m;
}

server{
    ...
    limit_conn perip 20;     //限制单个IP同时最多能持有10连接
    limit_conn perserver 100; //限制server的最大连接数
}
```

- limit_conn perip 20：对应的key是 $binary_remote_addr，表示限制单个IP同时最多能持有20个连接。
- limit_conn perserver 100：对应的key是 $server_name，表示虚拟主机(server) 同时能处理并发连接的总数。注意，只有当 request header 被后端server处理后，这个连接才进行计数。





## 缓存

### **1、浏览器缓存,静态资源缓存用expire**

```nginx
location ~ .*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|mp4|ogg|ogv|webm)$ {
    expires 7d;
}
location ~ .*\.(?:js|css)$ {
    expires 7d;
}
```



### 2、代理层缓存

```nginx
proxy_cache_path /data/cache/nginx/ levels=1:2 keys_zone=cache:512m inactive = 1d max_size=8g;
location / {
    location ~ \.(htm|html)?$ {
        proxy_cache cache;
        proxy_cache_key $uri$is_args$args;    //以此变量值做HASH,作为KEY
        add_header X-Cache $upstream_cache_status;
        proxy_cache_valid 200 10m;
        proxy_cache_valid any 1m;
        proxy_pass http://real_server;
       proxy_redirect off; 
    }
    location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)${
        root /data/webapps/edc;
        expires 3d;
        add_header Static Nginx-Proxy;
    }
}
```





## 黑白名单

### 1、不限流白名单

```nginx
geo $limit {
    122.16.11.0/24 0;
}

map $limit $limit_key{
    1 $binary_remote_addr;
    0 "";
}

limit_req_zone $limit_key zone=mylimit:10m rate=1r/s;

location / {
    limit_req zone=mylimit burst=1 nodelay;
    proxy_pass http://serveic3Cluster;
}
```



### 2、黑名单

```nginx
location / {
    deny 10.52.119.21;
    deny 11.12.123.1/24;
    allow 10.1.1.0/16;
    allow 1001:0dby::/32;
    deny all;
}
```







## **Nginx配置高可用的集群**

![高可用nginx集群](http://qiliu.luxiaobai.cn/img/%E9%AB%98%E5%8F%AF%E7%94%A8nginx%E9%9B%86%E7%BE%A4.png)

**Keepalived**

- 工作原理：vrrp协议实现
- 工作方式：抢占式和非抢占式

高可用：两台业务系统启动相同服务，如果有一台宕机，另一台自动接管，即高可用

安装配置keepalived：

```nginx
#两条服务器安装
yum install keepalived -y

##Ubuntu 安装keepalived
sudo apt-get install libssl-dev
sudo apt-get install openssl
sudo apt-get install libpopt-dev
sudo apt-get install keepalived

#配置keepalived.conf
sudo vim /etc/keepalived/keepalived.conf
global_defs {   #全局定义
    notifaction_email {
        1575018859@qq.com
    }
    notification_email_from sns-lvs@gmail.com
    smtp_server smtp.hysec.com
    smtp_connection_timeout 30
    router_id nginx_master         #设置Nginx master的ID，在一个网络应该是唯一的,访问到主机
}
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh"   ##最后手动执行下次脚本，以确保此脚本能够正常执行
    interval 2                                ##(检测脚本执行的间隔，单位是秒)
    weight 2
}
vrrp_instance VI_1 {
    state MASTER              #指定keepalived的角色，MASTER为主，BACKUP为备
    interface eth0           #当前进行vrrp通讯的网络接口卡（当前centos的网卡),使用 ifconfig查看
    virtual_router_id 66         #虚拟路由编号，主从要一致
    priority  100             #优先级，数值越大，获取处理请求的优先级越高
    advert_int  1               #检查间隔，默认为ls（vrrp组播周期秒数)
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script{
        chk_http_port           #调用检测脚本
    }
    virtual_ipaddress {
        121.199.76.44              ##定义虚拟IP（VIP），可多设，每行一个
    }
}


##创建Nginx服务监控脚本/usr/local/nginx/check_nginx.sh（主从服务器一致）
sudo vim /usr/local/src/check_nginx_pid.sh
#!/bin/bash

A=`ps -C nginx --no-header |wc -l`
if [ $A -eq 0 ];then
    #/usr/local/nginx/sbin/nginx            #重启Nginx
    sudo service nginx restart
    if [ `ps -C nginx --no-header | wc -l` -eq 0];then  #Nginx重启失败，则停掉keepalived服务
        killall keepalived
    fi
fi  

   
 
 ###另外检测脚本
 #!/bin/sh
 nginxpid=$(ps -C nginx --no-header|wc -l)
 #1.判断Nginx是否存活，如果不存活则尝试启动Nginx
 if [$nginxpid -eq 0 ];then
     systemctl start nginx
     sleep 3
     #2.等待3秒后再次获取一次Nginx状态
     nginxpid=$(ps -C nginx --no-header|wc -l)
     #3.再次进行判断,如Nginx还不存活则停止keepalived,让地址进行漂移,并退出脚本
     if[$nginxpid -eq 0];then
         systemctl stop keepalived
     fi
 fi  
```

日志存放位置：/var/log/messages

程序目录：/etc/keepalived/keepalivd.conf

 启动nginx和keepalived

```shell
systemctl start keepalived.service
```

一个master和多个worker的好处

1. 可以使用nginx  -s reload 热部署,利用nginx进行热部署操作
2. 每个worker是独立的进程,如果有其中的一个worker出现问题,其他worker独立的,继续进行争抢,实现请求过程,不会造成服务中断 
3. 设置多少个worker ?worker数和服务器的cpu数相等时最为适宜的 





**连接数worker_connection**

发送请求,占有worker的几个连接数:2个(静态)或者4个(动态)

nginx有一个master,  有4个worker,每个worker支持最大的连接数据1024,支持的最大并发数多少?

\##普通的静态访问最大并发数是:worker_connections * worker_processes / 2,

\##而如果是http作为反向代理,最大并发数:worker_connections * worker_processes / 4







```nginx
worker_processes  1;
# 错误日志路径，根据本机情况选择
error_log  logs/error.log;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    # 日志格式化，相关参数可上网查找
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       90;
    server_name  127.0.0.1;

        # 正常日志路径，根据本机情况选择
        access_log  logs/access.log;
        # 前端静态页面根目录 
        root "F:/EC3_TEST/ec-demo/www";
        index  index.html index.htm;#默认起始页
        # 后端代理转发
        location ^~ /ec-demo {
            proxy_pass http://127.0.0.1:8080/ec-demo-test;
            break;
        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|ico)$ {
            break;
        }

        location ~ .*\.(js|css|map|json|html)$ {
            break;
        }

        location ~ .*\.(woff|ttf|eot|svg|woff2)$ {
            break;
        }
        # 静态页面url后面带上.html后缀
        location / {
            rewrite   ^(.*?)/?$ /$1.html;
            break;
        }
    }
}

```



```nginx

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    server {
      listen 80;
      autoindex on;
      gzip on;
      gzip_min_length 1k;
      gzip_comp_level 9;
      gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
      gzip_vary on;
      gzip_disable "MSIE [1-6]\.";

      charset utf-8;
      #root /Users/marcos/Documents/工作文档/初始化/ec-demo/www;
      root /Users/lushengyang/Desktop/gillion/ec-demo/www;
      location ^~ /ec-demo {
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #proxy_pass  http://backendUpstream/http/ec-demo;
            proxy_pass http://127.0.0.1:8080/ec-demo;
            break;
         }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
          break;
        }

        location ~ .*\.(js|css|map|json|html)$ {
          break;
        }

        location ~ .*\.(js|css|map|json|html)$ {
          break;
        }

        location ~ .*\.(woff|ttf)$ {
          break;
        }

        location / {
          rewrite   ^(.*?)/?$ /$1.html;
              try_files $uri $uri/ /index.html;
          break;
        }
      }
}
```

