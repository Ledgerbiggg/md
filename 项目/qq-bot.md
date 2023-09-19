1. 启动mcl
* 关闭mcl
```sh
ps aux | grep mcl
```
* 进入目录
```sh
cd /root/mirai/root/mirai/mirai-console-loader-2.1.2/
```
* 后台启动mcl
```sh
nohup ./mcl > output.log 2>&1 &
```

2. 启动java程序


* 关闭bot
```sh
ps aux | grep qq-1.0.jar
```
* 进入jar包文件
```sh
cd mirai-console-loader-2.1.2/
```
* 后台启动
```sh
nohup  java -jar qq-1.0.jar > msg.log 2>&1 &
```











