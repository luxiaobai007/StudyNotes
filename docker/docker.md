[toc]



# Docker

## 概述

隔离：Docker核心思想！打包装箱！每个箱子是互相隔离的。

Docker通过隔离机制，可以将服务器利用到极致。

在容器技术出来之前，都是实验虚拟机技术

虚拟机：在window中装一个VMware，通过这个软件可以虚拟出来一台电脑或多台电脑！笨重！

虚拟机属于虚拟化技术，Docker容器技术，也是一种虚拟化技术！

```markdown
VM，Linux centos原生镜像（一个电脑！）隔离，需要开启多个虚拟机 几个G 启动几分钟
docker 隔离 镜像（最核心的环境 4M+jdk + mysql)十分小巧，运行镜像即可 几个M kB 秒级启动
```

Docker是基于Go语言开发的 开源项目



## 虚拟机技术缺点

1. 资源占用十分多
2. 冗余步骤多
3. 启动很慢



## 容器化技术

容器化不是模拟的一个完整的操作系统

### 比较Docker和虚拟机技术的不同

- 传统虚拟机，虚拟出一条硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件
- 容器内的应用直接运行在宿主机的内容，容器是没有自己的内核，也没有虚拟我们的硬件，轻便
- 每个容器间是互相隔离，每个容器内都有一个属于自己的文件系统，互不影响



## 优点

- **应用更快速的交付和部署**
  - 传统：一堆帮助文档，安装程序
  - Docker：打包镜像发布测试，一键运行

- **更便捷的升级和扩缩容**

- **更简单的系统运维**
  - 在容器化之后，开发、测试环境高度一致

- **更高效的计算资源利用**
  - Docker是内核级的虚拟化，可以在一个物理机上可以运行很多的容器实例！将服务器的性能压榨到极致



## Docker的基本组成

- **镜像（image)**:docker镜像就好比是一个模板，可以通过这个模板来参加容器服务，Tomcat镜像===> run ==> tomcat01容器（提供服务器)
- **容器（container)**: Docker利用容器技术，独立运行一个或者一个组也有，通过镜像来创建的。启动、停止、删除，基本命令！
- **仓库（repository):** 存放镜像的地方，分为公有仓库和私有仓库





## 安装Docker

```shell
##系统内核查看
root@iZbp13941xpzjmefjge9chZ:~# uname -r
4.4.0-151-generic

##系统版本查看
root@iZbp13941xpzjmefjge9chZ:~# cat /etc/os-release 
NAME="Ubuntu"
VERSION="16.04.6 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.6 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial


###
/var/lib/docker docker默认工作路径

```



## 底层原理

### **Docker是什么工作的？**

Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问！

DockerServer接收到Docker-Client的指令，就会执行这个命令！



#### **Docker为什么比VM快？**

1. docker比虚拟机更少的抽象层
2. docker利用的是宿主机的内核，VM需要是Guest OS

![docker1](http://qiliu.luxiaobai.cn/img/docker1.png)

### **帮助命令**

```shell
docker version    ##显示docker的版本信息
docker info       ##显示docker的系统信息，包括镜像和容器的数量
docker --help     #万能命令
```





## 镜像命令

**docker images**  查看本地主机上的所有镜像

```shell
root@iZbp13941xpzjmefjge9chZ:~# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   3 months ago   13.3kB

REPOSITORY  镜像的仓库源
TAG         镜像的标签
IMAGE ID    镜像的ID
CREATED     镜像创建的时间
SIZE        镜像的大小

docker images --help
##可选项
 -a, --all              #列出所有镜像
 -q, --quiet           #只显示镜像的ID

```



**docker search** 搜索镜像

```shell
root@iZbp13941xpzjmefjge9chZ:~# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11050     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   4192      [OK]       
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   821                  [OK]

##可选项 通过搜索来过滤
--filter=STARS=3000 #搜索出来的镜像STARS大于3000的
root@iZbp13941xpzjmefjge9chZ:~# docker search mysql --filter=STARS=3000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   11050     [OK]       
mariadb   MariaDB Server is a high performing open sou…   4192      [OK]  
```



docker pull下载镜像

```shell
##下载镜像 docker pull 镜像名[:tag]
root@iZbp13941xpzjmefjge9chZ:~# docker pull mysql
Using default tag: latest   ##如果不写tag，默认就是latest
latest: Pulling from library/mysql
b4d181a07f80: Pull complete   ##分层下载，docker image的核心，联合文件系统
a462b60610f5: Pull complete 
578fafb77ab8: Pull complete 
524046006037: Pull complete 
d0cbe54c8855: Pull complete 
aa18e05cc46d: Pull complete 
32ca814c833f: Pull complete 
9ecc8abdb7f5: Pull complete 
ad042b682e0f: Pull complete 
71d327c6bb78: Pull complete 
165d1d10a3fa: Pull complete 
2f40c47d0626: Pull complete 
Digest: sha256:52b8406e4c32b8cf0557f1b74517e14c5393aff5cf0384eff62d9e81f4985d4b  #签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest  ##真实地址

##等价
docker pull mysql
docker pull docker.io/library/mysql:latest
```



**docker rmi** 删除镜像

```shell
root@iZbp13941xpzjmefjge9chZ:~# docker rmi -f 镜像ID ##删除指定的镜像
root@iZbp13941xpzjmefjge9chZ:~# docker rmi -f 镜像ID 镜像ID 镜像ID ##删除多个镜像
root@iZbp13941xpzjmefjge9chZ:~# docker rmi -f $(docker images -aq) ##删除全部的镜像
```



## 容器命令

```shell
docker pull centos
```



### **新建容器并启动**

```shell
docker run [可选参数] image


##参数说明
--name="Name"  容器名字 Tomcat01 Tomcat02  用来区分容器
-d             后台方式运行
-it            使用交互方式运行，进入容器查看内容
-p             指定容器的端口 -p 8080:8080
   -p ip:主机端口:容器端口
  -p 主机端口:容器端口 （常用)
  -p 容器端口
  容器端口
-P            随机指定端口  

 
  
 ###测试、启动并进入
 root@iZbp13941xpzjmefjge9chZ:~# docker run -it centos /bin/bash
[root@51039269bfaa /]# ls ##查看容器内的centos 
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr 

#从容器中退出
[root@51039269bfaa /]# exit
exit
```



### **列出所有运行的容器**

```shell
##docker ps
     #列出当前正在运行的容器
-a   #列出当前正在运行的容器，带出历史运行过的容器
-n=? #显示最近创建的容器
-q   ##只显示容器的编号
root@iZbp13941xpzjmefjge9chZ:~# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@iZbp13941xpzjmefjge9chZ:~# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
51039269bfaa   centos         "/bin/bash"   3 minutes ago    Exited (0) 2 minutes ago              jolly_spence
307fb08f4d58   d1165f221234   "/hello"      53 minutes ago   Exited (0) 53 minutes ago             intelligent_banach
```



### 退出容器

```shell
exit   #直接容器停止并退出
 Ctrl + p + q ##容器不停止退出
```



### **删除容器**

```shell
docker rm 容器ID                  ##删除指定容器，不能删除正在运行的容器，强制删除rm -f
docker rm -f $(docker ps -aq)    ###删除所有容器
docker ps -a -q|xargs docker rm  #删除所有容器
```



### **启动和停止容器的操作**

```shell
docker start 容器ID    #启动容器
docker restart 容器ID  #重启容器
docker stop 容器ID     #停止当前正在运行的容器
docker kill 容器ID     #强制停止当前容器
```



### **常用其他命令**

#### **后台启动容器**

```shell
##命令：docker run -d 镜像名
root@iZbp13941xpzjmefjge9chZ:/home/lushengyang# docker run -d centos

##常见的坑，docker 容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
##Nginx 容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了
```



#### 查看日志

```shell
docker logs -f -t --tail 容器，没有日志

##自己编写一段shell脚本
root@iZbp13941xpzjmefjge9chZ:/# docker run -d centos /bin/sh -c "while true;do echo luxiaobai;sleep 1;done"

root@iZbp13941xpzjmefjge9chZ:/# docker ps
CONTAINER ID   IMAGE     
452edcd4ae75   centos    

##显示日志
-tf            ##显示日志
--tail number  ##要显示日志条数
docker logs -tf --tail 10 452edcd4ae75  ##最新10行
docker logs -tf 452edcd4ae75 ##显示所有日志
```



#### **查看容器中的进程信息**

```shell
### docker top 容器ID
oot@iZbp13941xpzjmefjge9chZ:/# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
452edcd4ae75   centos    "/bin/sh -c 'while t…"   6 minutes ago   Up 6 minutes             kind_wescoff
root@iZbp13941xpzjmefjge9chZ:/# docker top 452edcd4ae75
UID                 PID                 PPID                C                   STIME               TTY                 TIME                
root                12089               12066               0                   00:18               ?                   00:00:00            
root                12636               12089               0                   00:25               ?                   00:00:00 
```



#### **查看镜像的元数据**

```shell
###命令 docker inspect 容器ID

##测试
root@iZbp13941xpzjmefjge9chZ:/# docker inspect 452edcd4ae75
[
    {
        "Id": "452edcd4ae7511d18bb62b45e2b8c7c592d76a83242c15c3fbe86ec5fe7f3d4c",
        "Created": "2021-06-28T16:18:16.797980106Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do echo luxiaobai;sleep 1;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 12089,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-06-28T16:18:17.398169969Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/452edcd4ae7511d18bb62b45e2b8c7c592d76a83242c15c3fbe86ec5fe7f3d4c/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/452edcd4ae7511d18bb62b45e2b8c7c592d76a83242c15c3fbe86ec5fe7f3d4c/hostname",
        "HostsPath": "/var/lib/docker/containers/452edcd4ae7511d18bb62b45e2b8c7c592d76a83242c15c3fbe86ec5fe7f3d4c/hosts",
        "LogPath": "/var/lib/docker/containers/452edcd4ae7511d18bb62b45e2b8c7c592d76a83242c15c3fbe86ec5fe7f3d4c/452edcd4ae7511d18bb62b45e2b8c7c592d76a83242c15c3fbe86ec5fe7f3d4c-json.log",
        "Name": "/kind_wescoff",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/b271f66ca3d5bf59cc3b32a3c5a5aae8dcfd14673afb98f817ece54670d87427-init/diff:/var/lib/docker/overlay2/b3789e675973463071f8a020ddda675b97422797a9bb6f303179bd602485f46c/diff",
                "MergedDir": "/var/lib/docker/overlay2/b271f66ca3d5bf59cc3b32a3c5a5aae8dcfd14673afb98f817ece54670d87427/merged",
                "UpperDir": "/var/lib/docker/overlay2/b271f66ca3d5bf59cc3b32a3c5a5aae8dcfd14673afb98f817ece54670d87427/diff",
                "WorkDir": "/var/lib/docker/overlay2/b271f66ca3d5bf59cc3b32a3c5a5aae8dcfd14673afb98f817ece54670d87427/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "452edcd4ae75",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true;do echo luxiaobai;sleep 1;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "94cec82f5e861de71e1b5e30b8dd7baaa37728ae37ff00bd98684061965e487a",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/94cec82f5e86",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "eb4eac529802feacff450b8d6e304625ad4aa4d1be4a8d3b8c44c3bbd3ab06d9",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "7f0d4182b127999821c0e65e9daca7c0c63b9f43792a0124725de97a8c317581",
                    "EndpointID": "eb4eac529802feacff450b8d6e304625ad4aa4d1be4a8d3b8c44c3bbd3ab06d9",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```



#### **进入当前正在运行的容器**

```shell
####命令
docker exec -it 容器ID bashshell

#测试
root@iZbp13941xpzjmefjge9chZ:/# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
452edcd4ae75   centos    "/bin/sh -c 'while t…"   13 minutes ago   Up 13 minutes             kind_wescoff
root@iZbp13941xpzjmefjge9chZ:/# docker exec -it 452edcd4ae75 /bin/bash
[root@452edcd4ae75 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@452edcd4ae75 /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 16:18 ?        00:00:00 /bin/sh -c while true;do echo luxiaobai;sleep 1;done
root       851     0  1 16:32 pts/0    00:00:00 /bin/bash
root       877     1  0 16:32 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
root       878   851  0 16:32 pts/0    00:00:00 ps -ef

###方式2
docker attach 容器ID
docker attach 452edcd4ae75
正在执行当前的代码....

##docker exec  ##进入容器后开启一个新的终端，可以在里面操作
##docker attach  ##进入容器正在执行的容器，不会启动新的进程
```



#### **从容器内拷贝文件到主机上**

```shell
#####docker cp 容器id:容器内路径 目的的主机路径

##查看当前主机目录小
root@iZbp13941xpzjmefjge9chZ:/home# ls
betaPic  git  lushengyang  luxiaobai.java  project  software
root@iZbp13941xpzjmefjge9chZ:/home# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
56ec6968bcbd   centos    "/bin/bash"   41 seconds ago   Up 40 seconds             strange_brattain

###进入docker容器内部
root@iZbp13941xpzjmefjge9chZ:/home# docker attach 56ec6968bcbd
[root@56ec6968bcbd /]# cd /home/
[root@56ec6968bcbd home]# ls

##在容器内新建一个文件
[root@56ec6968bcbd home]# touch test.java
[root@56ec6968bcbd home]# ls
test.java
[root@56ec6968bcbd home]# exit
exit
root@iZbp13941xpzjmefjge9chZ:/home# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                     PORTS     NAMES
56ec6968bcbd   centos    "/bin/bash"   2 minutes ago   Exited (0) 3 seconds ago             strange_brattain

###将这个文件拷贝到主机
root@iZbp13941xpzjmefjge9chZ:/home# docker cp 56ec6968bcbd:/home/test.java /home
root@iZbp13941xpzjmefjge9chZ:/home# ls
betaPic  git  lushengyang  luxiaobai.java  project  software  test.java
```



### **常用命令小结**

```shell
attach     Attach to a running container              ##当前shell下attach连接指定镜像
  build      Build an image from a Dockerfile           ##通过Dockerfile定制镜像
  commit     Create a new image from a container changes    ##提交当前容器为新的镜像
  cp         Copy files/folders from the containers filesystem to the host path    ##从容器中拷贝指定文件或者目录到宿主机中
  create     Create a new container                          ###创建一个新的容器，同run，但不启动容器
  diff       Inspect changes on a container's filesystem     ##查看docker容器变化
  events     Get real time events from the server             ##从docker服务获取容器实时事件
  exec        Run a command in a running container           ##在已存在的容器上运行命令
  export      Export a container's filesystem as a tar archive ##导出容器的内容流作为一个tar归档文件[对应import]
  history     Show the history of an image                   ##展示一个镜像形成历史
  images      List images                                    ##列出系统当前镜像
  import      Import the contents from a tarball to create a filesystem image   ##从tar包中的内容创建一个新的文件系统映像
  info        Display system-wide information                 ##显示系统相关信息
  inspect     Return low-level information on Docker objects  ###查看容器详细信息
  kill        Kill one or more running containers                ###kill指定docker容器
  load        Load an image from a tar archive or STDIN       ##从一个tar包中加载一个镜像[对应save]
  login       Log in to a Docker registry                     ##注册或登录一个docker源服务器
  logout      Log out from a Docker registry                  ##从当前Docker registry退出
  logs        Fetch the logs of a container                    ##输出当前容器日志信息
  pause       Pause all processes within one or more containers   ##暂停容器
  port        List port mappings or a specific mapping for the container   ##查看映射端口对应的容器内部源端口
  ps          List containers                                     ##列出容器列表
  pull        Pull an image or a repository from a registry   ##从docker镜像源服务器拉取指定镜像或者库镜像
  push        Push an image or a repository to a registry    ##推送指定镜像或者库镜像至docker源服务器
  rename      Rename a container                            ##重新命名容器
  restart     Restart one or more containers                  ###重启运行的容器
  rm          Remove one or more containers                 ##移除一个或多个容器
  rmi         Remove one or more images                    ##移除一个或多个镜像[无容器使用该镜像才可删除，莫非需删除相关容器才可继续或者-f强制删除]
  run         Run a command in a new container               ##创建一个新的容器并运行一个命令
  save        Save one or more images to a tar archive (streamed to STDOUT by default)  ##保存一个镜像为一个tar包[对应load]
  search      Search the Docker Hub for images                 ##在docker hub 中搜索镜像
  start       Start one or more stopped containers                ##启动容器
  stats       Display a live stream of container(s) resource usage statistics  ##显示容器资源使用统计信息的实时流
  stop        Stop one or more running containers                ##停止容器
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE  ##给源中容器打标签
  top         Display the running processes of a container              ##查看容器中运行的进程信息
  unpause     Unpause all processes within one or more containers           ##取消暂停容器
  update      Update configuration of one or more containers              #更新一个或多个容器的配置
  version     Show the Docker version information                     ##查看docker版本号
  wait        Block until one or more containers stop, then print their exit codes  ##截取容器停止时的退出状态值
```



## **相关部署**

### **安装Nginx**

```shell
##1 搜索镜像
##2 下载镜像
##3 运行测试


## docker search nginx
## docker pull nginx

##-d 后台运行
##--name 给容器命名
##-p 外部端口:容器端口
## docker run -d --name nginx01 -p:3344:80 nginx  
4、 docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
18606c6a549f   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 3 seconds   0.0.0.0:3344->80/tcp   nginx01

5、curl localhost:3344


##进入容器内部
root@iZbp13941xpzjmefjge9chZ:/home# docker exec -it nginx01 /bin/bash
root@18606c6a549f:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@18606c6a549f:/# ls
bin   dev		   docker-entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc			 lib   media  opt  root  sbin  sys  usr
```



### **docker安装Tomcat**

```shell
##官方使用
docker run -it --rm tomcat:9.0   ##用完就删除，适合测试

#测试访问没有问题
root@iZbp13941xpzjmefjge9chZ:/home# docker run -d -p 3355:8080 --name tomcat01 tomcat

#进入容器
root@iZbp13941xpzjmefjge9chZ:/home# docker exec -it tomcat01 /bin/bash

##发现问题
  1、Linux命令少了
  2、没有webapps  阿里云镜像的原因，默认是最小的镜像，所有不必要的都删除了
  保证最小可运行环境
```



### **部署es+kibana**

```shell
##es暴露的端口很多
##es十分的耗内存
##es 的数据一般需要放置到安全目录！挂载
##--net somenetwork 网络配置
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag

root@iZbp13941xpzjmefjge9chZ:~# curl localhost:9200
{
  "name" : "564291baaf20",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "nLiUIKo0QIaudjKov7ln1A",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}



docker stats 查看CPU的状态
##增加内存的限制 修改配置文件 -e 环境配置修改
root@iZbp13941xpzjmefjge9chZ:~# docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xms512m" elasticsearch:7.6.2
```



### **可视化**

**portainer docker图形化界面管理工具 后台面板**

```shell
docker run -d -p 8088:9000 \ --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```



# **容器数据卷**

容器之间可以有一个数据共享的技术！Docker容器中产生的数据，同步到本地！

这就是卷技术！目录的挂载，将我们容器内的目录，挂载到Linux上面

![docker2](http://qiliu.luxiaobai.cn/img/docker2.png)

**容器的持久化和同步操作！容器间也是可以数据共享的**





## 使用数据卷

**直接使用命令来挂载 -v**

```shell
docker run -it -v 主机目录:容器内目录

#root@iZbp13941xpzjmefjge9chZ:/home#  docker run -it -v /home/test:/home centos /bin/bash
root@iZbp13941xpzjmefjge9chZ:/home/test# docker inspect d46651ae723b
```

![docker3](http://qiliu.luxiaobai.cn/img/docker3.png)

测试 文件同步

![docker4](http://qiliu.luxiaobai.cn/img/docker4.png)



## MySQL

mysql的数据持久化的问题

```shell
#获取镜像
root@iZbp13941xpzjmefjge9chZ:/# docker pull mysql:5.7

#运行容器，需要做数据挂载 
#安装启动MySQL，需要配置密码
#官方测试
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

##启动自己的
-d 后台运行
-p 端口映射
-v 卷挂载
-e 环境配置
--name 容器名字
root@iZbp13941xpzjmefjge9chZ:/home# docker run -d -p 3310:3306 -v /home/dockerImageConfig/mysql/conf:/etc/mysql/conf.d -v  /home/dockerImageConfig/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

##启动成功后用Navicat连接测试

##在本地测试成绩一个数据库，查看映射路径是否OK
```



## 具名挂载和匿名挂载

### 匿名挂载

```shell
-v 容器内路径
root@iZbp13941xpzjmefjge9chZ:/home# docker run -d -P --name nginx01 -v /ect/nginx nginx

#查看所有卷的情况
root@iZbp13941xpzjmefjge9chZ:/home# docker volume ls
local     3b0175e0b56dac42d8598d027d67d85442218d595e60be2e163c9fc5344427d7

#这种就是匿名挂载，在-v只写了容器内的路径，没有写容器外的路径
```

![docker5](http://qiliu.luxiaobai.cn/img/docker5.png)



### 具名挂载

```shell
root@iZbp13941xpzjmefjge9chZ:/home# docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
c7224fa5c8e2b6f3512456a6f6af8aac662485ea712eb80b0c35e4b35e9e2547
root@iZbp13941xpzjmefjge9chZ:/home# docker volume ls
DRIVER    VOLUME NAME
local     3b0175e0b56dac42d8598d027d67d85442218d595e60be2e163c9fc5344427d7
local     890c5abb31ab133d2d1a2dbd98b5f4cb119fbdee4a687ba4743f8a38db8f1e45
local     e1c62888504bb02c3b2f48895c14720b016a732f44a230efdbca0ad09528251a
local     juming-nginx

#通过-v 卷民:容器内路径
查看一下卷
root@iZbp13941xpzjmefjge9chZ:/home# docker volume inspect juming-nginx
[
    {
        "CreatedAt": "2021-06-30T00:53:32+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]
```

所有的docker容器内的卷，没有指定目录的情况下都在/var/lib/docker/volumes/juming-nginx

 通过具名挂载可以方便的找到我们的一个卷，大多数情况在使用具名挂载

```shell
##如何确定具名挂载还是匿名挂载，还是指定路径挂载
-v 容器内路径               #匿名挂载
-v 卷名:容器内路径           #具名挂载
-v /宿主机路径:容器内路径     #指定路径挂载
```

### 拓展

```shell
##通过-v容器内路径: ro rw 改变读写权限
ro readonly  #只读
rw readwrite #可读可写

#一旦这个设置了容器权限，容器对我们挂载出来的内容就有限定了
root@iZbp13941xpzjmefjge9chZ:/# docker run -d -P --name nginx02 -v juming-naing:/etc/nginx:ro nginx
root@iZbp13941xpzjmefjge9chZ:/# docker run -d -P --name nginx02 -v juming-naing:/etc/nginx:rw nginx

#ro 说明只能通过宿主机来操作，容器内部是无法操作！
```





# 初始Dockerfile

Dockerfile就是用来构建docker镜像的构建文件！命令脚本

通过这个脚本可以生成镜像，镜像是一层一层的，脚本一个个的命令，每个命令都是一层！

```shell
#创建一个dockerfile文件，名字随机，建议Dockerfile
##文件中的内容，指令（大写）参数
FROM centos

VOLUME ["/volume01","/volume02"]  ##数据卷目录 匿名挂载

CMD echo "---------end-------"
CMD /bin/bash


docker build -f /home/dockerImageConfig/docker-test-volume/dockerfile1 -t luxiaobai/centos:1.0 .
#这里的每个命令，就是镜像的一层
```

![docker6](http://qiliu.luxiaobai.cn/img/docker6.png)

![docker7](http://qiliu.luxiaobai.cn/img/docker7.png)



## 构建dockerFile步骤

1. 编写一个dockerfile文件
2. docker build构建成为一个镜像
3. docker run运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库)



### DockerFile构建过程

```shell
FROM                  # 基础镜像，一切从这里开始构建 centos
MAINTAINER            # 镜像是谁写的，姓名+邮箱
RUN                   # 镜像构建的时候需要运行的命令
ADD                   # 步骤，Tomcat镜像，这个Tomcat压缩包！添加内容
WORKDIR               # 镜像的工作目录
VOLUME                # 挂载的目录
EXPOSE                # 暴露端口配置
CMD                   # 指定这个容器启动的时候运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT            # 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD              # 当构建一个被继承DockerFile 这个时候就会运行ONBUILD 的指令，触发指令。
COPY                 # 类似ADD,将我们文件拷贝到镜像中
ENV                  # 构建的时候设置环境变量！
```

![docker10](http://qiliu.luxiaobai.cn/img/docker10.png)



### 基础知识

1. 每个保留关键字(指令)都是必须是大写字母
2. 执行从上到下顺序执行
3. \#表示注释
4. 每一个指令都会创建提交一个新的镜像层，并提交！

![docker11](http://qiliu.luxiaobai.cn/img/docker11.png)

dockerfile是面向开发的，发布项目，做镜像，就需要编写dockerfile文件

Docker镜像逐渐成为企业交付的标准。

开发、部署、运维

**DockerFile**：构建文件，定义了一切的步骤，源代码

**DockerImages**：通过DockerFile构建生成的镜像，最终发布和运行的产品

**Docker容器**：容器就是镜像运行起来提供服务

#### 实战测试

Docker Hub中99%镜像都是从这个基础镜像过来的FROM scratch,然后配置需要的软件和配置来进行的构建

![docker12](http://qiliu.luxiaobai.cn/img/docker12.png)

#### 创建一个自己的centos

```shell
#1、编写dockerfile的文件
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# vim mydockerfile-centos
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# cat mydockerfile-centos 
FROM centos
MAINTAINER luxiaobai<1575018859@qq.com>  ##制作者信息

ENV MYPATH /usr/local  ##默认目录
WORKDIR $MYPATH       ##工作目录

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80                ##暴露的端口

CMD echo $MYPATH
CMD echo "------end------"
CMD /bin/bash

##2、通过文件构建镜像
##命令 docker build -f dockerfile文件路径 -t 镜像名:[tag]
Successfully built 3c5ea5d5e56f
Successfully tagged mycentos:0.1


#3、测试运行
```

##### 列出本地进行的变更历史

![docker13](http://qiliu.luxiaobai.cn/img/docker13.png)



### CMD和ENTRYPOINT区别

```shell
CMD                   # 指定这个容器启动的时候运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT            # 指定这个容器启动的时候要运行的命令，可以追加命令
```

#### **测试cmd**

```shell
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# vim dockerfile-cmd-test
FROM centos
CMD ["ls","-a"]

#构建镜像
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# docker build -f dockerfile-cmd-test  -t cmdtest .

#run运行，发现ls -a命令生效
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# docker run 5adf8cc86d7e


#想追加一个命令 -l ls -al
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# docker run 5adf8cc86d7e -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.

#cmd的清理下 -l 替换了CMD["ls","-a"]命令， -l不是命令所以报错
```



#### **测试**ENTRYPOINT

```shell
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# cat dockerfile-cmd-entorypoint 
FROM centos
ENTRYPOINT ["ls","-a"]root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# docker build -f dockerfile-cmd-entorypoint -t entorypoit-test .

root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# docker run fa5f02c7f24a
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

#追加命令，是直接拼接在我们的ENTRYPOINT命令的后面
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/dockerfile# docker run fa5f02c7f24a -l
total 56
drwxr-xr-x   1 root root 4096 Jun 30 17:31 .
drwxr-xr-x   1 root root 4096 Jun 30 17:31 ..
-rwxr-xr-x   1 root root    0 Jun 30 17:31 .dockerenv
lrwxrwxrwx   1 root root    7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x   5 root root  340 Jun 30 17:31 dev
drwxr-xr-x   1 root root 4096 Jun 30 17:31 etc
drwxr-xr-x   2 root root 4096 Nov  3  2020 home
lrwxrwxrwx   1 root root    7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Nov  3  2020 lib64 -> usr/lib64
drwx------   2 root root 4096 Dec  4  2020 lost+found
drwxr-xr-x   2 root root 4096 Nov  3  2020 media
drwxr-xr-x   2 root root 4096 Nov  3  2020 mnt
drwxr-xr-x   2 root root 4096 Nov  3  2020 opt
dr-xr-xr-x 118 root root    0 Jun 30 17:31 proc
dr-xr-x---   2 root root 4096 Dec  4  2020 root
drwxr-xr-x  11 root root 4096 Dec  4  2020 run
lrwxrwxrwx   1 root root    8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Nov  3  2020 srv
dr-xr-xr-x  13 root root    0 Jun 29 17:43 sys
drwxrwxrwt   7 root root 4096 Dec  4  2020 tmp
drwxr-xr-x  12 root root 4096 Dec  4  2020 usr
drwxr-xr-x  20 root root 4096 Dec  4  2020 var
```



#### Tomcat 镜像

```shell
1、准备镜像文件tomcat压缩包，jdk的压缩包

2、编写dockerfile文件,官方命令 Dockerfile build会自动寻找这个文件
RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_231
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.48
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.48
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.48/bin/startup.sh && tail -F /url/local/apache-tomcat-9.0.48/bin/logs/catalina.out
```

#### 构建镜像

```shell
docker build -t diytomcat .

docker run -d -p 9090:8080 --name luxiaobaitomcat -v /home/software/build/tomcat/test:/usr/local/apache-tomcat-9.0.48/webapps/test -v /home/software/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.48/logs diytomcat
```

#### 启动镜像

```shell
docker run -d -p 9090:8080 --name luxiaobaitomcat -v /home/software/build/tomcat/test:/usr/local/apache-tomcat-9.0.48/webapps/test -v /home/software/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.48/logs diytomcat
```

访问测试

发布项目（做了卷挂载,可以直接在本地编写项目就可以发布了）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">
         
</web-app>
```

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>hello 路小白</title>
</head>
<body>
<%
System.out.println("-----my test web logs------");
%>
</body>
</html>
```





### 发布自己的镜像

#### **DockerHub**

```shell
dockerID :shengyanglu            password:s15759029165

root@iZbp13941xpzjmefjge9chZ:/home/software/build/tomcat/tomcatlogs# docker login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
  
  root@iZbp13941xpzjmefjge9chZ:/home/software/build/tomcat/tomcatlogs# docker login -u shengyanglu
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

登陆完毕后就可以提交镜像,一步,docker push

```shell
root@iZbp13941xpzjmefjge9chZ:/home/software/build/tomcat/tomcatlogs# docker push luxiaobai/diytomcat:1.0
The push refers to repository [docker.io/luxiaobai/diytomcat]
An image does not exist locally with the tag: luxiaobai/diytomcat

##错误解决
##重新给镜像增加一个tag进行push,自己发布的镜像,尽量增加版本号
root@iZbp13941xpzjmefjge9chZ:/home/software/build/tomcat/tomcatlogs# docker images
REPOSITORY            TAG       IMAGE ID       CREATED             SIZE
diytomcat             latest    23ebb86dc15c   34 minutes ago      691MB
root@iZbp13941xpzjmefjge9chZ:/home/software/build/tomcat/tomcatlogs# docker tag 23ebb86dc15c shengyanglu/tomcat:1.0
root@iZbp13941xpzjmefjge9chZ:~# docker push shengyanglu/tomcat:1.0
The push refers to repository [docker.io/shengyanglu/tomcat]
```



**发布在阿里云镜像服务**

1、登陆阿里云

2、找到容器镜像服务

3、参加命名空间

![docker14](http://qiliu.luxiaobai.cn/img/docker14.png)



4、创建容器镜像

![docker15](http://qiliu.luxiaobai.cn/img/docker15.png)



![docker16](http://qiliu.luxiaobai.cn/img/docker16.png)



![docker17](http://qiliu.luxiaobai.cn/img/docker17.png)

5、浏览阿里云信息

![docker18](http://qiliu.luxiaobai.cn/img/docker18.png)

![docker19](http://qiliu.luxiaobai.cn/img/docker19.png)





# 数据卷容器

## 多个MySQL同步数据

![docker8](http://qiliu.luxiaobai.cn/img/docker8.png)

```shell
启动3个容器
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/docker-test-volume# docker run -it --name docker02 --volumes-from docker01 luxiaobai/centos:1.0


root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/docker-test-volume# docker run -it --name docker02 --volumes-from docker01 luxiaobai/centos:1.0
```

![docker9](http://qiliu.luxiaobai.cn/img/docker9.png)



## 多个MySQL实现数据共享

```shell
root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/docker-test-volume# docker run -d -p 3310:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

root@iZbp13941xpzjmefjge9chZ:/home/dockerImageConfig/docker-test-volume# docker run -d -p 3311:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from mysql01 mysql:5.7
b885ef7bde9d3e64ce85fe6d4fc0ff74c9c44ef984ffb26609cb15544c850b8c
```

容器之间配置信息的时候，数据卷容器的生命周期一直持续到没有容器使用为止

但是一旦持久化到本地，这个时候，本地的数据是不会删除的！





# 镜像

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境的软件，包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

所有应用，直接打包成docker镜像，就可以直接跑起来

## 如何得到镜像

- 从远程仓库下载
- 拷贝
- 制作镜像DockerFile



## Docker镜像加载原理

###  **UnionFS(联合文件系统）**

UnionFS(联合文件系统):Union文件系统（UnionFS)是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite several directories into a single virtual filesystem)。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

**特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含底层的文件和目录**



### Docker进行加载原理

bootfs(boot file system)主要包含bootloader和kernel，bootloader主要是**引导加载**kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此次内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs(root file system),在bootfs之上，包含的就是Linux系统只能的/dev,/proc,/bin,/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

![docker20](http://qiliu.luxiaobai.cn/img/docker20.png)

### 特点

Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部！

这一层就是我们通常说的容器层，容器之下的都叫镜像层！

如何提交一个自己的镜像



### Commit镜像

```docker
docker commit 提交容器成为一个新的副本
docker commit -m="提交的描述信息" -a="作者" 容器ID 目标镜像名:[TAG]
```



## 测试

```shell
root@iZbp13941xpzjmefjge9chZ:~# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED        STATUS        PORTS                                            NAMES
0db517131d07   portainer/portainer   "/portainer"             19 hours ago   Up 19 hours   0.0.0.0:8088->9000/tcp                           portainer01
564291baaf20   elasticsearch:7.6.2   "/usr/local/bin/dock…"   20 hours ago   Up 20 hours   0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch02
root@iZbp13941xpzjmefjge9chZ:~# docker run -d -p 8080:8080 tomcat
b801c78e1d1d0cacafab8270dc1fbd81420e3cd5bc22a31c122d359188f4938b
root@iZbp13941xpzjmefjge9chZ:~# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                            NAMES
b801c78e1d1d   tomcat                "catalina.sh run"        6 seconds ago   Up 6 seconds   0.0.0.0:8080->8080/tcp                           condescending_lewin
0db517131d07   portainer/portainer   "/portainer"             19 hours ago    Up 19 hours    0.0.0.0:8088->9000/tcp                           portainer01
564291baaf20   elasticsearch:7.6.2   "/usr/local/bin/dock…"   20 hours ago    Up 20 hours    0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch02
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat /bin/bash
Error: No such container: tomcat
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it b801c78e1d1d /bin/bash
root@b801c78e1d1d:/usr/local/tomcat# ls
BUILDING.txt  CONTRIBUTING.md  LICENSE	NOTICE	README.md  RELEASE-NOTES  RUNNING.txt  bin  conf  lib  logs  native-jni-lib  temp  webapps  webapps.dist  work
root@b801c78e1d1d:/usr/local/tomcat# cd webapps
root@b801c78e1d1d:/usr/local/tomcat/webapps# ls
root@b801c78e1d1d:/usr/local/tomcat/webapps# cd ..
root@b801c78e1d1d:/usr/local/tomcat# cp -r webapps.dist/* webapps
root@b801c78e1d1d:/usr/local/tomcat# cd webapps
root@b801c78e1d1d:/usr/local/tomcat/webapps# ls
ROOT  docs  examples  host-manager  manager
root@b801c78e1d1d:/usr/local/tomcat/webapps# read escape sequence
root@iZbp13941xpzjmefjge9chZ:~# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                            NAMES
b801c78e1d1d   tomcat                "catalina.sh run"        3 minutes ago   Up 3 minutes   0.0.0.0:8080->8080/tcp                           condescending_lewin
0db517131d07   portainer/portainer   "/portainer"             20 hours ago    Up 20 hours    0.0.0.0:8088->9000/tcp                           portainer01
564291baaf20   elasticsearch:7.6.2   "/usr/local/bin/dock…"   20 hours ago    Up 20 hours    0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch02
root@iZbp13941xpzjmefjge9chZ:~# docker commit -m="add webapps" -a="luxiaobai" b801c78e1d1d tomcat02:1.0
sha256:f109f8c5a5b355420e7f747eed430c6e84ef0eb58e294f179f8ba95453012976
root@iZbp13941xpzjmefjge9chZ:~# docker images
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
tomcat02              1.0       f109f8c5a5b3   5 seconds ago   672MB
tomcat                9.0       b0bf9a4a7c93   3 days ago      667MB
```

![docker21](http://qiliu.luxiaobai.cn/img/docker21.png)

网卡有3个，3个网络

```shell
root@iZbp13941xpzjmefjge9chZ:~# docker run -d -P --name tomcat01 tomcat

##查看容器的内部网络地址 ip addr 容器启动时会得到一个eth0@if73 IP地址，docker分配的！
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
72: eth0@if73: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

root@iZbp13941xpzjmefjge9chZ:~# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.094 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.061 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.060 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.062 ms

linux可以ping通docker 容器内部   
```



### 原理

1. 每启动一个docker容器，docker就会给docker容器分配一个IP，只要安装docker，就会有一个网卡docker0，桥接模式，使用的技术是evth-pair技术！ 

在测试ip addr

![docker22](http://qiliu.luxiaobai.cn/img/docker22.png)

在启动一个容器,又多一对网卡

```docker
docker run -d -P --name tomcat02 tomcat

docker exec -it tomcat02 ip addr
```

![docker23](http://qiliu.luxiaobai.cn/img/docker23.png)

```docker
容器带来的网卡,都是一对对
evth-pair就是一对的虚拟设备接口,他们都是成对出阿信的,一段连着协议,一段彼此相连
evth-pair充当一个桥梁,连接各种虚拟网络设备的
OpenStac, Docker容器之间的连接,ovs的连接,都是使用evth-pair技术
```

测试tomcat01和tomcat02 是否可以ping通

```shell
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat02 ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.109 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.091 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.085 ms

结论:容器和容器之间是可以通信
```

tomcat0和 tomcat02是公用的一个路由器,docker路由的,dock儿会给我们的容器分配一个默认的可用IP

![docker24](http://qiliu.luxiaobai.cn/img/docker24.png)

Docker使用的是LInux的桥接，宿主机中是一个Docker容器的网桥 docker0。

![docker25](http://qiliu.luxiaobai.cn/img/docker25.png)

Docker中所有的网络接口都是虚拟的。虚拟的转发效率较高。

只要容器删除，对应网桥一对就没了

编写了一个微服务，database url=IP， 项目不重启， 数据库IP换掉了



### **--link**

````shell
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known

##如何解决 通过--link
root@iZbp13941xpzjmefjge9chZ:~# docker run -d -P --name tomcat03 --link tomcat02 tomcat
fa9c5be89fe3b09fa4b5e8a399c7a48ac25358565f83ef3def2c0574d1e0d0f7
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.114 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.081 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=3 ttl=64 time=0.114 ms

##反向链接 ping不通
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat02 ping tomcat03
ping: tomcat03: Name or service not known

````

```docker
docker inspect 容器ID
```

tomcat03 配置了Tomcat02的链接

![docker26](http://qiliu.luxiaobai.cn/img/docker26.png)

```shell
##查看hosts配置，
##--link 在hosts配置增加了配置
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat03 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	tomcat02 e39a86c061ad
172.17.0.4	fa9c5be89fe3
```

#### **真实开发不建议使用--link**

自定义网络！不适用docker0



## 自定义网络

查看所有的docker网络

```shell
docker network ls
```

### 网络模式

- bridge：桥接docker（默认，自己也使用bridge模式）
- none：不配置网络
- host：和宿主机共享网络
- container：容器内网络连通（用的少,局限很大）

**测试**

```shell
直接启动的命令 --net bridge这个就是docker0
docker run -d -P --name tomcat01 tomcat
docker run -d -P --name tomcat01 --net bridge tomcat

##docker0特点，默认 域名不能访问， --link可以打通连接


 #创建自定义网络
 #--driver bridge
#--subnet 192.168.0.0/16 192.168.0.2 ~ 192.168.255.255
#--gateway 192.168.0.1
root@iZbp13941xpzjmefjge9chZ:~# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

root@iZbp13941xpzjmefjge9chZ:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cff9f452f41d   bridge    bridge    local
062c6a870b07   host      host      local
e260b2dea0fa   mynet     bridge    local
33d58a5cc027   none      null      local
```

自己的网络

![docker27](http://qiliu.luxiaobai.cn/img/docker27.png)

```shell
[
    {
        "Name": "mynet",
        "Id": "e260b2dea0fa47e9f63dffaf482c7ba4aed40dffc23441b27cba0836eb1952cd",
        "Created": "2021-07-04T17:48:00.219455464+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "3a6fe28de52beca51522bf0d096ab478fe43c6eed12fd467bc2b5f0bf70fd713": {
                "Name": "tomcat-net-02",
                "EndpointID": "781f4501872fb769a33357d0a18eea89aa1e67f45711241913e527cdc093c21a",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "930735f99ea26d39e29eb0dc4ec2e3bae567265afdb554b5d0548cabb5eafd9b": {
                "Name": "tomcat-net-01",
                "EndpointID": "77859429bb2a01e5352dfabc4c40dea0d72e38b4b569cb15eec873f27fe56081",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

##再次测试ping连接
##不适用--link也可以ping名字
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat-net-01 ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.075 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.076 ms
^C
--- 192.168.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.075/0.075/0.076/0.008 ms
root@iZbp13941xpzjmefjge9chZ:~# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.073 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.092 ms
^C
--- tomcat-net-02 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.073/0.082/0.092/0.013 ms
```

自定义网络docker维护好对应的关系，推荐使用自定义网络

**自定义网络好处：**

redis--不同的集群使用不同的网络，保证集群是安全和健康的

mysql--不同的集群使用不同的网络，保证集群是安全和健康的

![docker28](http://qiliu.luxiaobai.cn/img/docker28.png)



## 网络连通

![docker29](http://qiliu.luxiaobai.cn/img/docker29.png)

![docker30](http://qiliu.luxiaobai.cn/img/docker30.png)

```shell
#测试 打通 tomcat01 -mynet
docker network connect mynet tomcat01
 docker network inspect mynet
# tomcat01放到了mynet网络下
#一个容器两个IP地址
#阿里云服务 公网IP 私网IP
```

![docker31](http://qiliu.luxiaobai.cn/img/docker31.png)

> 假设要跨网络操作别人，就需要使用docker network connect 连通！



# 部署Redis集群

```shell
##创建网卡
root@iZbp13941xpzjmefjge9chZ:~# docker network create redis --subnet 172.38.0.0/16
526879679ca5d3b346b59b45591309fb9b987b23863aaa08a87dd0d1d7dabd5e
root@iZbp13941xpzjmefjge9chZ:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cff9f452f41d   bridge    bridge    local
062c6a870b07   host      host      local
e260b2dea0fa   mynet     bridge    local
33d58a5cc027   none      null      local
526879679ca5   redis     bridge    local

 ##通过shell脚本创建6个Redis配置
 for port in $(seq 1 6); \
 do \
 mkdir -p /mydata/redis/node-${port}/conf
 touch /mydata/redis/node-${port}/conf/redis.conf
 cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
 port 6379
 bind 0.0.0.0
 cluster-enabled yes
 cluster-config-file nodes.conf
 cluster-node-timeout 5000
 cluster-announce-ip 172.38.0.1${port}
 cluster-announce-port 6379
 cluster-announce-bus-port 16379
 appendonly yes
 EOF
 done
 
 docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
 -v /mydata/redis/node-1/data:/data \
 -v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
 -d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
 
 docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
 -v /mydata/redis/node-6/data:/data \
 -v /mydata/redis/node-6/conf/redis.conf:/etc/redis/redis.conf \
 -d --net redis --ip 172.38.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
```





# Docker Compose

Docker Compose 管理容器，定义运行多个容器

**步骤**

1. 定义Dockerfile 保证项目在任何地方运行
2. 定义docker-compose.yml
3. 启动项目docker-compose up

**作用：批量容器编排**

Compose是Docker官方的开源项目，需要安装！

Dockerfile让程序在任何地方运行。 

```shell
##docker-compose.yaml

version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

## Compose

- 服务services，容器 应用 （web\redis\mysql...）
- 项目project. 一组关联的容器

### 安装Compose

#### 1、下载

```shell
###国外镜像有点慢
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#国内
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```



#### 2、授权

```shell
sudo chmod +x /usr/local/bin/docker-compose
```



#### 3、体验

官方地址：https://docs.docker.com/compose/gettingstarted/

python应用：计数器 Redis

1.  Docker 应用打包为镜像
2. Dockerfile应用打包为镜像
3. Docker-compose.yaml文件（定义整个服务，需要的环境 web/redis)
4. 启动Compose项目(docker-compose up) 

**一、准备相关环境**

```shell
1、创建文件目录
mkdir composetest
cd composetest

2、Python应用代码
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

    
3、引入依赖包，创建requirements.txt
flask
redis
```

**二、创建Dockerfile** 

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

**三、在Compose文件中定义services** docker-compose.yml

```shell
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

**四、构建和运行应用**

docker-compose up

流程：

1. 创建网络

2. 执行Docker-compose yaml

3. 启动服务

4. 1. Creating composetest_web_1   ... done
   2. Creating composetest_redis_1 ... done

![docker32](http://qiliu.luxiaobai.cn/img/docker32.png)

docker images

![docker33](http://qiliu.luxiaobai.cn/img/docker33.png)

```shell
root@iZbp13941xpzjmefjge9chZ:~# docker service ls
Error response from daemon: This node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
```

默认的服务名： 文件名_服务名_num

多个服务器、集群。 A B _num 副本数量

集群状态。服务都不可能只有一个运行实例、弹性 

kubectl service 负载均衡

3、网络规则

![docker34](http://qiliu.luxiaobai.cn/img/docker34.png)

10个服务-》项目（项目中的内容都在同个网络下，域名访问）

![docker35](http://qiliu.luxiaobai.cn/img/docker35.png)

如果在同一个网络下，可以通过域名访问。

停止： docker-compose down   ctrl+c

后台运行：docker-compose up -d

docker-compose 通过编写yaml配置文件，可以通过compose一键启动所有服务、停止

 

**yaml规则**

https://docs.docker.com/compose/compose-file/compose-file-v3/

```yaml
#3层
version: '' #版本
services:   #服务
    服务1: web
        #服务配置 
        images
        build
        network
        ...
    服务2：redis
    ...
    服务3：redis
#其他配置 网络/卷、全局规则
volumes:
networks:
configs: 
```

![docker36](http://qiliu.luxiaobai.cn/img/docker36.png)



# **开源项目**

## **博客**

下载程序、安装数据库、配置

compose应用==》一键启动

```yaml
#创建应用目录
mkdir my_wordpress
cd my_wordpress

##编写docker-compose.yml文件
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}    
```

1. 下载项目（docker-compose.yml)
2. 如果需要文件。Dockerfile
3. 文件准备齐全（一键启动）



### **实战**

1. 编写项目微服务
2. dockerfile 构建镜像
3. docker-compose.yaml编排项目
4. 服务器运行 docker-compose up 

将Dockerfile、jar包、docker-compose.yml文件放在同一个目录下luxiaobaiapp，使用docker-compose up将项目启动。

假设项目要重新部署打包

```shell
docker-compose up --build
```



# Docker Stack

docker-compose 单机部署项目!

Docker Stack部署,集群部署

```shell
#单机
docker-compose up -d wordpress.yaml

#集群
docker stack deploy wordpress.yaml

```

![docker37](http://qiliu.luxiaobai.cn/img/docker37.png)



### docker Secret

安全! 配置密码, 证书!

![docker38](http://qiliu.luxiaobai.cn/img/docker38.png)

### Docker Config

配置

![docker39](http://qiliu.luxiaobai.cn/img/docker39.png)



官方文档https://docs.docker.com/engine/swarm/

### **准备**

4台服务器

![docker40](http://qiliu.luxiaobai.cn/img/docker40.png)

![docker41](http://qiliu.luxiaobai.cn/img/docker41.png)

![docker42](http://qiliu.luxiaobai.cn/img/docker42.png)

4台服务器!!1主3从!!

121.199.21.27

**安装Docker**

```shell
##按照gcc相关环境
1、yum -y install gcc
2、yum -y install gcc-c++
##卸载旧版本
3、sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
##安装软件包
yum install -y yum-utils

##设置镜像仓库 国内
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo


##更新yum软件包索引
yum makecache fast

##安装Docker CE
yum install -y docker-ce docker-ce-cli containerd.io

###启动docker 
systemctl start docker

docker version


###配置镜像加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://bf2wmr0q.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



### 工作模式

![docker43](http://qiliu.luxiaobai.cn/img/docker43.png)

### **搭建集群**

![docker44](http://qiliu.luxiaobai.cn/img/docker44.png)

![docker46](http://qiliu.luxiaobai.cn/img/docker46.png)

![docker45](http://qiliu.luxiaobai.cn/img/docker45.png)

初始化节点 docker swarm init

docker swarm join 加入一个节点

docker2加入一个节点，作为工作节点

```shell
docker swarm join --token SWMTKN-1-4cbdpajpqpnlfrwa9fnwrxsd4vtlm9stczpggq31m8bbr29qil-954m54sn65mxt6m39e9uemx6f 172.16.90.113:2377
```

![docker47](http://qiliu.luxiaobai.cn/img/docker47.png)

docker1

生成一个work token,让其他节点(docker3)作为工作节点加入

```shell
docker swarm join-token worker
```

生成一个manager token，让其他节点(docker4）作为管理加入

```shell
docker swarm join-token manager
```



![docker48](http://qiliu.luxiaobai.cn/img/docker48.png)

1. 生成主节点  docker swarm init  --advertise-addr 172.16.90.113
2. 通过docker swarm join-token work/manager 生成对应的工作/管理 token，让其他节点加入进来

双主双从！





## Raft协议

双主双从：假设一个节点挂了，其他节点是否可用！

Raft协议：保证大多数节点存活才可以用，只要>1，集群至少大于3台

**实验**

1. 将docker1停止宕机！双主，另外一个主节点也不能使用了

![docker49](http://qiliu.luxiaobai.cn/img/docker49.png)

2. 可以将其他节点离开

![docker50](http://qiliu.luxiaobai.cn/img/docker50.png)

3. work就是工作的、管理节点操作！docker1、3、4设置为了管理节点

集群，可用！3个主节点 >1台管理节点存活

Raft协议：保证大多数节点存活，才可以使用，高可用！

**体会**

弹性、扩缩容！集群！

集群：swarm docker service

容器=>服务！

集群：高可用

docker service --help

创建服务、动态扩展服务、动态更新服务

![docker51](http://qiliu.luxiaobai.cn/img/docker51.png)

#### **灰度发布：金丝雀发布！**

![docker52](http://qiliu.luxiaobai.cn/img/docker52.png)

```shell
docker run 容器启动！不具有扩缩容器
docker service 服务！ 具有扩缩容器，滚动更新
```

**查看服务**

```shell
docker service ps
docker service ls
```

#### 动态扩缩容

```shell
#生成3个Nginx服务，随机分配到各个节点上
##服务，集群中任意的节点都可以访问，服务可以有多个副本动态扩缩容实现高可用
docker service update --replicas 10 my-nginx


##scale也是动态扩缩容
docker service scale my-nginx=5
```

**总结**

**swarm**

集群的管理和编号.docker可以初始化一个swarm集群,其他节点可以加入.(管理、工作者)

**Node**

就是一个docker节点,多个节点就组成一个网络集群.(管理、工作者)

**Service**

任务,可以在管理节点或者工作节点来运行.核心!用户访问!

**Task**

容器内的命令,细节任务

命令->管理->api->调度->工作节点(创建Task容器维护创建)

![docker53](http://qiliu.luxiaobai.cn/img/docker53.png)

调整service以什么方式运行

```shell
--mode string
Service mode (replicated or global)(default "replicated")

docker service create --mode replicated --name mytom tomcat:7 默认的

docker service create --mode global --name haha alpine ping baidu.com
#日志收集
每一个节点有自己的日志收集器,过滤.把所有日志最终再传给日志中心.
服务监控,状态性能
```

**拓展:网络模式:** "PublishMode": "ingress"

Swarm:

 Overlay:

 ingress:特殊的Overlay网络!负载均衡的功能! IPVS VIP