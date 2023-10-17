# git版本控制(本地有文件,远程仓库有文件)
1. 初始本地仓库
```sh
git init
```
2. 添加文件
```sh
git add .
```
3. 提交到本地仓库
```sh
    git commit -m "init project"
```
4. 绑定远程分支
```sh
git remote add origin https://github.com/Ledgerhhhh/AlgorithmNotes.git
```
5. 重命名当前分支
```sh
git branch -m master main
```
6. 当前分支有文件,远程分支也有文件,就这样合并
```sh
git pull origin main --allow-unrelated-histories
```
7. 设置分支的对应关系
```sh
git branch --set-upstream-to=origin/main main
```
8. 拉取
```sh
git pull
```
9. 推送
```sh
git push
```









