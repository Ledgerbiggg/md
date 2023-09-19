# ngnix

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





































































































































