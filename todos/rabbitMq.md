# Rabbitmq
## 简介mq
* 消息中间件是基于队列与消息传递技术，在网络环境中为应用系统提供同步或异步. 可靠的消息传输的支撑性软件系统——百度百科
## 应用场景
### 异步处理
> 用户注册后，需要发注册邮件和注册短信，传统的做法有两种

* 串行方式：将注册信息写入数据库后，发送注册邮件，再发送注册短信，以上三个任务全部完成后才返回给客户端。这有一个问题是，邮件，短信并不是必须的，它只是一个通知，而这种做法让客户端等待没有必要等待的东西。  
![](https://img-blog.csdnimg.cn/6e7e963a78dc4c84b087231bdf9662ac.png)
* 并行方式：将注册信息写入数据库后，发送邮件的同时，发送短信，以上三个任务完成后，返回给客户端，并行的方式能提高处理的时间。
![](https://img-blog.csdnimg.cn/76357a42e3584a4fb67ac94425d69f5d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX1_lpYvmlpfnmoTljaHljaE=,size_20,color_FFFFFF,t_70,g_se,x_16)

* 引入消息队列后，把发送邮件，短信不是必须的业务逻辑异步处理。
![](https://img-blog.csdnimg.cn/1b52cbeff2d64e2b8297c19af3b1726d.png)

### 应用解耦
> 双11是购物狂节，用户下单后，订单系统需要通知库存系统，传统的做法就是订单系统调用库存系统的接口。

![](https://img-blog.csdnimg.cn/0d837d84487e441dbbf74590fa076422.png)

* 这种做法有一个缺点:
    * 当库存系统出现故障时，订单就会失败。
    * 订单系统和库存系统高耦合。

* 引入消息队列


![](https://img-blog.csdnimg.cn/41889a3c066e47b7896fe57bba4d1d23.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX1_lpYvmlpfnmoTljaHljaE=,size_17,color_FFFFFF,t_70,g_se,x_16)

### 流量削峰
> 秒杀活动，一般会因为流量过大，导致应用挂掉，为了解决这个问题，一般在应用前端加入消息队列。

1. 用户的请求，服务器收到之后，首先写入消息队列，加入消息队列长度超过最大值，则直接抛弃用户请求或跳转到错误页面。

2. 秒杀业务根据消息队列中的请求信息，再做后续处理。

## 消息队列优缺点
* 优点
    * 解耦
    * 异步
    * 削峰
* 缺点
* 系统可用性降低
系统引入的外部依赖越多，越容易挂掉。本来你就是 A 系统调用 BCD 三个系统的接口就好了，人 ABCD 四个系统好好的，没啥问题，你偏加个 MQ 进来，万一 MQ 挂了咋整，MQ 一挂，整套系统崩溃的，你不就完了？如何保证消息队列的高可用，可以点击这里查看。

* 系统复杂度提高
硬生生加个 MQ 进来，你怎么**保证消息没有重复消费**？怎么**处理消息丢失的情况**？怎么保证消息传递的顺序性？头大头大，问题一大堆，痛苦不已。

* 一致性问题
A 系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是，要是 BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，咋整？你这数据就不一致了。

## 常用消息中间件

* AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。Erlang中的实现有RabbitMQ等。

* JMS即Java消息服务（Java Message Service）应用程序接口，是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。

## RabbitMQ的集群架构
![](https://img-blog.csdnimg.cn/098cbe5314ba487eadf04a964c214c8f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX1_lpYvmlpfnmoTljaHljaE=,size_19,color_FFFFFF,t_70,g_se,x_16#pic_center)

## 安装
* 参照我的另一个笔记(docker常用镜像配置)

## 管理界面介绍
* 第一次访问需要登录，默认的账号密码为：guest/guest

!()[https://img-blog.csdnimg.cn/3e19c22214044c128d0c88f27b9b12de.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX1_lpYvmlpfnmoTljaHljaE=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center]

* connections：无论生产者还是消费者，都需要与RabbitMQ建立连接后才可以完成消息的生产和消费，在这里可以查看连接情况
* channels：通道，建立连接后，会形成通道，消息的投递获取依赖通道。
* Exchanges：交换机，用来实现消息的路由
* Queues：队列，即消息队列，消息存放在队列中，等待消费，消费后被移除队列。

## 用户权限
* 超级管理员(administrator)
    * 可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

* 监控者(monitoring)
    * 可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)

* 策略制定者(policymaker)
    * 可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。

* 普通管理者(management)
    * 仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。

* 其他
    * 无法登陆管理控制台，通常就是普通的生产者和消费者。

## RabbitMQ的工作原理介绍

![](https://img-blog.csdnimg.cn/c9dac21006264c5abc63d5c2ea4b5b50.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX1_lpYvmlpfnmoTljaHljaE=,size_19,color_FFFFFF,t_70,g_se,x_16)


* Broker：消息队列服务进程，此进程包括两个部分：Exchange和Queue
* Exchange：消息队列交换机，按一定的规则将消息路由转发到某个队列，对消息进行过虑。
* Queue：消息队列，存储消息的队列，消息到达队列并转发给指定的消费者
* Producer：消息生产者，即生产方客户端，生产方客户端将消息发送
* Consumer：消息消费者，即消费方客户端，接收MQ转发的消息。

![](https://img-blog.csdnimg.cn/20e90392b4f1404d982478fabf813c97.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX1_lpYvmlpfnmoTljaHljaE=,size_20,color_FFFFFF,t_70,g_se,x_16)

## 消息流程
生产者发送消息流程：
1. 生产者和Broker建立TCP连接。
2. 生产者和Broker建立通道。
3. 生产者通过通道消息发送给Broker，由Exchange将消息进行转发。
4. Exchange将消息转发到指定的Queue（队列）

消费者接收消息流程：
1. 消费者和Broker建立TCP连接
2. 消费者和Broker建立通道
3. 消费者监听指定的Queue（队列）
4. 当有消息到达Queue时Broker默认将消息推送给消费者。
5. 消费者接收到消息。
6. ack回复给mq说消息收到了

## RabbitMQ 交换机类型
交换机类型	| 描述	| 使用场景
--- | --- | ---
直连交换机 (Direct)	| 将消息发送到指定的队列，需要指定路由键（Routing Key），与队列绑定的路由键完全匹配才会被转发	| 单一的消息路由，例如任务分发，日志传递等
主题交换机 (Topic)	| 使用通配符的方式进行消息路由，根据路由键的模式匹配来决定转发	| 需要灵活的消息路由，可以根据多种属性进行筛选和传递，例如新闻分类、设备状态等
扇形交换机 (Fanout)	| 将消息广播到绑定在交换机上的所有队列	| 需要将消息广播到多个消费者，例如实现发布/订阅模式，广播通知等
标题交换机 (Headers)	| 类似于主题交换机，但通过匹配消息头的键值对来进行路由	| 基于消息头的键值对进行复杂的路由，适用于某些特殊的情况，如需要根据多个头信息进行匹配的场景






























































