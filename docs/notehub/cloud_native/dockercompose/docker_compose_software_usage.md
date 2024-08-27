---
title: docker-compose常用软件安装汇总
date: 2022-04-25 20:46:32
tags: 
  - DockerCompose
  - Docker
---
## 安装YApi

- 方式一：

```yaml
version: '3'

services:
  yapi-web:
    image: liuqingzheng/yapi:latest
    container_name: yapi-web
    ports:
      - 3000:3000
    environment:
      - YAPI_ADMIN_ACCOUNT=466640681@qq.com
      - YAPI_ADMIN_PASSWORD=Aa123456!
      - YAPI_CLOSE_REGISTER=true
      - YAPI_DB_SERVERNAME=yapi-mongo
      - YAPI_DB_PORT=27017
      - YAPI_DB_DATABASE=yapi
      - YAPI_MAIL_ENABLE=false
      - YAPI_LDAP_LOGIN_ENABLE=false
      - YAPI_PLUGINS=[]
    depends_on:
      - yapi-mongo
    links:
      - yapi-mongo
    restart: unless-stopped
  yapi-mongo:
    image: mongo:latest
    container_name: yapi-mongo
    volumes:
      - ./data/db:/data/db
    expose:
      - 27017
    restart: unless-stopped
```

```yaml
docker-compose up -d # 启动
docker-compose stop  # 停止
docker-compose rm    # 删除

http://127.0.0.1:3000/
输入邮箱：466640681@qq.com
输入密码：Aa123456!
```

- 方式二：
  - 参考链接：[https://github.com/Ryan-Miao/docker-yapi]

---

