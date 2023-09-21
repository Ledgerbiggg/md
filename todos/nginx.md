# ngnix

## 正向代理与反向代理概述

* 正向代理    
    * 正向代理代理的是客户端，需要在客户端配置，我们访问的还是真实的服务器地址

* 反向代理
    * 反向代理代理的是服务器端，客户端不需要任何配置，客户端只需要将请求发送给反向代理服务器即可，代理服务器将请求分发给真实的服务器，获取数据后将数据转发给你。隐藏了真实服务器，有点像网关。

* 区别与总结
    * 正向代理代理的是客户端，需要为每一个客户端都做一个代理服务器，客户端访问的路径是目标服务器
反向代理代理的是真实服务器，客户端不需要做任何的配置，访问的路径是代理服务器，由代理服务器将请求转发到真实服务器


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





































































































































