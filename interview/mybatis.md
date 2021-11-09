[toc]



# 为什么能直接调用userMapper接口的方法

1. 注册Mapper
2. 通过`new SqlSessionFactoryBuilder().build(inputStream);`构建SqlSessionFactory.在builder方法中讲配置文件中的内容转换为Configuration中的对象属性.
3. 获取Mapper,调用接口方法.
   - 通过`sqlSession = factory.openSession();`创建`SqlSession`对象
   - 通过`UserMapper userMapper = sqlSession.getMapper(UserMapper.class);`获取mapper代理对象
   - 通过`userMapper.selectById(1)`获取数据库结果集
   - 在获取Mapper时,主要通过反射的(动态代理),来获取Mapper接口的对象, 最终调用的是MapperProxy的invoke方法.

```java
 public class MybatisApplication {
        public static final String URL = "jdbc:mysql://localhost:3306/mblog";
        public static final String USER = "root";
        public static final String PASSWORD = "123456";
    
        public static void main(String[] args) {
            String resource = "mybatis-config.xml";
            InputStream inputStream = null;
            SqlSession sqlSession = null;
            try {
                inputStream = Resources.getResourceAsStream(resource);
                SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
                sqlSession = sqlSessionFactory.openSession();
                
                UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
                System.out.println(userMapper.selectById(1));
    
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                sqlSession.close();
            }
        }
```









# MyBatis和Hibernate有什么区别

1. **sql优化方面**

   - Hibernate 不需要编写大量的 SQL，就可以完全映射，提供了日志、缓存、级联（级联比 MyBatis 强大）等特性，此外还提供 HQL（Hibernate Query Language）对 POJO 进行操作。但会多消耗性能。
   - MyBatis 手动编写 SQL，支持动态 SQL、处理列表、动态生成表名、支持存储过程。工作量相对大些。

2. **开发方面**

   - MyBatis 是一个半自动映射的框架, 需要手动匹配 POJO、SQL 和映射关系。

   - Hibernate是一个全表映射的框架,只需提高POJO和映射关系即可.

3. **Hibernate优势**

   - Hibernate 的 DAO 层开发比 MyBatis 简单，Mybatis 需要维护 SQL 和结果映射。
   - Hibernate 对对象的维护和缓存要比 MyBatis 好，对增删改查的对象的维护要方便。
   - Hibernate 数据库移植性很好，MyBatis 的数据库移植性不好，不同的数据库需要写不同 SQL
   - Hibernate 有更好的二级缓存机制，可以使用第三方缓存。MyBatis 本身提供的缓存机制不佳。

4. **Mybatis优势**

   - MyBatis 可以进行更为细致的 SQL 优化，可以减少查询字段。
   - MyBatis 容易掌握，而 Hibernate 门槛较高。



# MyBatis- Plus有了解吗



























#### 相关资料来源

[1]: https://www.jianshu.com/p/f54b147b6041
[2]: https://segmentfault.com/a/1190000038748480

