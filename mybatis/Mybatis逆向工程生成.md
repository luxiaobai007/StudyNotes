---
MyBatis逆向工程生成---STS实现
---

[toc]

**1、下载MyBatis-generator 插件**

![mybatis1](http://qiliu.luxiaobai.cn/img/mybatis1.png)

![mybatis2](http://qiliu.luxiaobai.cn/img/mybatis2.png)

**2、**[**http://mybatis.org/generator/configreference/xmlconfig.html**](http://mybatis.org/generator/configreference/xmlconfig.html)**查找mybatis逆向工程的配置文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
  <classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />

  <context id="DB2Tables" targetRuntime="MyBatis3">
    <jdbcConnection driverClass="COM.ibm.db2.jdbc.app.DB2Driver"
        connectionURL="jdbc:db2:TEST"
        userId="db2admin"
        password="db2admin">
    </jdbcConnection>

    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

    <javaModelGenerator targetPackage="test.model" targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

    <sqlMapGenerator targetPackage="test.xml"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

    <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

    <table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
      <property name="useActualColumnNames" value="true"/>
      <generatedKey column="ID" sqlStatement="DB2" identity="true" />
      <columnOverride column="DATE_FIELD" property="startDate" />
      <ignoreColumn column="FRED" />
      <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
    </table>

  </context>
</generatorConfiguration>
```

**3、在resources目录下新建generatorConfig.xml文件,并修改相关配置**

![mybatis3](http://qiliu.luxiaobai.cn/img/mybatis3.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<classPathEntry
		location="/opt/apache-maven-3.6.2/repository/mysql/mysql-connector-java/8.0.25/mysql-connector-java-8.0.25-sources.jar" />

	<context id="DB2Tables" targetRuntime="MyBatis3">
		<!-- 不生成注释 -->
		<commentGenerator>
			<property name="suppressDate" value="true" />
			<property name="suppressAllComments" value="true" />
		</commentGenerator>

		<jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
			connectionURL="jdbc:mysql://localhost:3306/WEBCHAT?useSSL=false" userId="root"
			password="Sal123456">
		</jdbcConnection>

		<javaTypeResolver>
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver>

		<javaModelGenerator
			targetPackage="com.luxiaobai.web_chat.pojo"
			targetProject="WebChat/src/main/java" />

		<!-- xxxMapper.xml -->
		<sqlMapGenerator targetPackage="com.luxiaobai.mapper"
			targetProject="WebChat/src/main/resources" />

		<!-- mapper接口位置 -->
		<javaClientGenerator type="XMLMAPPER"
			targetPackage="com.luxiaobai.web_chat.mapper" targetProject="WebChat/src/main/java" />

		<!-- 实体类 -->
		<table tableName="chat_msg" domainObjectName="ChatMsg"
			enableCountByExample="false" enableSelectByExample="false"
			enableDeleteByExample="false" enableUpdateByExample="false"></table>
		<table tableName="friends_request"
			domainObjectName="FriendsRequest" enableCountByExample="false"
			enableSelectByExample="false" enableDeleteByExample="false"
			enableUpdateByExample="false"></table>
		<table tableName="my_friends" domainObjectName="MyFriends"
			enableCountByExample="false" enableSelectByExample="false"
			enableDeleteByExample="false" enableUpdateByExample="false"></table>
		<table tableName="t_users" domainObjectName="User"
			enableCountByExample="false" enableSelectByExample="false"
			enableDeleteByExample="false" enableUpdateByExample="false"></table>

	</context>
</generatorConfiguration>
```

**4、在pom.xml文件中引入逆向工程文件的插件**

```xml
<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.2</version>
		</dependency>
  
  <plugin>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.3.2</version>
			</plugin>
```

**5、在generatorConfig.xml右键Run AS运行即可**