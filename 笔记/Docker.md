# docker
## Docker 是一个开源的应用容器引擎
* Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
* 容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。
## Docker的应用场景
* Web 应用的自动化打包和发布。
* 自动化测试和持续集成、发布。
* 在服务型环境中部署和调整数据库或其他的后台应用。
* 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。
## Docker 的优点
* Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

1. 快速，一致地交付您的应用程序
2. 响应式部署和扩展
3. 在同一硬件上运行更多工作负载

## Docker Hello World
```sh
runoob@runoob:~$ docker run ubuntu:15.10 /bin/echo "Hello world"
Hello world
```
* 各个参数解析：
    * docker: Docker 的二进制执行文件。

    * run: 与前面的 docker 组合来运行一个容器。

    * ubuntu:15.10 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。

    * /bin/echo "Hello world": 在启动的容器里执行的命令
## 常用命令
1. 拉取镜像
* 这将从 Docker 镜像仓库中拉取指定名称和标签的镜像到本地。
```sh
docker pull <image_name>:<tag>
```
2. 运行容器
* -d：在后台运行容器；-p：端口映射；--name：指定容器名称。
```sh
docker run -d -p <host_port>:<container_port> --name <container_name> <image_name>
```
3. 执行命令
* -it：以交互模式运行一个新命令。   
```sh
docker exec -it <container_name> <command>
```
4. 停止容器
```sh
docker stop <container_name>
```
5. 启动容器
```sh
docker start <container_name>
```
6. 重启容器
```sh
docker restart <container_name>
```
7. 查看运行中容器
```sh
docker ps
```
8. 查看所有容器（包括停止的）
```sh
docker ps -a
```
9. 删除容器
```sh
docker rm <container_name>
```
10. 查看镜像
```sh
docker images
```
11. 删除镜像
```sh
docker rmi <image_name>:<tag>
```
12. 挂载卷
* 这将把主机文件系统上的路径挂载到容器内部，实现持久化存储。
```sh
docker run -v <host_path>:<container_path> <image_name>
```
13. 查看容器日志
```sh
docker logs <container_name>
```
14. 后台运行并进入容器
```sh
docker cp <container_name>:<container_path> <host_path>
```
15. 从容器复制文件到主机
```sh
docker cp <container_name>:<container_path> <host_path>
```
16. 构建镜像
```sh
docker build -t <image_name> <path_to_dockerfile>
```
17. 查看容器内进程
```sh
docker top <container_name>
```
18. 查看容器详细信息
```sh
docker inspect <container_name>
```
19. 暂停容器
```sh
docker pause <container_name>
```
20. 恢复容器
```sh
docker unpause <container_name>
```
21. 设置环境变量
```sh
docker run -e <VAR_NAME>=<value> <image_name>
```
22. 进入容器交互模式
```sh
docker exec -it <container_name> /bin/bash
```
23. 导出容器为镜像
```sh
docker exec -it <container_name> /bin/bash
```
24. 查看容器使用的资源情况
##  docker网络  
1. docker容器在默认情况下,一般会分配一个独立的network-namespace,也就是网络类型中的Bridge模式.
* 在使用Bridge时就涉及到了一个问题,既然它有独立的namesapce,这就需要一种技术使容器内的端口可以在主机上访问到,这种技术就是端口映射,docker可以指定你想把容器内的某一个端口可以在容器所在主机上的某一个端口它俩之间做一个映射,当你在访问主机上的端口时,其实就是访问容器里面的端口.

##  docker部署第一个后台的应用程序
1. 构建镜像(jar包)
*  后台应用打包好jar包

2. 编写Dockerfile
* touch Dockerfile

```sh
# 使用一个基础的 Java 镜像
FROM openjdk:11-jre-slim

# 设置工作目录
WORKDIR /app
# 复制 JAR 文件到容器中 
COPY app.jar /app/app.jar 
# 复制 配置文件到容器中
COPY application.yml /app/application.yml
# 设置工作目录 
WORKDIR /app 
# 启动 Java 应用程序
CMD ["bash", "-c", "java -jar app.jar"] 
```
3. 构建容器
```sh
docker build -t app:1.0 .
```
5. 运行容器
```sh
docker run -d --name app -p 9999:9999 app:1.0
```











































