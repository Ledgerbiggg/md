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
扇形交换机 (Fanout)	| 将消息广播到绑定在交换机上的所有队列	| 需要将消息广播到多个消费者，例如实现发布/订阅模式，广播通知等
主题交换机 (Topic)	| 使用通配符的方式进行消息路由，根据路由键的模式匹配来决定转发	| 需要灵活的消息路由，可以根据多种属性进行筛选和传递，例如新闻分类、设备状态等
标题交换机 (Headers)	| 类似于主题交换机，但通过匹配消息头的键值对来进行路由	| 基于消息头的键值对进行复杂的路由，适用于某些特殊的情况，如需要根据多个头信息进行匹配的场景

### 介绍一下规则
\* 表示一个单词
\# 用来表示任意数量的单词
* 例子 
    * 通配绑定键是跟队列进行绑定的，举个例子
        * 队列Q1绑定键是*.TT.* 队列Q2绑定键为TT。#
        * 如果消息携带的路由键是A.TT.B,那么队列Q1将会收到
        * 如果消息携带的路由键是TT.B.A,那么队列Q2将会收到

## Rabbitmq的使用(直连)
1. 引入依赖
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

2. 配置文件的修改
```yml
server:
  port: 8888

spring:
  application:
    name: test_code
  rabbitmq:
    host: 106.54.9.19
    port: 5672
    username: admin
    password: admin
    virtual-host: ledger

rabbitmq:
  exchange: TestDirectExchange
  queen: TestDirectQueue
  routing: TestDirectRouting
  lonely: lonelyDirectExchange
```

3. 配置交换机,队列,绑定
 ```java
@Configuration
public class RabbitConfig {

    @Value("${rabbitmq.exchange}")
    private String exchange;
    @Value("${rabbitmq.queen}")
    private String queen;
    @Value("${rabbitmq.routing}")
    private String routing;
    @Value("${rabbitmq.lonely}")
    private String lonely;


    @Bean
    public DirectExchange TestDirectExchange() {
        //配置交换机
        return new DirectExchange(exchange,true,false);
    }

    @Bean
    public Queue TestDirectQueue() {
        //配置直连交换机的名字和是否持久化(默认是不是持久化)
        return new Queue(queen, true);
    }

    @Bean
    public Binding binding() {
        return BindingBuilder.bind(TestDirectQueue()).to(TestDirectExchange()).with(routing);
    }

    @Bean
    public DirectExchange lonelyDirectExchange() {
        return new DirectExchange(lonely);
    }

}
 ```
 4. 发送消息
 ```java
    @GetMapping("/sendDirectMessage")
    public String sendDirectMessage(Student student){
        //发送消息(传入字符串是不会乱码的)
        String jsonString = JSON.toJSONString(student);
        rabbitTemplate.convertAndSend(exchange,routing,jsonString);
        return "数据完成收集";
    }
 ```
5. 接收消息
```java
    @RabbitHandler
    public void process(String message){
        Student student = JSON.parseObject(message, Student.class);
        System.out.println(student);
    }
```
## Rabbitmq的使用(扇形)
1. 绑定
* 一个交换机可绑定多个队列，不需要使用路由来绑定

```java
@Configuration
public class RabbitFanConfig {

    @Value("${fanout.exchange}")
    private String exchange;
    @Value("${fanout.queen}")
    private String queen;


    @Bean
    public FanoutExchange TestFanoutExchange() {
        //配置交换机
        return new FanoutExchange(exchange,true,false);
    }

    @Bean
    public Queue TestFanoutQueue() {
        //配置直连交换机的名字和是否持久化(默认是不是持久化)
        return new Queue(queen, true);
    }
    @Bean
    public Queue TestFanoutQueue2() {
        //配置直连交换机的名字和是否持久化(默认是不是持久化)
        return new Queue(queen+"2", true);
    }

    @Bean
    public Queue TestFanoutQueue3() {
        //配置直连交换机的名字和是否持久化(默认是不是持久化)
        return new Queue(queen+"3", true);
    }
    @Bean
    public Binding fanBinding() {
        return BindingBuilder
                .bind(TestFanoutQueue())
                .to(TestFanoutExchange());
    }
    @Bean
    public Binding fanBinding2() {
        return BindingBuilder
                .bind(TestFanoutQueue2())
                .to(TestFanoutExchange());
    }

    @Bean
    public Binding fanBinding3() {
        return BindingBuilder
                .bind(TestFanoutQueue3())
                .to(TestFanoutExchange());
    }

}

```
2. 发送消息(不需要绑定路由)
```java
    @GetMapping("/sendFanMessage")
    public String sendFanMessage(Student student){
        //发送消息(传入字符串是不会乱码的)
        String jsonString = JSON.toJSONString(student);
        rabbitTemplate.convertAndSend(fanoutExchange,jsonString);
        return "数据完成收集";
    }
```
3. 其他大差不差
## Rabbitmq的使用(主题)

* .表示一个字符
* \#表示任意字符

1. 绑定
```java
@Configuration
public class RabbitTopicConfig {

    @Value("${topic.exchange}")
    private String exchange;
    @Value("${topic.queen}")
    private String queen;
    @Value("${topic.routing}")
    private String routing;

    @Bean
    public TopicExchange TestFTopicExchange() {
        //配置交换机
        return new TopicExchange(exchange,true,false);
    }

    @Bean
    public Queue TestTopicQueue() {
        //配置直连交换机的名字和是否持久化(默认是不是持久化)
        return new Queue(queen+".test1", true);
    }
    @Bean
    public Queue TestTopicQueue2() {
        //配置直连交换机的名字和是否持久化(默认是不是持久化)
        return new Queue(queen+".ledger", true);
    }

    @Bean
    public Binding topicBinding() {
        //TestTopicQueue
        return BindingBuilder
                .bind(TestTopicQueue())
                .to(TestFTopicExchange())
                .with(routing+".#");
    }

    @Bean
    public Binding topicBinding2() {
        //TestTopicQueue
        return BindingBuilder
                .bind(TestTopicQueue2())
                .to(TestFTopicExchange())
                .with(routing+".ledger");
    }
}

```
* 发送消息
    * 发送到topicExchange这个交换机 topRouting+".ledger" 这个路由上面去 
    * 有匹配字符是这个的就会接受到消息(routing+".ledger")和(routing+".#")队列
```java

    @GetMapping("/sendTopicMessage")
    public String sendTopicMessage(Student student) {
        //发送消息(传入字符串是不会乱码的)
        String jsonString = JSON.toJSONString(student);
        rabbitTemplate.convertAndSend(topicExchange, topRouting+".ledger", jsonString);
        return "数据完成收集";
    }
```

## tips
* 队列和交换机的创建时机是在服务启动的时候创建的，重复创还能不会报错，同名的不同配置会报错
* 消息的发送和接受所对应的类型要一样(这块都是String)
```java

    //发送消息
    @GetMapping("/sendDirectMessage")
    public String sendDirectMessage(Student student){
        //发送消息(传入字符串是不会乱码的)
        String jsonString = JSON.toJSONString(student);
        rabbitTemplate.convertAndSend(exchange,routing,jsonString);
        return "数据完成收集";
    }

    //接受消息
    @RabbitHandler
    public void process(String message){
        Student student = JSON.parseObject(message, Student.class);
        System.out.println(student);
    }
```
* 发送消息的时候交换机不存在会报错，路由不存在不会报错，消息会丢失
* 不绑定路由的话，不会发送到直连或者主题交换机，消息会丢失
    * 即这个方法只适合扇形交换机使用
```java
rabbitTemplate.convertAndSend(exchange, jsonString);
```
## 消息回调
* 消息的确认(生产者推送消息成功，消费者接受消息成功)

## 生产者消息确认的回调机制
1. 配置消息确认的配置项
    * publisher-confirm-type
    * publisher-returns
```yml
  rabbitmq:
    host: 106.54.9.19
    port: 5672
    username: admin
    password: admin
    virtual-host: ledger
    # 确认消息已经发送到交换机
    publisher-confirm-type: correlated
    # 确认消息已经被投递到队列
    publisher-returns: true
```
2. 配置相关的回调(直接修改里面的RabbitTemplate)
```java
@Configuration
public class RabbitConfig {

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate();

        rabbitTemplate.setConnectionFactory(connectionFactory);

        //设置开启Mandatory,才能触发回调函数,无论消息推送结果怎么样都强制调用回调函数
        rabbitTemplate.setMandatory(true);
        //消息投递到交换机触发的回调
        rabbitTemplate.setConfirmCallback((correlationData, b, s) -> {
            System.out.println("callback   相关数据" + correlationData);
            System.out.println("callback   确认原因" + b);
            System.out.println("callback   原因" + s);
        });
        //消息投递到队列失败就会触发这个回调
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
            @Override
            public void returnedMessage(ReturnedMessage returnedMessage) {
                byte[] body = returnedMessage.getMessage().getBody();
                System.out.println(new String(body));
                System.out.println("callback    returnedMessage   相关数据" + returnedMessage.getMessage());
                System.out.println("callback    returnedMessage   确认原因" + returnedMessage.getReplyCode());
                System.out.println("callback    returnedMessage   原因" + returnedMessage.getReplyText()); 
            }
        });
        return rabbitTemplate;
    }
}
```
* tip:
    * setConfirmCallback
    1. 这个回调肯定会执行,用来检查是不是已经送到交换机,交换机没有找到就会false

    * setReturnsCallback
    1. 这个回调不一定执行,会在没有队列接受到交换机消息的时候执行

    * 场合
    1. 交换机没有但是对队列 setConfirmCallback(false) setReturnsCallback(不执行)
    2. 有交换机但是没对队列 setConfirmCallback(true) setReturnsCallback(false)
    3. 交换机没有也没对队列 setConfirmCallback(false) setReturnsCallback(不执行)
    4. 交换机和队列都有 setConfirmCallback(true) setReturnsCallback(不执行)

## 消费者消息确认机制
1. 自动确认(默认)
* 接受到消息就算你默认收到消息,队列中的消息就会消失
2. 手动确认,在处理完成消息之后,要给服务器发送确认的消息

手动确认 | 内容
--- | ---
basic.ack |  用于肯定确认
basic.nack | 用于否定确认 (注意:这是AMQPO-9-1的RabbitMQ扩展)
basic.reiect  | 用于否定确认，但与 basic,nack 相比有一个限制:一次只能拒绝单条消息

* 使用拒绝后重新入列这个确认模式要谨慎，因为一般都是出现异常的时候，catch 异常再拒绝入列，选择是否重入列。

### 步骤
1. 创建监听的配置类
```java
@Configuration
public class MessageListenerConfig {
    @Resource
    private CachingConnectionFactory connectionFactory;

    @Resource
    private TestGetRabbitmqMessage testGetRabbitmqMessage;

    @Value("${rabbitmq.queen}")
    private String queen;

    @Bean
    public SimpleMessageListenerContainer simpleMessageListenerContainer() {
        // 创建一个 SimpleMessageListenerContainer 实例
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        // 设置并发消费者数量
        container.setConcurrentConsumers(1);
        // 设置最大并发消费者数量
        container.setMaxConcurrentConsumers(1);
        // 设置消息确认模式为手动确认
        container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        // 设置队列的名称(即监听哪个队列)
        container.setQueueNames(queen);
        // 设置消息监听器，用于处理接收到的消息
        container.setMessageListener(testGetRabbitmqMessage);
        return container;
    }
}

```
2. 手动确认消息
    * 这个监听的类要实现ChannelAwareMessageListener接口
    * 实现onMessage方法,里面有两个参数(Message,Channel),Message是消息主体,Channel是信道,用来给消息做确认或者拒绝的
    * deliveryTag这个是消息的标签message.getMessageProperties().getDeliveryTag()获取
    * basicAck(deliveryTag, true)第二个参数表示是否批量处理消息
    * basicReject(deliveryTag, false)拒绝消息,第二个参数表示是否重新入队
    * basicNack(deliveryTag, false,false)批量拒绝消息,第二个参数是表示是否重新入队,第三个参数是表示是否批量处理

```java
@Component
public class TestGetRabbitmqMessage implements ChannelAwareMessageListener {

    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        long deliveryTag = message.getMessageProperties().getDeliveryTag();

        try {
            byte[] body = message.getBody();
            String str = new String(body);

            Student student = JSON.parseObject(str, Student.class);

            System.out.println("接受到数据onMessage" + student);

            System.out.println("消息是来自" + message.getMessageProperties().getConsumerQueue());
            assert channel != null;
            channel.basicAck(deliveryTag, true);
        } catch (Exception e) {
            //根据标签来拒绝消息,第二个参数是是否重新进入队伍
            channel.basicReject(deliveryTag, false);
            throw new RuntimeException(e);
        }
    }
}

```




















































