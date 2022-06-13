---
JPA的findAll()方法查询优化问题
---



# 场景

> 在调用查询统计记录接口时，查询返回的结果响应时间缓慢



# 分析

- 通过debug调试发现在进行查询`findAll()`接口时，发现耗时在2-3s左右。

- 查看需要查询的实体类数据，可能是返回的`List<String> rids`长度过长导致的

- 不进行findAll()进行全表查询，重新定义一个新的实体，只返回需要的属性。

- 通过重新编写SQL语句可以实习查询效率的优化。但是对于条件查询将无法实现。

  ```sql
  @Query("SELECT new SearchRecordStaticsticBO(s.userKey, s.status, s.dataCenter, s.total, s.craetedAt, s.appId, s.appName FROM SearchRecordDO AS s)")
  ```

  

- 采用**视图映射实体**来作为临时表进行数据的查询和统计。



# 实现

1. 对于需要使用的数据属性创建一个视图

   ```sql
   CREATE VIEW sgksearch_record AS SELECT id, u_key, status, data_center, total, created_at, app_id, app_name FROM sgk_search_record;
   ```

   

2. 对应创建响应的表

   ```java
   @Setter
   @AllArgsConstructor
   @NoArgsConstructor
   @Entity
   @Immutable
   @Table(name = Constant.TABLE_PREFIX + "search_record")
   public class SearchRecordStatisticBO {
       @Id
       @GeneratedValue(strategy = IDENTITY)
       private Long id;
       @Column(name = "u_key")
       private String userKey;
       @Enumerated(EnumType.STRING)
       private SearchRecordDO.Status status;
       @Enumerated(EnumType.STRING)
       private DataCenter dataCenter;
       private Integer total;
       private LocalDateTime createdAt;
       private String appId;
       private String appName;
   
   }
   ```

   

3. 同时相应的创建对用的dao层 `SearchRecordStaticsticRepostiory`和`Specification`进行条件查询

==注意点==

在进行创建视图时，别忘了加个主键id，否则后面对数据进行分组统计时，将会你设置的@Id属性值进行去重在分组。

