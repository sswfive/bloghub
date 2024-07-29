---
title: centos7安装git
date: 2022-05-03 06:52:01
tags: 
  - Git
---
Centos7中yum仓库的默认git版本比较低，需要先卸载了在安装新版本，具体步骤如下：
```bash
# 安装centos7 WANDisco仓库
yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm

# 安装Git
yum install git -y

git version
```