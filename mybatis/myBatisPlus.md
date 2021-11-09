

# 概述

Mybatis就是简化JDBC操作.

在MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生

### 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

## 

> 引入 `MyBatis-Plus` 之后请不要再次引入 `MyBatis` 以及 `MyBatis-Spring`，以避免因版本差异导致的问题。

# 快速开始

## 依赖导入

```xml
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.0</version>
        </dependency>
```

##  配置

```java
spring.datasource.username=root
spring.datasource.password=Sal123456
spring.datasource.url=jdbc:mysql://localhost:3306/mybatisplus?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

#mysql 8 驱动不同 驱动不同com.mysql.cj.jdbc.Driver, 需要增加是去配置 serverTimezone=GMT%2B8
```



==传统方法pojo-dao(连接mybatis,配置mapper.xml文件)-service-controller==

使用mybatis-plus之后

- pojo

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class User {
      private long id;
      private String name;
      private int age;
      private String email;
  }
  
  ```

  

- mapper接口

  ```java
  //在对应的mapper上面继承基本的类BaseMappper
  @Repository//代表持久战层
  public interface UserMapper extends BaseMapper<User> {
      //所有的CRUD操作都已经编写完成
      //不在需要配置一堆文件
  }
  ```

  >注意点: 需要在主启动类上去扫描我们的mapper包下的所有接口`@MapperScan("com.luxiaobai.mybatisplus.mapper")`

- 使用

  ```java
  	@Test
  	void contextLoads() {
  		//查询全部用户
  		//参数是一个Wrapper， 条件构造器， 不用就null
  		List<User> users = userMapper.selectList(null);
  		users.forEach(System.out::println);
  	}
  ```

  

# 配置日志

现在sql是不可见,需要查看日志信息来定位问题

```properties
#配置日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

![image-20210902111315825](/Users/lushengyang/Desktop/LSY/StudeyNotes/image/image-20210902111315825.png)



# CRUD扩展

```java
@Test
	public void testInsert(){
		User user = new User();
		user.setName("路小白");
		user.setAge(21);
		user.setEmail("lsy@dingtalk.com");
		int insert = userMapper.insert(user);
		System.out.println(insert);
	}
```



## 主键生成策略

>默认ID_WORKER全局唯一ID



分布式系统唯一ID生成: https://www.cnblogs.com/haoxinyue/p/5208136.html

### 雪花算法(**Twitter的snowflake算法**)

snowflake是Twitter开源的分布式ID生成算法，结果是一个long型的ID。其核心思想是：使用41bit作为毫秒数，10bit作为机器的ID（5个bit是数据中心，5个bit的机器ID），12bit作为毫秒内的流水号（意味着每个节点在每毫秒可以产生 4096 个 ID），最后还有一个符号位，永远是0。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    //对应数据库中的主键（uuid、自增id、雪花算法、redis、zookeeper)
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private String name;
    
    private Integer age;
    
    private String email;
}
```



```java
public enum IdType{
  AUTO(0); //数据库id自增
  NONE(1); //未设置主键
  INPUT(2); //手动输入
  ID_WORKER(3); //默认的全局唯一id
  UUID(4);//全局唯一ID uuid
  ID_WORKER_STR(5);//ID_WORKER 字符串表示法
}
```



## 自动填充

创建时间、修改时间

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    //对应数据库中的主键（uuid、自增id、雪花算法、redis、zookeeper)
    @TableId(type = IdType.AUTO)
    private Long id;

    private String name;

    private Integer age;

    private String email;

    //字段添加填充内容
    @TableField(fill= FieldFill.INSERT)
    private Date createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
}



@Slf4j
@Component //一定不要忘记把处理器加到IOC容器中！
public class MyMetaObjectHandler implements MetaObjectHandler {
    //插入时的填充策略
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill .....");
        this.setFieldValByName("createTime", new Date(), metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }

    //更新时的填充策略
    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill....");
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
```





## 乐观锁

> 当要更新一条记录的时候，希望这条记录没有被别人更新
>乐观锁实现方式：
>
>> - 取出记录时，获取当前version
>> - 更新时，带上这个version
>> - 执行更新时， set version = newVersion where version = oldVersion
>> - 如果version不对，就更新失败





















































































































