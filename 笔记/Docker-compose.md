# Docker-compose

## Compose介绍
* Docker Compose是一个用来定义和运行复杂应用的Docker工具。一个使用Docker容器的应用，通常由多个容器组成。使用Docker Compose不再需要使用shell脚本来启动容器。 
Compose 通过一个配置文件来管理多个Docker容器，在配置文件中，所有的容器通过services来定义，然后使用docker-compose脚本来启动，停止和重启应用，和应用中的服务以及所有依赖服务的容器，非常适合组合使用多个容器进行开发的场景。
## Compose和Docker兼容性

compose文件格式版本 | 	docker版本
--- | ---
3.4   |	17.09.0+
3.3   |	17.06.0+
3.2   |	17.04.0+
3.1   |	1.13.1+
3.0   |	1.13.0+
2.3   |	17.06.0+
2.2   |	1.13.0+
2.1   |	1.12.0+
2.0   |	1.10.0+
1.0   |	1.9.1.+

## 安装docker-compose
* 使用github下载
1.  下载Docker Compose：
```sh
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```
2. 赋予执行权限
```sh
sudo chmod +x /usr/local/bin/docker-compose
```
3. 赋予执行权限
```sh
docker-compose --version
```
## docker-compose.yml文件
### 定义
* 是一个用于定义多容器应用配置的YAML文件。它允许你在单个文件中定义多个Docker容器、网络设置、卷映射以及其他相关配置，以便通过一组命令来管理这些容器的创建、启动、停止和销毁。

1. 定义多个服务：在一个文件中定义应用程序的多个服务（容器），并指定它们之间的关系和依赖。每个服务都可以基于不同的镜像来创建。

2. 配置容器参数：指定容器运行时的参数，例如映射的端口、环境变量、命令等。

3. 创建网络：Docker Compose会自动创建一个网络，允许定义的服务之间进行通信。

4. 管理卷：可以指定卷映射，将主机的目录或文件映射到容器内部，实现数据持久化。

5. 一键管理：通过一组简单的命令（如docker-compose up、docker-compose down等），可以一次性启动、停止、销毁整个应用程序的多个容器

* 这个文件的结构由YAML格式定义，提供了一种清晰且易于维护的方式来描述多容器应用的配置。通过将所有配置集中在一个文件中，简化了部署和管理多个容器的过程，使得协作和开发更加方便。

* 总之，docker-compose.yml 是用于定义多容器应用程序的配置文件，它简化了多容器应用的部署和管理流程。

## docker-compose使用示例

### 定义应用
1. 目录结构
```sh
your-compose-app/
├── dist/ # 前端文件夹(vue打包文件)
├── app.jar # 后端应用(java后端程序)
├── application.yml # 后端配置文件(主机配置)
├── Dockerfile # Docker打包镜像文件(打包镜像的配置文件)
├── docker-compose.yml # docker-compose批量启动文件(批量启动配置)
├── nginx.conf # nginx配置文件(挂载用)
```
2. 编写nginx.conf配置文件
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
3. 编写Dockerfile文件
```sh
# 使用一个基础的 Java 镜像
FROM openjdk:11-jre-slim

# 设置工作目录
WORKDIR /app

# 复制 JAR 文件到容器中
COPY english.jar /app/english.jar
COPY application.yml /app/application.yml

# 定义启动命令
CMD ["java", "-jar", "english.jar"]

```
2. 编写docker-compose.yml(只使用了ngnix和jar镜像的构建)
```yml
version: '3'
services:
  # 启动nginx的镜像 docker run -p 80:80 \
  # -v ./nginx.conf:/etc/nginx/nginx.conf \
  # -v ./dist:/usr/share/nginx/html \
  # --name compose-nginx-1 nginx
  nginx:
    # 镜像名称
    image: nginx
    # 端口映射
    ports:
      - "80:80"
    # 挂载卷
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro  # 映射 NGINX 配置文件(ro表示容器内部只读,保证数据的安全性)
      - ./dist:/usr/share/nginx/html:ro # 映射 静态资源类dist 
    container_name: nginx

  # 打包java的镜像配置 docker build -t java-app .
  # 运行镜像的配置 docker run -p 9999:9999 \ 
  # -v ./application.yml:/app/application.yml \
  # --name compose-java-app-1 english
  java-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "9999:9999"
    volumes:
      - ./application.yml:/app/application.yml:ro  # 映射 Java 应用程序配置文件
    container_name: english
```
3. 启动
```sh
docker-compose up -d
```
4. 关闭
```sh
docker-compose down
```
































