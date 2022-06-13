---
微服务和SpringCloud
---

[toc]

-----

# 微服务版本选择

## Spring boot和SpringCloud的版本选择

### Spring boot版本选择

[Git源码地址](https://github.com/spring-projects/spring-boot/releases/)

[Spring boot2.x的新特性](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes)



### SpringCloud版本选择

[Git源码地址](https://github.com/spring-projects/spring-cloud)

[官网](https://spring.io/projects/spring-cloud)



### SpringCloud和spring boot之间的依赖关系查看

[地址](https://spring.io/projects/spring-cloud#overview)

[详细查看](https://start.spring.io/actuator/info)



### 学习版本选择

| cloud         | Hoxton.SR1    |
| ------------- | ------------- |
| boot          | 2.2.2.RELEASE |
| cloud alibaba | 2.1.0.RELEASE |
| Java          | Java8         |
| Maven         | 3.5及以上     |
| Mysql         | 5.7及以上     |



### IDEA 配置RunDashBoard

```xml
<component name="RunDashboard">
    <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
    </option>
    <option name="ruleStates">
      <list>
        <RuleState>
          <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
        </RuleState>
        <RuleState>
          <option name="name" value="StatusDashboardGroupingRule" />
        </RuleState>
      </list>
    </option>
  </component>
```





# Cloud各种组件的停更/升级/替换

![image-20220413140323270](http://qiliu.luxiaobai.cn/img/image-20220413140323270.png)



## 参考资料

[Springcloud](https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/)

[SpringCloud中文文档](https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md)

[Springboot](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/)



# 架构编码构建

==约定优于配置==



```pom

    <!-- 统一管理jar包版本 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
    </properties>
 
    <!-- 子模块继承之后，提供作用：锁定版本+子modlue不用写groupId和version  -->
    <dependencyManagement>
        <dependencies>
            <!--spring boot 2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud Hoxton.SR1-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba 2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.spring.boot.version}</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                    <addResources>true</addResources>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```



## 热部署Devtools

1. 添加Devtools依赖到module

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
       <scope>runtime</scope>
       <optional>true</optional>
   </dependency>
   ```

2. 添加插件到父类总工程中

   ```xml
   <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <configuration>
                       <fork>true</fork>
                       <addResources>true</addResources>
                   </configuration>
               </plugin>
           </plugins>
       </build>
   ```

3. 开启自动编译的选项

   <img src="http://qiliu.luxiaobai.cn/img/image-20220414164340683.png" alt="image-20220414164340683" style="zoom:60%;" />

4. `shift+command+alt+/ `点击`Registry`

![image-20220414164809040](http://qiliu.luxiaobai.cn/img/image-20220414164809040.png)



## RESTTemplate

> RestTemplate提供了多种便捷访问远程HTTP服务的方法,是一种简单便捷的访问resetful服务模板类,是Spring提供的用于访问Reset服务的客户端模板工具集

[官网地址](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)

- url:REST请求地址
- request Map:请求参数
- ResponseBean.class: HTTP响应转换被转换成的对象类型





# [Eureka](./注册中心/Eureka)



# [Zookeeper](./注册中心/Zookeeper)



# [Consul](./注册中心/Consul)



## 三个注册中心异同点

| 组件名    | 语言 | CAP  | 服务健康检查 | 对外暴露接口 | Springcloud集成 |
| --------- | ---- | ---- | ------------ | ------------ | --------------- |
| Eureka    | Java | AP   | 可配支持     | HTTP         | 已集成          |
| Consul    | Go   | CP   | 支持         | HTTP/DNS     | 已集成          |
| Zookeeper | Java | CP   | 支持         | 客户端       | 已集成          |



## CAP理论

- C:Consistency(强一致性)
- A:Availability(可用性)
- P:Partition tolerance(分区容错性)

==CAP理论关注粒度是数据,而不是整体系统设计的策略==

**最多只能同时较好的满足两个**

==CAP理论的核心是: 一个分布式系统不可能同时很好的满足一致性、可用性和分区容错性这三个需求==

**根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：** 

- CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
- CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。 
- AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

![image-20220426112520826](http://qiliu.luxiaobai.cn/img/image-20220426112520826.png)



## AP(Eureka)

### AP架构

> 当网络分区出现后，为了保证可用性，系统B 可以返回旧值 ，保证系统的可用性。 

1. 当没有出现网络分区时,系统A与系统B的数据一致性,X=1
2. 将系统A的X修改为2, X=2
3. 当出现网络分区后,系统A与系统B之间的数据同步失败,系统B的X=1
4. 当客户端请求系统B时,**为了保证可用性,此时系统B应返回旧值,X=1**

![image-20220426112739159](http://qiliu.luxiaobai.cn/img/image-20220426112739159.png)

==结论：违背了一致性C的要求，只满足可用性和分区容错，即AP==



## CP(Zookeeper/Consul)

### CP架构

>当网络分区出现后，为了保证一致性，就必须拒接请求，否则无法保证一致性

1. 当没有出现网络分区时,系统A与系统B的数据一致,X=1
2. 将系统A的X修改为2,X=2
3. 当出现网络分区时,系统A与系统B之间的数据同步数据失败,系统B的X=1
4. 当客户端请求系统B时,**为了保证一致性,此时系统B应拒绝服务请求,但会错误码或错误信息**

![image-20220426113153264](http://qiliu.luxiaobai.cn/img/image-20220426113153264.png)

==结论：违背了可用性A的要求，只满足一致性和分区容错，即CP==















































































































































































































































































































































































































































































































































