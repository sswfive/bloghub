---
title: gitflow的使用
date: 2023-04-15 19:53:38
tags: 
  - Git
  - Tools
---
[参考链接：git-flow 备忘录](https://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)
## 安装
```bash
# Mac install
brew install git-flow-avh

# Linux  install
apt-get install git-flow  # ubuntu
yum install git-flow      # Centos

# window install
wget -q -O - --no-check-certificate https://raw.github.com/petervanderdoes/gitflow-avh/develop/contrib/gitflow-installer.sh install stable | bash
```

## 常用命令
```bash
# 初始化(前提： 在一个现有的git库内)
git flow init 

# 新增一个分支(基于develop分支创建)
git flow feature start myfeature

# 完成一个分支特性（合并到devlop分支上,并删除分支myfeature）
git flow feature finish myfeature 

# 发布一个新分支到服务器上(在此功能特性没完成时，将本地分支推送到远端仓库)
git push --set-upstream orgin myfeature
git add . && git commit -m "feat: xxxxx"
git flow feature publish myfeature

# 拉取一个发布的新分支特性（在协同时，一般用到）
git flow feature pull origin myfeature
或
git flow feature track myfeature  # 根据在origin上的新特性分支
```
