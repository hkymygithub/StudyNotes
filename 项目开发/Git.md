# Git

### 配置
#### 显示当前的 git 配置信息
```
git config --list
```
#### 编辑 git 配置文件
```
# 针对当前仓库
git config -e
# 针对系统所有仓库
git config -e --global
```
#### 设置提交代码时的用户信息
```
git config --global user.name [userName]
git config --global user.email [email]
```
### 生成ssh密钥
```
ssh-keygen -t rsa -C [email]
```
### 初始化仓库
```
# 当前仓库
git init
# 指定仓库
git init [dirName]
```
### 拷贝远程仓库
```
git clone [url]
```
### 添加文件到暂存区
```
# 添加某个文件
git add [file_1] [file_2] ...
# 添加指定目录
git add [dir]
# 添加所有文件
git add .
```
### 查看仓库当前的状态
```
git status
```
### 比较暂存区和工作区的差异
```
# 尚未缓存的改动
git diff
# 查看已缓存的改动
git diff --cached
# 查看已缓存的与未缓存的所有改动
git diff HEAD
# 显示修改摘要
git diff --stat
```