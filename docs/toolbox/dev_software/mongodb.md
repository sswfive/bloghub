---
title: Mongodb的安装指南
date: 2022-07-28 23:15:08
tags:
  - SoftwareInstall
---
## 安装说明
- 基于Linux7.8安装
- 版本：社区版：mongodb-linux-x86_64-rhel70-5.0.9
- 数据目录：/usr/local/mongodb/data/
- 配置文件：/usr/local/mongodb/conf/mongodb.conf
- 日志文件:  /usr/local/mongodb/logs/mongodb.log

官网安装链接：https://www.mongodb.com/docs/manual/administration/install-on-linux/

## 安装指南
### 单机部署
1. 下载安装mongodb
>百度网盘下载地址：链接：https://pan.baidu.com/s/1QTrVmX8xSNHMaxyXwMiGZQ?pwd=fb9g 
>提取码：fb9g 
>--来自百度网盘超级会员V7的分享

```bash 
cd /data/software
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.9.tgz

mkdir -p /usr/local/mongodb
tar -zxvf mongodb-linux-x86_64-rhel70-5.0.9.tgz -C /usr/local/mongodb
```

2. 添加环境变量
```bash 
vim /etc/profile

export PATH=/usr/local/mongodb/bin:$PATH

source /etc/profile
```

3.创建数据、日志、配置文件目录
```bash
mkdir -p /usr/local/mongodb/data
mkdir -p /usr/local/mongodb/logs
mkdir -p /usr/local/mongodb/conf
```

4.编写配置文件
```bash
cd /usr/local/mongodb/conf
vim mongodb.conf

bind_ip = 192.168.146.38
port = 27017
#后台运行
fork = true
## 数据目录
dbpath=/usr/local/mongodb/data/mongodb
## 日志路径
logpath=/usr/local/mongodb/data/logs/mongod.log
## is log Rotate mongoDB 3.0
logappend = true
#logRotate = true
## 开启日志
journal = true
## 使用小文件进行开发
smallfiles = true
5. 启动测试
mongod --config /usr/local/mongodb/conf/mongodb.conf
# >>>logs
about to fork child process, waiting until server is ready for connections.
forked process: 17802
child process started successfully, parent exiting
6.查看进程
netstat -unltp | grep mongo
# 或
netstat -lanp | grep "27017"
```

### 扩展
#### 创建管理员
● mongo
```bash
-- 进入管理数据库
use admin;
-- 创建管理员, 设置验证
db.createUser({user:'admin',pwd:'123456', roles:[{role:'dbAdminAnyDatabase',db:'admin'},{role:'root',db:'admin'}]})
```
#### 创建非管理员
● mongo
```bash
-- 进入用户数据库
user test;
-- 创建用户
db.createUser(
    {
        user: "test",
        pwd: "123456",

        roles: [{ "role": "readWrite", "db": "test" }, { "role": "dbAdmin", "db": "test" }],
        "mechanisms": [
            "SCRAM-SHA-1",
            "SCRAM-SHA-256"
        ]
    }
)
```