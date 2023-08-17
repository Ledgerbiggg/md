

# gulimall

## 远程服务调用

```yml
spring:
  application:
    name: gulimall-product
  cloud:
    nacos:
       discovery:
          server-addr: localhost:8848
```

### nacos注册中心

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202306262034943.png)

## 远程配置文件

### bootstrap.yml

```yml
spring:
  application:
    name: gulimall-product
  cloud:
    nacos:
      server-addr: localhost:8848
      config:
        namespace: 2986ae7c-6ee8-4236-93e0-2deabc455259
        extension-configs:
          - data-id: datasource.yaml
            group: dev
            refresh: true # 同步刷新数据
          - data-id: mybatis-plus.yaml
            group: dev
            refresh: true
      discovery:
        server-addr: localhost:8848
```

### nacos配置中心

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202306262030812.png)



## pojo

### 限制用户输入

* @NotNull
* @Null
* @NotEmpty
* @Min
* @Pattern
* @URL

```java
/**
 * 品牌
 * 
 * @author ledger
 * @email ledgerhhh@gmail.com
 * @date 2023-06-17 11:27:45
 */
@Data
@TableName("pms_brand")
public class BrandEntity implements Serializable {
	private static final long serialVersionUID = 1L;

	/**
	 * 品牌id
	 */
	@TableId
	@NotNull(message = "修改必须指定品牌id",groups = {UpdateGroup.class})
	@Null(message = "新增不能指定id",groups = {AddGroup.class})
	private Long brandId;
	/**
	 * 品牌名
	 */
	@NotBlank(message = "品牌名不能为空",groups = {AddGroup.class, UpdateGroup.class})
	private String name;
	/**
	 * 品牌logo地址
	 */
	@NotEmpty(message = "品牌logo不能为空",groups = {AddGroup.class})
	@URL(message = "logo必须是一个合法的地址")
	private String logo;
	/**
	 * 介绍
	 */
	private String descript;
	/**
	 * 显示状态[0-不显示；1-显示]
	 */
	@ListValue(vals={0,1},groups = {AddGroup.class})
	private Integer showStatus;
	/**
	 * 检索首字母
	 */

	@NotNull(message = "增加必须指定排序",groups = {AddGroup.class})
	@Pattern(regexp = "//^[a-zA-Z]$", message = "检索首字母必须是一个英文字母",groups = {UpdateGroup.class,AddGroup.class})
	private String firstLetter;
	/**
	 * 排序
	 */
	@NotNull(message = "增加必须指定排序",groups = {AddGroup.class})
	@Min(value = 0, message = "排序必须大于等于0",groups = {UpdateGroup.class,AddGroup.class})
	private Integer sort;

}

```

* tip:	
  1. message 报错消息
  2. groups 分组消息

## controller



```java
public interface AddGroup {
}
```

```java
public interface updateGroup {
}
```



* 创建两个分组，代表add和update操作(分类组的方法不能使用没有设置分组条件的字段限制注解)

```java

    /**
     * 保存
     */
    @RequestMapping("/save")
    //@RequiresPermissions("product:brand:save")
    public R save(@Validated({AddGroup.class}) @RequestBody BrandEntity brand) {
        brandService.save(brand);

        return R.ok();
    }

    /**
     * 修改
     */
    @RequestMapping("/update")
    //@RequiresPermissions("product:brand:update")
    public R update(@Validated({UpdateGroup.class}) @RequestBody BrandEntity brand) {
        brandService.updateById(brand);
        return R.ok();
    }
```



## oss签名直传

```xml
        <dependency>
            <groupId>com.aliyun.oss</groupId>
            <artifactId>aliyun-sdk-oss</artifactId>
            <version>3.15.1</version>
        </dependency>
```



```yml
aliyun:
  oss:
    access_key_id: LTAI5tSd****
    access_key_secret: bJyOcffhbo8Kp****
    endpoint: oss-cn-shenzhen.aliyuncs.com
    bucket_name: ledger-for-project
    dir: ledger/
```

```java
/**
 * @author ledger
 * @version 1.0
 **/
@RestController
@RequestMapping("/thirdparty")
public class OssController {

    @Value("${aliyun.oss.endpoint}")
    private String endpoint;

    @Value("${aliyun.oss.bucket_name}")
    private String bucket;

    @Value("${aliyun.oss.access_key_id}")
    private String accessId;

    @Value("${aliyun.oss.access_key_secret}")
    private String accessKey;

    @Value("${aliyun.oss.dir}")
    private String dir;

    @GetMapping("/oss/policy")
    public R oss() {
        // 创建ossClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessId, accessKey);
        try {
            long expireTime = 30;
            long expireEndTime = System.currentTimeMillis() + expireTime * 1000;
            Date expiration = new Date(expireEndTime);
            PolicyConditions policyConds = new PolicyConditions();
            policyConds.addConditionItem(PolicyConditions.COND_CONTENT_LENGTH_RANGE, 0, 1048576000);
            policyConds.addConditionItem(MatchMode.StartWith, PolicyConditions.COND_KEY, dir);

            String postPolicy = ossClient.generatePostPolicy(expiration, policyConds);
            byte[] binaryData = postPolicy.getBytes("utf-8");
            String encodedPolicy = BinaryUtil.toBase64String(binaryData);
            String postSignature = ossClient.calculatePostSignature(postPolicy);

            Map<String, String> respMap = new LinkedHashMap<String, String>();
            respMap.put("accessId", accessId);
            respMap.put("policy", encodedPolicy);
            respMap.put("signature", postSignature);
            respMap.put("dir", dir);
            respMap.put("host", "https://"+bucket+"."+endpoint);
            respMap.put("expire", String.valueOf(expireEndTime / 1000));
            return R.ok().put("data", respMap);
        } catch (Exception e) {
            // Assert.fail(e.getMessage());
            throw new RuntimeException(e);
        }
    }
}

```



## mp分页查询配置

```java
@Configuration
@EnableTransactionManagement
@MapperScan("com.ledger.gulimall.product.dao")
public class MybatisConfig {
    //引入分页插件
    @Bean
    public MybatisPlusInterceptor paginationInnerInterceptor(){
        MybatisPlusInterceptor mybatisPlusInterceptor=new MybatisPlusInterceptor();
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
        //设置请求页面大于最大页的操作
        paginationInnerInterceptor.setOverflow(true);
        //设置最大单页限制数量
        paginationInnerInterceptor.setMaxLimit(1000L);
        mybatisPlusInterceptor.addInnerInterceptor(paginationInnerInterceptor);
        return mybatisPlusInterceptor;
    }
}

```



## 使用jackson来进行时间格式化

```java
spring:
  datasource:
    username: root
    password: admin123
    url: jdbc:mysql://localhost:3306/gulimall_pms?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
```

## 网关配置

```yml
spring:
  application:
    name: gulimall-gateway
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
      config:
        group: dev
        namespace: ff2103e2-0b21-439c-bdb6-f52309ca0730
        file-extension: yml
    gateway:
      routes:
        - id: member_router
          uri: lb://gulimall-member
          predicates:
            - Path=/api/member/**
          filters:
            - RewritePath=/api/(?<segment>.*),/$\{segment}

        - id: ware-route
          uri: lb://gulimall-ware
          predicates:
            - Path=/api/ware/**
          filters:
            - RewritePath=/api/(?<segment>.*),/$\{segment}

        - id: product_route
          uri: lb://gulimall-product
          predicates:
            - Path=/api/product/**
          filters:
            - RewritePath=/api/(?<segment>.*),/$\{segment}

        - id: thirdparty_route
          uri: lb://gulimall-third-party
          predicates:
            - Path=/api/thirdparty/**
          filters:
            - RewritePath=/api/(?<segment>.*),/$\{segment}

        - id: admin_route
          uri: lb://renren-fast
          predicates:
            - Path=/api/**
          filters:
            - RewritePath=/api/(?<segment>.*),/renren-fast/$\{segment}
```

















































