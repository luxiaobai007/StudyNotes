---
MyBatis基础学习(一)
---

[toc]

# 基本概念

## Mvn仓库

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
</dependency>
```



## 持久化

### **数据持久化**

- 持久化就是将程序的数据在持久状态和瞬时状态转化的过程
- 内存:断电即失
- 数据库(JDBC),Io文件持久化.





### **持久层**

Dao层,Service,Controller层..

- 完成持久化工作的代码块
- 层界限十分明显

## 优点

- 简单易学
- 灵活
- sql和代码的分离，提高了可维护性。
- 提供映射标签，支持对象与数据库的orm字段关系映射
- 提供对象关系映射标签，支持对象关系组建维护
- 提供xml标签，支持编写动态sql。 



## 实战入门

**pom.xml**

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
</dependency>
<!--junit-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```



**mybatis-config.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="Sal123456"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```



**编写mybatis工具类**

```java
package com.luxiaobai.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

//sqlSessionFoctory --> sqlSession
public class MybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            //1、使用获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
             sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

//    既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```



**实体类**

```java
package com.luxiaobai.pojo;

/**
 * @author ：luxiaobai
 * @date ：Created in 2021/7/24 00:29
 * @description：用户实体
 * @modified By：`
 * @version: 1.0
 */

public class User {
	private int id;
	private String name;
	private String pwd;

	public User() {
		super();
		// TODO Auto-generated constructor stub
	}

	public User(int id, String name, String pwd) {
		super();
		this.id = id;
		this.name = name;
		this.pwd = pwd;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getPwd() {
		return pwd;
	}

	public void setPwd(String pwd) {
		this.pwd = pwd;
	}

	@Override
	public String toString() {
		return "User [id=" + id + ", name=" + name + ", pwd=" + pwd + "]";
	}
	
}
```

**Dao接口**

```java
package com.luxiaobai.dao;

import java.util.List;

import com.luxiaobai.pojo.User;

public interface UserDao {
	List<User> getUserList();
}
```

**mapper**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 <!-- namespace=绑定一个对应的Dao/Mapper接口 --> 
<mapper namespace="com.luxiaobai.dao.UserDao">
	<select id="getUserList" resultType="com.luxiaobai.pojo.User">
		select * from mybatis.user; 
	</select>
</mapper>
```



**测试**

```xml
org.apache.ibatis.binding.BindingException: Type interface com.luxiaobai.dao.UserDao is not known to the MapperRegistry.


<!-- 在build中配置resources，来防止资源导出失败问题-->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```



## CRUD

**1、namespace**

namespace中的包名要和Dao/mapper接口的包名一致！

**2、select**

- id:对应namespace中的方法名
- resultType：SQL语句执行的返回值
- parameterType: 参数类型

**3、Insert**

**4、update**

**5、Delete**

- **增删改需要提交事务**



**万能Map**

假设：实体类，或者数据库中的表、字段或者参数过多



```java
int addUser2(Map<String, Object> map);

<insert id="addUser2" parameterType="map">
   insert into mybatis.user (id, name, pwd) values (#{userId}, #{userName}, #{userPasswd})
</insert>

@Test
public void addUser2(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    Map<String, Object> userMap = new HashMap<>();
    userMap.put("userId", 6);
    userMap.put("userName", "ks");
    userMap.put("userPasswd", "admin");
    userMapper.addUser2(userMap);
    sqlSession.commit();
    sqlSession.commit();
}
```

Map传递参数，直接在SQL中取出key即可， [parameterType="map"]

对象传递参数，直接在SQL中取对象的属性即可 [parameterType="Object"]

只有一个基本类型参数的情况下，可以直接在SQL中取到

**多个参数用map，或者注解**



## 模糊查询

```java
##使用这种防止SQL注入
<select id="getUserBylike" resultType="com.luxiaobai.pojo.User">
   select * from mybatis.user where name like "%"#{value}"%"
</select>
@Test
public void getUsersLike(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    List<User> userlist = userMapper.getUserBylike("xiao");
    for (User user : userlist) {
        System.out.println(user);
    }
    sqlSession.close();
}


##不推荐
<select id="getUserBylike" resultType="com.luxiaobai.pojo.User">
   select * from mybatis.user where name like #{value}
</select>

@Test
public void getUsersLike(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    List<User> userlist = userMapper.getUserBylike("%xiao%");
    for (User user : userlist) {
        System.out.println(user);
    }
    sqlSession.close();
}
```





# 配置解析

## 核心配置

**mybatis-config.xml**

```markdown
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```



**环境配置（environments）**

MyBatis可以配置 成适应多种环境

**注意：尽管可以配置多个环境，但每个SqlSessionFactory实例只能选择一种环境。**

Mybatis默认的事务管理器就是JDBC，连接池：POOLED

**属性（properties）**

通过properties 属性来实现引用配置文件。

可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。[db.properties]

- 编写db.properties配置文件

  ```xml
  driver=com.mysql.cj.jdbc.Driver
  url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"
  username=root
  password=Sal123456
  ```

  

- 在核心配置中引入

  ```xml
  <properties resource="db.properties">
      <property name="username" value="root"/>
      <property name="password" value="Sal123456"/>
  </properties>
  ```

  

- 可以直接引入外部文件

- 可以在其中增加一些属性配置

- 如果两个文件有同一个字段，优先使用外部的配置文件（db.properties)

**类型别名（typeAliases）**

- 类名别名是为JAVA类型设置一个短的名字
- 意义：仅在与用来减少类完全限定名的冗余

```xml
<!--    给实体类起别名-->
<typeAliases>
    <typeAlias type="com.luxiaobai.pojo.User" alias="User"/>
</typeAliases>
```

也可以指定包名，MyBatis会在包名下面搜索需要的Java Bean，如：扫描实体类的包，它的默认别名就为这个类的类名，首字母小写（推荐，大写也可以）

```xml
<!--    给实体类起别名-->
    <typeAliases>
        <package name="com.luxiaobai.pojo"/>
    </typeAliases>
```

在实体类比较少的时候，使用第一种方式

实体类十分多，建议使用第二种

第一种可以DIY别名，第二种则不行



## **设置（setting）**

![截屏2021-11-05 23.31.58](http://qiliu.luxiaobai.cn/img/%E6%88%AA%E5%B1%8F2021-11-05%2023.31.58.png)

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

## **其他配置**

- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)

- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)

- [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)

- - mybatis-generator-core
  - mybatis-plus
  - 通用mapper

**映射器（mappers)**

MapperRegistry:注册绑定我们的Mapper文件

**方式一：**

```xml
<!--每一个Mapper.xml 都需要在mybatis核心配置文件中注册 -->
<mappers>
    <mapper resource="com/luxiaobai/dao/UserMapper.xml"/>
</mappers>
```

**方式二：使用class文件绑定注册**

```xml
<mappers>
   <mapper class="com.luxiaobai.dao.UserMapper" />
 </mappers>
```

注意点：

- 接口和它的mapper配置文件必须同名
- 接口和它的mapper配置文件必须在同一个包下

**方式三：使用扫描包进行诸如绑定**

```xml
<mappers>
        <package name="com.luxiaobai.dao.UserMapper"/>
    </mappers>
```

注意点：

- 接口和它的mapper配置文件必须同名
- 接口和它的mapper配置文件必须在同一个包下

## **生命周期和作用域（scope）** 

生命周期和作用域至关重要，错误的使用会导致非常严重的并发问题

**SqlSessionFactoryBuilder:**

- 一旦触及了SqlSessionFactory,就不再需要它了
- 局部变量

## **SqlSessionFactory:**

- 可以想象为：数据库连接池 
- SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**
- **SqlSessionFactory 的最佳作用域是应用作用域**
- **最简单的就是使用单例模式或者静态单例模式**

## **SqlSession:** 

- 连接到连接池的一个请求
- SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域
- 用完之后关闭，否则资源被占用

> **解决属性名和字段名不一致的问题**

- 起别名

```sql
select id,name,pwd as password from mybatis.user where id = #{id}
```

## **resultMap**

**结果集映射**

```xml
<resultMap id="UserMap" type="User">
    <!--      column数据库中的字段， property实体类中的属性-->
    <!--        <result column="id" property="id"/>-->
    <!--        <result column="name" property="name"/>-->
    <result column="pwd" property="password"/>
</resultMap>


<select id="getUserById" resultMap="UserMap" parameterType="int">
    select * from mybatis.user where id = #{id}
</select>
```

==resultMap元素是MyBatis中最重要最强大的元素==



# 日志

## 日志工厂

- SLF4J 
-  LOG4J 【掌握】
-  LOG4J2 
- JDK_LOGGING 
- COMMONS_LOGGING 
- STDOUT_LOGGING 【掌握】
-  NO_LOGGING

mybatis中具体使用那一个日志实现，在设置中设定

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

**STDOUT_LOGGING标准日志输出**

![mybatis10](http://qiliu.luxiaobai.cn/img/mybatis10.png)

**LOG4J**

-  Log4j是[Apache](https://baike.baidu.com/item/Apache/8512995)的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是[控制台](https://baike.baidu.com/item/控制台/2438626)、文件、[GUI](https://baike.baidu.com/item/GUI)组件
- 可以控制每一条日志的输出格式；
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程
- 可以通过一个[配置文件](https://baike.baidu.com/item/配置文件/286550)来灵活地进行配置，而不需要修改应用的代码。

1. 导入

   ```xml
   <dependency>
       <groupId>log4j</groupId>
       <artifactId>log4j</artifactId>
       <version>1.2.17</version>
   </dependency> 
   ```

   

2. log4j.properties

   ```properties
   log4j.rootLogger=DEBUG,console,file
   
   #控制台输出的相关设置
   log4j.appender.console = org.apache.log4j.ConsoleAppender
   log4j.appender.console.Target = System.out
   log4j.appender.console.Threshold=DEBUG
   log4j.appender.console.layout = org.apache.log4j.PatternLayout
   log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
   
   #文件输出的相关设置
   log4j.appender.file = org.apache.log4j.RollingFileAppender
   log4j.appender.file.File=./log/luxiaobai.log
   log4j.appender.file.MaxFileSize=10mb
   log4j.appender.file.Threshold=DEBUG
   log4j.appender.file.layout=org.apache.log4j.PatternLayout
   log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
   
   #日志输出级别
   log4j.logger.org.mybatis=DEBUG
   log4j.logger.java.sql=DEBUG
   log4j.logger.java.sql.Statement=DEBUG
   log4j.logger.java.sql.ResultSet=DEBUG
   log4j.logger.java.sql.PreparedStatement=DEBUG
   ```

   

3. 配置log4j为日志的实现

```xml
<settings>
   <setting name="logImpl" value="LOG4J"/>
</settings>
```

**简单使用**

1. 在要使用log4j的类中，导入包import org.apache.log4j.Logger;
2. 日志对象，参数为当前类的class  

```java
static Logger logger = Logger.getLogger(UserMapperTest.class);
```

