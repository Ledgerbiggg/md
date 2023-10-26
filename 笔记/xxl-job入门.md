## XXL-Job快速入门+详细教程
* 1 概念
```html
XXL-JOB是一个轻量级分布式任务调度平台
详细说明：XXL-JOB是一个任务调度框架，通过引入XXL-JOB相关的依赖，按照相关格式撰写代码后，可在其可视化界面进行任务的启动，执行，中止以及包含了日志记录与查询和任务状态监控
如果将XXL-JOB形容为一个人的话，每一个引入xxl-job的微服务就相当于一个独立的人(执行器)，而按照相关约定格式撰写的Handler为餐桌上的食物，可视化界面则可以决定哪个执行器(人)，吃东西或者不吃某个东西(定时任务)，在什么时间吃(Corn表达式控制或者执行或终止或者；立即开始)；
```

### 开整!!!!
* 1. 引入依赖
```xml
        <dependency>
            <groupId>com.xuxueli</groupId>
            <artifactId>xxl-job-core</artifactId>
            <version>2.3.0</version> <!-- 根据实际版本号填写 -->
        </dependency>
```
* 2. 配置XXL-Job Executor























































