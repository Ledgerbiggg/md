# redis
* 创建卷
```sh
mkdir -p /data/redis/conf/
mkdir -p /data/redis/data/
```
* 修改文件权限为可编辑 表示所有文件
```sh
chmod -R 777 /data/redis/
```
* 修改配置文件
```sh
cd /data/redis/conf
```
* 启动
```sh
sudo docker run -p 6379:6379 --name redis --restart always -v /data/redis/conf:/etc/redis/redis.conf -v /data/redis/data:/data -d redis redis-server /etc/redis/redis.conf 
```
* ps:redis-server /etc/redis/redis.conf以配置文件启动redis，加载容器内的conf文件，最终找到的是挂载的目录/home/docker/redis/conf
# pgsql

* 挂卷
```sh
mkdir -p /data/postgres
```
* 启动
```sh
docker run --name postgres --restart always -e POSTGRES_PASSWORD=123456 -p 5432:5432 -v /data/postgres:/var/lib/postgresql/data -d postgres:12
```
ps:postgres版本较高低版本navicat链接默认数据库会出问题
# rabbitmq

* 挂卷
```sh
mkdir -p /data/rabbitmq/{data,conf,log}
```
* 设置权限
```sh
chmod -R 777 /data/rabbitmq
```
* 临时运行
```sh
docker run --privileged=true \
-d -p 5672:5672 -p 15672:15672 \
--name rabbitmq \
--restart=always --hostname=rabbitmqhost -e RABBITMQ_DEFAULT_VHOST=my_vhost \
-e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin \
rabbitmq:3.9.12-management
```
* 复制配置文件
```sh
docker cp rabbitmq:/etc/rabbitmq/ /data/rabbitmq/conf
```
* 删除容器
```sh
docker rm -f rabbitmq
```
* 运行
```sh
docker run --privileged=true \
-d -p 5672:5672 -p 15672:15672 \
--name rabbitmq -v /data/rabbitmq/data:/var/lib/rabbitmq -v /data/rabbitmq/conf:/etc/rabbitmq -v /data/rabbitmq/log:/var/log/rabbitmq \
--restart=always --hostname=rabbitmqhost -e RABBITMQ_DEFAULT_VHOST=my_vhost -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin \
rabbitmq:3.9.12-management
```
* 进入容器
```sh
docker exec -it rabbitmq bash
```
* 启动插件
```sh
rabbitmq-plugins enable rabbitmq_management
```
ps:可以使用admin和admin来访问
# nacos
* 挂卷
```sh
mkdir -p /data/nacos/{logs,init.d}
```
* 修改权限
```sh
chmod -R /data/nacos/
```
* 配置文件
```sh
vim /data/nacos/init.d/custom.properties      
```
* 写入
```sh
server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848
 
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://xx.xx.xx.x:3306/nacos-config? characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true #这里需要修改端口
db.user=root #用户名
db.password=root #密码
 
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false
management.metrics.export.elastic.enabled=false
management.metrics.export.influx.enabled=false
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i
nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
nacos.naming.distro.taskDispatchThreadCount=1
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.naming.expireInstance=true
```

* 运行
```sh
docker  run \
--name nacos -d \
-p 8848:8848 \
--privileged=true \
--restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v /data/nacos/logs:/home/nacos/logs \
-v /data/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties \
nacos/nacos-server
```
# 内网穿透
## 服务端
* 配置文件
```sh
mkdir -p /data/frp/ && cat >>  /data/frp/frps.ini 
```
* 配置
```sh
vim /data/frp/frps.ini
```
```ini
[common]
bind_port = 7000									
# 启用面板
dashboard_port = 7500
# 面板登录名和密码
dashboard_user = user								
dashboard_pwd = password
# 使用http代理并使用8888端口进行穿透
vhost_http_port = 8888
# 使用https代理并使用9999端口进行穿透
vhost_https_port = 9999
# 日志路径
log_file = ./frps.log
# 日志级别
log_level = info
# 日志最大保存天数
log_max_days = 2
# 认证超时时间
authentication_timeout = 900
# 认证token，客户端需要和此对应
token = ledgerhhh 	
```
* 运行
```sh
docker run  -d \
--restart=always \
--network host \
-v /data/frp/frps.ini:/data/frp/frps.ini \
--name frps snowdreamtech/frps
```								
## 客户端
* 配置文件
```sh
mkdir -p /data/frp/ && cat >>  /data/frp/frpc.ini 
```
* 配置
```sh
vim /data/frp/frpc.ini
```
* 配置
```ini
[common]
server_addr = frps服务器的IP地址
server_port = frps服务器的端口
token = frps服务器的token

[ssh]
type = 协议
local_ip = 127.0.0.1				## 可配置本机ip或者配置其他主机上的ip
local_port = 22						## 可配置本机端口或者配置其他主机上的端口
remote_port = 8000					## 从本地端口映射到frps服务器上的端口
```
* 运行
```sh
docker run  -d \
--restart=always \
--network host \
-v /data/frp/frpc.ini:/etc/frp/frpc.ini \
--name frpc snowdreamtech/frpc
```

























