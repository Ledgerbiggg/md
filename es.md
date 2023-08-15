# elasticsearch
## 简介
* E：EalsticSearch 搜索和分析的功能
* L：Logstach 搜集数据的功能，类似于flume（使用方法几乎跟flume一模一样），是日志收集系统
* K：Kibana 数据可视化（分析），可以用图表的方式来去展示，文不如表，表不如图，是数据可视化平

* 日志统计使用ELK来方便管理,存入大量的日志记录,使用Kibana 就可以直接看到日志情况

## 全文检索
* 简单说就是,扫描文章的每一个词语,对词语加索引,索引指向文章,反映出这个文章连词条出现的位置和次数

## 倒排索引
* 以前是根据ID查内容，倒排索引之后是根据内容查ID，然后再拿着ID去查询出来真正需要的东西。

## Lucene
* 是一个各种建立倒排索引的方法合集,但是不是分布式的,es底层是Lucene,而且是分布式的

## es由来
* Lucene不是分布式的,如果数据太多就要增加机器
* 有机器挂了数据就丢失了

### 优点

1. 分布式的功能
2. 数据高可用，集群高可用
3. API更简单
4. API更高级。
5. 支持的语言很多
6. 支持PB级别的数据
7. 完成搜索的功能和分析功能

## 核心概念

* NRT(Near Realtime)近实时
    * 快
* cluster: 集群
    * clustername 默认值就是ElasticSearch,如果这个值是一样的就属于同一个集群，不一样的值就是不一样的集群
* Node: 节点
    * 集群中的一台机器
* index: 索引
    * index类似于我们Mysql里面的一个数据库
* type: 类型
    * 好比数据库里面的一张表
* document: 文档
    * 一个文档就是一条记录
    * 好比表里面的一条数据
* Field: 字段
    * 一个document有一个或者多个field组成
    * 好比关系型数据库中列的概念
* shard: 分片 
    * 一台服务器，无法存储大量的数据，ES把一个index里面的数据，分为多个shard，分布式的存储在各个服务器上面。
* replica: 副本
![](https://img-blog.csdnimg.cn/20190419153200263.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9qZW5yZXkuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

* tip: 
    * Keyword 类型不会分词直接根据字符串内容建立反向索引，
    * Text 类型在存入 Elasticsearch 的时候，会先分词，然后根据分词后的内容建立反向索引。

## Kibana操作
### !!!!7.0版本开始，Elasticsearch移除了多个文档类型的概念，推荐使用单一类型_doc来表示所有文档。

## Kibana操作 
```sql
# 检查健康状态 green为健康
GET _cat/health 
# 查看索引
GET _cat/indices
# 获取所有的索引的详细
GET _all
# 增加一个索引
PUT ledger
# 查看索引
GET _cat/indices
# 删除索引
DELETE ledger
# 查询
GET /ledger/_doc/1
# 修改
PUT /ledger/_doc/1
{
  "name":"ledger1111",
  "price":444444441,
  "tags":[154,1541]
}
# put全局修改,数据会丢失
PUT /ledger/_doc/1
{
  "name":"ledger1111"
}
# post局部更新
POST /ledger/_doc/_update
{
  "doc": {
    "name": "ledger191"
  }
}
# 删除
DELETE /ledger/_doc/1
# 全索引扫秒
GET /ledger/_search
{
  "query": {
    "match_all": {}
  }
}
```
## DSL语言
```sql
# 精确查询
GET ledger/_search
{
  "query": {
  "match": {
      "name":"ledger1"
    }
  }
}
# 模糊查询加上根据
# price降序排序
# _source字段筛选
# from 起始页
# size 长度
GET ledger/_search
{
  "query": {
  "match": {
      "name":"ledger"
    }
  },
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ],
  "from": 0,
  "size": 2，
  "_source": ["name","price"]
}
# bool合并多个布尔条件
# {bool}>{must、match_phrase、filter、must_not、should}所有的结合
    # [must_not]>{match、match_phrase、term、range、exists}必须不满足这个才给你放回
    # [must]>{match、match_phrase、term、range、exists}必须满足这个才给你返回
    # [should]>{match、match_phrase、term、range、exists}满足任意就可以，都不满足也会返回，但是分数降低
    # {filter}>{term、range、exists}文档过滤，结果已经搜索出来了
    # {match_phrase}>字段  字段的短文匹配
    # {match}>字段名:值 字段的短文匹配
    # {term}>字段名:值 字段的精确匹配
    # {range}>{字段名}>gte;lte 范围匹配
    # {exists}>filed:字段名 存在某个字段

# 使用bool
GET ledger/_search
{
  "query": {
    "bool": {
      "must": [
          {
            "match": {
            "name": "ledger"
          }
        }
      ],
      "filter": {
        "range": {
          "price": {
            "gte": 1,
            "lte": 1
          }
        }
      },
      "must_not": [
        {
          "match": {
            "name": "hjhh"
          }
        }
      ],
      "should": [
          {
          "match": {
          "name": "ledger"
          }
        },
        {
          "term": {
            "name": {
              "value": "ledger"
            }
          }
        }
      ]
    }
  }
}
```


### 输出结果
```json
{
  "took" : 2,  // 查询执行花费的时间，单位毫秒
  "timed_out" : false,  // 查询是否超时，这里为 false 表示未超时
  "_shards" : {
    "total" : 1,  // 总分片数
    "successful" : 1,  // 成功的分片数
    "skipped" : 0,  // 跳过的分片数
    "failed" : 0  // 失败的分片数
  },
  "hits" : {
    "total" : {
      "value" : 1,  // 总匹配文档数
      "relation" : "eq"  // 匹配关系，这里表示 "等于" 关系
    },
    "max_score" : 0.40059417,  // 最高的相关性得分
    "hits" : [
      {
        "_index" : "ledger",  // 索引名称
        "_type" : "_doc",  // 文档类型
        "_id" : "1",  // 文档 ID
        "_score" : 0.40059417,  // 文档的相关性得分
        "_source" : {  // 文档的原始数据
          "name" : "ledger big1"  // 文档中的字段数据
        }
      }
    ]
  }
}

```
### 和query统计的属性
```json
{
  "query":{
    "match_all": {}
  },
  "from": 0,
  "size": 2, 
  //高亮
  "highlight": {
    "fields": {
      "name": {}
    }
  }
}
```










