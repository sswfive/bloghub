---
title: Docker安装常用软件汇总
date: 2022-04-02 10:41:59
tags: 
  - Docker
---
## docker-mysql-安装
- 安装指令和指令参数说明
```shell
docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

-p 3306:3306  : 将容器的3306端口映射到主机的3306端口

-v  -v $PWD/logs:/logs  : 将主机当前目录下的conf/my.cnf 挂载到容器的/etc/mysql/my.cnf

-v  -v $PWD/data:/var/lib/mysql   : 将主机当前目录下的data目录挂载到容器的/var/lib/mysql

-e  -e MYSQL_ROOT_PASSWORD=123456  : 初始化root用户的密码
```

- 进入容器配置
```shell
docker exec -it 容器id /bin/bash

# 进入MySQL
mysql -uroot -p123456;

#　建立用户并授权
GRANT ALL PRIVILEGES ON  *.* TO 'root'@'%' IDENTIFIED BY 'root123' WITH GRANT OPTION;

flush privileges;
```

---

- https://github.com/Ryan-Miao/docker-yapi
