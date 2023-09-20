## docker配置常用镜像

## Elasticsearch & ik分词器  &  Kibana

* **Elasticsearch:**
  Elasticsearch 是一个分布式的搜索和分析引擎，用于存储、搜索和分析大量数据。

* **IK 分词器:**
  IK 分词器（IK Analyzer）是 Elasticsearch 中用于中文分词的一种分析器。

* **Kibana:**
  Kibana 是 Elasticsearch 的可视化工具

### Elasticsearch

1. 安装镜像

```sh
docker pull elasticsearch:7.7.1
```

2. 创建挂载目录

```sh
# 配置文件
mkdir -p /data/elasticsearch/conf
# 数据文件
mkdir -p /data/elasticsearch/data
# 插件
mkdir -p /data/elasticsearch/plugins
# 将http.host: 0.0.0.0写入elasticsearch.yml，允许通过网络中的任何计算机访问 Elasticsearch 的 HTTP 接口
echo "http.host: 0.0.0.0" >> /data/elasticsearch/conf/elasticsearch.yml
```

3. 文件夹赋权

```sh
# -R表示递归赋权
chmod -R 777 /data/elasticsearch/
```

4. 运行

* 9200 端口是 Elasticsearch 的 HTTP REST API 默认监听端口；9300 端口是 Elasticsearch 的节点间通信端口
* 命令行参数，单机版运行
* 最小堆内存和最大堆内存

```sh
docker run --name elasticsearch --restart=always \
-p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms256m -Xmx256m" \
-v /data/elasticsearch/conf/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.7.1
```

### ik分词器

* [github地址]: https://github.com/medcl/elasticsearch-analysis-ik/releases?q=7.&amp;expanded=true

* tip：
1. 版本一定要和es的一致
2. 下载zip的文件,自己电脑上面解压,再上传到服务器

* 自定义分词器

```sh
 vi /data/elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml
```

```sh
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--
        	用户可以在这里配置远程扩展字典 
        	http://192.168.56.10/es/fenci.txt 这里为nginx对应的路径
        -->
        <entry key="remote_ext_dict">http://192.168.56.10/es/fenci.txt</entry>
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

### Kibana

1. 安装镜像

```sh’
docker pull kibana:7.4.2
```

2. 运行

```sh
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://106.54.9.19:9200 -p 5601:5601 -d kibana:7.4.2
```

## mysql

* 拉取

```sh
docker pull mysql
```

* 运行

```sh
docker run -d --restart=always --name mysql -p 13306:3306 -e TZ=Asia/Shanghai -e MYSQL_ROOT_PASSWORD=123456 mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci

```

* 创建文件夹

```sh
mkdir -p /data/mysql/var/lib/
mkdir -p /data/mysql/etc/
mkdir -p /data/mysql/var/log/

```

* 复制卷（一个一个来）

```sh
docker cp mysql:/var/lib/mysql /data/mysql/var/lib/
docker cp mysql:/etc/mysql /data/mysql/etc/
docker cp mysql:/var/log  /data/mysql/var/log/

```

* 关闭容器

```sh
docker rm -f mysql

```

* 运行容器

```sh
docker run  -p 13306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-e TZ=Asia/Shanghai \
-v /data/mysql/data:/var/lib/mysql:rw  \
-v /data/mysql/log:/var/log/mysql:rw  \
-v /data/mysql/conf/my.cnf:/etc/mysql/my.cnf:rw  \
--name mysql \
--restart=always \
-d mysql

```

* 进入

```sh
docker exec -it mysql bash
```

* 操作

```sh
mysql -u root -p123456
```

* 改密码

```sh
alter user 'root'@'%' identified with mysql_native_password by 'root';
flush privileges;
```

* exit

```sh
exit
```

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

# ngnix
1. 拉区镜像
```sh
docker pull nginx
```
2. 编写ngnix.config
```sh
mkdir -p /data/nginx
touch nginx.conf
```
* 内容
```conf
# 使用 "user" 指令来指定运行 Nginx 的用户，通常为 "nginx" 用户。
user nginx;

# "worker_processes" 指定了 Nginx 启动的工作进程数，"auto" 表示根据可用 CPU 核心数自动设置。
worker_processes auto;

# 定义错误日志的位置和文件名。
error_log /var/log/nginx/error.log;

# 配置事件模块，用于控制 Nginx 的事件处理。
events {
    # "worker_connections" 指定每个工作进程的最大并发连接数。
    worker_connections 1024;
}

# 配置 HTTP 模块，包含所有关于 HTTP 服务的设置。
http {
    # 包含 MIME 类型的配置文件，用于定义文件类型和对应的 MIME 类型。
    include /etc/nginx/mime.types;
    
    # 默认 MIME 类型，当无法确定文件类型时会使用该类型。
    default_type application/octet-stream;

    # 定义一个 HTTP 服务器块。
    server {
        # 监听端口为 80，用于处理传入的 HTTP 请求。
        listen 80;
        
        # 定义服务器名，通常是域名，可以是多个域名，用空格分隔。
        server_name localhost;

        # 配置请求的处理逻辑。
        location / {
            # 指定根目录，用于查找请求的资源文件。
            root /usr/share/nginx/html;
            
            # 默认索引文件名。
            index index.html;
        }
    }
}
```
3. 文件权限设置
```sh
chmod 777 -R /home/ubuntu/font/dist/
```
4. 启动并挂卷
```sh
docker run -d -p 80:80 --name nginx \
-v /home/ubuntu/font/dist/:/usr/share/nginx/html \
-v /data/nginx/nginx.conf:/etc/nginx/nginx.conf \
nginx
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
-v /data/frp/frps.ini:/etc/frp/frps.ini \
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

























