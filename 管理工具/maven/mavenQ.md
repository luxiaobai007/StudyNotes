[toc]



# Maven是什么

M a v e n 是 ⼀ 个 项 ⽬ 管 理 ⼯ 具，它 包 含 了 ⼀ 个 项 ⽬ 对 象 模 型（ P r o j e c t O b j e c t M o d e l ），反 映 在 配 置 中，就 是 ⼀ 个 p o m . x m l ⽂ 件 。是 ⼀ 组 标 准 集 合，⼀ 个 项 ⽬ 的 ⽣ 命 周 期 、⼀ 个 依 赖 管 理 系 统，另 外 还 包 括 定 义在 项 ⽬ ⽣ 命 周 期 阶 段 的 插 件 ( p l u g i n ) 以 及 ⽬ 标 ( g o a l ) 。

核心:

- **依赖管理**：对 jar 的统⼀管理(Maven 提供了⼀个 Maven 的中央仓库，https://mvnrepository.com/，当我们在项⽬中添加完依赖之后，Maven 会⾃动去中央仓库下载相关的依赖，并且解决依赖的依赖问题)
- **项⽬构建**：对项⽬进⾏编译、测试、打包、部署、上传到私服等





## 常用命令

| 常用命令    | 中文含义 | 说明                                      |
| ----------- | -------- | ----------------------------------------- |
| mvn clean   | 清理     | 用来清理已经编译好的文件                  |
| mvn compile | 编译     | 将Java代码编译成Class文件                 |
| mvn test    | 测试     | 项目测试                                  |
| mvn package | 打包     | 根据用户的配置,将项目打包成jar包或者war包 |
| mvn install | 安装     | 手动向本地仓库安装一个jar                 |
| mvn deploy  | 上传     | 将jar上传到私服                           |



# **dependencies和dependencyManagement,plugins和pluginManagement有什么区别?**

dependencyManagement 是表⽰依赖 jar 包的声明，即你在项⽬中的 dependencyManagement 下声明了依赖，maven 不会加载该依赖，dependencyManagement 声明可以被继承。

dependencyManagement 的⼀个使⽤案例是当有⽗⼦项⽬的时候，⽗项⽬中可以利⽤ dependencyManagement 声明⼦项⽬中需要⽤到的依赖 jar 包，之后，当某个或者某⼏个⼦项⽬需要加载该插件的时候，就可以在⼦项⽬中 dependencies 节点只配置groupId 和 artifactId 就可以完成插件的引⽤。

dependencyManagement 主要是**为了统⼀管理插件**，**确保所有⼦项⽬使⽤的插件版本保持⼀致**，类似的还有 plugins 和pluginManagement。



# 如何制定编码

在properties中制定project.build.sourceEncoding

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```



# 如何打包一个可以直接运行的Springboot 的jar包

可以使用spring-boot-maven-plugin插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



## 如果引入了第三方jar包,如何打包?

首先添加依赖

```xml
<dependency>
    <groupId>io.github.dunwu</groupId>
    <artifactId>dunwu-common</artifactId>
    <version>1.0.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/src/main/resources/lib/dunwu-common-1.0.0.jar</systemPath>
</dependency>
```

接着配置spring-boot-maven-plugin插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                   </goals>
                </execution>
            </executions>
            <configuration>
                <includeSystemScope>true</includeSystemScope>
            </configuration>
        </plugin>
    </plugins>
</build>
```



# 如何指定maven构建时的JDK版本

## properties方式

```xml
<project>
...
    <properties>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>
...
</project>
```



## 使用maven-compiler-plugin插件,并指定source和target版本

```xml
<build>
...
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.3</version>
        <configuration>
            <source>1.7</source>
            <target>1.7</target>
        </configuration>
    </plugin>
</plugins>
...
</build>
```



# 如何避免将dependency打包到构件中?

指定 maven dependency 的 scope 为 provided ，这意味着：依赖关系将在运⾏时由其容器或 JDK 提供。具有此范围的依赖关系不会传递，也不会捆绑在诸如 WAR 之类的包中，也不会包含在运⾏时类路径中



# **如何跳过单元测试**

执⾏ mvn package 或 mvn install 时，会⾃动编译所有单元测试(src/test/java ⽬录下的代码)，如何跳过这⼀步？

在执⾏命令的后⾯，添加命令⾏参数 **-Dmaven.test.skip=true** 或者 **-DskipTests=true**



# 如何排除依赖

如何排除依赖⼀个依赖关系？⽐⽅项⽬中使⽤的 libA 依赖某个库的 1.0 版。libB 以来某个库的 2.0 版，如今想统⼀使⽤ 2.0版，怎样去掉 1.0 版的依赖？

## 通过 exclusion 排除指定依赖

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.10.0</version>
    <exclusions>
        <exclusion>
            <artifactId>log4j-api</artifactId>
            <groupId>org.apache.logging.log4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

og4j-core 本⾝是依赖了 log4j-api的，但是因为⼀些其他的模块也依赖了 log4j-api，并且两个 log4j-api版本不同，所以我们使⽤标签排除掉 log4j-core 所依赖的 log4j-api，这样Maven就不会下载 log4j-core 所依赖的 log4j-api了，也就保证了我们的项⽬中只有⼀个版本的 log4j-api。

Maven Helper这个插件可以查看项目中哪些依赖的Jar包冲突了,在pom.xml文件的底部会多出一个Dependency Analyzer









































































































































































































