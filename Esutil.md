## es操作的工具类

### 依赖包
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.7.1</version>
</dependency>
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>7.7.1</version>
</dependency>
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.19</version>
</dependency>
```

### es连接的配置
```java
public class ElasticSearchConnection {
    private volatile static RestHighLevelClient client;

    public static RestHighLevelClient getClient(){
        if(client==null){
            synchronized (ElasticSearchConnection.class){
                if(client==null){
                    HttpHost http = new HttpHost("106.54.9.19", 9200, "http");
                    RestClientBuilder builder = RestClient.builder(http);
                    client=new RestHighLevelClient(builder);
                }
            }
        }
        return client;
    }
    public static void closeClient(){
        if (client != null) {
            try {
                client.close();
            } catch (IOException e) {
               throw new RuntimeException(e);
            }
        }
    }
}
```

### es工具类
```java
public class EsUtil {
    /**
     * 根据类型返回对应的list的封装类
     * @param index 索引
     * @param type 类型的类
     * @param from 起始
     * @param size 大小
     * @param match 匹配的映射
     * @param <T> 查询的泛型
     * @return 查询的泛型集合
     */
    public static <T> List<T> SearchFromEs(String index, Class<T> type, Integer from, Integer size, Map<String, String> match) {
        // 获取连接到 Elasticsearch 的客户端
        RestHighLevelClient client = ElasticSearchConnection.getClient();
        // 创建一个搜索请求，指定要搜索的索引
        SearchRequest searchRequest = new SearchRequest(index);
        // 创建一个搜索源构建器，用于构建查询
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        // 构建一个匹配所有文档的查询
        if (MapUtil.isEmpty(match)) {
            searchSourceBuilder.query(QueryBuilders.matchAllQuery());
        } else {
            BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
            match.forEach((s, s2) -> boolQueryBuilder.must(QueryBuilders.matchQuery(s, s2)));
            //构建查询条件
            searchSourceBuilder.query(boolQueryBuilder);
        }

        // 添加排序条件，根据 "price" 字段降序排序
        searchSourceBuilder.sort(SortBuilders.fieldSort("price").order(SortOrder.DESC));
        // 设置分页参数，从第 0 条开始取 10 条数据
        searchSourceBuilder.from(from);
        searchSourceBuilder.size(size);
        // 将搜索源构建器设置为搜索请求的查询源
        searchRequest.source(searchSourceBuilder);
        // 执行搜索请求，获取搜索响应
        SearchResponse searchResponse = null;
        try {
            searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("查询es数据库出错");
        }
        // 从搜索响应中获取命中的文档数组
        SearchHit[] hits = searchResponse.getHits().getHits();
        // 遍历搜索结果并输出
        if(hits.length==0) {
            return null;
        }
        ArrayList<T> list = new ArrayList<>();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();
            T o = JSON.parseObject(sourceAsString, type);
            list.add(o);
        }
        // 关闭 Elasticsearch 客户端连接
        ElasticSearchConnection.closeClient();
        return list;
    }

    /**
     * 批量插入数据
     * @param index 索引
     * @param entityList 数据集合
     * @param <T> 数据泛型
     */
    public static <T> void insertIntoEs(String index, List<T> entityList){
        GetIndexRequest getIndexRequest = new GetIndexRequest(index);

        RestHighLevelClient client = ElasticSearchConnection.getClient();
        //异步发起一个查询索引是否存在的请求
        client.indices().existsAsync(getIndexRequest, RequestOptions.DEFAULT, new ActionListener<>() {
            @Override
            public void onResponse(Boolean aBoolean) {
                if(aBoolean){
                    System.out.println("es存在这个索引");
                }else {
                    System.out.println("es不存在这个索引,要新建");
                    CreateIndexRequest createIndexRequest = new CreateIndexRequest(index);
                    try {
                        CreateIndexResponse createIndexResponse = client.indices().create(createIndexRequest, RequestOptions.DEFAULT);
                    } catch (IOException e) {
                        e.printStackTrace();
                        throw new RuntimeException("新建es索引失败");
                    }
                }
                //创建批量操作请求
                BulkRequest bulkRequest = new BulkRequest();
                for (T t : entityList) {
                    //指定索引
                    IndexRequest indexRequest = new IndexRequest(index);
                    //新增map
                    indexRequest.source(BeanUtil.beanToMap(t));
                    //添加
                    bulkRequest.add(indexRequest);
                }
                //客户端异步执行
                client.bulkAsync(bulkRequest, RequestOptions.DEFAULT, new ActionListener<BulkResponse>() {
                    @Override
                    public void onResponse(BulkResponse bulkItemResponses) {
                        if(bulkItemResponses.hasFailures()) {
                            throw new RuntimeException("批量插入es失败");
                        }
                    }
                    @Override
                    public void onFailure(Exception e) {
                        e.printStackTrace();
                        throw new RuntimeException("批量插入es失败");
                    }
                });
            }
            @Override
            public void onFailure(Exception e) {
                e.printStackTrace();
            }
        });
        ElasticSearchConnection.closeClient();
    }

    /**
     * 根据id删除文档
     * @param index 文档索引
     * @param id 文档id
     */
    public static void deleteDocFromEs(String index,String id){
        RestHighLevelClient client = ElasticSearchConnection.getClient();
        //删除的请求
        DeleteRequest deleteRequest = new DeleteRequest(index,id);
        //异步删除,获得返回值
        client.deleteAsync(deleteRequest, RequestOptions.DEFAULT, new ActionListener<DeleteResponse>() {
            @Override
            public void onResponse(DeleteResponse deleteResponse) {
                if(RestStatus.OK!=deleteResponse.status()){
                    throw new RuntimeException("删除es文档失败");
                }
            }
            @Override
            public void onFailure(Exception e) {
                e.printStackTrace();
                throw new RuntimeException("删除es文档失败");
            }
        });
        ElasticSearchConnection.closeClient();
    }
    /**
     * 根据id来修改文档
     * @param index 索引
     * @param id id
     * @param entity 实体类
     * @param <T> 实体类泛型
     * @return 修改结果布尔
     */
    public static <T> Boolean updateDocInEs(String index,String id,T entity){
        RestHighLevelClient client = ElasticSearchConnection.getClient();
        //创建一个修改的文档
        UpdateRequest updateRequest = new UpdateRequest();
        //实体类转化为map
        updateRequest.doc(BeanUtil.beanToMap(entity));
        UpdateResponse update;
        try {
            //跟新的响应
            update = client.update(updateRequest, RequestOptions.DEFAULT);
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("删除索引失败");
        }
        ElasticSearchConnection.closeClient();
        return update.status()==RestStatus.OK?Boolean.TRUE:Boolean.FALSE;
    }
}
```