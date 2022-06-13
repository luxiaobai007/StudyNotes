# ES 相关命令

```shell
PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}

GET /megacorp/employee/2
GET /megacorp/employee/_search

GET /megacorp/employee/_search?q=last_name:Smith

GET /megacorp/employee/_search
{
  "query" : {
    "match": {
      "last_name": "Smith"
    }
  }
}

#过滤器 filter
GET /megacorp/employee/_search
{
  "query" : {
    "bool" : {
      "must" : {
        "match" : {
          "last_name" : "smith"
        }
      },
      "filter" : {
        "range" : {
          "age" : {
            "gt" : 30
          }
        }
    }
    }
  }
}

GET /megacorp/employee/_search
{
  "query" : {
    "match": {
      "about": "rock climbing"
    }
  }
}

##短语搜索
GET /megacorp/employee/_search
{
  "query" : {
    "match_phrase": {
      "about": "rock climbing"
    }
  }
}

#高亮搜索 highlight
GET /megacorp/employee/_search
{
  "query" : {
    "match": {
      "about": "rock climbing"
    }
  },
  "highlight": {
    "fields": {
      "about": {}
    }
  }
}


#聚合
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}


#对聚会字段添加优化
PUT /megacorp/employee/_mapping?include_type_name=true
{
  "properties": {
    "interests": {
      "type" : "text",
      "fielddata" : true
    }
  }
}

GET /megacorp/employee/_search
{
  "query" : {
    "match" : {
      "last_name" : "smith"
    }
  },
  "aggs" : {
    "all_interests" : {
      "terms": {
        "field": "interests",
        "size": 10
      }
    }
  }
}

#聚合分级汇总
GET /megacorp/employee/_search
{
  "aggs":{
    "all_interestes" : {
      "terms": {
        "field": "interests",
        "size": 10
      },
      "aggs":{
      "avg_age":{
        "avg": {
          "field": "age"
        }
      }
    }
    }
    
  }
}

#集群健康
GET /_cluster/health


PUT /blogs
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}


##显示索引简化信息
GET _cat/indices

##显示索引详细信息
GET _cat/indices?v

##查看s开头的索引
GET _cat/indices/s*?v

#创建有10个分片的index
PUT student
{
  "mappings" : {
    "properties" : {
      "uid": {
        "type" : "integer"
      },
      "name" : {
        "type" : "keyword"
      },
      "age" : {
        "type" : "integer"
      }
    }
  },
  "settings" : {
    "index" : {
      "number_of_shards" : 10,
      "number_of_replicas" : 1
    }
  }
}

#添加记录
POST student/_doc/1?routing=1
{
  "uid": 1,
  "name": "张三",
  "age": 10
}

#查询中带上explain为true，可查看文档属于那个shard
GET student/_search?explain=true

GET student/_search
{
  "query": {
    "match": {
      "uid": 1
    }
  },
  "explain": true
}


GET student/_doc/1?_source=name,age

GET .tasks/_search

##分词
GET student/_analyze?analyzer=ik_max_word&pretty=true
{
  "text": "FaceBook重新更名为Meta，开创元宇宙"
}


POST /my_store/products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20, "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30, "productID" : "QQPX-R-3956-#aD8" }


GET my_store/_search
{
  "query" : {
    "constant_score": {
      "filter": {
        "term": {
          "price": "20"
        }
      },
      "boost": 1.2
    }
  }
}

PUT poems

DELETE poems

PUT poems
{
  "mappings": {
      "properties":{
        "title":{
          "type": "keyword"
        },
        "author":{
          "type":"keyword"
        },
        "dynasty":{
          "type": "keyword"
        },
        "words":{
          "type":"integer"
        },
        "content":{
          "type": "text"
        }
      }
  }
}


PUT poems/_doc/1
{
  "title":"静夜思",
  "author":"李白",
  "dynasty":"唐",
  "words": 20,
  "content":"床前明月光，疑是地上霜。举头望明月，低头思故乡"
}


##创建三组测试数据的mapping
DELETE shakespeare
PUT /shakespeare?include_type_name=true
{
  "mappings": {
    "doc" : {
      "properties":{
        "speaker":{
          "type": "keyword"
        },
        "play_name":{
          "type": "keyword"
        },
        "line_id":{
          "type": "integer"
        },
        "speech_number":{
          "type": "integer"
        }
      }
    }
  }
}

PUT /logstash-2021?include_type_name=true
{
  "mappings": {
    "logs": {
      "properties":{
        "geo": {
          "properties":{
            "corrdinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  }
}

PUT /logstash2-2021?include_type_name=true
{
  "mappings": {
    "logs":{
      "properties":{
        "geo": {
          "properties":{
            "corrdinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  }
}


GET /_cat/indices
GET /bank/account/_search

#查询所有并account_number字段升序排序
GET /bank/account/_search
{
  "query":{
    "match_all": {}
  },
  "sort":[
      {
        "account_number": "asc"
      }
    ]
}

#范围查询 from和size
#从from开始，之后的size个数据
GET  /bank/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "account_number": {
        "order": "asc"
      }
    }
  ],
  "from": 10,
  "size": 10
}

#query match
GET /bank/_search
{
  "query": {
    "match": {
      "address": "mill lane"
    }
  }
}


#bool条件查询
###age必须40，且state不是ID的
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "age": 10
        }}
      ],
      "must_not": [
        {
          "match": {
            "state": "ID"
          }
        }
      ]
    }
  }
}


GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        {"match_all": {}}
      ],
      "filter": [
        {
          "range": {
            "balance": {
              "gte": 20000,
              "lte": 30000
            }
          }
        }
      ]
    }
  }
}



PUT /phones/
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "images": {
        "type": "keyword",
        "index": false
      },
      "price":{
        "type": "float"
      }
    }
  }
}

#查看索引关系
GET /phones/_mapping

#随机生成id
POST /phones/_doc
{
  "title": "魅族手环",
  "images": "http://qiliu.luxiaobai.cn/meizu.jpg",
  "price": 4699.00
}

##自定义ID  自定义ID值不能重复，否则数据将会被覆盖
POST /phones/_doc/phone_2
{
  "title": "小米手机",
  "images": "http://qiliu.luxiaobai.cn/xioami.jpg",
  "Saleable": true
}

GET /phones/_search
#根据id查找
GET /phones/_doc/phone_2

#匹配查询
GET /phones/_search
{
  "query": {
    "match": {
      "title": {
        "query": "小米手机电视",
        "minimum_should_match": "60%"
      }
    }
  }
}


##多字段查询
GET phones/_search
{
  "query": {
    "multi_match": {
      "query": "小米",
      "fields": ["title", "subTitle"]
    }
  }
}

##词条查询
#可分割的最小词条 title为字段名 ["字段值"]
GET /phones/_search
{
  "query": {
    "terms": {
      "title": [
        "小米",
        "手机"
      ]
    }
  }
}

#结果过滤
#excludes:不显示的字段 includes:显示的字段
GET /phones/_search
{
  "_source": {
    "excludes": "images"
  },
  "query": {
    "terms": {
      "title": [
        "小米",
        "电视"
      ]
    }
  }
}

#布尔查询
#bool 把各种其他查询通过must(与)、must_not(非)、should(或)的方式进行组合
#标题一定有小米，或者价格有2699，4699
GET /phones/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "小米"
          }
        }
      ],
      "should": [
        {
          "terms": {
            "price": [
              "2699",
              "4699"
            ]
          }
        }
      ]
    }
  }
}

#范围查询
#2799<=price<=3899
GET /phones/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 2799,
        "lte": 3899
      }
    }
  }
}

POST _analyze
{
  "analyzer": "ik_max_word",
  "text": "我是路小白"
}

##match进行分词器分析所谓的分词，就是把当前的value进行分词 模糊查询
##term是代表完全匹配，即不进行分词器分析 精确查询
GET /phones/_search
{
  "query": {
    "match": {
      "title": "手环"
    }
  }
}

GET /phones/_search
{
  "query": {
    "term": {
      "title": "小米手环"
    }
  }
}

POST /phones/_doc
{
  "title":"OPPO手机",
  "images": "1234232321.jpg"
}
#模糊查询 fuzziness
#标题为oppo默认允许错误一个字母， 最大为两个字母正确标题OPPO
GET /phones/_search
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "oppe",
        "fuzziness": "auto"
      }
    }
  }
}


##过滤filter
GET /phones/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "小米"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "price": {
              "gte": 2699,
              "lte": 4699
            }
          }
        }
      ]
    }
  }
}


##排序  默认：desc倒序，   升序：asc
GET /phones/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "price": {
              "gte": 1000,
              "lte": 5000
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    },
    {
      "_id":{
        "order": "asc"
      }
    }
  ]
}


###聚合
PUT /cars 
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "color":{
        "type": "keyword"
      },
      "make":{
        "type": "keyword"
      }
    }
  }
}

#批量添加数据
POST /cars/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }

GET /cars/_search

#聚合为桶
GET /cars/_search
{
  "aggs": {
    "color": {
      "terms": {
        "field": "color"
      }
    }
  }
}

#桶内度量
GET /cars/_search
{
  "size": 0, 
  "aggs": {
    "color_bucket": {
      "terms": {
        "field": "color",
        "size": 10
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}

#桶内嵌套桶
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "color": {
      "terms": {
        "field": "color",
        "size": 10
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        },
        "mark":{
          "terms": {
            "field": "make",
            "size": 10
          }
        }
      }
    }
  }
}


#阶梯分组
GET /cars/_search
{
  "aggs": {
    "price_histogram": {
      "histogram": {
        "field": "price",
        "interval": 5000,
        "min_doc_count": 1
      }
    }
  }
}


#范围分组
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "price_range": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 5000,
            "to": 15000
          },
          {
            "from": 15000,
            "to": 20000
          },
          {
            "from": 20000,
            "to": 25000
          },
          {
            "from": 25000,
            "to":30000
          },
          {
            "from": 30000,
            "to":35000
          },
          {
            "from": 35000,
            "to": 40000
          }
        ]
      }
    }
  }
}

```

>1. match进行分词器分析, 所谓的分词，就是把当前的value进行分词
>2. term是代表完全匹配，即不进行分词器分析









## 参考资源

[使用kibana操作elasticsearch7.X教程](https://www.cnblogs.com/strict/p/12642146.html)