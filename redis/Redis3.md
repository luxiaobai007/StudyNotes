---
title: Redis6入门学习(三)
---

[toc]

# **Redis_Jedis_测试**

## **Jedis所需要的jar包**

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```



## **连接Redis注意事项**

禁用Linux的防火墙：Linux(CentOS7)里执行命令

```shell
systemctl stop/disable firewalld.service 
```

Redis.conf中

```shell
#bind 127.0.0.1
protected-mode no
```



## **Jedis常用操作**

**测试程序**

```java
public class ConnectRedis {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        String pong = jedis.ping();
        System.out.println("连接成功" + pong);
        //测试相关数据类型
        //Key
        jedis.set("k1","v1");
        jedis.set("k2","v2");
        jedis.set("k3","v3");
        Set<String> keys = jedis.keys("*");
        System.out.println(keys.size());
        for (String key : keys) {
            System.out.println(key);
        }
        System.out.println("Key API exists " + jedis.exists("k1"));
        System.out.println("Key API ttl " + jedis.ttl("k1"));
        System.out.println("Key API get " +jedis.get("k1"));

        //String
        jedis.mset("str1","v1","str2","v2","str3","v3");
        System.out.println("String API >>mset >>> mget " + jedis.mget("str1","str2","str3"));

        //List
        jedis.lpush("mylist","l1","l2","l3");
        List<String> mylist = jedis.lrange("mylist", 0, -1);
        for (String s : mylist) {
            System.out.println("List API lpush>>lrange: " + s);
        }


        //set
        jedis.sadd("orders","order01","order02","order3");
        Set<String> orders = jedis.smembers("orders");
        for (String order : orders) {
            System.out.println("Set API>>sadd>>smembers " + order);
        }
        jedis.srem("orders","order02");


        //hash
        jedis.hset("hash","userName","lisi");
        System.out.println("hash API:hset>>hget :" + jedis.hget("hash","userName"));
        HashMap<String, String> map = new HashMap<>();
        map.put("telphone","15759029165");
        map.put("address","longyan");
        map.put("email","1575018859@qq.com");
        jedis.hmset("hash2",map);
        List<String> result = jedis.hmget("hash2", "telphone", "email");
        for (String s : result) {
            System.out.println("hash " + s);
        }

        //zset
        jedis.zadd("zset01",100d,"z3");
        jedis.zadd("zset01",90d,"l4");
        jedis.zadd("zset01",80d,"w5");
        jedis.zadd("zset01",70d,"z5");
        Set<String> zset01 = jedis.zrange("zset01", 0, -1);
        for (String s : zset01) {
            System.out.println("zset>> " + s);
        }
        jedis.close();
    }
}
```



## Redis-Jedis-实例

### **手机验证码**

输入手机号，点击发送后随机生成6位数字码，2分钟有效

输入验证码，点击验证，返回成功或失败

每个手机号每天只能输入3次

```java
public class PhoneCode {
    public static void main(String[] args) {
        verifyCode("15759029165");
        //getRedisCode("15759029165","438209");
    }

    //1 生成手机验证码
    public static String getCode() {
        Random random = new Random();
        String code = "";
        for (int i = 0; i < 6; i++) {
            int ran = random.nextInt(10);
            code += ran;
        }
        return code;
    }

    // 2 每个手机每天只能发送3次，验证码放到redis中，设置过期时间
    public static void verifyCode(String phone) {
        Jedis jedis = new Jedis("121.199.76.44", 6379);
        //拼接Key
        //手机发送次数key
        String countKey = "VerifyCode" + phone + ":count";
        //验证码key
        String codeKey = "VerifyCode" + phone + ":code";

        String count = jedis.get(countKey);
        if (count == null) {
            //没有发送次数，第一次发送
            //设置发送次数是1
            jedis.setex(countKey, 24 * 60 * 60, "1");
        } else if (Integer.parseInt(count) <= 2) {
            jedis.incr(countKey);
        }else if (Integer.parseInt(count)>2){
            System.out.println("今天发送次数已经超过3次");
            jedis.close();
        }

        //发送验证码放到redis里面
        String vcode = getCode();
        jedis.setex(codeKey,60,vcode);
        jedis.close();
    }

    //3 验证码校验
    public static void getRedisCode(String phone,String code){
        Jedis jedis = new Jedis("121.199.76.44",6379);
        String codeKey = "VerifyCode" + phone + ":code";
        String redisCode = jedis.get(codeKey);
        if (redisCode.equals(code)){
            System.out.println("成功");
        }else {
            System.out.println("失败");
        }

        jedis.close();
    }
}
```



## **Redis与SpringBoot整合**

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.luxiaobai</groupId>
    <artifactId>spring-redis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-redis</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- spring2.X集成redis所需common-pool2-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.6.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### **appplication.properties**

```properties
#Redis服务器地址
spring.redis.host=192.168.0.101
spring.redis.port=6379

#Redis数据库索引（默认为0）
spring.redis.database=0
#连接超时时间（毫秒）
spring.redis.timeout=1800000
#连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=20
#最大阻塞等待时间（负数表示没有限制）
spring.redis.lettuce.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.lettuce.pool.min-idle=0
```

### **配置类**

```java
@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setConnectionFactory(factory);
        //key序列化方式
        template.setKeySerializer(redisSerializer);
        //value序列化
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //value hashmap序列化
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        //解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // 配置序列化（解决乱码的问题）,过期时间600秒
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }
}
```



### controller

```java
@RestController
@RequestMapping("/redisTest")
public class RedisTestController {
    @Autowired
    private RedisTemplate redisTemplate;

    @GetMapping
    public String testRedist(){
        //设置值到redis
        redisTemplate.opsForValue().set("name","lucy");
        //从redis取值
        String name = (String) redisTemplate.opsForValue().get("name");
        return name;
    }
}
```





# Redis事务----锁机制-秒杀

## **Redis的事务定义**

Redis事务是一个单独的隔离操作：**事务中的所有命令都会序列化、按顺序地执行**。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

Redis事务的主要**作用**就是**串联多个命令防止别的命令插队。**



## **Multi、Exec、discard**

从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。

组队的过程中可以通过discard来放弃组队。  

![redistran](http://qiliu.luxiaobai.cn/img/redistran.png)

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) OK
组队成功,提交成功

127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set m1 v1
QUEUED
127.0.0.1:6379(TX)> set m2
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379(TX)> set m3 v3
QUEUED
127.0.0.1:6379(TX)> exec
(error) EXECABORT Transaction discarded because of previous errors.
组队阶段报错,提交失败

127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set m1 v1
QUEUED
127.0.0.1:6379(TX)> incr m1
QUEUED
127.0.0.1:6379(TX)> set m2 v2
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
组队成功,提交有成功有失败情况
```



## **事务的错误处理**

组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。

![redistran1](/Users/lushengyang/Desktop/LSY/StudeyNotes/redis/Redis3.assets/redistran1.png)

如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。

![redistran2](http://qiliu.luxiaobai.cn/img/redistran2.png)

## **事务冲突问题**

**实例**

一个请求想给金额减8000

一个请求想给金额减5000

一个请求想给金额减1000

![redistan3](http://qiliu.luxiaobai.cn/img/redistan3.png)

### 悲观锁

![redistran4](http://qiliu.luxiaobai.cn/img/redistran4.png)

> **悲观锁(Pessimistic Lock)**, 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。



### 乐观锁

> 乐观锁(Optimistic Lock), 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用**版本号等机制**。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种check-and-set机制实现事务的。

![乐观锁](http://qiliu.luxiaobai.cn/img/%E4%B9%90%E8%A7%82%E9%94%81.png)





### WATCH key[key...]

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

```java
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> decrby balance 10
QUEUED
127.0.0.1:6379(TX)> incrby debt 10
QUEUED
127.0.0.1:6379(TX)> exec
1) (integer) -10
2) (integer) 10
```



### **unwatch**

取消 WATCH 命令对所有 key 的监视。

如果在执行 WATCH 命令之后，EXEC 命令或DISCARD 命令先被执行了的话，那么就不需要再执行UNWATCH 了。



## **Redis事务三特性**

- **单独的隔离操作**

- - 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

- **没有隔离级别的概念**

- - n 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行

- **不保证原子性**

- - n 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚



## **Redis-事务-秒杀案例**

### **解决计数器和人员记录的事务操作**

![redistran5](http://qiliu.luxiaobai.cn/img/redistran5.png)



### **Redis事务--秒杀并发模拟**

#### **工具ab模拟测试**

```shell
#ubuntu
sudo apt-get install apache2-utils 

#CentOS
sudo yum -y install httpd-tools
#联网
yum install httpd-tools
```

**vim postfile 模拟表单提交参数,以&符号结尾;存放当前目录。**

```shell
prodid=0101&
ab -n 2000 -c 200 -k -p ~/postfile -T application/x-www-form-urlencoded http://127.0.0.1:8081/Seckill/doseckill
```



## **超卖问题**

<img src="http://qiliu.luxiaobai.cn/img/redischao1.png" alt="redischao1" style="zoom: 67%;" />

**利用乐观锁淘汰用户，解决超卖问题**

<img src="http://qiliu.luxiaobai.cn/img/rediscaho2.png" alt="rediscaho2" style="zoom:67%;" />

**连接超时，通过连接池解决**

节省每次连接redis服务带来的消耗，把连接好的实例反复利用。通过参数管理连接的行为

```java
//连接池工具类
public class JedisPoolUtil {
    private static volatile JedisPool jedisPool = null;

    private JedisPoolUtil() {

    }

    public static JedisPool getJedisPoolInstance() {
        if (null == jedisPool) {
            synchronized (JedisPoolUtil.class) {
                if (null == jedisPool) {
                    JedisPoolConfig poolConfig = new JedisPoolConfig();
                    poolConfig.setMaxTotal(200);
                    poolConfig.setMaxTotal(32);
                    poolConfig.setMaxWaitMillis(100 * 1000);
                    poolConfig.setTestOnBorrow(true);//ping PONG

                    jedisPool = new JedisPool(poolConfig, "121.199.76.44", 6379, 6000);
                }
            }
        }
        return jedisPool;
    }

    public static void release(JedisPool jedisPool, Jedis jedis) {
        if (null != jedis) {
            jedisPool.returnResource(jedis);
        }
    }
}
```

**连接池参数**

- MaxTotal：控制一个pool可分配多少个jedis实例，通过pool.getResource()来获取；如果赋值为-1，则表示不限制；如果pool已经分配了MaxTotal个jedis实例，则此时pool的状态为exhausted。
- maxIdle：控制一个pool最多有多少个状态为idle(空闲)的jedis实例；
- MaxWaitMillis：表示当borrow一个jedis实例时，最大的等待毫秒数，如果超过等待时间，则直接抛JedisConnectionException；
- testOnBorrow：获得一个jedis实例的时候是否检查连接可用性（ping()）；如果为true，则得到的jedis实例均是可用的；



## **库存遗留问题**

![redisyiliu](http://qiliu.luxiaobai.cn/img/redisyiliu.png)



# LUA脚本

![LUA](http://qiliu.luxiaobai.cn/img/LUA.png)

>Lua 是一个小巧的[脚本语言](http://baike.baidu.com/item/脚本语言)，Lua脚本可以很容易的被C/C++ 代码调用，也可以反过来调用C/C++的函数，Lua并没有提供强大的库，一个完整的Lua解释器不过200k，所以Lua不适合作为开发独立应用程序的语言，而是作为嵌入式脚本语言。很多应用程序、游戏使用LUA作为自己的嵌入式脚本语言，以此来实现可配置性、可扩展性。
>
>这其中包括魔兽争霸地图、魔兽世界、博德之门、愤怒的小鸟等众多游戏插件或外挂。
>
>https://www.w3cschool.cn/lua/



**ubuntu 安装**

```shell
#下载编译包
curl -R -O http://www.lua.org/ftp/lua-5.3.0.tar.gz
tar zxf lua-5.3.0.tar.gz
cd lua-5.3.0
make linux
make install
```



![lua1](http://qiliu.luxiaobai.cn/img/lua1.png)

错误原因就是缺少依赖包libreadline-dev

```shell
centos: yum install readline-devel
Ubuntu: sudo apt-get install libreadline-dev.
```



## **LUA脚本在Redis中的优势**

将复杂的或者多步的redis操作，写为一个脚本，一次提交给redis执行，减少反复连接redis的次数。提升性能。LUA脚本是类似redis事务，有一定的原子性，不会被其他命令插队，可以完成一些redis事务性的操作。但是注意redis的lua脚本功能，只有在Redis 2.6以上的版本才可以使用。利用lua脚本淘汰用户，解决超卖问题。

redis 2.6版本以后，通过lua脚本解决争抢问题，实际上是redis 利用其单线程的特性，用任务队列的方式解决多任务并发问题

![lua2](http://qiliu.luxiaobai.cn/img/lua2.png)



```lua
local userid=KEYS[1]; 
local prodid=KEYS[2];
local qtkey="sk:"..prodid..":qt";
local usersKey="sk:"..prodid..":usr'; 
local userExists=redis.call("sismember",usersKey,userid);
if tonumber(userExists)==1 then 
  return 2;
end
local num= redis.call("get" ,qtkey);
if tonumber(num)<=0 then 
  return 0; 
else 
  redis.call("decr",qtkey);
  redis.call("sadd",usersKey,userid);
end
return 1;
```

```java
public class SecKill_redisByScript {
    private static final Logger logger = LoggerFactory.getLogger(SecKill_redisByScript.class);

    public static void main(String[] args) throws IOException {
        JedisPool jedisPool = JedisPoolUtil.getJedisPoolInstance();
        Jedis jedis = jedisPool.getResource();
        System.out.println(jedis.ping());
        Set<HostAndPort> set = new HashSet<HostAndPort>();

        //doSecKill("201","sk:0101");
    }

    static String secKillScript = "local userid=KEYS[1];\r\n" +
            "local prodid=KEYS[2];\r\n" +
            "local qtkey='sk:'..prodid..\":qt\";\r\n" +
            "local usersKey='sk:'..prodid..\":usr\";\r\n" +
            "local userExists=redis.call(\"sismember\",usersKey,userid);\r\n" +
            "if tonumber(userExists)==1 then \r\n" +
            "   return 2;\r\n" +
            "end\r\n" +
            "local num= redis.call(\"get\" ,qtkey);\r\n" +
            "if tonumber(num)<=0 then \r\n" +
            "   return 0;\r\n" +
            "else \r\n" +
            "   redis.call(\"decr\",qtkey);\r\n" +
            "   redis.call(\"sadd\",usersKey,userid);\r\n" +
            "end\r\n" +
            "return 1";

    static String secKillScript2 =
            "local userExists=redis.call(\"sismember\",\"{sk}:0101:usr\",userid);\r\n" +
                    " return 1";

    public static boolean doSecKill(String uid, String prodid) throws IOException {
        JedisPool jedisPool = JedisPoolUtil.getJedisPoolInstance();
        Jedis jedis = jedisPool.getResource();
        String sha1 = jedis.scriptLoad(secKillScript);
        Object result = jedis.evalsha(sha1, 2, uid, prodid);
        String reString = String.valueOf(result);
        if ("0".equals(reString)) {
            System.out.println("已抢空！！");
        } else if ("1".equals(reString)) {
            System.out.println("抢购成功！！");
        } else if ("2".equals(reString)) {
            System.out.println("该用户已抢过！！");
        } else {
            System.out.println("抢购异常！！");
        }
        jedis.close();
        return true;
    }
}
```


