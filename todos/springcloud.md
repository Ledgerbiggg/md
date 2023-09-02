# SpringCloud
## 简介
* Spring cloud 是一系列框架的有序集合。它利用 spring boot 的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用 spring boot 的开发风格做到一键启动和部署。

##  优缺点
* 优点
1. 每个服务足够内聚，足够小，代码容易理解、开发效率提高；
2. 服务之间可以独立部署，微服务架构让持续部署成为可能；
3. 精细度业务控制，每个服务可以各自进行负载均衡扩展和数据库扩展，而且，每个服务可以根据自己的需要部署到合适的硬件服务器上；
4. 容易扩大开发团队，可以针对每个服务组件开发团队；
5. 提高容错性，一个服务的内存泄露并不会让整个系统瘫痪；
6. 每个服务可以用不同的技术开发，系统不会被长期限制在某个技术栈上。
* 缺点
1. 开发人员要处理分布式系统的复杂性：
设计服务之间的通信机制，需要考虑分布式事务等问题；
涉及多个服务直接的自动化测试；
服务管理的复杂性（PS：现在docker的出现适合解决这个问题）；
2. 对于业务数据和处理能力不是很明确的创业公司，不适合微服务架构模式，这时候最重要的是快速开发、快速部署、快速试错。
* 应用场景
    * 适合业务复杂度较大、并发量较高的场景
* SpringCloud与Dubbo对比
    * 架构完整度上，Dubbo只有服务注册/服务治理（zookeeper）两个模块，SpringCloud已经有二十多个模块，而且还在增加；
    * SpringCloud的社区活跃度高；但Dubbo有高质量的中文文档
    * Dubbo服务间的通讯采用的RPC,SpringCloud则是Http的Rset,RPC对业务接口有强依赖性，需要通讯双方有一样的接口，REST更为轻量化，只通过一个约定进行规范，不存在代码间的耦合。
## springcloud子项目
1. Spring Cloud Config（Spring）
配置管理工具包，让你可以把配置放到远程服务器，集中化管理集群配置，目前支持本地存储、Git以及Subversion。
2. Spring Cloud Bus(Spring)
事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署。
3. Eureka（Netflix）
云端服务发现，一个基于 REST 的服务，用于定位服务，以实现云端中间层服务发现和故障转移。
4. Hystrix（Netflix）
熔断器，容错管理工具，旨在通过熔断机制控制服务和第三方库的节点,从而对延迟和故障提供更强大的容错能力。
5. Zuul（Netflix）
Zuul 是在云平台上提供动态路由,监控,弹性,安全等边缘服务的框架。Zuul 相当于是设备和 Netflix 流应用的 Web 网站后端所有请求的前门。
6. Archaius（Netflix）
配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。
7. Consul（HashiCorp）
封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。
8. Spring Cloud for Cloud Foundry(Pivotal)
通过Oauth2协议绑定服务到CloudFoundry，CloudFoundry是VMware推出的开源PaaS云平台。
9. Spring Cloud Sleuth（Spring）
日志收集工具包，封装了Dapper和log-based追踪以及Zipkin和HTrace操作，为SpringCloud应用实现了一种分布式追踪解决方案。
10. Spring Cloud Data Flow（Pivotal）
大数据操作工具，作为Spring XD的替代产品，它是一个混合计算模型，结合了流数据与批量数据的处理方式。
11. Spring Cloud Security（Spring）
基于spring security的安全工具包，为你的应用程序添加安全控制。
12. Spring Cloud Zookeeper（Spring）
操作Zookeeper的工具包，用于使用zookeeper方式的服务发现和配置管理。
13. Spring Cloud Stream（Spring）
数据流操作开发包，封装了与Redis,Rabbit、Kafka等发送接收消息。
14. Spring Cloud CLI（Spring）
基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件。
15. Ribbon（Netflix）
提供云端负载均衡，有多种负载均衡策略可供选择，可配合服务发现和断路器使用。
16. Turbine（Netflix）
Turbine是聚合服务器发送事件流数据的一个工具，用来监控集群下hystrix的metrics情况。
17. Feign（OpenFeign）
Feign是一种声明式、模板化的HTTP客户端。
18. Spring Cloud Task（Spring）
提供云端计划任务管理、任务调度。
19. Spring Cloud Connectors（Spring）
便于云端应用程序在各种PaaS平台连接到后端，如：数据库和消息代理服务。
20. Spring Cloud Cluster（Spring）
提供Leadership选举，如：Zookeeper, Redis, Hazelcast, Consul等常见状态模式的抽象和实现。
21. Spring Cloud Starters（Pivotal）
Spring Boot式的启动项目，为Spring Cloud提供开箱即用的依赖管理。

## 项目搭建
1. 父工程继承,打包写pom不会参与打包成为jar,新增子模块是test-user,常用依赖直接写在父类
```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.15</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

     <packaging>pom</packaging>

    <modules>
        <module>test-user</module>
    </modules>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
2. 子工程继承父工程
```xml
   <parent>
        <groupId>com.ledger</groupId>
        <artifactId>springcloud_test</artifactId>
        <version>spring-cloud</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```
## Feign/Ribbon/OpenFeign
### 概念介绍 
* Feign:声明式 REST 客户端–Feign 通过使用 JAX-RS（就是一种使用注解来实现 RESTful 的技术）或 SpringMVC 注解的装饰方式，生成接口的动态实现。旨在自动封装服务调用客户端，减少代码开发量
* Ribbon:实现负载均衡，从一个服务的多台机器中选择一台。

### Nginx与Ribbon
1. Nginx 是客户端所有请求统一交给 nginx，由 nginx 进行实现负载均衡请求转发，属于服务器端负载均衡。既请求由 nginx 服务器端进行转发。
2. Ribbon 是从 eureka 注册中心服务器端上获取服务注册信息列表，缓存到本地，然后在本地实现轮询负载均衡策略。
3. Nginx 适合于服务器端实现负载均衡 比如 Tomcat ，Ribbon 适合与在微服务中 RPC 远程调用实现本地服务负载均衡，比如 Dubbo、SpringCloud 中都是采用本地负载均衡。

* Ribbon算法：实现Irule接口（choose方法），自带七种算法。

1. RoundRobinRule：轮询；(默认)
2. RandomRule：随机；
3. AvailabilityFilteringRule：会先过滤掉由于多次访问故障而处于断路器状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问；
4. WeightedResponseTimeRule：根据平均响应时间计算所有服务的权重，响应时间越快的服务权重越大被选中的概率越大。刚启动时如果统计信息不足，则使用RoundRobinRule（轮询）策略，等统计信息足够，会切换到WeightedResponseTimeRule；
5. RetryRule：先按照RoundRobinRule（轮询）策略获取服务，如果获取服务失败则在指定时间内进行重试，获取可用的服务；
6. BestAvailableRule：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务；
7. ZoneAvoidanceRule：复合判断Server所在区域的性能和Server的可用性选择服务器。

* OpenFeign:Feign的强化版，内置了ribbon，简化了Eureka和Ribbon的使用。

## Hystrix
* 使一种服务雪崩解决方案    
    * 服务雪崩：一个服务失败（调用时间过长或失败），导致整条链路的服务都失败的情形，我们称之为服务雪崩。
解决服务雪崩的方式有下面几种：

* 解决方法
1. 应用扩容：加机器、升级硬件
2. 流量控制： 限流（超出限定流量，返回让用户稍后再试）
3. 缓存：将用户访问量大的数据放入缓存中，减少数据库访问。
4. 服务降级：
    1. 超时：当下游的服务响应过慢，下游服务主动停掉一些不太重要的业务，释放出服务器资源，增加响应速度
    2. 程序运行异常：当下游的服务因为某种原因不可用，上游主动调用本地的一些降级逻辑，避免卡顿，迅速返回给用户
5. 服务熔断：不调用该失败的服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用


* 而Hystrix的出现为微服务提供了服务降级与熔断的方案。

### 原理
* hystrix熔断触发：
1. 当失败的调用到一定阀值，时间窗 5 秒内 20 次调用失败（默认），就会启动熔断机制。
2. 此时断路器将会开启，当开启的时候，所有请求都不会进行转发
* hystrix熔断恢复：
1. 一段时间之后(默认是5秒)，这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功,断路器会关闭，若失败，继续开启。

## OpenFeign的基本使用方法

* 后面nacos的使用中会整合Feign+hystrix,有完整的代码实例，这里就只做介绍不贴代码
1. 首先，在pom.xml中引入openfeign和nacos的依赖
2. 在nacos注册中心端提供一个可以调用的接口，并在application中添加注释@EnableDiscoveryClient
3. 在nacosclient客户端入口建立一个接口，@FeignClient括号里填写的是注册中心的application-name（application.yml中配置的），然后在controller中调用这个客户端。application中添加注解@EnableDiscoveryClient和@EnableFeignClients


## Nacos搭建使用
1. 引入依赖
* 控制spring-cloud版本
```xml
    <properties>
        <java.version>11</java.version>
        <!-- 控制spring-cloud版本的一致性 -->
        <spring-cloud.version>2021.0.1</spring-cloud.version>
        <!-- alibaba-cloud版本统一控制 -->
        <alibaba-cloud.version>2.1.0.RELEASE</alibaba-cloud.version>
    </properties>
```
* 依赖配置
```xml
<!--发现中心-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.2.6.RELEASE</version>
            <!--openfeign里面有复载均衡，所以这里必须注释掉，不然会冲突-->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--配置中心-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>

        <!--openfeign依赖 Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <!--openFeign使用时需要搭配loadbalancer负载均衡-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-loadbalancer</artifactId>
        </dependency>

        <!--hystrix依赖 spring-cloud-starter-hystrix已经被废弃了-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
```
* 这是spring-cloud的依赖配置管理
    * 将所有的org.springframework.cloud包下面的依赖版本和com.alibaba.cloud下面的依赖版本统一管理,就不用写version也可以得到很好的兼容
```xml
    <dependencyManagement>
        <!-- 版本控制 -->
        <dependencies>
            <!-- org.springframework.cloud包下面的版本控制 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- com.alibaba.cloud包下面的版本控制 -->
             <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${alibaba-cloud.version}</version> <!-- 指定所需的版本号 -->
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
2. 配置相关依赖
```yml
server:
  port: 9000
spring:
  application:
    # 服务名称
    name: test-product
  cloud:
    nacos:
      discovery:
        # nacos注册中心地址
        server-addr: 106.54.9.19:8848
        # 集群位置
        cluster-name: NB
```
3. 开启使用Discovery和Hystrix
```java
@EnableDiscoveryClient
@EnableHystrix
@SpringBootApplication
public class TestProductApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestProductApplication.class, args);
    }
}
```
4. 复制项目启动 
* 配置另外一个地址的启动端口
    * 命令行参数是:-Dserver.port=8001
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230902112255.png)

* 启动
![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/20230902112151.png)

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
  3. 不同namespace下的服务互相不可见(一般是做环境的区别)

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
  
## Nacos配置管理
1. 引入依赖
```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
```
2. 配置**bootstrap.yml**
* 配置地址  spring.cloud.nacos.server-addr: 106.54.9.19:8848 
* 文件名后缀    spring.cloud.config.file-extension: yaml
* 命名空间  spring.cloud.config.namespace: 85e7bf18-84c9-42e9-a5f6-46de7fec3215
* 配置环境  spring.
* 远程nacos中的配置中心的命名(85e7bf18-84c9-42e9-a5f6-46de7fec3215)空间下面的文件是
    * {服务名称}-{环境名称}.{后缀名称}
```yml
spring:
  application:
    # 服务名称
    name: test-product
  profiles:
    # 环境
    active: dev
  cloud:
    nacos:
      server-addr: 106.54.9.19:8848
      # nacos地址
      discovery:
        # 集群位置
        cluster-name: NB
        # 命名空间
        namespace: 85e7bf18-84c9-42e9-a5f6-46de7fec3215
      config:
        # 文件后缀
        file-extension: yaml
        # nacos命名空间(得看你配置文件里面有没有设置命名空间)
        namespace: 85e7bf18-84c9-42e9-a5f6-46de7fec3215
```
3. **@RefreshScope注解**
* 这个注解加在使用变量的类上面
* 如果是修改数据源配置等会影响spring容器里面的Bean的配置,建议使用主动配置数据源的操作
```java
@RestController
@RefreshScope
public class ProductController {

    @Value("${ledger}")
    private String ledger;

    //配置降级方法
    @HystrixCommand(
            fallbackMethod = "timeOutInvoke",
            commandProperties = {
            //设置服务调用超时10秒时触发服务降级
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "10000")
    })
    @GetMapping("/hello")
    public String invoke(){
        return ledger;
    }

    public String timeOutInvoke() {
        return "系统繁忙，请稍后再试";
    }
}
```
## 多环境配置共享
* 微服务启动时会从nacos读取多个配置文件：
  * [spring.application.name]-[spring.profiles.active].yaml，例如：userservice-dev.yaml
  * [spring.application.name].yaml，例如：userservice.yaml

* 无论profile如何变化，[spring.application.name].yaml这个文件一定会加载，因此多环境共享配置可以写入这个文件

* 当本地配置文件和远程配置文件有共用的配置属性的话，默认先使用远程的配置文件,远程配置文件，环境配置的优先级最高

![](https://image-bed-for-ledgerhhh.oss-cn-beijing.aliyuncs.com/image/202306091702542.png)


## openFeign客户端依赖(简化发送请求的过程)
1. 导入Feign客户端依赖
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```
2. 配置注解在启动类上面TestProductApplication
```java
@EnableFeignClients
public class TestProductApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestProductApplication.class, args);
    }
}
```
2. 配置一个userservice的发送请求的接口
```java
@FeignClient(value = "test-user")
public interface userService {
    @GetMapping("/getUser")
    List<User> getUser();
}
```
3. 配置bean
```java
@Data
@TableName("spring_cloud_test")
public class User {
    @TableId
    private String id;
    private String userName;
    private int age;
    private boolean gender;
}
```
4. controller
```java
@RestController
@RefreshScope
@Slf4j
public class ProductController {

    @Value("${ledger}")
    private String ledger;
    @Resource
    private userService userService;

    //配置降级方法
    @HystrixCommand(
            fallbackMethod = "timeOutInvoke",
            commandProperties = {
            //设置服务调用超时10秒时触发服务降级
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "10000")
    })
    @GetMapping("/hello")
    public String invoke(){
        List<User> user = userService.getUser();
        return user.toString();
    }

    public String timeOutInvoke() {
        return "系统繁忙，请稍后再试";
    }
}
```
## 自定义Feign的配置

* Feign运行自定义配置来覆盖默认配置，可以修改的配置如下

| 类型                | 作用             | 说明                                                   |
| ------------------- | ---------------- | ------------------------------------------------------ |
| feign.Logger.Level  | 修改日志级别     | 包含四种不同的级别：NONE、BASIC、HEADERS、FULL         |
| feign.codec.Decoder | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为java对象 |
| feign.codec.Encoder | 请求参数编码     | 将请求参数编码，便于通过http请求发送                   |
| feign. Contract     | 支持的注解格式   | 默认是SpringMVC的注解                                  |
| feign. Retryer      | 失败重试机制     | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试 |

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





























