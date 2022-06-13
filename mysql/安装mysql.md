# CentOS 安装Mysql

1、安装依赖包

```shell
sudo yum install libaio
```

2、检查 MySQL 是否已安装

```shell
sudo yum list installed | grep mysql
或者
rpm -qa|grep -i mysql
有就先全部卸载
sudo yum -y remove mysql-libs.x86_64
```

3、wget下载

```shell
sudo yum -y install wget
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
```

4、添加 MySQL Yum Repository 到你的系统 repository 列表中，执行

```shell
sudo yum localinstall mysql-community-release-el7-5.noarch.rpm

##验证是否添加成功
sudo yum repolist enabled | grep "mysql.*-community.*"
```

5、安装

```shell
yum install mysql-community-server

sudo systemctl start  mysqld
sudo systemctl status  mysqld 
```

6、设置密码

```mysql
 set password for 'root'@'localhost' =password('root');
```

7、配置文件/etc/my.cnf,进行编码配置

```shell
[mysql]
default-character-set =utf8
```

8、把在所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户

```mysql
grant all privileges on *.* to root@'%'identified by 'root';
```





### 内网安装

下载[MySQL-client-5.5.52-1.linux2.6.x86_64.rpm](https://downloads.mysql.com/archives/get/p/23/file/MySQL-client-5.5.52-1.linux2.6.x86_64.rpm) [MySQL-server-5.5.52-1.linux2.6.x86_64.rpm](https://downloads.mysql.com/archives/get/p/23/file/MySQL-server-5.5.52-1.linux2.6.x86_64.rpm)

```shell
rpm -ivh MySQL-server-5.5.52-1.linux2.6.x86_64.rpm
rpm -ivh MySQL-client-5.5.52-1.linux2.6.x86_64.rpm
ps -ef | grep mysql
##查看用户组
cat /etc/passwd | grep mysql
cat /etc/group | grep mysql
mysqladmin --version
###mysql后台启动
service mysql start
###设置密码
/usr/bin/mysqladmin -u root password 123456
###设置开机自启动
chkconfig mysql on
chkconfig --list | grep mysql
###查看运行级别
cat /etc/inittab
ntsysv
```



![image-20220125155316312](/Users/lushengyang/Library/Application Support/typora-user-images/image-20220125155316312.png)

![image-20220125153709103](/Users/lushengyang/Library/Application Support/typora-user-images/image-20220125153709103.png)



```shell
##CentOS卸载mysql
rpm -qa | grep -i mysql #查看mysql所有已安装的组件
yum -y remove mysql-*　
find / -name mysql #查看与mysql相关的目录
```





































































#### 相关资料来源

[1]: https://www.cnblogs.com/braveym/p/10868751.html

