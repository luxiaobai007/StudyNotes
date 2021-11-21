---
MyBatis基础学习(二)
---

[toc]



----

# 分页

减少数据的处理量



## 使用Limit分页

```sql
语法：
select * from user limit startIndex,pageSize;
select * from user limit 3;  #[0,n]
```

## 使用mybatis分页

1. 接口

   ```java
   List<User> getUserByLimit(Map<String, Integer> map);
   ```

2. Mapper.xml

   ```xml
   <select id="getUserByLimit" parameterType="map" resultType="user">
      select * from mybatis.user limit #{startIndex}, #{pageSize}
   </select>
   ```

3. 测试

```java
@Test
public void getUserListByLimit(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    HashMap<String, Integer> map = new HashMap<String, Integer>();
    map.put("startIndex", 1);
    map.put("pageSize", 2);
    List<User> userByLimit = userMapper.getUserByLimit(map);
    for (User user : userByLimit) {
        logger.info("user>>>" + user);
    }
    sqlSession.close();
}
```



## RowBounds分页

不再使用SQL实现分页

1. 接口

   ```java
   List<User> getUserBYRowBounds();
   ```

2. mapper.xml

   ```xml
   <select id="getUserBYRowBounds" resultType="user">
      select * from mybatis.user
   </select>
   ```

3. 测试

```java
@Test
public void getUserBYRowBounds(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    //RowBounds实现
    RowBounds rowBounds = new RowBounds(1, 2);
    //通过JAVA代码层面实现分页
    List<User> userList = sqlSession.selectList("com.luxiaobai.dao.UserMapper.getUserBYRowBounds",null, rowBounds);
    for (User user : userList) {
        logger.info(user);
    }
    sqlSession.close();
    
}
```

## 分页插件

![plus](http://qiliu.luxiaobai.cn/img/plus.png)

### **使用注解开发**

#### **面向接口编程**

**根本原因： 解 耦，可拓展，提供复用，分层开发中，上层不用管具体的实现，大家都遵守共同的标准，使得开发变得容易，规范性更好。**



1. 注解在接口上实现

   ```java
   @Select("select * from user")
   List<User> getUsres();
   ```

2. 在核心配置文件中绑定接口

   ```xml
   <mappers>
       <mapper class="com.luxiaobai.dao.UserMapper"/>
   </mappers>
   ```

3. 测试

```java
@Test
    public void test(){
       SqlSession sqlSession = MybatisUtils.getSqlSession();
//       底层主要应用反射
       UserMapper user = sqlSession.getMapper(UserMapper.class);
       List<User> userList = user.getUsres();

       for (User user1 : userList) {
           System.out.println(user1);
       }
       sqlSession.close();
   }
```

>本质：反射机制实现
>
>底层：动态代理！



### mybatis执行的流程

![流程](http://qiliu.luxiaobai.cn/img/%E6%B5%81%E7%A8%8B.png)



### **使用注解CRUD**

在工具类创建的时候显示自动提交事务

```java
public static SqlSession getSqlSession() {
    return sqlSessionFactory.openSession(true);
}
```

编写接口，增加注解

```java
@Select("select * from user")
List<User> getUsres();

//方法存在多个参数，所有参数前面必须用@Param注解
@Select("select * from user where id = #{id}")
User getUserById(@Param("id") int id);

@Insert("insert into user(id,name,pwd) values (#{id},#{name},#{pwd})")
int addUser(User user);

@Update("update user set name=#{name}, pwd=#{pwd} where id=#{id}")
int updateUser(User user);

@Delete("delete from user where id=#{uid}")
int deleUser(@Param("uid") int id);
```



#### **关于@Param()注解**

- 基本类型的参数或者 String类型，需要加上
- 引用类型不需要加
- 如果只有一个基本类型的话，可以忽略，但是建议加上
- 在SQL中引用的就是@Param("uid")中设定的属性名uid

#### **#{}和${}**

- \#{}是预编译处理，${}是字符串替换。
- Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，调用 PreparedStatement 的 set 方法来赋值；
- Mybatis 在处理${}时，就是把${}替换成变量的值。
- 使用#{}可以有效的防止 SQL 注入，提高系统安全性。



## Lombok

```java
@Data：  无参构造、get、set、tostring、hashcode、equals
@AllArgsConstructor
@NoArgsConstructor
@EqualsAndHashCode
@ToString 
```



## **多对一处理**

- 多个学生，对应一个老师
- 对于学生这边而言，关联，多个学生，关联一个老师 【多对一】
- 对于老师而言，集合 一个老师，有很多学生 【一对多】



sql

```sql
create table `teacher` (
    `id` int(10) not null,
    `name` varchar(30) default null,
    primary key (`id`)
) ENGINE =INNODB DEFAULT CHARSET =UTF8;

INSERT INTO teacher(id, name) values(1, '王老师');

create table `student`(
    `id` int(10) not null ,
    `name` varchar(20) default null,
    `tid` int(10) default null,
    primary key (`id`),
    key `fktid` (`tid`),
    constraint `fktid` foreign key (`tid`) references `teacher` (`id`)
);

INSERT INTO student(id, name,tid) values(1, '小迷', '1');
INSERT INTO student(id, name,tid) values(2, '小红', '1');
INSERT INTO student(id, name,tid) values(3, '小明', '1');
INSERT INTO student(id, name,tid) values(4, '小光', '1');
INSERT INTO student(id, name,tid) values(5, '小蓝', '1');
INSERT INTO student(id, name,tid) values(6, '小白', '1');
INSERT INTO student(id, name,tid) values(7, '小黑', '1');
```



**根据查询嵌套处理**

```xml
<!--    查询所有学生信息-->
<!--    根据查询出来的学生的tid，寻找对应的老师！-->
<select id="getStudentList" resultMap="StudentTeacher">
    select *
    from student
</select>

<resultMap id="StudentTeacher" type="Student">
    <result property="id" column="id"/>
    <result property="name" column="name"/>
    <!--        复杂的属性，需要单独处理 对象：association  集合：collection-->
    <association proper ty="teacher" column="tid" javaType="Teacher" select="getTeacher"/>

</resultMap>

<select id="getTeacher" resultType="Teacher" parameterType="int">
    select *
    from teacher
    where id = #{id}
</select>
```



**根据结果嵌套处理**

```xml
 <!--    按照结果嵌套处理-->
<select id="getStudentList2" resultMap="StudentTeacher2">
    select s.id sid, s.name sname, t.name tname from student s , teacher t where s.tid = t.id;
</select>
    
    <resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid" />
        <result property="name" column="sname" />
        <association property="teacher" javaType="Teacher">
            <result property="name" column="tname" />
        </association>
    </resultMap>
```



**Mysq多对一查询方式**

- 子查询
- 联表查询

**一对多**

**实体类**

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    private int id;
    private String name;
    private int tid;
}


@Data
@NoArgsConstructor
@AllArgsConstructor
public class Teacher {
    private int id;
    private String name;

    private List<Student> students;
}
```



**按结果嵌套处理**

```xml
<select id="getTeacher" resultMap="TeacherStudent">
    select s.id sid, s.name sname, t.id tid, t.name tname
    from student s,
         teacher t
    where s.tid = t.id
      and t.id = #{tid}
</select>
<resultMap id="TeacherStudent" type="Teacher">
    <result property="id" column="id"/>
    <result property="name" column="tname"/>
    <!--        复杂属性，需要单独处理 对象：association 集合：collection-->
    <!--        javaType="" 指定属性的类型-->
    <!--        集合中的泛型信息，使用ofType获取-->
    <collection property="students" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="tid"/>
    </collection>
</resultMap>
```



**按照查询嵌套处理**

```sql
<select id="getTeacherList2" resultMap="TeacherStudent2">
    select *
    from teacher
    where id = #{tid }
</select>
<resultMap id="TeacherStudent2" type="Teacher">
    <collection property="students" javaType="ArrayList" ofType="Student" select="getStudentByteacherId"
                column="id"/>
</resultMap>
<select id="getStudentByteacherId" resultType="Student">
    select *
    from student
    where tid = #{tid}
</select>
```

**小结**

1. 关联 - association [多对一]

2. 集合 -  collection [一对多]

3. javaType & ofType

4. 1. javaType 用来指定实体类中属性的类型
   2. ofType 用来指定映射到List或者集合中的pojo类型，泛型中的约束类型！



>**面试高频**
>
>- MySQL引擎
>- InnoDB底层原理
>- 索引
>- 索引优化



# 动态SQL

**根据不同的条件生成不同的SQL语句**

```sql
##借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。
if
choose (when, otherwise)
trim (where, set)
foreach
```



```sql
CREATE TABLE `blog`
(
    `id`          VARCHAR(50)  NOT NULL COMMENT '博客id',
    `title`       VARCHAR(100) NOT NULL COMMENT '博客标题',
    `author`      VARCHAR(30)  NOT NULL COMMENT '博客作者',
    `create_time` DATETIME     NOT NULL COMMENT '创建时间',
    `views`       INT(30)      NOT NULL COMMENT '浏览量'
) ENGINE = INNODB
  DEFAULT CHARSET = utf8;
```



### IF

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from blog where 1=1
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</select>
```

### Choose(when, otherwise)

```xml
<select id="queryBlogChoose" parameterType="map" resultType="blog">
    select * from blog
    <where>
        <choose>
            <when test="title != null">
                title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
```



### trim(where, set)

```xml
<select id="queryBlogIf2" parameterType="map" resultType="blog">
    select * from blog
    <where>
        <if test="title != null">
            title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </where>
</select>
<update id="updateBlog" parameterType="map">
    update blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author}
        </if>
    </set>
    where id=#{id}
</update>
```

 **trim标签**

```xml
　select * from user
 <trim prefix="WHERE" prefixoverride="AND |OR">
　　　　<if test="name != null and name.length()>0"> AND name=#{name}</if>
　　　　<if test="gender != null and gender.length()>0"> AND gender=#{gender}</if>
　　</trim>
  sql:select * from user where 空格 name='xx' and gender = 'xx'
```

- prefix：前缀
- prefixoverride：去掉第一个and或者是or

```xml
update user
　　<trim prefix="set" suffixoverride="," suffix=" where id = #{id} ">
　　　　<if test="name != null and name.length()>0"> name=#{name} , </if>
　　　　<if test="gender != null and gender.length()>0"> gender=#{gender} ,  </if>
　　</trim>
  sql:update user set name='xx', gender='xx' 空格 where id='x'
```

- suffixoverride:去掉最后一个逗号
- suffix：后缀

### Foreach

```java
<!--    select * from blog where 1=1 and (id1 or id=2 or id=3)-->
<!--    传递一个map，map中可以存在一个集合-->
    <select id="queryBlogForeach" parameterType="map" resultType="blog">
        select * from blog
        <where>
            <foreach collection="ids" item="id" separator="or" open="and (" close=")">
                id=#{id}
            </foreach>
        </where>
    </select>
    
 @Test
public void queryBlogForeach(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);
    HashMap map = new HashMap();

    ArrayList<Integer> ids = new ArrayList<>();
    ids.add(1);
    ids.add(2);
    map.put("ids", ids);

    List<Blog> blogList = blogMapper.queryBlogForeach(map);
    for (Blog blog : blogList) {
        System.out.println(blog.toString());
    }
    sqlSession.close();
}
```

动态SQL就是拼接SQL语句，只要保证SQL的正确性，按照SQL的格式 ，去排列组合

- 在MySQL中写出完整的SQL，在对应修改成为动态SQL



### **SQL片段**

1. 使用SQL标签抽取公共的部分

   ```xml
   <sql id="sql-if">
       <if test="title != null">
           and title = #{title}
       </if>
       <if test="author != null">
           and author = #{author}
       </if>
   </sql>
   ```

   

2. 在需要的地方使用include标签引用即可

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from blog where 1=1
   <include refid="sql-if"></include>
</select>
```

**注意事项：**

- 最好基于单表来定义SQL片段
- 不要存在where标签





# 缓存

## **简介**

1. ### 什么是缓存【cache】

2. 1. 存在内存中的临时数据
   2. 将用户经常出现的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库数据文件）查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题

3. ### 为什么使用缓存

4. 1. 减少和数据库的交互次数，减少系统开销，提高系统效率

5. ### 什么样的数据能使用缓存

6. 1. 经常查询并且不经常改变的数据

## **Mybatis缓存**

- mybatis包含一个非常强大的查询缓存特性，可以非常方便的定制和配置缓存，缓存可以极大的提升查询效率

- mybatis系统中默认定义了两级缓存：**一级缓存**和**二级缓存**

- - 默认情况情况下，只有一级缓存开启（SqlSession级别的缓存，也称为本地缓存）
  - 二级缓存需要手动开启和配置，它是基于namespace级别的缓存
  - 为了提高扩展性，mybatis定义了缓存接口Cache，通过实行Cache接口来自定义二级缓存

### **一级缓存**

- 一级缓存也叫本地缓存：SqlSession

- - 与数据库同一次会话期间查询到的数据会放在本地缓存中
  - 以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库

### **缓存失效的情况：**

1.  查询不同的东西
2. 增删改操作，可能会改变原来的数据，所以必定会刷新缓存
3. 查询不同的Mapper.xml
4. 手动清理缓存

```java
sqlSession.clearCache()
```



### **二级缓存**

- 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存

- 基于namespace级别的缓存，一个名称空间，对应一个二级缓存

- 工作机制

- - 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中
  - 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中
  - 新的会话查询信息，就可以从二级缓存中获取内容；
  - 不同的mapper查出的数据放在自己对应的缓存（map)中



步骤：

1. 开启全局缓存

   ```xml
   <!--显示的开启全局缓存-->
   <setting name="cacheEnabled" value="true"/>
   ```

2. 在要使用二级缓存的mapper中开启

```xml
<!--在当前Mapper.xml中使用二级缓存-->
<cache eviction="FIFO"
       flushInterval="60000"
       size="512"
       readOnly="true"/>
```

- **只要开启了二级缓存，在同一个Mapper下就有效**
- **所有的数据都会先放在一级缓存中**
- **只有当会话提交，或者关闭的时候，才会提交到二级缓存中** 





### 自定义缓存

```xml
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>
```

在mapper中指定我们的ehcache缓存实现

```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache" />
```

ehcache.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">

    <diskStore path="./tmpdir/Tmp_EhCache"/>
    <defaultCache
            eternal="false"
            maxElementsInMemory="10000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="259200"
            memoryStoreEvictionPolicy="LRU"/>
    <cache
            name="cloud_user"
            eternal="false"
            maxElementsInMemory="5000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="1800"
            memoryStoreEvictionPolicy="LRU"/>
</ehcache>
```

![cache1](http://qiliu.luxiaobai.cn/img/cache1.png)

![cache2](http://qiliu.luxiaobai.cn/img/cache2.png)



# **Mybatis自定义插件**

1. 依赖注入

   ```xml
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus-boot-starter</artifactId>
       <version>3.4.0</version>
   </dependency>
   ```

2. 编写Interceptor的实现类

3. 使用@Intercepts注解完成插件签名

```java
package com.luxiaobai.dao;

import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.plugin.*;

import java.util.Properties;

/**
 * @author ：luxiaobai
 * @date ：Created in 2021/7/28 14:56
 * @description：自定义插件
 * @modified By：`
 * @version: 1.0
 */

/**
 * type:要拦截的那个对象
 * method：要拦截的那个方法
 * args:拦截方法的参数
 *   完成插件签名：告诉mybatis当前插件用来拦截那个对象的那个方法
 */

@Intercepts({
        @Signature(type = StatementHandler.class, method = "parameterize", args = java.sql.Statement.class)
})
public class MyFirstPlugin implements Interceptor {
    /**
     * intercept：拦截： 拦截目标对象的目标方法的执行
     *
     * @param invocation
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        //执行目标方法
        Object proceed = invocation.proceed();
        //返回执行后的返回值
        return proceed;
    }

    /**
     * plugin:包装目标对象的：包装：为目标对象创建一个代理对象
     *
     * @param target
     * @return
     */
    @Override
    public Object plugin(Object target) {
//        借助Plugin的wrap方法来使用当前interceptor包装我们的目标对象
        Object wrap = Plugin.wrap(target, this);
//        返回为当前target创建的动态代理
        return wrap;
    }

    /**
     * 将插件注册时的property属性设置进来
     *
     * @param properties
     */
    @Override
    public void setProperties(Properties properties) {
        System.out.println("插件配置的信息：" + properties);
    }
}
```

​	4. 将写好的插件注册到全局配置文件中

```xml
<!--    注册插件-->
<plugins>
    <plugin interceptor="com.luxiaobai.dao.MyFirstPlugin"></plugin>
</plugins>
```

