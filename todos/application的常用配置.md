## application的常用的配置
```yml
spring:
  datasource:
    driver-class-name: org.postgresql.Driver  # 数据库驱动类名
    url: jdbc:postgresql://localhost:5432/excel_for_english  # 数据库连接 URL
    username: postgres  # 数据库用户名
    password: 123456  # 数据库密码
    hikari:
      maximum-pool-size: 20  # 连接池最大连接数
      minimum-idle: 5  # 连接池最小空闲连接数
      connection-timeout: 30000  # 连接超时时间，30秒
      idle-timeout: 600000  # 空闲连接超时时间，10分钟
      max-lifetime: 1800000  # 连接最大生存时间，30分钟

redis:
  host: localhost  # Redis 服务器主机名
  port: 6379       # Redis 服务器端口号
  password: 123456 # Redis 访问密码
  database: 0      # Redis 数据库编号

server:
  port: 9999       # Spring Boot 应用服务器端口号

mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    # 配置 MyBatis-Plus 的日志实现为 org.apache.ibatis.logging.stdout.StdOutImpl
  type-aliases-package: com.ledger.es_test1.domain
    # 指定 MyBatis 的类型别名包，用于自动注册别名
  mapper-locations: classpath*:mapper/*.xml
    # 指定 MyBatis Mapper XML 文件的位置模式，用于映射 SQL 语句和 Java 接口

common:
  path: "D:\\360TEST\\"
  fileName: "ledger.xlsx"
  sheetName: "EnglishWord"
```



