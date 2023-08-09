# ElasticSearch（库、表、记录） **ELK**

> 搜索功能

* 和数据库对比
 
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307061553875.png)


![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307061600156.png)
## 基础

1. 安装
2. 生态圈
3. 分词器 ik
4. RestFull操作 ES
5. CRUD
6. springboot 集成 es(从原理分析)
7. 爬虫爬取数据
8. 实战，模拟全文搜索

以后只要使用，需要用到搜索。就可以使用es（大数据量的情况下可以使用）

## 熟悉目录

* bin 启动文件
* config 配置文件
  * log4j2  日志配置文件
  * jvm.option java虚拟机的相关配置
  * elasticsearch.yml elasticsearch的配置文件!  默认 9200端口 ！ 跨域
* lib 相关jar包
* modules 功能模块
* plugins 插件！ ik分词器(可以将文本成词语)
  * 最少切分：ik_smart
  * 最细切分：ik_max_word
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307061858813.png)


## 启动

* elasticsearch.bat

## 使用可视化工具(一个插件)

* elasticsearch-head-master

1. npm i

2. 修改es的配置文件es.xml

   ```yml
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   ```

3. 重启elasticsearch.bat

## ELK （es、logstash、kibana）

## es核心

### 索引（就是数据库）

### 字段类型（mapping）

### 文档（doucment）

### 分片(倒排索引)

## IK分词器

1. 下载IK分词器
2. 将IK放到es的插件下面
3. 重启es，插件被加载进去

### 查看分词效果

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307051941455.png)

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307051942989.png)

method | url地址 | 描述
--- | --- | --
PUT  | localhost:9200/索引名称/类型名称/文档id    | 创建文档(指定文档id)
POST  | localhost:9200/索引名称/类型名称    | 创建文档( 随机文档id )
POST  | localhost:9200/索引名称/类型名称/文档id/_update    | 修改文档
DELETE  | localhost:9200/索引名称/类型名称/文档id    | 删除文档
GET  | localhost:9200/索引名称/类型名称/文档id    | 查询文档通过文档id
POST  | localhost:9200/索引名称/类型名称/_search    | 查询所有数据

### 创建一个索引
* put /索引名/类型名/文档id
* {请求体}
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052014734.png)

* 索引增加
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052017230.png)

* 完成自动增加索引，数据也成功添加了，这就是从我说大家初期可以将其当做数据库学的原因

### 数据的基本类型(有点像数据库建库)

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052026475.png)

> 如果自己的文档字段没有指定，那么es就会给我们默认配置字段类型

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052036286.png)

> 扩展：通过命令索引情况 通过get _cat/可以es当前的很多信息

* 修改索引
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052042034.png)

* 查询索引

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052057949.png)

* 删除索引
```sh
DELETE test1
```

## 关于文档的基本操作(基本操作)
* 基本操作
1. 添加一条数据
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052053086.png)

2. 查找数据
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052057949.png)

3. 更新操作(推荐使用过这个_update，put不全给值的话就会置空)
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052105186.png)

4. 简单的搜索(传入参数匹配)
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052113057.png)


5. 复杂操作的搜索(排序，分页，高亮，模糊查询，精准查询！)
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052119810.png)

* hit：索引和文档的信息，查询的结果总数，然后就是查询出来的具体的文档，数据中的数据都可以遍历出来
  * 分数：我们可以通过来判断谁更加符合结果
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052125606.png)

* 匹配（match）：根据字段匹配数据
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052137498.png)
* 过滤（_source）：配置参数
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052129787.png)
* 排序（sort）：根据某个字段进行排序
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052137539.png)
* 分页（from,size）：从第几个数据开始，返回第几个数据
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052138873.png)
* 布尔（bool）：
  * （should）几组条件满足一个就可以返回了，好比or，
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052148222.png)
  * （must）每个条件都要满足，好比and，
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052151310.png)
  * （must_not）不能满足以下所有条件
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052154735.png)
  * （filter）：过滤条件
    * gt 大于
    * gte 大于等于
    * lt 小于
    * lte 小于等于
    ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052157449.png)
  * 匹配多个条件(多个关键字使用空格隔开)
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052203035.png)
  * 精确查询
    * term查询时直接通过倒排索引指定的词条进程精准的查找的
  * 关于分词
    * term，直接精确的查找
    * match，会使用分词器解析(先分析文档，然后通过分析的文档进行查询)
  * 两个类型 text keyword
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052216603.png)
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052216634.png)
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052226913.png)
  * 多个条件同时匹配
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052230335.png)
* 高亮效果
![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052236071.png)
  * 自定义高亮
  ![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307052240626.png)

### 总结
* 这些其实MySQL也可以做，只是MySQL 效率比较低!
  * 匹配
  * 按照条件匹配
  * 精确匹配
  * 区间范围匹配
  * 匹配字段过滤
  * 多条件查询
  * 高亮查询

* tip：在java里面，所有的方法和对象都是这里面的key
### 集成springboot
1. 引入依赖
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```

2. 新建对象
```JAVA
@Configuration
public class ElasticSearchConfig {
    @Bean
    public RestHighLevelClient restHighLevelClient() {
        return new RestHighLevelClient(
                RestClient.builder(
                        (new HttpHost("localhost", 9200, "http"))));

    }
}

```
3. 测试
```java
@SpringBootTest
class Es01ApiApplicationTests {
    @Autowired
    @Qualifier("restHighLevelClient")
    private RestHighLevelClient client;

    @Test
    void testCreateIndex() throws IOException {
        //创建索引请求
        CreateIndexRequest request = new CreateIndexRequest("yb");
        //执行请求indicesClient,请求之后获得响应
        CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);
        System.out.println(createIndexResponse);
    }

    @Test
    void testExistIndex() throws IOException {
        GetIndexRequest request = new GetIndexRequest("yb");
        boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }

    /**
     * 测试添加文档
     *
     * @throws IOException
     */
    @Test
    void testAddDoc() throws IOException {
        User user = new User();

        user.setName("ledger");
        user.setAge(111);
        IndexRequest indexRequest = new IndexRequest("");
        indexRequest.id("1");
        indexRequest.timeout(TimeValue.timeValueSeconds(2));
        indexRequest.source(JSON.toJSONString(user), XContentType.JSON);
        indexRequest.index("yb");
        IndexResponse index = client.index(indexRequest, RequestOptions.DEFAULT);
        System.out.println(index);

    }

    @Test
    void testIsExists() throws IOException {
        GetRequest request = new GetRequest("yb", "1");
        request.fetchSourceContext(new FetchSourceContext(false));
        request.storedFields("_none_");
        boolean exists = client.exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }

    @Test
    void testGetDoc() throws IOException {
        GetRequest request = new GetRequest("yb", "1");
        GetResponse response = client.get(request, RequestOptions.DEFAULT);
        System.out.println(response.getSourceAsString());
    }

    @Test
    void testUpdateDoc() throws IOException {
        UpdateRequest request = new UpdateRequest("yb", "1");
        request.timeout(TimeValue.timeValueSeconds(2));
        User user = new User("999999", 111);
        request.doc(JSON.toJSONString(user), XContentType.JSON);
        UpdateResponse update = client.update(request, RequestOptions.DEFAULT);
        System.out.println(update);
    }

    @Test
    void testDeleteRequest() throws IOException {
        DeleteRequest deleteRequest = new DeleteRequest("yb", "1");
        deleteRequest.timeout(TimeValue.timeValueSeconds(1));
        DeleteResponse delete = client.delete(deleteRequest, RequestOptions.DEFAULT);
        System.out.println(delete);
    }

    /**
     * 批量插入数据
     */
    @Test
    void testBulkRequest() throws IOException {
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.timeout(TimeValue.timeValueSeconds(2));
        ArrayList<User> users = new ArrayList<>();
        users.add(new User("ledger", 122));
        users.add(new User("ledger2", 555));
        users.add(new User("ledge452", 5565));
        for (int i = 0; i < users.size(); i++) {
            User user=users.get(i);
            bulkRequest.add(new IndexRequest("yb").source(JSON.toJSONString(user), XContentType.JSON));
        }
        BulkResponse responses = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println(responses);

    }

    /**
     * SearchRequest 搜索请求
     * SearchSourceBuilder 条件构造
     * HighlightBuilder 构建
     * MatchAllQueryBuilder 匹配所有
     *  xxx QueryBuilder 对应所有的命令
     * @throws IOException
     */
    @Test
    void  testSearch() throws IOException {
        SearchRequest searchRequest = new SearchRequest("yb");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //精确匹配
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("name", "ledger");
        sourceBuilder.query(termQueryBuilder);
        sourceBuilder.timeout(TimeValue.timeValueSeconds(2));
        searchRequest.source(sourceBuilder);
        sourceBuilder.from(1);
        sourceBuilder.size(1);
        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        System.out.println(search);
        SearchHit[] hits = search.getHits().getHits();
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsMap());
        }

    }

}
```
3. 关闭对象

## 实战

### 爬虫
> 数据问题？数据库获取，消息队列获取，都可以成为数据源，爬虫
爬取数据：(获取请求返回的页面信息，筛选出我们想要的数就可以了)



### 前后端分离


### 搜索高亮























