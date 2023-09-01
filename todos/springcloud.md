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


























































