[toc]

-----

# Mysql 文件存放位置

|       路径        |           解释            |                    备注                    |
| :---------------: | :-----------------------: | :----------------------------------------: |
|  /var/lib/mysql/  | mysql数据库文件的存放路径 | /var/lib/mysql/iZbp13941xpzjmefjge9chZ.pid |
| /usr/share/mysql  |       配置文件目录        |         Mysql.server命令及配置文件         |
|     /usr/bin      |       相关命令目录        |         Mysqladmin mysqldump等命令         |
| /etc/init.d/mysql |       启停相关脚本        |                                            |



# 修改配置文件

```shell
cp /usr/share/mysql/my-huge.cnf /etc/my.cnf
###修改字符集
#查看字符集配置
show variables like '%char%'

####修改字符集
[client]
#password	= your_password
port		= 3306
socket		= /var/lib/mysql/mysql.sock
default-character-set=utf8 	####utf8
 
[mysqld]
port		= 3306
character_set_server=utf8			####utf8
character_set_client=utf8		####utf8
collation-server=utf8_general_ci		####utf8
socket		= /var/lib/mysql/mysql.sock
skip-external-locking
key_buffer_size = 384M
max_allowed_packet = 1M
table_open_cache = 512
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size = 32M 
thread_concurrency = 8
 
log-bin=mysql-bin
 
server-id	= 1
 
[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash
default-character-set=utf8   ####utf8
# Remove the next comment character if you are not familiar with SQL
#safe-updates

[myisamchk]
key_buffer_size = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
```

> 默认客户端和服务器都用了latin1,所以会乱码

![image-20220125174835909](http://qiliu.luxiaobai.cn/img/image-20220125174835909.png)







## 存储引擎

![image-20220304152648892](http://qiliu.luxiaobai.cn/img/image-20220304152648892.png)

### Percona

percona为Mysql数据器进行了改进,提高了在高负载情况下的InnoDB的性能

该公司新建了一款存储引擎: `xtradb`完全可以替代Innodb,并且在性能和并发上做得更好

- 阿里大部分mysql数据库其实使用的percona的原型加以修改
- AliSql+AliRedis 





## 性能优化分析

```
select * from user where name = '' and email=''
create index idx_user_name on user(name, email)
```

- 关联查询太多join(设计缺陷)
- 服务器调优及各个参数设置(缓冲、线程数)



### 常用的Join查询

内连接: `select * from A Inner join B ON A.key = b.key`

![image-20220304154718757](http://qiliu.luxiaobai.cn/img/image-20220304154718757.png)

左连接: `select * from A left join B ON A.key = B.key`

![image-20220304154857506](http://qiliu.luxiaobai.cn/img/image-20220304154857506.png)

 

右连接: `select * from A right join B ON A.key = B.key`

![image-20220304155108230](http://qiliu.luxiaobai.cn/img/image-20220304155108230.png)



`select * from A left join B on A.key = B.key where B.Key IS NULL`

![image-20220304155248820](http://qiliu.luxiaobai.cn/img/image-20220304155248820.png)



`select * from A right join B on A.key=B.key where A.key is NULL`

![image-20220304155439900](http://qiliu.luxiaobai.cn/img/image-20220304155439900.png)

 

全连接:`select * from A FULL outer join B on A.key = B.key` === `select * from A left join B on A.key = B.key union select * from A right join B on A.key = B.key`

![image-20220304155604306](http://qiliu.luxiaobai.cn/img/image-20220304155604306.png)



`select * from A FULL outer join B on A.key = B.key Where A.key is Null OR B.key is NULL` ==== `select * from A leftjoin B on A.key = B.key where B.key is null union select * from A right join B on A.key = B.key wehre A.key is null`

![image-20220304155809742](/Users/lushengyang/Library/Application Support/typora-user-images/image-20220304155809742.png)





### 索引

**排好序的快速查找数据结构**

   

















