 # springcloud

## 微服务技术栈
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306070843160.png)

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306070844663.png)


# 实用篇

* 微服务治理
* Docker
* 异步通信
* 分布式搜索

# 高级篇

* 微服务保护
* 分布式事务
* 分布式缓存
* 多级缓存
* 可靠消息服务

# 面试篇

* Nacos源码
* Sentinel源码
* redis热点问题

# 实用篇
## 认识微服务
1. 单体架构
* 将业务的所有功能在一个项目中开发，打成一个包部署

优点：架构简单，部署成本低

缺点：耦合度高

2. 分布式架构
* 根据业务功能对系统进行拆分，每个业务独立项目开发，称为一个服务

优点：耦合度低，有利于服务的升级和拓展

服务治理：

* 服务拆分粒度如何
* 服务集群地址如何维护
* 服务之间乳化泵实现远程调用
* 服务健康状态如何感知

3. 微服务

微服务是一种经历过良好架构设计的分布式架构方案，微服务架构特征：

* 单一职责：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发
* 面向服务：微服务对外暴露业务接口
* 自治：团队独立、技术独立、数据独立、部署独立


## 微服务技术对比

微服务这种方案。有springcloud和Dubbo

* 微服务集群
* 注册中心(维护微服务每个节点信息和状态)
* 配置中心(统一管理微服务配置)
* 服务网关(访问微服务节点)

### 微服务技术对比
 | Dubbo | SpringCloud | SpringCloudAlibaba
---  | --- | --- | --- 
注册中心  | zookeeper、Redis | Eureka、Consul | Nacos、Eureka
服务远程调用 | Dubbo协议 | Feign (http协议) | Dubbo、Feign
配置中心 | 无 | SpringCloudConfig | SpringCloudConfig、Nacos
服务网关 | 无 | SpringCloudGateway、Zuul | SpringCloudGateway、Zuul
服务监控和保护 | dubbo-admin，功能弱 | Hystrix | Sentinel

### 企业需求

* 4种情况

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306070926822.png)

### springcloud
* SpringCloud是目前国内使用最广泛的微服务框架。官网地址: https://spring.io/projects/spring-cloud。
* SpringCloud集成了各种微服务功能组件，并基于SpringBoot实现了这些组件的自动装配，从而提供了良好的开箱即用体验:

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306070928258.png)

## 服务的拆分以及远程调用
### 服务拆分的注意事项
1. 不同的微服务，不要重复开发相同业务
2. 微服务独立，不要访问其他微服务的数据库
3. 微服务可以将自己的业务暴露为借口，供其他微服务使用

* 总结：

基于RestTemplate发起的http请求实现远程调用http请求做远程调用是与语言无关的调用，只要知道对方.的ip、端口、接口路径、请求参数即可。

## 微服务的远程调用
* 服务的提供者：一次业务中，被其他的微服务调用的服务(提供接口给其他微服务)
* 服务的消费者：一次业务，调用其他微服务的服务(调用其他微服务提供的接口)


### 服务调用关系

* 服务提供者:暴露接口给其它微服务调用
* 服务消费者:调用其它微服务提供的接口
* 提供者与消费者角色其实是相对的
* 一个服务可以同时是服务提供者和服务消费者

## Eureka注册中心

### 服务调用出现的问题

* 服务消费者该如何获取服务提供者的地址信息
* 如果有多个服务的提供者，消费者该如何选选择
* 消费者如何得知服务者的健康状态

## Eureka的作用

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306071029502.png)


* 消费者该如何获取服务提供者具体信息?
    1. 服务提供者启动时向eureka注册自己的信息
    2. eureka保存这些信息
    3. 消费者根据服务名称向eureka拉取提供者信息

* 如果有多个服务提供者，消费者该如何选择?
    1. 服务消费者利用负载均衡算法，从服务列表中挑选一个
* 消费者如何感知服务提供者健康状态?
    1. 服务提供者会每隔30秒向EurekaServer发送心跳请求，报告健康状态
    2. eureka会更新记录服务列表信息，心跳不正常会被剔除
    3. 消费者就可以拉取到最新的信息

### 总结
在Eureka架构中，微服务角色有两类:
* EurekaServer: 服务端，注册中心
    1. 记录服务信息
    2. 心跳监控
* EurekaClient:客户端
    * Provider:服务提供者，例如案例中的 user-service
    * 注册自己的信息到EurekaServer
    * 每隔30秒向EurekaServer发送心跳
* consumer:服务消费者，例如案例中的order-service
    * 根据服务名称从EurekaServer拉取服务列表
    * 基于服务列表做负载均衡，选中一个微服务后发起远程调用


## eureka注册中心

* 搭建EurekaServer服务步骤如下

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306071040836.png)

### 搭建注册中心
1. 配置文件
```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
```
2. 启动项目的注解
```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServer2Application {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer2Application.class, args);
    }

}
```
3. 配置文件
```yml
server:
  port: 10086 # 服务端口

spring:
  application:
    name: eurekaserver #eureka服务名
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10086/eureka # 注册中心地址
    fetch-registry: false # 不让Eureka Server从其他实例获取注册表信息。(防止控制台报错)
```
### 服务注册
1. 引入依赖
```xml
   <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <!--eureka客户端依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- nacos客户端依赖包 -->
<!--        <dependency>-->
<!--            <groupId>com.alibaba.cloud</groupId>-->
<!--            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>-->
<!--        </dependency>-->
<!--        &lt;!&ndash;nacos的配置管理依赖&ndash;&gt;-->
<!--        <dependency>-->
<!--            <groupId>com.alibaba.cloud</groupId>-->
<!--            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>-->
<!--        </dependency>-->
    </dependencies>
```
2. 配置相关配置
```yml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/cloud_user?useSSL=false&serverTimezone=UTC
    username: root
    password: admin123
    driver-class-name: com.mysql.cj.jdbc.Driver
  application:
    name: user-service
mybatis:
  type-aliases-package: com.ledger.user.pojo
  configuration:
    map-underscore-to-camel-case: true
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
      fetch-registry: false
logging:
  level:
    com.ledger: debug

```
3. 增加注解
```java
@EnableEurekaServer
@SpringBootApplication
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }

}
```
### 完成orderService的服务拉取
1. 修改orderservice的代码，修改访问的url路径，用服务名替代ip，端口
```java
@Service
public class OrderServiceImpl implements OrderService {

    @Resource
    private OrderMapper orderMapper;
    @Resource
    private RestTemplate restTemplate;

    @Override
    public Order getOrderById(Long id) {
        Order order = orderMapper.getOrderById(id);
        String url = "http://userservice/user/" + order.getUserId();
        User user = restTemplate.getForObject(url, User.class);
        order.setUser(user);
        return order;
    }
}
```
2. 在order-service项目的配置类中的增加**负载均衡**的注解
```java
@MapperScan("com.ledger.mapper")
@SpringBootApplication
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        //发送请求的模板
        return new RestTemplate();
    }
    @Bean
    public IRule RandomRule() {
        //随机负载均衡
        return new RandomRule();
    }
}
```
## 饥饿加载
* Ribbon默认是采用懒加载，即第一次访问时才会去创建LoadBalanceClient，请求时间会很长而饥饿加载则会在项目启动时创建，降低第一次访问的耗时，通过下面配置开启饥饿加载:
```yml
ribbon:
eager-Toad:
enabled: true # 开启饥饿加载
clients: userservice # 指定对userservice这个服务饥饿加载
```
## nacos

### 服务注册到Nacos
1. 在cloud-demo父工程中添加spring-cloud-alilibaba的管理依赖
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.5.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

2. 注释掉orderservice里面原有的eureka依赖，添加nacos的客户端依赖 

```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```
3. 配置相关配置选项
```yml
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos服务地址
```
## Nocos 服务分级存储模型
1. 修改application.yml,添加如下内容
```yml
      discovery:
        cluster-name: SH # 集群名称是杭州
```
2. 在Nacos控制台看到如下
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091424759.png)

## Nacos-NacosRule负载均衡
### NacosRule负载均衡策略

* 优先选择同集群服务实例列表
* 本地集群找不到提供者，才去其它集群寻找，并且会报警告
* 确定了可用实例列表后，再采用**随机**负载均衡挑选实例

## 根据权重实现负载均衡
1. 调整权重
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091442031.png)
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091443468.png)

### 总结
* 实例的权重控制
    1. Nacos控制台可以设置实例的权重值，0~1之间
    2. 同集群内的多个实例，权重越高被访问的频率越高
    3. 权重设置为0则完全不会被访问

## Nacos 环境隔离(namespace)
1. 新建命名空间
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091453733.png)

2. 配置文件更改命名空间的配置
```yml
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos服务地址
      discovery:
        cluster-name: HZ # 集群名称是杭州
        namespace: 15792aaa-bb43-4309-bdf0-6850bdca8183 # nacos命名空间
```
3. 注意点
* 不同命名空间之间的服务不能相互访问，orderservice访问userservice服务就会报错，说是找不到实例

### 总结
* Nacos环境隔离
    1. 每个namespace都有唯一id
    2. 服务设置namespace时要写id而不是名称
    3. 不同namespace下的服务互相不可见

## nacos注册中心的细节分析

1. nacos注册中心的细节分析
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091501599.png)

### 步骤
1. 更改实例为非临时实例
```yml
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos服务地址
      discovery:
        cluster-name: HZ # 集群名称是杭州
        namespace: 15792aaa-bb43-4309-bdf0-6850bdca8183 # nacos命名空间
        ephemeral: false # 是否是临时的
```
2. nacos服务器就会主动询问定时非临时的服务器的健康状态
3. 对于临时的服务器，nacos会采用心跳检测的方法，判断服务的健康状态
4. nacos对于服务的消费者采用推拉结合的方法，来提供消费者准确的地址信息
    * 服务消费者会有一个注册信息的缓存，避免每次都会向注册中心拉取消息
    * 消费者定时拉取nacos注册中心的注册信息
    * 当注册中心有信息变更或者服务健康状态变换，nacos会直接向服务消费者推送信息，确保消息的时效性

### 总结
1. acos与eureka的共同点
    * 都支持服务注册和服务拉取
    * 都支持服务提供者心跳方式做健康检测
2. Nacos与Eureka的区别
    1. Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式
    2. 临时实例心跳不正常会被剔除，非临时实例则不会被剔除
    3. Nacos支持服务列表变更的消息推送模式，服务列表更新更及时
    4. Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式

# springcloud微服务架构

## Nacos配置管理

### 统一配置管理
* 配置更改热更新

1. 新建配置
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091535716.png)
2. 配置文件
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091534175.png)
3. 配置文件的获取步骤
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091539390.png)
    1. 引入Nacos的配置管理客户端依赖:
```xml
           <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
```
    2. 在userservice里面添加一个bootstrap.yml文件，这个文件是引导文件，优先级高于application.xml:

```yml
    spring:
  application:
    name: userservice
  profiles:
    active: dev # 开发环境
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos服务地址
      config:
        file-extension: yaml # 文件后缀
        namespace: 15792aaa-bb43-4309-bdf0-6850bdca8183 # nacos命名空间(得看你配置文件里面有没有设置命名空间)
```
### 总结

* 将配置交给Nacos管理的步骤
    1. 在Nacos中添加配置文件
    2. 在微服务中引入nacos的config依赖
    3. 在微服务中添加bootstrap.yml，配置nacos地址、当前环境、服务名称、文件后缀名。这些决定了程序启动时去nacos读取哪个文件

## 配置的热更新

1. 第一种方式。增加这个@RefreshScope注解在使用到配置文件里面的pattern.dateformat这个上面

```java
@RestController
@RefreshScope
public class UerController {
    @Resource
    private UserService userService;
    @Value("${pattern.dateformat}")
    private String pattern;

    @GetMapping("/now")
    public String now() {
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(pattern));
    }

    @GetMapping("/user/{id}")
    public User user(@PathVariable long id) {
        return userService.getUserById(id);
    }
}
```
2. 第二种方式。
* 新建一个存储这种常量的类来代替@Value注解，然后将这个类配置到spring容器中，声明一个前缀
```java
@Data
@Component
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    @Value("${pattern.dateformat}")
    private String dateformat;
}
```
## 多环境配置共享
* 微服务启动时会从nacos读取多个配置文件：
    * [spring.application.name]-[spring.profiles.active].yaml，例如：userservice-dev.yaml
    * [spring.application.name].yaml，例如：userservice.yaml

* 无论profile如何变化，[spring.application.name].yaml这个文件一定会加载，因此多环境共享配置可以写入这个文件

* 当本地配置文件和远程配置文件有共用的配置属性的话，默认先使用远程的配置文件,远程配置文件，环境配置的优先级最高
    ![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091702542.png)

## nacos集群搭建
* 搭建集群的基本步骤
    1. 搭建数据库集群，初始化数据库表结构

    2. nacos集群配置
        * cluster.conf文件(将cluster.conf.example文件后缀去掉)

        ![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091922896.png)

    3. 配置nacos
        * application.properties

        ![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091924493.png)

    4. 启动nacos集群

        命令行或者双击startup.cmd

    5. nginx反向代理(增加nginx.conf文件)
    ![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091927633.png)

    6. 修改nacos的注册地址
    ```yml
      cloud:
    nacos:
      server-addr: localhost:8848 # nacos服务地址
      discovery:
        cluster-name: HZ # 集群名称是杭州
        namespace: 15792aaa-bb43-4309-bdf0-6850bdca8183 # nacos命名空间
        ephemeral: false # 是否是临时的
    ```

## Feign客户端依赖(简化发送请求的过程)
1. 导入Feign客户端依赖
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```
2. 配置一个userservice的发送请求的接口
```java
@FeignClient("userservice")
public interface UserClient {
    @GetMapping("/user/{id}")
    User getUserById(@PathVariable Long id);
}
```

3. 在orderservice里面直接调用就行了
```java
    @Resource
    private UserClient userClient;

    @Override
    public Order getOrderById(Long id) {
        Order order = orderMapper.getOrderById(id);
        User user = userClient.getUserById(order.getUserId());
        order.setUser(user);
        return order;
    }
```
## 自定义Feign的配置

* Feign运行自定义配置来覆盖默认配置，可以修改的配置如下

类型 | 作用 | 说明
--- | --- | ---
feign.Logger.Level	| 修改日志级别	| 包含四种不同的级别：NONE、BASIC、HEADERS、FULL
feign.codec.Decoder	| 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为java对象
feign.codec.Encoder	| 请求参数编码	| 将请求参数编码，便于通过http请求发送
feign. Contract	| 支持的注解格式	| 默认是SpringMVC的注解
feign. Retryer |	失败重试机制	| 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试

* 配置日志等级
1. 第一种方法(配置文件直接写配置)
```yml
feign:
  client:
    config:
      default: # default表示全局的配置，可以写成单个服务
        loggerLevel: FULL
```
2. 第二种方法(使用java配置类)
    1. 先写配置类
        ```java
        public class DefaultFeignConfiguration {
            @Bean
            public Logger.Level logLevel() {
                return Logger.Level.BASIC;
            }
        }
        ```
    2. 启动类增加配置
        ```java
            @MapperScan("com.ledger.mapper")
            @SpringBootApplication
            @EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration.class)//表示全局的配置
            public class OrderServiceApplication {
            
                public static void main(String[] args) {
                    SpringApplication.run(com.ledger.OrderServiceApplication.class, args);
                }

                @Bean
                @LoadBalanced
                public RestTemplate restTemplate() {
                    //发送请求的模板
                    return new RestTemplate();
                }
        }
        ```
### 总结
* Feign的日志配置:
1. 方式一是配置文件，feign.client.config.xxx.loggerLevel
    * 如果xxx是default则代表全局
    * 如果xxx是服务名称，例如userservice则代表某服务
2. 方式二是java代码配置Logger.Level这个Bean
    * 如果在@EnableFeignClients注解声明则代表全局
    * 如果在@FeignClient注解中声明则代表某服务

## Feign的性能优化
### Feign底层的客户端实现：
* URLConnection：默认实现，不支持连接池
* Apache HttpClient ：支持连接池
* OKHttp：支持连接池

### 因此优化Feign的性能主要包括：
* 使用连接池代替默认的URLConnection
* 日志级别，最好用basic或none

### 使用HttpClient示例
1. 引入依赖
```xml
         <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-httpclient</artifactId>
        </dependency>
```

2. 配置连接池
```yml
feign:
  httpclient:
    enabled: true # 开启httpclient
    max-connections: 200 # 最大连接数
```
### 总结

* Feign的优化：
    1. 日志级别尽量用basic
    2. 使用HttpClient或OKHttp代替URLConnection
        * 引入feign-httpClient依赖
        * 配置文件开启httpClient功能，设置连接池参数

## Feign的最佳实践

1. 方式一(继承): 给消费者非FeignClient和提供者的congtroller定义一个统一的父接口作为标准

2. 将FeignClient抽取为独立模块，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用✔✔✔
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306092121038.png)


## 抽取FeignClient
* 实现最佳实践方式二的步骤如下：
    1. 首先创建一个module，命名为feign-api，然后引入feign的starter依赖
    2. 将order-service中编写的UserClient、User、DefaultFeignConfiguration都复制到feign-api项目中
    3. 在order-service中引入feign-api的依赖!!!!
    4. 修改order-service中的所有与上述三个组件有关的import部分，改成导入feign-api中的包
    5. 重启测试

* tips： 
    *  当将Feign中发送请求的接口抽离出来之后，order的包扫描会出现问题，这个问题的解决方法有两种
    1. 方式一：指定FeignClient所在包
        ```java
            @EnableFeignClients(basePackages = "cn.itcast.feign.clients")
        ```
    2. 方式二：指定FeignClient字节码
        ```java
            @EnableFeignClients(basePackages = "cn.itcast.feign.clients")
        ```
## 同一网关Gateway
###  为什么需要网关

1. 网关功能
    * 身份认证和权限校验
    * 服务路由、负载均衡
    * 请求限流

### 搭建网关的服务

1. 创建新的module，引入网关依赖和服务发现依赖
```xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
```

2. 编写路由配置和nacos地址
```yml
server:
  port: 10010
spring:
  application:
    name: gateway
  cloud:
    nacos:
      server-addr: localhost:8848
    gateway:
      routes:
        - id: user-service
          uri: lb://userservice # 路由的目标地址 http就是固定地址
          predicates:
            - Path=/user/** # 这个是匹配规则，只要以/user/开头就会符合要求
```
## 路由断言工厂
### 网关路由可以配置的内容包括：
* 路由id：路由唯一标示
* uri：路由目的地，支持lb和http两种
* predicates：路由断言，判断请求是否符合要求，符合则转发到路由目的地
* filters：路由过滤器，处理请求或响应

* 断言工厂
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306101031163.png)

### 总结
* PredicateFactory的作用是什么？

    读取用户定义的断言条件，对请求做出判断

* Path=/user/**是什么含义？

    路径是以/user开头的就认为是符合的

## 路由过滤器GatewayFilter

* GatewayFilter是网关提供的一种路由过滤器，可以对进入网关的请求和微服务返回的响应做处理

### 过滤器工厂
* 过滤器的作用是什么？
    1. 对路由的请求或响应做加工处理，比如添加请求头
    2. 配置在路由下的过滤器只对当前路由的请求生效
* defaultFilters的作用是什么？
    1. 对所有路由都生效的过滤器

## 全局过滤器(处理进入微服务的响应)
* 全局过滤器的作用也是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样。
* 区别在于GatewayFilter通过配置定义，处理逻辑是固定的。而GlobalFilter的逻辑需要自己写代码实现。
* 定义方式是实现GlobalFilter接口。

```java
@Component
public class AuthorizeFilter implements GlobalFilter , Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //1.获取请求参数
        ServerHttpRequest request = exchange.getRequest();
        MultiValueMap<String, String> queryParams = request.getQueryParams();
        //判断参数里面是不是有auth这个参数，并且等于admin
        if("admin".equals(queryParams.getFirst("auth"))){
            //有这个参数就放行
            return chain.filter(exchange);
        }
        //设置响应状态码为401
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        //拦截响应回去
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        //拦截过滤器的执行顺序
        return 0;
    }
}
```
## 路由器的执行顺序

* 每一个过滤器都必须指定一个int类型的order值，order值越小，优先级越高，执行顺序越靠前。
* GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
* 路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增。
* 当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。

### 总结

* 路由过滤器、defaultFilter、全局过滤器的执行顺序？
1. order值越小，优先级越高
2. 当order值一样时，顺序是defaultFilter最先，然后是局部的路由过滤器，最后是全局过滤器

## 跨域问题处理

* 跨域问题：浏览器禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题
* 解决方案：CORS

```yml
spring:
  cloud:
    gateway:
      # 。。。
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOrigins: # 允许哪些网站的跨域请求 
              - "http://localhost:8090"
              - "http://www.leyou.com"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
```
### 总结
* CORS跨域要配置的参数包括哪几个？
    * 允许哪些域名跨域？
    * 允许哪些请求头？
    * 允许哪些请求方式？
    * 是否允许使用cookie？
    * 有效期是多久？

# Docker

## 初识Docker

* Docker是一个快速交付应用、运行应用的技术：

    1. 可以将程序及其依赖、运行环境一起打包为一个镜像，可以迁移到任意Linux操作系统
    2. 运行时利用沙箱机制形成隔离容器，各个应用互不干扰
    3. 启动、移除都可以通过一行命令完成，方便快捷

## Docker和虚拟机的差异：

* docker是一个系统进程；虚拟机是在操作系统中的操作系统
* docker体积小、启动速度快、性能好；虚拟机体积大、启动速度慢、性能一般

## 容器和镜像
* 镜像（Image）：Docker将应用程序及其所需的依赖、函数库、环境、配置等文件打包在一起，称为镜像。
* 容器（Container）：镜像中的应用程序运行后形成的进程就是容器，只是Docker会给容器做隔离，对外不可见。

## Docker架构
* Docker是一个CS架构的程序，由两部分组成：
    * 服务端(server)：Docker守护进程，负责处理Docker指令，管理镜像、容器等
    * 客户端(client)：通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令

### 总结
* 镜像：
    * 将应用程序及其依赖、环境、配置打包在一起
* 容器：
    * 镜像运行起来就是容器，一个镜像可以运行多个容器
* Docker结构：
    * 服务端：接收命令或远程请求，操作镜像或容器
    * 客户端：发送命令或者请求到Docker服务端
* DockerHub：
    * 一个镜像托管的服务器，类似的还有阿里云镜像服务，统称为DockerRegistry

## Docker安装
# 0.安装Docker

Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。

Docker CE 分为 `stable` `test` 和 `nightly` 三个更新频道。

官方网站上有各种环境下的 [安装指南](https://docs.docker.com/install/)，这里主要介绍 Docker CE 在 CentOS上的安装。

# 1.CentOS安装Docker

Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10， CentOS 7 满足最低内核的要求，所以我们在CentOS 7安装Docker。



## 1.1.卸载（可选）

如果之前安装过旧版本的Docker，可以使用下面命令卸载：

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```



## 1.2.安装docker

首先需要大家虚拟机联网，安装yum工具

```sh
yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
```



然后更新本地镜像源：

```shell
# 设置docker镜像源
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
```





然后输入命令：

```shell
yum install -y docker-ce
```

docker-ce为社区免费版本。稍等片刻，docker即可安装成功。



## 1.3.启动docker

Docker应用需要用到各种端口，逐一去修改防火墙设置。非常麻烦，因此建议大家直接关闭防火墙！

启动docker前，一定要关闭防火墙后！！

启动docker前，一定要关闭防火墙后！！

启动docker前，一定要关闭防火墙后！！



```sh
# 关闭
systemctl stop firewalld
# 禁止开机启动防火墙
systemctl disable firewalld
```



通过命令启动docker：

```sh
systemctl start docker  # 启动docker服务

systemctl stop docker  # 停止docker服务

systemctl restart docker  # 重启docker服务
```



然后输入命令，可以查看docker版本：

```
docker -v
```

如图：

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306101400233.png)


## 1.4.配置镜像加速

docker官方镜像仓库网速较差，我们需要设置国内镜像服务：

参考阿里云的镜像加速文档：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

## Docker的基操

### 镜像的相关命令

* 镜像名称一般分成两个组成部分[repository]:[tag]
* 在没有指定tag时，默认使用最新版本，代表最新版本的镜像

### 总结

* 镜像操作有哪些？
    * docker images 展示所有镜像
    * docker rmi  删除镜像
    * docker pull 拉取镜像
    * docker push 推送镜像
    * docker save  保存镜像成为tar文件
    * docker load 加载tar文件成为镜像

## 容器相关命令
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306101430171.png)

### 案例(创建一个nginx容器)

* 步骤一：去docker hub 查看Nginx的容器运行命令
    * docker run --name 容器的名字 -p 服务器的端口:容器的端口 -d 后台运行(镜像名称nginx:[tar]，不加标签代表最新版本)
```sh
docker run --name my_nginx -p 80:80 -d nginx
```
* 步骤二：查看容器日志的命令：
    * docker logs
    * 添加 -f 参数可以持续查看日志
* 步骤三：查看容器状态：
    * docker ps

### 案例修改容器中的文件(不推荐直接修改容器的文件)
* 步骤一：进入容器。进入我们刚刚创建的nginx容器的命令为：
    ```sh
    docker exec -it mn bash
    ```
* 步骤二：命令解读：
    * docker exec ：进入容器内部，执行一个命令
    * -it : 给当前进入的容器创建一个标准输入、输出终端，允许我们与容器交互
    * mn ：要进入的容器的名称
    * bash：进入容器后执行的命令，bash是一个linux终端交互命令

* 步骤二：进入nginx的HTML所在目录 /usr/share/nginx/html
    ```sh
    cd /usr/share/nginx/html
    ```
* 步骤三：修改index.html的内容
    ```sh
    sed -i 's#Welcome to nginx#ledger是大帅比哈哈哈#g' index.html
    sed -i 's#<head>#<head><meta charset="utf-8">#g' index.html
    ```
## 练习在docker容器中使用redis
* 进入redis容器，并执行redis-cli客户端命令，存入num=666
    1. 步骤一：进入redis容器
    ```sh
    docker exec -it redis bash
    ```
    2. 步骤二：执行redis-cli客户端命令
    ```sh
    redis-cli
    ```
    3. 步骤三：设置数据num=666
    ```sh
    set number 666
    ```
## 容器与数据耦合的问题

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306101914798.png)

### 数据卷
* 数据卷（volume）是一个虚拟目录，指向宿主机文件系统中的某个目录。
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306101923879.png)

* 操作数据卷
    * 数据卷操作的基本语法如下(二级指令)：
    ```sh
    docker volume [COMMAND]
    ```

 命令  |  作用
 --- | ---
 create |	创建一个volume
 inspect |	显示一个或多个volume的信息
 ls	 |	列出所有的volume
 prune	|	删除未使用的volume
 rm	 |	删除一个或多个指定的volume

## 数据卷挂载案例(创建一个nginx容器，修改容器的html目录内的index.html文件内容)
```sh
docker run --name mn -p 80:80 -v html:/usr/share/nginx/html -d nginx
```
docker run ：就是创建并运行容器

-- name mn ：给容器起个名字叫mn

-v html:/root/htm ：把html数据卷挂载到容器内的/root/html这个目录中

-p 8080:80 ：把宿主机的8080端口映射到容器内的80端口

nginx ：镜像名称

1. 步骤
    * 创建容器并挂载数据卷到容器内的html目录
    ```sh
    docker run --name mn -p 80:80 -v html:/usr/share/nginx/html -d nginx
    ```

    * 进入html数据卷所在位置，并修改html内容
    ```sh
    docker volume inspect html
    ```
    运行这个指令之后会出来关于html卷的详细信息
    ```sh
    [
    {
        "CreatedAt": "2023-06-10T19:28:21+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/html/_data",
        "Name": "html",
        "Options": null,
        "Scope": "local"
    }
    ]
    ```
    * 执行如下指令
    ```sh
    cd /var/lib/docker/volumes/html/_data/index.html
    ```
    * 使用windows上面的vscode修改这个目录文件

## 案例(创建并运行一个mysql容器，将宿主机目录直接挂在到容器)

-v [宿主机目录]:[容器内目录]

-v [宿主机文件]:[容器内文件]

* 实现思路如下：
1. 在将课前资料中的mysql.tar文件上传到虚拟机，通过load命令加载为镜像
2. 创建目录/tmp/mysql/data
3. 创建目录/tmp/mysql/conf，将课前资料提供的hmy.cnf文件上传到/tmp/mysql/conf
4. 去DockerHub查阅资料，创建并运行MySQL容器，要求：
    * 挂载/tmp/mysql/data到mysql容器内数据存储目录
    * 挂载/tmp/mysql/conf/hmy.cnf到mysql容器的配置文件
    * 设置MySQL密码
```sh
docker run --name mysql -e MYSQL_ROOT_PASSWORD=123  -p 1111:3306 -v /root/tmp/mysql/conf/hmy.cnf:/etc/mysql/conf.d/hmy.cnf -v /root/tmp/mysql/data:/var/lib/mysql -d mysql:5.7.25
```
### 总结
1. docker run的命令中通过 -v 参数挂载文件或目录到容器中：

* -v volume名称:容器内目录

    * -v 宿主机文件:容器内文件
    * -v 宿主机目录:容器内目录

2. 数据卷挂载与目录直接挂载的
    * 数据卷挂载耦合度低，由docker来管理目录，但是目录较深，不好找
    * 目录挂载耦合度高，需要我们自己管理目录，不过目录容易寻找查看

## 自定义镜像
* 镜像结构
    * 镜像是将应用程序及其所需要的系统函数库，环境，配置，依赖打包而成

* 自下而上：
    1. 基础镜像
        * 应用依赖的系统函数库、环境、配置、文件等
    2. 层（ Layer ）
        * 在BaseImage基础上添加安装包、依赖、配置等，每次操作都形成新的一层。
    3. 入口（Entrypoint）
        * 镜像运行入口，一般是程序启动的脚本和参数

## 什么是dockerfile
* Dockerfile就是一个文本文件，其中包含一个个的指令(Instruction)，用指令来说明要执行什么操作来构建镜像。每一个指令都会形成一层Layer。

指令 | 说明 | 示例
--- | --- | ---
FROM | 指定基础镜像 | FROM centos:6
ENV	| 设置环境变量，可在后面指令使用 | ENV key value
COPY | 拷贝本地文件到镜像的指定目录 | COPY ./mysql-5.7.rpm /tmp
RUN | 执行Linux的shell命令，一般是安装过程的命令 | RUN yum install gcc
EXPOSE | 指定容器运行时监听的端口，是给镜像使用者看的 | EXPOSE 8080
ENTRYPOINT | 镜像中应用的启动命令，容器运行时调用 | ENTRYPOINT java -jar xx.jar

## 案例(基于Ubuntu镜像构建一个新镜像，运行一个java项目)
* 步骤1：新建一个空文件夹docker-demo
* 步骤2：拷贝课前资料中的docker-demo.jar文件到docker-demo这个目录(项目的jar包)
* 步骤3：拷贝课前资料中的jdk8.tar.gz文件到docker-demo这个目录(jdk依赖)
* 步骤4：拷贝课前资料提供的Dockerfile到docker-demo这个目录(基础的layer层)

* 步骤5：进入docker-demo(进入所在文件夹，可以直接用docker build 在当前文件夹下面使用文件夹下面的所有文件来构建镜像)
* 步骤6：运行命令：
    ```sh
     docker build -t javaweb:1.0 . #这个点表示使用当前文件夹下面的dockerfile文件来构建自己的镜像
    ```

* tips：可以使用别人已经构建好的基础镜像来构建自己的镜像
如下的dockerfile文件
    ```sh
    # 指定基础镜像
    # FROM ubuntu:16.04
    # # 配置环境变量，JDK的安装目录
    # ENV JAVA_DIR=/usr/local

    # # 拷贝jdk和java项目的包
    # COPY ./jdk8.tar.gz $JAVA_DIR/
    ```


    # # 安装JDK
    # RUN cd $JAVA_DIR \
    #  && tar -xf ./jdk8.tar.gz \
    #  && mv ./jdk1.8.0_144 ./java8
    
    # # 配置环境变量
    # ENV JAVA_HOME=$JAVA_DIR/java8
    # ENV PATH=$PATH:$JAVA_HOME/bin
    FROM openjdk:8-alpine
    
    COPY ./docker-demo.jar /tmp/app.jar
    # 暴露端口
    EXPOSE 8090
    # 入口，java项目的启动命令
    ENTRYPOINT java -jar /tmp/app.jar
    ```
* 使用别人构建好的基础镜像来构建镜像，快速且轻量(FROM openjdk:8-alpine)

## DockerCompose
* 什么是DockerCompose
    * Docker Compose可以基于Compose文件帮我们快速的部署分布式应用，而无需手动一个个创建和运行容器！
    * Compose文件是一个文本文件，通过指令定义集群中的每个容器如何运行。

### 总结
* 帮助我们快速部署分布式应用，无需一个个微服务去构建镜像和部署













