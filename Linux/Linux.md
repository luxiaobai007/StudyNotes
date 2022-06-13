[toc]

# 虚拟机网络

1. 桥连接:Linux可以和其他的系统通行,但是可能造成IP冲突
2. NAT模式:网络地址转换方式:Linux可以访问外网,不会造成IP冲突
3. 主机模式:你的Linux是一个独立的主机,不能访问外网.



# Linux目录结构

```java
/bin (/usr/bin、/usr/local/bin) 是Binary的缩写,存放最经常使用的命令
/sbin(/usr/sbin、/usr/local/sbin) s就是Super User的意思,存放系统管理员使用的系统管理程序
/home 存放普通用户的主目录,在Linux中每个用户都有一个自己的目录,一般以用户账号命名
/root 该目录为系统管理员,也称作超级权限者的用户主目录 
/lib 系统开机所需要最基本的动态连接共享库,其作用类似于windows里的DLL文件,几乎所有的应用程序都需要用到这些共享库
/lost+found 这个目录一般情况下是空的,当系统非法关机后,就存放一些文件
/etc  所有系统管理所需要的配置文件和子目录
/usr 这是一个非常重要的目录,用户的很多应用程序和文件都放在这个目录下,类似于windows下的programfiles目录
/boot 存放的是启动Linux时使用的一些核心文件,包括一些连接文件以及镜像文件
/proc 这个是一个虚拟的目录,它是系统内存的映射,访问这个目录来获取系统信息
/srv service缩写,该目录存放一些服务启动之后需要提取的数据
/sys Linux2.6内核的一个很大的变化,该目录下安装了2.6内核中新出现的一个文件系统
/tmp 存放临时文件
/dev 类似于windows的设备管理器,把所有硬件用文件的形式存储
/media Linux系统会自动识别一些设备,如U盘、光驱等,当识别后,Linux会把是识别的设备挂载到这个目录下
/mnt 系统提供该目录是为了让用户临时挂载别的文件系统的,我们可以将外部的存储挂载在/mnt/上,然后进入该目录就可以查看里面的内容了.
/opt 给主机额外安装软件所摆放的目录,如安装oracle数据库就可放到该目录下.默认为空
/usr/local 这是另一个给主机额外安装软件所安装的目录.一般是通过编译源码方式安装的程序
/var 这个目录存放着在不断扩充着的东西,习惯将修改的目录放在这个目录下.包括各种日志文件
/selinux SELinux是一种安全子系统,它能控制程序只能访问特定文件.
```



# 相关指令

## Vi和Vim

所有的Linux系统都内建vi文本编辑器

Vim具有程序编辑的能力,可以看作是vi的增强版本,可以主动的以字体颜色辨别语法的正确性,方便程序设计.



### 模式

#### 正常模式

以vim打开一个档案就直接进入一般模式了.这个模式中,你可以使用上下左右按键来移动光标,可以使用删除字符或删除整行来处理档案内容,也可以复制、粘贴



#### 插入模式/编辑模式

按下i,l,o,O,a,A,r,R等任何一个字母之后才会进入编辑模式,一般来说按i即可



#### 命令行模式

在这个模式当中,可以完成读取、存盘、替换、离开vim、显示行号等的动作则是在此模式中达成的



#### 快捷键

1. 拷贝当前行: yy, 拷贝当前行向下的5行 5yy,并粘贴.

2. 删除当前行: dd, 删除当前行向下的5行 5dd

3. 在文件中查找某个单词[命令行下/关键字, 回车 查找, 输入n就是查找下一个]

4. 设置文件的行号,取消文件的行号 [命令行下:set nu 和 :set nonu]

5. 编辑/etc/profile文件, 使用快捷键到底文档的最末行[G]和最首行[gg]

6. 在一个文件中输入“hello",然后又撤销这个动作 u

7. 编辑 /etc/profile 文档,并将光标移动到20行 

8. 1. 设置行号 :set nu
   2. 输入20
   3.  shift + g



## 关机&重启命令

```shell
shutdown 
 shutdown -h now :立即关机
 shutdown -h 1 : 1分钟后关机
 shutdown -r now 立即重启
 
halt  关机,作用同上
reboot 重启
sync 把内存的数据同步到磁盘
```

> **注意细节:当关机或重启时,都应该先执行sync指令,把**内存的数据同步到磁盘**,防止数据丢失**



## 用户管理

### **添加用户**

- useradd [选项] 用户名

- useradd -d 指定目录 新的用户名 给新创建的用户指定家目录

- passwd 用户名	更改用户密码 

- userdel 		删除用户

- - 删除用户 保留家目录 userdel 用户
  - 删除用户 家目录也删除 userdel -r 用户



```shell
###增加用户
sudo adduser luxiaobai
##将用户添加到sudo组
sudo usermod -aG sudo luxiaobai
###删除用户  userdel/deluser ubuntu上推荐deluser
sudo deluser username ##删除用户而不删除用户文件
sudo deluser --remove-home username ##删除用户的家目录和邮件使用

```





### 查询用户信息

id 用户

```shell
id root
```



### 用户组

#### **查看当前用户/登陆用户**

```shell
whoami/who am i
```



#### **增加组**

```shell
groupadd 组名
```



#### **删除组**

```shell
groupdel 组名
```



#### 指定用户到用户组

```shell
useradd -g 用户组 用户名


###root提权
useradd -g wheel luxiaobai
###修改权限
usermod -g wheel luxiaobai
```



### 用户配置文件(/etc/passwd)

用户的配置文件,记录用户的各种信息

用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录shell



### 组配置文件(组信息):/etc/group

记录Linux包含的组的信息

组名:口令:组标识号:组内用户列表



### 口令配置文件(密码和登录信息):/etc/shadow

登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志  



## 服务器安全设置

1. 禁止非root用户执行`/etc/rc.d/init.d/`下的系统命令

   ```shell
   chmod -R 700 /etc/rc.d/init.d/*
   chmod -R 777 /etc/rc.d/init.d/* #恢复默认设置
   ```

   

2. 限制不同文件的权限

   ```shell
   chattr +a .bash_history #避免删除.bash_history或者重定向到/dev/null
   chattr +i .bash_history
   ```

   

3. 使用yum update更新系统时不升级内核,只更新软件包

   ```
   vi /etc/yum.conf
   ##在[main]的最后添加
   exclude=kernel*
   
   #直接在yum的命令后加上如下的参数
   yum --exclude=kernel* update 
   
   #查看系统版本
   cat /etc/issue
   
   查看内核版本
   uname -a
   ```

   

4. 关闭CentOS自动更新

   ```shell
   chkconfig --list yum-updatesd ##显示当前系统状态
   ```

   

5. 删除MySQL历史记录.如果用SQL语句修改了数据库密码,也会因.mysql_history而泄露

6. 隐藏服务器系统信息

   ```shell
   mv /etc/issue /etc/issuebak
   ```

   

7. CentOS系统优化

   ```shell
   vi /etc/profile
   ulimit -c unlimited
   ulimit -s unlimited
   ulimit -SHn 65535
   ulimit -S -c 0
   export LC_ALL=C
   source /etc/profile
   ulimit -a #显示当前的各种用户进程限制
   ```

   





## 实用指令

### 指定运行级别

0:关机

1:单用户[找回丢失密码]

2:多用户没有网络服务

3:多用户状态有网络服务

4:系统未使用保留给用户

5:图形界面

6:系统重启

常用运行级别3和5,要修改默认运行级别可改文件/etc/inittab的id:5:initdefault:这一行中的数字

```shell
init[0123456]
```



# **面试题**

## **如何找回root密码**

思路:进入到单用户模式,然后修改root密码.因为进入单用户模式,root不需要密码就可以登录.

开机=>在引导时输入回车键=>在界面输入e ,在看到一个新的界面高亮选中第二行(kernal),=>在输入一个e,这行最后输入 1=>在输入一个回车键=>再次输入b, 这时就会进入单用户模式

![imageshell](http://qiliu.luxiaobai.cn/img/imageshell-1931525.png)

按Ctrl+x执行,进入单用户模式

chroot /sysroot

passwd root 修改密码

touch /.autorelabel生效

Ctrl+D,执行reboot重启生效

rw 末尾添加 init = /bin/sh





# 文件目录类

## mkdir

```shell
mkdir -p 目录    创建多级目录
rmdir 目录         删除的是空目录,如果目录下有内容则不可以删除
cp -r 递归复制整个文件夹  \cp -r 目录 目录 强制覆盖
```



## rm

```shell
rm -r: 递归删除整个文件夹
rm -f: 强制删除不提示
```



## cat

```shell
cat -n :显示行号
cat -n /etc/profile | more   分页显示
```



## more

```shell
space :向下翻一页
Enter:向下翻一行
q: 代表立刻离开more,不再显示该文件内容
Ctrl+F  向下滚动一屏
Ctrl+B 返回上一屏
more /etc/profile
```



## less

```shell
space :向下翻一页
pagedown :向下翻一页
pageup :向上翻动一页
/字符串:向下搜寻[字符串]的功能:n:向下查找;N:向上查找
?字符串:向上搜寻[字符串]的功能 n:向上查找;N:向下查找
q:离开less
```



## **>指令和>>指令**

 \>输出重定向:覆盖原来的文件内容

 \>>追加:不会覆盖原来的内容,追加到文件尾部

- ls -l > 文件 			列表的内容写入文件a.txt中(覆盖写)
- ls -al >> 文件		列表的内容追加到文件aa.txt的末尾
- cat 文件1 > 文件2 	文件1的内容覆盖到文件2	
- echo "内容" >> 文件 



## head指令

head -n 5 文件	查看文件头部前5行内容



## In指令

软链接也叫符号链接

ln -s [原文件或目录] [软链接名] 给原文件创建一个软链接

ln -s /root linkToRoot



## date指令 显示当前日期

```shell
date		显示当前时间
date + %Y	显示当前年份
date+%m	显示当前月份
date+%d		显示当前是那一天
```



### 设置日期

```shell
date -s 字符串时间
date -s "2018-10-10 11:22:22"
```



## cal指令 查看日历指令

```shell
cal [选项] 	不叫选项,显示本月日历
cal 2020		显示2020的一整年日历
```



## 搜索查找类

### **find指令**

find [搜索范围] [选项]

-  -name<查询方式>		按照指定的文件名查找模式查找文件
-  -user<用户名>		        查找属于指定用户名所有文件
-  -size<文件大小>		按照指定的文件大小查找文件 (+n :大于多少 -n小于多少  n等于 )

```shell
find /home -name hello.txt   *.txt
find /opt -user root
find / -size +20M
```



### locate指令

快速定位文件路径.利用事先建立的系统中所有文件名称及路径的locate数据库实现快速定位给定的文件.locate指令无需遍历整个文件系统,查询速度较快.为保证查询结果的准确度,管理员必须定期更新locate时刻.



#### locate 搜索文件

locate指令基于数据库进行查询,所以第一次运行前,必须使用updatedb指定创建locate数据库

```shell
updatedb
locate hello.txt
```



### grep指令和管道符合 |

过滤查找,管道符, “|”, 表示将前一个命令的处理结果输出传递给后面的命令处理

grep [选项] 查找内容 源文件

- -n 显示匹配行及行号
- -i 忽略字母大小写reboot

```shell
cat hello.txt | grep -ni hello 
```



## 压缩和解压缩类

**gzip 文件 :只能将文件压缩为\*.gz文件 不会保留原来的文件**

**gunzip 文件.gz   解压缩文件**

**zip [选项] XXX.zip 将要压缩的内容**

-r: 递归压缩,即压缩目录

**unzip [选项] XXX.zip 解压缩文件**

-d<目录> : 指定解压后文件的存放目录

```shell
zip -r mypackage.zip /home

unzip -d /home/lxb mypackage.zip
```



### **tar指令**

tar [选项] XXX.tar.gz 打包的内容  : 打包目录,压缩后的文件格式.tar.gz)

- -c: 产生.tar打包文件
- -v: 显示详细信息
- -f: 指定压缩后的文件名
- -z: 打包同时压缩
- -x: 解压.tar文件

```shell
lushengyang@luxiaobaideMacBook-Pro tar % ls
a.txt	b.txt
lushengyang@luxiaobaideMacBook-Pro tar % tar -zcvf a.tar.gz a.txt b.txt 
a a.txt
a b.txt
lushengyang@luxiaobaideMacBook-Pro tar % ls
a.tar.gz	a.txt		b.txt

lushengyang@luxiaobaideMacBook-Pro tarTest % tar -zxvf a.tar.gz 
x a.txt
x b.txt
```

![shell](http://qiliu.luxiaobai.cn/img/shell.png)



## **组管理和权限管理**

### **查看文件的所有者**

```shell
ls -ahl
```

### **修改文件所有者**

```shell
chown 用户名 文件名
```



### **组的创建**

```shell
groupadd 组名
```



### **修改文件所在的组**

```shell
chgrp 组名 文件名
```



### **改变用户所在组**

```shell
usermod -g 组名 用户名

usermod -d 目录名 用户名 改变该用户登录的初始目录
```





## 权限的基本介绍

![linux1](http://qiliu.luxiaobai.cn/img/linux1.png)

**0-9位说明**

### **-rw-r--r--**

- 第0位确定文件类型(d, -, l, c, b)
- 第1-3位确定所有者(该文件所有者)拥有该文件的权限. -- User
- 第4-6位确定所属组(同用户组的)拥有该文件的权限 --- Group
- 第7-9位确定其他用户拥有该文件的权限 。  -- Other



### **rwx作用到文件**

1. [r]代表可读
2. [w] 可写,可以修改,但是不代表可以删除该文件,删除一个文件的前提条件是对该文件所在的目录有写权限,才能删除
3. [x]代表可执行(execute):可以被执行



### **rwx作用到目录**

1. [r]: 可读:ls查看目录内容

2. [w]: 可写,可以修改,目录内创建+删除+重命名目录

3. [x]: 代表可执行,可以进入该目录



### **修改权限 chmod**

u:所有者	g:所有组	o:其他人 a:所有人(u、g、o的总和)

1.  chmod u=rwx,g=rx,0=x 文件目录名
2. chmod o+w 文件目录名
3. chmod a-x 文件目录名

```shell
r=4 w=2 x=1 rwx=4+2+1=7
chmod u=rwx,g=rx,o=x 文件目录名
相当于 chmod 751 文件目录名
```



### **修改文件所有者 chown**

```shell
chown newowner file  :改变文件的所有者

chown newowner:newgroup file 改变用户的所有者和所有组

\- R 如果是目录,则使其下所有子文件或目录递归生效
```





### **修改文件所在组chgrp**

```shell
chgrp newgroup file 改变文件的所有组
```



## **实践**

```shell
police	bandit

jack	jerry	警察组(police)

xh xq		土匪组(bandit)
```



### 创建组

```shell
groupadd police
groupadd bandit
```



创建用户并加入对应的组

```shell
useradd -g police jeck
useradd -g police jerry

useradd -g bandit xh
useradd -g bandit xq

passwd jeck
passwd jerry
passwd xh
passwd xq
```



jack创建一个文件,自己可以读写,本组人可以读,其他组没人任务权限

```shell
vim ject01.txt
 chmod 640 jeck01.txt 
```



jack修改该文件,让其他组人可以读,本族人可以读写

```shell
chmod 0=r,g=rw jack01.txt
```



xh投靠警察 看看是否可以 读写

```shell
usermod -g police xh
chmod g=rx  jack
```



## **定时任务调度**

###   **crontab**

```shell
-e	编辑crontab定时任务

-l	查询crontab任务

-r	删除当前用户所有的crontab任务
```





### **5个占位符说明 \* \* \* \* \***

第1个 * : 一小时当中的第几分钟	0~59

第2个 * : 一天当中的第几小时 	0~23

第3个 * : 一个月当中的第几天	1-31

第4个 * : 一年当中的第几个月	1~12

第5个 *  : 一周当中的星期几	0~7(0和7都代表星期日)





## **磁盘分区和挂载**

### **window分区的方式:**

1. #### mbr分区:

2. 1. 最多支持4个主分区
   2. 系统只能安装在主分区
   3. 扩展分区要占一个主分区
   4. MBR最大只支持2TB,但拥有最好的兼容性

3. #### gtp分区:

4. 1. 支持无限多个主分区(但操作系统可能限制,比如windows下最多128个分区)
   2. 最大支持18EB的大容量(EB=1024PB, PB=1024TB)
   3. windows7 64位以后支持gtp



### **Linux分区**

硬盘说明

1. Linux硬盘分IDE硬盘和SCSI硬盘,目前基本上是SCSI硬盘
2. IDE硬盘,驱动器标识符为:“hdx~”,其中“hd"表明分区所在设备的类型 .“x"为盘号(a为基本盘,b为基本从属盘,c为辅助主盘,d为辅助从属盘). “~”代表分区,前四个分区用数字1到4表示,他妈是主分区或扩展分区,从5开始就是逻辑分区.例如:hda3表示为第一个IDE硬盘上的第三个主分区或扩展分区,hdb2表示为第二个IDE硬盘上的第二个主分区或扩展分区.
3. SCSI硬盘则标识为“sdx~", "sd"表示分区所在设备的类型的,其余和IDE硬盘一样.

```shell
lsblk -f .     #查看系统的分区和挂载的情况
lsblk
```



### **如何增加一块硬盘**

1. 虚拟机添加硬盘

2. 分区	fdisk /dev/sdb

3. 格式化 .  mkds -t ext4 /dev/sdb1

4. 挂载 .   创建挂载目录 /home/newdisk .  挂载 mount /dev/sdb1 /home/newdisk .  (临时挂载)

5. 设置可以自动挂载 (永久挂载)

6. 1. vim /etc/fstab       /dev/sdb1   /home/newdisk    ext4   defaults 0 0
   2. mount -a

7. 解除挂载 umount /dev/sdb1 或者  umount /newdisk

 

### **磁盘情况查询**

系统磁盘使用情况

```shell
df -h
```



查询指定目录的磁盘占用情况

```shell
du -h 目录
```

 -s 指定目录占用大小汇总

-h 带计量单位

-a 含文件

--max-depth=1 子目录深度

-c 列出明细的同时,增加汇总值



统计文件下的文件个数

```shell
ls -l /home/ | grep "^-" | wc -l
```



统计文件下的目录个数

```shell
ls -l /home/ | grep "^d" | wc -l
```



统计文件下的目录个数,及子目录个数

```shell
ls -lR /home/ | grep "^d " | wc -l
```



以树状目录显示

```shell
tree 目录
```

-a 显示所有文件和目录

-d 显示目录名称而非内容

-L 层级个数

```shell
tree -L 2
```



## 网络配置

- NAT:网络地址转换模式:该模式下Linux可以访问外网，不会造成IP地址冲突，实际工作中推荐使用此种方式。
- 桥连接模式:此网络连接方式，虚拟机中的Linux是可以和其他的系统主机通讯的，因为Linux系统的IP和虚拟机所在物理机器IP在同一个IP地址段，并且是自动分配的，所以可能会出现虚拟机IP地址和其他系统主机IP地址冲突（局域网内主机数量越多，出现概率越大）。
- 仅主机模式:当设置为Host-only上网时，虚拟机只能和主机进行通信，不可以上网，也不可以和其他机器进行通信，此时主机使用VMnet1与虚拟机通信。



### 指定固定的IP. Centos

```shell
vi /et /sysconfig/network-scripts/ifcfg-eth0 (网卡配置文件)
```

![Linux2](http://qiliu.luxiaobai.cn/img/Linux2.png)

```shell
service network restart
```



### **进程管理**

```shell
ps -a:	显示当前终端的所有进程信息

ps -u:	以用户的格式显示进程信息

ps -x:	显示后台进程运行的参数
```



![linux3](http://qiliu.luxiaobai.cn/img/linux3.png)



#### **kill/killall**

```shell
kill [选项] 进程号

killall 进程名称

-9 表示强迫进程立即停止
```



#### **查看进程树pstree**

```shell
pstree [选项] 

-p:显示进程PID

-u:显示进程的所属用户
```



## **服务(service)管理**

### **service管理指令:**

service 服务名 start | stop | restart | reload | status

systemctl 

```shell
1、setup ##图形界面设置那些服务器自启动
2、ls -l /etc/init.d/  列出系统有哪些服务
```





#### **chkconfig指令**

可以给每个服务的各个运行级别设置自启动/关闭,需要重启生效

1. 查看服务 chkconfig --list | grep xxx
2. chkconfig 服务名 -- list
3. chkconfig --level 5 服务名 on/off



### **动态监控进程**

#### **top**

-d  秒数: 每隔几秒更新

\- i:        不显示任何闲置或者僵死进程

-p:        通过制定监控进程ID来仅仅监控某个进程的状态

在监控模式下,输入u,在输入用户名,就可以查看指定用户的进程 在监控模式下,输入k,在输入PID,就可以关闭指定进程



#### **交互操作说明:**

- P :以CPU使用率排序,默认就是此项
- M:以内存的使用率排序
- N:以PID排序
- q:退出top

![linux4](http://qiliu.luxiaobai.cn/img/linux4.png)





## **监控网络状态**

### **查看系统网络情况netst**

```shell
netstat [选项]

-an 按一定顺序排列输出

-p 显示那个进程在调用

netstat -anp .   ###查看所有的网络服务 netstat -anp | grep sshd
```



### **RPM 和YUM**

#### **rpm包的简单查询指令**

查询已安装的rpm列表

```shell
  rpm -qa | grep  xx
```



rpm -qa:查询所安装的所有rpm软件包 rpm  -ql 软件包名:查询软件包中的文件 rpm -qa | more rpm -q 软件包名:查询软件包是否安装 rpm -qi 软件包名:查询软件包信息 rpm -qf 文件全路径名:查询文件所属的软件包 rpm -qf /etc/passswd



#### **卸载rpm包**

rpm -e RPM包的名称

--nodeps 强制删除,但是一般不推荐这样做,依赖于该软件包的程序可能无法运行

rpm -e --nodeps foo



#### **安装rpm包**

rpm -ivh RPM包全路径名称

-i=install  安装

v=verbose 提示

h=hash 进度条



#### **yum**

查询yum服务器是否有需要安装的软件

yum  list | grep xx软件列表

安装指定的yum包

yum install xxx 下载安装



#### **搭建JAVAEE环境**

tomcat服务安装

1.  解压缩到/opt
2. 启动tomcat ./startup.sh
3. 开放端口 vim  /etc/sysconfig/iptables 





























































