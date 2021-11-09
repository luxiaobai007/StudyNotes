

[toc]



# 批量导入txt文本（文本大小达到G级以上）

## 导入方法

1. bulk：ES本地支持的批量导入方式，如果是txt文本需要进行数据格式转换为json文件
2. logstash： ELK生态中的另一个产品，可以将数据文本 转换为ES的数据源
3. SpringData-ES的Java方式。



## ES Bulk Processtor

> BulkProcessor提供了一个简单的接口来实现批量提交请求（多种请求，如IndexRequest，DeleteRequest），且可根据请求数量、大小或固定频率进行flush提交。flush方式可选同步或异步

官方示例

```java
 public BulkProcessor bulkProcessor() throws UnknownHostException {
        Settings settings = Settings.builder().put("cluster.name", "elasticsearch").build();
        //客户端连接的采用端口为9300，否则获取不到对应的节点
        Client client = new PreBuiltTransportClient(settings)
                .addTransportAddress(new TransportAddress(InetAddress.getByName("localhost"), Integer.parseInt("9300")));

        BulkProcessor bulkProcessor = BulkProcessor.builder(
                        client,//添加搜索客户端
                        new BulkProcessor.Listener() {
                            @Override
                            public void beforeBulk(long executionId,
                                                   BulkRequest request) {
                                //批量执行之前被调用

                            }

                            @Override
                            public void afterBulk(long executionId,
                                                  BulkRequest request,
                                                  BulkResponse response) {
                                //批量执行之后被调用 检查是否有一些失败的请求

                            }

                            @Override
                            public void afterBulk(long executionId,
                                                  BulkRequest request,
                                                  Throwable failure) {//
                                System.out.println("error>>> " + failure);
                            }
                        })
                .setBulkActions(10000)//10000个请求执行一次批量请求
                .setBulkSize(new ByteSizeValue(1, ByteSizeUnit.GB))//每1G冲洗一次批量
                .setFlushInterval(TimeValue.timeValueSeconds(5))//无论请求数量多少，每5秒冲洗一次批量
                .setConcurrentRequests(1)//设置并发请求的数量,值 0 表示仅允许执行单个请求,值 1 表示允许在累积新的批量请求时执行 1 个并发请求。
                .setBackoffPolicy(
                        /**
                         *
                         * 设置一个自定义的退订策略，最初将等待 100m，成倍增加，重述多达三次。
                         * 每当一个或多个批量项目请求失败时，都会尝试重试，这表明可用于处理请求的计算资源太少。
                         * 要禁用后退，通过。EsRejectedExecutionExceptionBackoffPolicy.noBackoff()
                         */
                        BackoffPolicy.exponentialBackoff(TimeValue.timeValueMillis(100), 3))
                .build();

        return bulkProcessor;
    }

```





















#### 参考

[ElasticSearch导入txt文本或者json文本](https://www.cnblogs.com/ttzsqwq/p/11077574.html)

[将普通文本文件导入Elasticsearch](https://www.pianshen.com/article/5399591779/)

[ELK之logstash使用-----logstash将txt文本的接送数据导入到ES中](https://blog.csdn.net/weixin_44993313/article/details/106340355)

[elasticsearch使用BulkProcesssor导入txt大文件](https://qyi.io/archives/771.html)

[Elasticsearch Bulk Processor](https://www.elastic.co/guide/en/elasticsearch/client/java-api/2.3/java-docs-bulk-processor.html)

[Elasticsearch-BulkProcessor浅析](https://blog.csdn.net/baichoufei90/article/details/97117025)