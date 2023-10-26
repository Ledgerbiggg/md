## Jenkins

### docker运行jenkins(一定是lts,不然得后悔)
1. 挂卷
```sh
mkdir -p /data/jenkins_home
```
2. 文件夹给权限
```sh
chomd -R 777 /data/jenkins_home
```
3. 将maven下载并解压到这个/data/jenkins_home

4. 运行镜像
```sh
docker run -d \
-p 8080:8080 \
-v /data/jenkins_home:/var/jenkins_home \
--name jenkins \
jenkins/jenkins:lts

```
4. 访问ip:8080
5. docker logs jenkins 获取密码
6. 安装推荐插件(第一个)
7. 等待一会
8. plugins下载maven的插件

### 运行项目
1. 修改maven的配置路径(因为已经挂载了,可以直接写/var/jenkins_home/mavenxxx版本)
2. 添加maven(ADDitem)
3. 添加git的仓库地址
4. 修改主分支是*/mian
5. 运行项目

### 将打包的java包转移到服务器上面
1. 下载插件(publish over ssh)
2. manage/configure 去配置ssh连接的目标
* Name (随便取名字)
* HostName (主机地址)
* Username (root)
* Remote Directory (/)
3. 点开高级
* Use password authentication, or use a different key(勾选)
* 点击Test Configuration如果成功就表示可以连接
4. 构建后操作可以下拉选择服务器了
* SSH Server填写主机名称
* Transfers填写转移
    * Source files 上传的jar包(**/**.jar)
    * Remove prefix 文件上传会给一个默认的/target前缀,使用这个选项消除 (/target)
    * Remote directory 文件夹的名字
    
 

















