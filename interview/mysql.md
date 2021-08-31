[toc]





# 事务

事务是指是程序中一系列严密的逻辑操作，而且所有操作必须全部成功完成，否则在每个操作中所作的所有更改都会被撤消。

**原子性**（Atomicity）**：**操作这些指令时，要么全部执行成功，要么全部不执行。只要其中一个指令执行失败，所有的指令都执行失败，数据进行回滚，回到执行指令前的数据状态。

 **● 一致性**（Consistency）**：**事务的执行使数据从一个状态转换为另一个状态，但是对于整个数据的完整性保持稳定。

 **● 隔离性**（Isolation）**：**隔离性是当多个用户并发访问[数据库](https://cloud.tencent.com/solution/database?from=10680)时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。

 **● 持久性**（Durability）**：**当事务正确完成后，它对于数据的改变是永久性的。



# Mysql select的执行流程

整个过程涉及到Mysql的组成结构、Innodb的页结构以及Mysql的索引原理

mysql的组成结构

![image-20210826170141046](/Users/lushengyang/Library/Application Support/typora-user-images/image-20210826170141046.png)



## 连接器

用于Mysql和客户端进行连接，当我们在terminal中输入：`mysql -u username -p`后，Mysql客户端就会跟你服务器上的Mysql进行友好的TCP三次握手，双方状态都变为established之后，连接器开始验证你输入的用户名和密码；

![image-20210826171137853](/Users/lushengyang/Library/Application Support/typora-user-images/image-20210826171137853.png)

连接器在处理连接请求的时候，除了会验证用户名密码以外，还会去检验用户权限



## 查询缓存

**首先声明Mysql8.0之后查询缓存模块被拿掉了**，查询缓存仅对于select语句，当查询缓存开启的时候，比如对表A的select * from A;该语句和查询结果会以HashMap的形式，将查询语句（select * from A）作为key，结果作为value存储起来，当你下一次进行查询的select查询的时候，如果是**一模一样**的查询语句，则会命中，且Mysql不往下执行，直接返回结果。当然即便A表的查询经过缓存，但是任意时候对A表进行了增删改操作，这条与A表相关的缓存也会被清空。这里就有一个问题！在查询缓存打开的情况下，每一张表进行了增删改之后都要检查缓存，看看是否需要删除记录，这对相应的操作性能有影响，所以看起来这是一个不错的功能，但并不推荐使用，更绝的是8.0之后就直接拿掉了。



## 分析器

如果查询缓存关闭或是没有命中缓存的情况下，SQL语句会进入分析器，

分析器做两个事：

​	一是词义分析，也就是说根据你这个SQL字符和空格组成的字符串，对关键字进行分析，比如发现有“select”，Mysql就知道这是一个查询语句，发现from后面的“food”，就知道food是表名；在确定了这个SQL字符串是来干嘛的之后，就开始第二件事，

语义分析，这就是我们熟悉的语法检查，一旦发现错误就报出：You have an error in your sql syntax。

![image-20210826172053388](/Users/lushengyang/Library/Application Support/typora-user-images/image-20210826172053388.png)



## 优化器

优化器会在连表查询的时候确定怎样的查询顺序比较好，或是有多个索引的时候决定用哪一个索引，不用哪一个索引等等，它会根据执行的效率进行判断。当优化器决定了最终的执行方案后，就会交由执行器进行执行。



## 执行器

在执行`select * from food where id = 3`之前，执行器会先判断当前的登陆用户是否有权限访问user这个表。根据优化器的分析如果是全表扫描的话就会调用InnoDB执行引擎调取第一行，记录之后调取“下一行”接口……直至找到；如果是通过索引查找，下文会详细介绍查找过程，最后执行器将所有结果的集合返回给用户。



## 数据页

当目前为止这个SQL语句算是走完了Mysql的Server端，进而来到了存储引擎，从版本5.5.5之后，Mysql默认的存储引擎就是InnoDB，这里我们就以它为例，进行介绍。上面我们说了，执行器在查询SQL语句的时候，会调InnoDB的接口一行一行查询，这里有一个问题，InnoDB当中是如何执行的呢？如果查询条件中有索引呢？为了更准确的描述这条语句的执行，我们先了解一下，InnoDB中数据的存储结构和索引的实现原理。
 InnoDB当中数据是按照页的方式进行存储，我们看到下图，就是InnoDB将数据按照页的结构进行存储。暂时分不清没关系，我们看到User Records就是用户的数据，在它下面Free Space字面意思是空闲空间，其实可以理解成User Records的备胎空间，新一条数据存储进来就从Free Space当中拿一点空间来给到User Records。





# 慢SQL有遇到过吗?是怎么解决的?

## 慢SQL?

MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录MySQL中查询时间超过（大于）设置阈值（long_query_time）的语句，记录到慢查询日志中。

其中，long_query_time的默认值是10，单位是秒，也就是说默认情况下，你的SQL查询时间超过10秒就算慢SQL了。



## 如何开启慢SQL日志

在MySQL中，慢SQL日志默认是未开启的，也就说就算出现了慢SQL，也不会告诉你的，如果需要知道哪些SQL是慢SQL，需要我们手动开启慢SQL日志的。

关于慢SQL是否开启，我们可以通过下面这个命令来查看：

```mysql
show variables like '%slow_query_log%';
```

![image-20210827141131676](/Users/lushengyang/Desktop/LSY/document/image/image-20210827141131676.png)

开启慢SQL日志

```mysql
set global slow_query_log = 1;
```

需要注意，这里开启的是我们当前的数据库，并且，我们重启数据库后会失效的。

![image-20210827141403886](/Users/lushengyang/Desktop/LSY/document/image/image-20210827141403886.png)

```mysql
#修改慢SQL的默认时间
set long_query_time = 3;
```

![image-20210827141528155](/Users/lushengyang/Desktop/LSY/document/image/image-20210827141528155.png)



如果想要永久的生效,还需要修改MySql的配置文件**mysql.conf**文件

```mysql
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/lxb-slow.log
long_query_time=3
log_output=FILE
```

注意：不同操作系统，配置有些区别

```shell
#Linux操场系统
在mysql配置文件my.cnf中增加
log-slow-queries=/var/lib/mysql/slowquery.log (指定日志文件存放位置，可以为空，系统会给一个缺省的文件host_name-slow.log)
long_query_time=2 (记录超过的时间，默认为10s)
log-queries-not-using-indexes (log下来没有使用索引的query,可以根据情况决定是否开启)
log-long-format (如果设置了，所有没有使用索引的查询也将被记录)


#Windows系统中
在my.ini的[mysqld]添加如下语句：
log-slow-queries = E:\web\mysql\log\mysqlslowquery.log
long_query_time = 3(其他参数如上)
```



查看慢SQL出现了多少条

```mysql
show global status like '%Slow_queries';
```



### 如何定位慢SQL?

- 系统级表象:
  - 使用**sar**命令和**top**命令查看当前系统的状态
  - 也可以使用**Prometheus**和**Grafana**监控工具查看当前系统状态
  - **CPU**消耗严重
  - **IO**等待严重
  - 页面响应时间过长
  - 项目日志出现超时等错误
- SQL语句表象:
  - **SQL**语句冗长
  - **SQL**语句执行时间过长
  - **SQL**从全表扫描中获取数据
  - 执行计划中的**rows**和**cost**很大



### 熟悉慢SQL日志分析工具吗

如果开启了慢SQL日志后，可能会有大量的慢SQL日志产生，此时再用肉眼看，那是不太现实的，所以大佬们就给我搞了个工具：`mysqldumpslow`。

`mysqldumpslow`能将相同的慢SQL归类，并统计出相同的SQL执行的次数，每次执行耗时多久、总耗时，每次返回的行数、总行数，以及客户端连接信息等。

通过命令

```mysql
mysqldumpslow --help
[root@iZbp13941xpzjmefjge9chZ ~]# mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                al: average lock time
                ar: average rows sent
                at: average query time
                 c: count
                 l: lock time
                 r: rows sent
                 t: query time  
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
```

常用的参数:

```shell
-s 指定输出的排序方式
   t  : 根据query time(执行时间)进行排序；
   at : 根据average query time(平均执行时间)进行排序；（默认使用的方式）
   l  : 根据lock time(锁定时间)进行排序；
   al : 根据average lock time(平均锁定时间)进行排序；
   r  : 根据rows(扫描的行数)进行排序；
   ar : 根据average rows(扫描的平均行数)进行排序；
   c  : 根据日志中出现的总次数进行排序；
-t 指定输出的sql语句条数；
-a 不进行抽象显示(默认会将数字抽象为N，字符串抽象为S)；
-g 满足指定条件，与grep相似；
-h 用来指定主机名(指定打开文件，通常慢查询日志名称为“主机名-slow.log”，用-h exp则表示打开exp-slow.log文件)；
```

```shell
mysqldumpslow -s c slow.log
应该是mysqldumpslow最简单的一种形式，其中
-s参数是以什么方式排序的意思，
	c指代的是以总数从大到小的方式排序。
-s的常用子参数有：
	c: 相同查询以查询条数和从大到小排序。
	t: 以查询总时间的方式从大到小排序。
	l: 以查询锁的总时间的方式从大到小排序。
	at: 以查询平均时间的方式从大到小排序。
	al: 以查询锁平均时间的方式从大到小排序。
```

```shell
# 得到返回记录集最多的10 个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log

# 得到访问次数最多的10 个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log

# 得到按照时间排序的前10 条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow.log

# 另外建议在使用这些命令时结合| 和more 使用，否则有可能出现爆屏情况
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log | more
```



# mysql如何动态增加索引





# mysql增加字段





# 自增主键

1. 如果定义了主键(primary key),则InnoDB会选择主键作为聚集索引
2. 如果没有显式定义主键,,则InnoDB会选择第一个不包含null值的唯一索引作为主键索引
3. 如果也没有这样的唯一索引,则InnoDB会选择内置6字节长的ROWID作为隐含的聚集索引.



# 为什么使用数据索引能提供效率

- 数据索引的存储是有序的 
- 在有序的情况下，通过索引查询⼀个数据是⽆需遍历索引记录的 
- 极端情况下，数据索引的查询效率为⼆分法查询效率，趋近于 log2(N) 



# 什么情况下应不建索引或少建索引

1. 表记录太少
2. 经常插入、删除、修改的表
3. 数据重复切分布平均的表字段
4. 经常和主字段一块查询单主字段索引值比较多的表字段



# 什么是表分区

表分区，是指根据⼀定规则，将数据库中的⼀张表分解成多个更⼩的，容易管理的部分。从逻辑上看，只有⼀张表，但是底层却 是由多个物理分区组成。 



# mysql优化

- 开启查询缓存，优化查询 
- explain你的select查询，这可以帮你分析你的查询语句或是表结构的性能瓶颈。EXPLAIN 的查询结果还会告诉你你的索引主键被如何利⽤的，你的数据表是如何被搜索和排序的 

- 当只要⼀⾏数据时使⽤limit 1，MySQL数据库引擎会在找到⼀条数据后停⽌搜索，⽽不是继续往后查少下⼀条符合记录的 数据

- 为搜索字段建索引 

- 使⽤ ENUM ⽽不是 VARCHAR。如果你有⼀个字段，⽐如“性别”，“国家”，“⺠族”，“状态”或“部⻔”，你知道这些字段的取值是有限⽽且固定的，那么，你应该使⽤ ENUM ⽽不是VARCHAR 

- Prepared StatementsPrepared Statements很像存储过程，是⼀种运⾏在后台的SQL语句集合，我们可以从使⽤ prepared statements 获得很多好处，⽆论是性能问题还是安全问题。Prepared Statements 可以检查⼀些你绑定好的变量，这样可以保护你的程序不会受到“SQL注⼊式”攻击 

- 垂直分表 

- 选择正确的存储引擎 







## Hash索引和B+树索引有什么区别或优劣

**hash索引**底层就是hash表,进⾏查找时,调⽤⼀次hash函数就可以获取到相应的键值,之后进⾏回表查询获得实际数据.

**B+树**底层实现原理多路平衡查找树.对于每⼀次的查询都是从根节点出发,查找到叶⼦节点⽅可以获得所查键值,然后根据查询判断是否需要回表 查询数据. 



- hash索引进⾏等值查询更快(⼀般情况下),但是却⽆法进⾏范围查询. 
- hash索引不⽀持使⽤索引进⾏排序
- hash索引不⽀持模糊查询以及多列索引的最左前缀匹配.原理也是因为hash函数的不可预测.
- hash索引任何时候都避免不了回表查询数据,⽽B+树在符合某些条件(聚簇索引,覆盖索引等)的时候可以只通过索引完成查询. 
- hash索引虽然在等值查询上较快,但是不稳定.性能不可预测,当某个键值存在⼤量重复的时候,发⽣hash碰撞,此时效率可能极差.⽽B+树的查询效率⽐较稳定,对于所有的查询都是从根节点到叶⼦节点,且树的⾼度较低. 





# MySQL的联合查询 













































#### 相关资源来源

[1]: https://www.jianshu.com/p/338092a0a8c6

