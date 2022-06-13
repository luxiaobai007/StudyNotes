Shiro入门

[toc]

 

# 权限管理

> 基本上涉及到用户参与的系统都要进行权限管理,权限管理属于系统安全的范畴,权限管理实现`对用户访问系统的控制`,安装安全规则或者安全策略控制用户可以访问而且只能访问自己被授权的资源

权限管理包括用户**身份认证**和**授权**两部分,简称==认证授权==.对于需要访问控制的资源用户首先经过身份认证,认证通过后用户具有该资源的访问权限方可访问



## 身份认证

==身份认证==:判断一个用户是否为合法用户的处理过程.



## 授权

==授权==,即访问控制,控制谁能访问哪些资源.主体进行身份认证后需要分配权限方可访问系统的资源,对于某些资源没有权限是无法访问的.



# Shiro

官方网站:https://shiro.apache.org/

> **Apache Shiro™** is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management. With Shiro’s easy-to-understand API, you can quickly and easily secure any application – from the smallest mobile applications to the largest web and enterprise applications.







# Shiro核心架构

![image-20211129210958030](http://qiliu.luxiaobai.cn/img/image-20211129210958030.png)



##  Subject

==Subject即主体==,外部应用与subject进行交互,subject记录了当前操作用户,将用户的概念理解为当前操作的主体.Subject在shiro中是一个接口,接口中定义了很多认证授权相关的方法,外部程序通过subject进行认证授权,而subject是通过SecurityManager安全管理器进程认证授权



## SecurityManager

==SecurityManager即安全管理器==,对全部的Subject进行安全管理,是shiro的核心.通过Security Manager可以完成subject的认证、授权等. 实质上SecurityManager是通过Authenticator进行认证,通过Authorizer进行授权,通过SessionManager进行会话管理等.

==SecurManager是一个接口,继承了Authenticator, Authorizer, SessionManager这三个接口.==

## Authenticator

==Authenticator即认证器==,对用户身份进行认证,Authenticator是一个接口,shiro提供ModularRealmAuthenticator实现类,通过ModularRealmAuthenticator基本上可以满足大多数需求,也可以自定义认证器.

## Authorizer

==Authorizer即授权器==,用户通过认证器认证通过,在访问功能时需要通过授权器判断用户是否有此功能的操作权限.

## Realm

==Realm即领域==,相当于datasource数据源,securityManager进行安全认证需要通过Realm获取用户权限数据.,比如: 如果用户身份数据在数据库那么realm就需要从数据库获取用户身份信息

**不要把realm理解成只是从数据源取数据,在realm中还有认证授权校验的相关的代码**

## SessionManager

==sessionManager即会话管理==



## SessionDAO

==sessionDAO即会话dao==,是对session会话操作的一套接口,比如要将session存储到数据库,可以通过jdbc将会话存储到数据库



## CacheManager

==CacheManager即缓存管理==,将用户权限数据存储在缓存,这样可以提供性能



## Cryptography

==Cryptography即密码管理==, 提供的一套加密/解密的组件,比如提供常用的散列、加/解密等功能.



# Shiro的认证

## 认证

> 身份认证,判断一个用户是否为合法用户的处理过程.最常用的就是通过核对用户输入的用户名和口令,看其是否与系统中存储的该用户的用户名与口令一致,来判断用户身份是正确



## 认证的关键对象

- Subject: 主体

  访问系统的用户,主体可以是用户、程序等,进行认证的都被称为主体;

- Principal: 身份信息

  是主体进行身份认证的标识,必须具有**唯一性,**如用户名、手机号、Email等,一个主体可以有多个身份,但是必须有一个主身份(Primary Principal)

- credential: 凭证信息

  是只有主体自己知道的安全信息,如密码、证书等.



## 认证流程

![image-20211129214100250](http://qiliu.luxiaobai.cn/img/image-20211129214100250.png)





## 认证开发

####  依赖引入

```xml
<dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.5.3</version>
        </dependency>
```



#### Shirt.ini

```ini
[users]
luxiaobai=123456
admin=admin
```



```java

/**
 * @author ：luxiaobai
 * @date ：Created in 2021/12/4 11:20
 * @description：
 * @modified By：`
 * @version: 1.0
 */

public class TestAuthenticator {
    public static void main(String[] args) throws Exception {
        //1、创建安全管理器
        DefaultSecurityManager securityManager = new DefaultSecurityManager();

        //2、给安全管理器设置realm
        securityManager.setRealm(new IniRealm("classpath:shiro.ini"));
        //3、SecurityUtils给全局安全工具类设置安全管理器
        SecurityUtils.setSecurityManager(securityManager);

        //4、关键对象Subject
        Subject subject = SecurityUtils.getSubject();

        //5、创建令牌
        UsernamePasswordToken token = new UsernamePasswordToken("admin", "admin");

        try {
            System.out.println("认证之前：" + subject.isAuthenticated());
            subject.login(token);
            System.out.println("认证之后：" + subject.isAuthenticated());
        } catch (IncorrectCredentialsException e) {
            e.printStackTrace();
            throw new IncorrectCredentialsException("密码错误");
        } catch (UnknownAccountException e){
            e.printStackTrace();
            throw new UnknownAccountException("用户名错误");
        }
    }
}

```





## 源码分析

### 用户名验证

由`subject.login(token)`debug模式进入源码分析在`simpleAccountRealm`类中的`doGetAuthenticationInfo`方法下完成了用户名的认证

![image-20211204120619651](http://qiliu.luxiaobai.cn/img/image-20211204120619651.png)





### 密码验证

密码的最终验证在`AuthenticatingRealm`中的`assertCredentialsMatch`方法完成的密码验证

![image-20211204121037144](http://qiliu.luxiaobai.cn/img/image-20211204121037144.png) 



### 总结

>需要在数据库中完成用户名的认证,则只需要取实现doGetAuthenticationInfo的方法进行用户名的验证即可,密码校验由shiro中的Realm(域)自动完成



## Realm的完整实现

![image-20211204121922178](http://qiliu.luxiaobai.cn/img/image-20211204121922178.png)



### 真正实现的类SimpleAccountRealm

![image-20211204122139448](http://qiliu.luxiaobai.cn/img/image-20211204122139448.png)



- **AuthenticatingRealm** 认证realm doGetAuthenticationInfo
- **AuthorizingRealm** 授权realm  doGetAuthorizationInfo



## 自定义Realm

```java

public class CustomerRealm extends AuthorizingRealm {

    /**
     * 授权
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    /**
     * 认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //在token中获取用户名
        String principal = (String) authenticationToken.getPrincipal();
        //根据用户名在数据库中查询
        if ("luxiaobai".equals(principal)){
            //参数1： 返回数据库中正确的用户名 //参数2： 返回数据库中正确的密码 //参数3： 提供档期Realm的名字 this.getName
            return new SimpleAuthenticationInfo(principal,"123456", this.getName());
        }
        return null;
    }
}
```



### 测试自定义Realm的实现

```java

    @Test
    void TestCustomRealmAuthenticator(){
        //设置安全管理器
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        //设置自定义Realm
        defaultSecurityManager.setRealm(new CustomerRealm());
        //将安全工具类设置安全管理器
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        //获取token
        UsernamePasswordToken token = new UsernamePasswordToken("luxiaobai", "123456");

        //获取subject
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(token);
            System.out.println(subject.isAuthenticated());
        } catch (IncorrectCredentialsException e) {
            e.printStackTrace();
            System.out.println("密码错误");
        } catch (UnknownAccountException e){
            System.out.println("用户名错误");
        }
    }

}
```



## 使用MD5和Salt

> **MD5算法**: 
>
> ​	作用: 一般用来加密 或者 签名(校验和)
>
> ​	特点: MD5算法不可逆 内容相同无论执行多少次md5生成结果始终是一致
>
> ​	生成结果: 始终是一个16进制32位长度字符串 



### MD5使用

```java
 @Test
    void testMd5Sal(){
        //md5
        Md5Hash md5Hash = new Md5Hash();
        md5Hash.setBytes("123".getBytes());
        String s = md5Hash.toHex();
        System.out.println(s);

        //md5算法正确使用
        Md5Hash md5Hash1 = new Md5Hash("123");
        System.out.println(md5Hash1.toHex());

        //md5+salt
        Md5Hash md5Hash2 = new Md5Hash("123", "fsdfkasdj*&fsd0");
        System.out.println(md5Hash2.toHex());

        //md5+salt+hash散列
        Md5Hash md5Hash3 = new Md5Hash("123", "fsdfjkj5$%*~", 1024);
        System.out.println(md5Hash3.toHex());

    }
```





![image-20211204130629315](http://qiliu.luxiaobai.cn/img/image-20211204130629315.png)





### 实现自定义RealmMD5

```java

public class CustomerMd5Realm extends AuthorizingRealm {
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String principal = (String) authenticationToken.getPrincipal();
        if ("luxiaobai".equals(principal)) {
            //参数1： 返回数据库中正确的用户名 //参数2： 返回数据库中md+salt+hash //参数3： salt //参数4 当前realm的名字
            return new SimpleAuthenticationInfo(principal,
                    "512c2ee9b9063c198885026b9a0888d4",
                    ByteSource.Util.bytes("fsdfkasdj*&fsd0"),
                    this.getName());
        }
        return null;
    }
}
```



### 测试方法

```java
 @Test
    void testRealmMd5(){
        //设置管理器
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        //注入自定义realm
        CustomerMd5Realm customerMd5Realm = new CustomerMd5Realm();
        //设置Realm使用hash凭证匹配器
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        //使用的算法
        hashedCredentialsMatcher.setHashAlgorithmName("md5");
        //使用散列
        hashedCredentialsMatcher.setHashIterations(1024);

        customerMd5Realm.setCredentialsMatcher(hashedCredentialsMatcher);
        //注入realm
        defaultSecurityManager.setRealm(customerMd5Realm);

        //注入安全管理器
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        //获取subject
        Subject subject = SecurityUtils.getSubject();
        //token
        UsernamePasswordToken token = new UsernamePasswordToken("luxiaobai", "123456");

        try {
            subject.login(token);
        } catch (UnknownAccountException e) {
            e.printStackTrace();
            System.out.println("用户名错误！");
        }catch (IncorrectCredentialsException e){
            e.printStackTrace();
            System.out.println("密码错误！");
        }
    }
```



# Shiro的授权

> 授权,即访问控制,控制谁能访问哪些资源.主体进行身份认证后需要分配权限方可访问系统的资源,对于某些资源没有权限是无法访问的



## 关键对象

==授权可简单理解为Who对What(which)进行How操作==

**Who,即主体(Subject)**,主体需要访问系统中的资源

**What,即资源(Resource)**,如系统菜单、页面、按钮、类方法、系统商品信息等.资源包括**资源类型和资源实例**,比如**商品信息为资源类型**,类型为t01的商品为**资源实例**

**How, 权限/许可(Permission)**,规定了主体对资源的操作许可,权限离开资源没有意义,如用户查询权限、用户添加权限、某个类方法的调用权限、编号为001用户的修改权限等,通过权限可知主体对哪些资源都有哪些操作许可



## 授权流程

![image-20211205102356202](http://qiliu.luxiaobai.cn/img/image-20211205102356202.png)



## 授权方式

- 基于角色的访问控制

  - RBAC基于角色的访问控制(Role-Based Access Control)是以角色为中心进行访问控制

    ```java
    if(subject.hasRole('admin')){
      //操作说明资源
    }
    ```

    

- 基于资源的访问控制

  - RBAC基于资源的访问控制(Resource-Based Access Control)是以资源为中心进行访问控制

    ```java
    if(subject.isPermission('user:update:01')){//资源实例
      //对01用户进行修改
    }
    if(subject.isPermission('user:update:*')){//资源类型
      //所有进行修改 
    }
    ```

  ### 权限字符串

  权限字符串的规则: ==资源标识符: 操作: 资源实例标识符==,即对那个资源的那个实例具有什么操作,`:`是资源/操作/实例的分隔符,权限字符串也可以使用`*`通配符.

  例子:

  - 用户创建权限: user:create,或user:create:*
  - 用户修改实例001的权限:user:update:001
  - 用户实例001的所有权限: user: *:001



shiro中授权编程实现方式

- 编程式

  ```java
  Subject subject = SecurityUtils.getSubject();
  if(subject.hasRole('admin')){
    //有权限
  }else{
    //无权限
  }
  ```

  

- 注解式

  ```java
  @RequiresRoles('admin')
  public void hello(){
    
  }
  ```

  

- 标签式

  ```java
  JSP/GSP 标签:在JSP/GSP页面通过相应的标签完成:
  <shiro:hasRole name="admin">
    <!-有权限->
  </shiro:hasRole>
  注意:Tymeleaf中使用shiro需要额外集成
  ```

  

  # 开发授权

  ```java
    @Override
      protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
          String primaryPrincipal = (String) principalCollection.getPrimaryPrincipal();
          System.out.println("用户信息：" + primaryPrincipal);
          //根据当前的用户信息 用户名 在看数据库中获取当前用户名的角色信息，以及权限信息
          SimpleAuthorizationInfo simpleAuthenticationInfo = new SimpleAuthorizationInfo();
          //将数据库中查询的角色信息赋值给权限对象
          simpleAuthenticationInfo.addRole("admin");
          simpleAuthenticationInfo.addRole("user");
  
          //将数据库中查询的权限信息赋值给权限对象
          simpleAuthenticationInfo.addStringPermission("user:*:01");
          simpleAuthenticationInfo.addStringPermission("product:create");
          return simpleAuthenticationInfo;
      }
  
  
   //授权操作
          if (subject.isAuthenticated()){
              //基于角色的权限控制
              System.out.println(subject.hasRole("admin"));
  
              //基于多角色的权限控制
              System.out.println(subject.hasAllRoles(Arrays.asList("admin","super")));
  
              //是否具有一个角色
              boolean[] booleans = subject.hasRoles(Arrays.asList("admin", "super", "user"));
              for (boolean aBoolean : booleans) {
                  System.out.println(aBoolean);
              }
  
              System.out.println("=============================================");
              //基于权限字符串的访问控制，资源标识符：操作：资源类型
              System.out.println("权限：" + subject.isPermitted("user:*:*"));
              System.out.println("权限：" + subject.isPermitted("product:create:02"));
  
              //分别具有哪些权限
              boolean[] permitted = subject.isPermitted("user:*:01", "order:*:02");
              for (boolean b : permitted) {
                  System.out.println(b);
              }
  
              //通知具有哪些权限
              boolean permittedAll = subject.isPermittedAll("user:*:01", "order:create");
              System.out.println("同时具有权限：" + permittedAll);
          }
  ```

  

  



# 整合Spring boot实战

## 思路 

![image-20211205113259803](http://qiliu.luxiaobai.cn/img/image-20211205113259803.png)





## 实战





















































































































































































































































