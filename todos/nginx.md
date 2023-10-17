# ngnix

## 正向代理与反向代理概述

* 正向代理

    * 正向代理代理的是客户端，需要在客户端配置，我们访问的还是真实的服务器地址

* 反向代理
    * 反向代理代理的是服务器端，客户端不需要任何配置，客户端只需要将请求发送给反向代理服务器即可，代理服务器将请求分发给真实的服务器，获取数据后将数据转发给你。隐藏了真实服务器，有点像网关。
### 区别与总结
* 最根本的区别是代理的对象不同
    * 正向代理代理的是客户端，需要为每一个客户端都做一个代理服务器，客户端访问的路径是目标服务器
    * 反向代理代理的是真实服务器，客户端不需要做任何的配置，访问的路径是代理服务器，由代理服务器将请求转发到真实服务器


## 常用命令
```sh
# 1、启动nginx
./nginx
# 2、关闭nginx
./nginx -s stop
# 3、重新加载nginx (nginx.conf)
./nginx -s reload
# 4、查看版本号
./nginx -v
```
## 配置文件
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
* Nginx配置文件(nginx.conf)
    * 默认在Linux上安装的Nginx，配置文件在安装的nginx目录下的conf目录下，名字叫做nginx.conf

* nginx.conf主要由三部分组成
    * 全局块，
    * events块
    * http块
![](http://c2cpicdw.qpic.cn/offpic_new/228664584//228664584-1432737184-B43E3E22A8B612629842D97DF745E43F/0?term=2&is_origin=0)

## 全局块
* 就是配置文件从头开始到events块之间的内容，主要设置的是影响nginx服务器整体运行的配置指令比如worker_process, 值越大，可以支持的并发处理量也越多，但是还是和服务器的硬件相关

## 3events块
* events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

上述例子就表示每个 work process 支持的最大连接数为 1024.
这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置

## http块
* 包括http全局块，以及多个server块

## http全局块
* http 全局块配置的指令包括文件引入、 MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

## server全局块
* 最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。
```conf
  #这一行表示这个server块监听的端口是80，只要有请求访问了80端口，此server块就处理请求
  listen       80;
  #  表示这个server块代表的虚拟主机的名字
  server_name  localhost;
```
## location块
* 一个 server 块可以配置多个 location 块。
主要作用是根据请求地址路径的匹配，匹配成功进行特定的处理
这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

```conf
# 表示如果请求路径是/就是用这个location块进行处理
location / {
            root   html;
            index  index.html index.htm;
        }
```


















































































































