---
title: Centos的yum源误删解决方法
date: 2022-07-06 23:54:37
tags: 
  - 运维手记
---
## 事出有因

- 更换yum源，手误yum源误删

## Bug现场

```
(base) [root@automlgpu4 bin]# yum 
-bash: /usr/bin/yum: No such file or directory
```

## **迎刃而解**

```
# step1: 查看centos版本信息
$ rpm -q centos-release
[output]: centos-release-7-9.2009.1.el7.centos.x86_64

# step2 找到对应的源信息，链接如下：
http://mirrors.163.com/

# step3:执行下面命令
rpm -ivh --nodeps http://mirrors.163.com/centos/7.9.2009/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
rpm -ivh --nodeps http://mirrors.163.com/centos/7.9.2009/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm
rpm -ivh --nodeps http://mirrors.163.com/centos/7.9.2009/os/x86_64/Packages/yum-3.4.3-168.el7.centos.noarch.rpm

# step4
cd /etc/yum.repo.d/
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
yum repolist
yum clean all
yum makecache
```

**记录时间：2021年6月29日17:23:22**
