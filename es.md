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
















