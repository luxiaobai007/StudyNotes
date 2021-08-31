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

















#### 相关资料来源

[1]: https://www.cnblogs.com/braveym/p/10868751.html

