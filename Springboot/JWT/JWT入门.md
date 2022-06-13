---
 JWT入门学习
---

[toc]

# JWT

## 简介

**JSON Web 令牌**

官网网址:https://jwt.io

> JSON Web Token (JWT) is an open standard ([RFC 7519](https://tools.ietf.org/html/rfc7519)) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the **HMAC** algorithm) or a public/private key pair using **RSA** or **ECDSA**.

通过JSON形式作为Web应用中的令牌,用于在各方之间安全地将信息作为JSON对象传输.在数据传输过程中还可以完成数据加密、签名等相关处理



## 基于传统的Session认证

HTTP协议是一种无状态的协议,因此每次请求时,要再一次的进行用户认证.为了识别是那个用户发出的请求,采用的是Session-cookie的机制.在服务器中存储用户登录信息,保存在session中,同时将登陆信息响应给浏览器,保存为cookie,以便下次请求再次发送,用于识别身份信息.



### 暴露的问题

1. 由于session都是保存在内存中,随着认证用户的增多,服务端的开销会明显增大
2. 用户认证之后,服务端做认证记录,意味着用户下次请求必须在这台服务器上,才可以拿到授权资源,这样在分布式的应用上,相应的限制了负载均衡的能力,限制了应用的扩展能力.需要实现session共享机制
3. 基于cookie来进行用户识别,容易被截获,容易受到跨站请求伪造的攻击(CSRF)



## 基于JWT认证

![image-20211125212133101](http://qiliu.luxiaobai.cn/img/image-20211125212133101.png)

### 认证流程

1. 前端将用户名和密码发送给后端
2. 后端核对用户名和密码成功后,将用户ID等其他信息作为JWT Payload(负载),将其与头部分别进行Base64编码拼接后签名,形成一个JWT(Token),形成的JWT就是一个形同lll.zzz.xxx的字符串
3. 后端将JTW字符串作为登陆成功的返回结果返回给前端.前端可以将返回的结果保存在`localStorage`或`sessionStorage`上,退出登陆时前端删除保存的JWT即可
4. 前端在每次请求时将JWT放入HTTP Header中的Authorization位.(解决XSS和XSRF问题)
5. 后端检查是否存在,如存在验证JWT的有效性.例如,检查签名是否正确;检查Token是否过期;检查Token的接收方是否是自己(可选)
6. 验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作,返回相应结果.



## 优势

- 简洁(Compact):可以通过URL,POST参数或者在HTTP header发送,因为数据量小,传输速度也很快
- 自包含(Self-contained): 负载中包含了所有用户所需要的信息,避免了多次查询数据库
- 因为Token是以JSON加密的形式保存在客户端的,所以JWT是跨语言的,原则上任何web形式都支持
- 不需要在服务端保存会话信息,特别适用于分布式微服务





## JWT结构

```json
token string =====> header.payload.signature		 token

##令牌组成
1、标头(Header)
2、有效载荷(Payload)
3、签名(Signature)
JWT通常如下所示: xxxx.yyyyy.zzzzz 即Header.Payload.Signature




#Header
-标头通常由两部分组成: 令牌的类型(即JWT)和所使用的签名算法,例如HMAC SHA256或RSA. 它会使用Base64编码组成JWT结构的第一部分
-注意: Base64是一种编码,也就是说,它是可以被翻译回原来的样子,并不是一种加密过程.
{
  "alg": "SHA256",
  "typ": "JWT"
}


###Payload 不要放敏感信息
-令牌的第二部分是有效负载,其中包含声明. 声明是有关实体(通常是用户)和其他数据的声明,同样的,它会使用Base64编码组成JWT结构的第二部分

{
  "sub": "123456789",
  "name": "John Doe",
  "admin": true
}

###Signature
-前面两部分都是使用Base64进行编码的,即前端可以解开里面的信息. Signature需要使用编码后的header和payload以及我们提提供的一个密钥,然后使用header中指定的签名算法(HS256)进行签名. 签名的作用是保证JWT没有被篡改
如:
HMACSHA256(base64UrlEncoder(header) + "." + base64UrlEncode(payload),secret);

##签名目的
-最后一步签名的过程,实际上是对头部以及负载内容进行签名,防止内容被篡改.如果有人对头部以及负载的内容解码之后进行修改,在进行编码,最后加上之前的签名组合形成新的JTW的话,那么服务器会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的.涂哥要对新的头部和负载进行签名,在不知道服务器加密时用的密钥的话,得出来的签名也是不一样的.


##信息安全问题
Base64是一种编码,是可逆的.
--所以,在JWT中,不应该在负载里面加入任何敏感的数据.一般的,在JWT中,传输的是用户的User ID,这个值实际上不是什么敏感内容,一般情况下被知道也是安全,但是像密码这样的内容就不能放在JWT中了.如果将用户的密码放在了JWT中,那么怀有恶意的第三方通过Base64解码就能知道用户密码.因此JWT适用于向Web应用传递一些非敏感信息.JWT还经常用于设计用户认证和授权系统,甚至实现Web应用的单点登录.
```



## 使用JWT

```xml
<dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.18.2</version>
        </dependency>
```

> 生成token

```java
@Test
    void contextLoads() {
        Calendar instance = Calendar.getInstance();
        instance.add(Calendar.SECOND, 90);
        //生成令牌 链式调用
        String token = JWT.create()
                //payload
                .withClaim("username", "张三")
                .withClaim("username", 1)
                //设置过期时间
                .withExpiresAt(instance.getTime())
                //设置签名 保密 复杂
                .sign(Algorithm.HMAC256("token@HDLSDJDKF"));
        System.out.println("输出令牌：" + token);
    }

输出令牌：eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MzgwNjQ2MTAsInVzZXJuYW1lIjoxfQ.-nUfEvCRcc_03dBjJp2EvuP93IL2Ns-ReParKO1u3G8
```

> 验签对象

```java
 @Test
    void checkJWT(){
        //构建验签对象
        JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("token@HDLSDJDKF")).build();
        DecodedJWT verify = jwtVerifier.verify("eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MzgwNjUxOTYsInVzZXJJRCI6MSwidXNlcm5hbWUiOiLlvKDkuIkifQ.gNbzYHZsOMyd8rEGyPtTImRFnTQb6dhOa5jMzZq_lL0");
        Claim username = verify.getClaim("username");
        Claim userID = verify.getClaim("userID");
        //注意类型问题，否则拿不到对应的数据为null
        System.out.println(userID.asInt());
        System.out.println(username.asString());
    }
```



> **常见异常信息**
>
> -SignatureVerificationException:			签名不一致异常
>
> -TokenExpireException:							令牌过期异常
>
> -AlgorithmMismatchException:				算法不匹配异常
>
> -InvalidClaimException:								失效的payload异常





## 封装工具类

```java
public class JwtUtils {
    private static final String SECRECY = "luxiaobai@lsy";

    /**
     * 生成token
     * @param map 传入payload
     */
    public static String getToken(Map<String, String> map){
        JWTCreator.Builder builder = JWT.create();
        map.forEach(builder::withClaim);
        Calendar instance = Calendar.getInstance();
        instance.add(Calendar.SECOND, 90);
        builder.withExpiresAt(instance.getTime());
        return builder.sign(Algorithm.HMAC256(SECRECY));
    }

    /**
     * 验证token
     */
    public static void checkToken(String token){
       JWT.require(Algorithm.HMAC256(SECRECY)).build().verify(token);
    }

    /**
     * 获取token中的payload
     */
    public static DecodedJWT getPayload(String token){
        return JWT.require(Algorithm.HMAC256(SECRECY)).build().verify(token);
    }
}

```





#  Springboot整合JWT

```xml
<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.18.2</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.23</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.26</version>
        </dependency>
```

```properties
server.port=8999

spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/jwt?characterEncoding=utf-8
spring.datasource.username=root
spring.datasource.password=Sal12345

mybatis.type-aliases-package=com.luxiaobai.entity
mybatis.mapper-locations=classpath:com/luxiaobai/mapper/*.xml

logging.level.com.luxiaobai.dao=debug
```

### 主要代码

#### JwtInterceptor

```java
public class JWTInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //获取请求头中的令牌
        String token = request.getHeader("token");
        Map<String, Object> map = new HashMap<>();
        try {
            //验证令牌
            JwtUtils.getPayload(token);
            return true;
        } catch (TokenExpiredException e) {
            e.printStackTrace();
            map.put("msg", "token过期");
        } catch (AlgorithmMismatchException e) {
            e.printStackTrace();
            map.put("msg", "算法不一致！");
        } catch (Exception e) {
            e.printStackTrace();
            map.put("msg", "token无效");
        }
        map.put("state", false);
        //将map转换为json jackson
        String json = new ObjectMapper().writeValueAsString(map);
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().println(json);
        return false;

    }
}

```



#### Config

```java
@Component
public class InterceptorConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new JWTInterceptor())
                .addPathPatterns("/user/test")
                .excludePathPatterns("/user/login");
    }
}

```



#### Controller

```java
@RestController
@Slf4j
public class TestController {

    @Autowired
    private UserServiceImpl userService;

    @GetMapping("/user/login")
    public Map<String, Object> login(User user) {
        log.info(("用户名[{}]" + user.getName()));
        log.info("密码[{}]" + user.getPassword());
        Map<String, Object> map = new HashMap<>();
        try {
            User userDb = userService.login(user);
            //生成令牌
            Map<String, String> payload = new HashMap<>();
            payload.put("userId", userDb.getId());
            payload.put("userName", userDb.getName());
            String token = JwtUtils.getToken(payload);
            map.put("token", token);
            map.put("state", true);
            map.put("msg", "认证成功");
        } catch (Exception e) {
            map.put("state", false);
            map.put("msg", e.getMessage());
        }
        return map;
    }

    @PostMapping("/user/test")
    public Map<String, Object> test(HttpServletRequest request) {
        String token = request.getHeader("token");
        log.info("token:[{}]", token);
        DecodedJWT payload = JwtUtils.getPayload(token);
        log.info("用户ID:[{}]", payload.getClaim("userId"));
        log.info("用户名:[{}]", payload.getClaim("userName"));
        Map<String, Object> map = new HashMap<>();
        //转换为拦截器  实际业务逻辑
//        try {
//            //验证令牌
//            DecodedJWT payload = JwtUtils.getPayload(token);
//            map.put("state", true);
//            map.put("msg", "认证成功");
//            return map;
//        } catch (TokenExpiredException e) {
//            e.printStackTrace();
//            map.put("msg", "token过期");
//        } catch (AlgorithmMismatchException e) {
//            e.printStackTrace();
//            map.put("msg", "算法不一致！");
//        } catch (Exception e) {
//            e.printStackTrace();
//            map.put("msg", "token无效");
//        }
        map.put("state", true);
        return map;
    }
}

```





#### 源代码

[1]:https://github.com/luxiaobai007/JWT.git  JWT源代码及学习笔记

