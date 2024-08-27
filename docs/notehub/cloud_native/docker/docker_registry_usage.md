---
title: docker-registry搭建与使用
date: 2022-04-10 19:24:19
tags: 
  - Docker
---
## Docker Hub
- 是docker 官方维护的一个公共仓库
- 当拉取镜像比较慢时，可以配置镜像加速器：[http://docker-cn.com/](http://docker-cn.com/)
### 注册

- 在 [https://cloud.docker.com](https://cloud.docker.com/) 免费注册一个 Docker 账号。
### 登录

- 通过执行`docker login`命令交互式的输入用户名及密码来完成在命令行界面登录 Docker Hub。
### 注销

- 你可以通过`docker logout`退出登录。 拉取镜像
### 拉取镜像 

- 通过`docker search`命令来查找官方仓库中的镜像
- `通过docker pull`命令来将它下载到本地。
```bash
$ docker search centos
NAME                                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                                          The official build of CentOS.                   465       [OK]
tianon/centos                                   CentOS 5 and 6, created using rinse instea...   28
blalor/centos                                   Bare-bones base CentOS 6.5 image                6                    [OK]
saltstack/centos-6-minimal                                                                      6                    [OK]
tutum/centos-6.4                                DEPRECATED. Use tutum/centos:6.4 instead. ...   5                    [OK]
```

- 官方的镜像说明是官方项目组创建和维护的，`automated`资源允许用户验证镜像的来源和内容。
- 根据是否是官方提供，可将镜像资源分为两类。
   - 一种是类似 centos 这样的镜像，被称为基础镜像或根镜像。这些基础镜像由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。
   - 还有一种类型，比如 tianon/centos 镜像，它是由 Docker 的用户创建并维护的，往往带有用户名称前缀。可以通过前缀`username/`来指定使用某个用户提供的镜像，比如 tianon 用户。
- 在查找的时候通过`--filter=stars=N`参数可以指定仅显示收藏数量为 N 以上的镜像。下载官方 centos 镜像到本地。
```bash
$ docker pull centos
Pulling repository centos
0b443ba03958: Download complete
539c0211cd76: Download complete
511136ea3c5a: Download complete
7064731afe90: Download complete
```
### 推送镜像

- 登录后通过`docker push`命令来将自己的镜像推送到 Docker Hub。
```bash
$ docker tag ubuntu:17.10 username/ubuntu:17.10
$ docker image ls
REPOSITORY                                               TAG                    IMAGE ID            CREATED             SIZE
ubuntu                                                   17.10                  275d79972a86        6 days ago          94.6MB
username/ubuntu                                          17.10                  275d79972a86        6 days ago          94.6MB
$ docker push username/ubuntu:17.10
$ docker search username
NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
username/ubuntu
```
## 私有仓库
### 搭建流程

- `docker-registry`是官方提供的工具，可以用于构建私有的镜像仓库。本文内容基于 docker-registry v2.x 版本。你可以通过获取官方 registry 镜像来运行。
```bash
$ docker run -d -p 30001:5000 --restart=always --name registry docker.io/registry
```

- 配置：
```bash
vim /etc/docker/daemon.json

{
    "registry-mirrors":["https://docker.mirrors.ustc.edu.cn"],
    "insecure-registries":["127.0.0.1:30001"]  # 服务器ip和端口
}
```

- 重启docker服务
```bash
systemctl restart docker
```

- 这将使用官方的`registry`镜像来启动私有仓库。默认情况下，仓库会被创建在容器的`/var/lib/registry`目录下。你可以通过 -v 参数来将镜像文件存放在本地的指定路径。例如下面的例子将上传的镜像放到本地的 /opt/data/registry 目录。
```bash
$ docker run -d -p 30001:5000 --restart=always -v /opt/data/registry:/var/lib/registry docker.io/registry
```

- 访问：[http://127.0.0.1:30001/v2/_catalog](http://127.0.0.1:30001/v2/_catalog)
```json
{
"repositories": [
]
}
```

- 登录
```bash
docker login -u 用户名 -p 密码
```

- 上传镜像到仓库
```bash
docker tag frontend_docker:v1 127.0.0.1:30001/frontend_docker:v1  # 打标签
docker push 127.0.0.1:30001/frontend_docker:v1  

{
"repositories": [
"frontend_docker"
]
}
```

- 拉取镜像
```bash
docker pull 127.0.0.1:30001/frontend_docker:v1
```
### 在私有仓库上传、搜索、下载镜像

- 创建好私有仓库之后，就可以使用`docker tag`来标记一个镜像，然后推送它到仓库。例如私有仓库地址为 127.0.0.1:5000。先在本机查看已有的镜像。
```
$ docker image ls
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

- 使用`docker tag`将 ubuntu:latest 这个镜像标记为 127.0.0.1:5000/ubuntu:latest。 
   - 格式为 docker tag IMAGE[:TAG] [REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]
```
$ docker tag ubuntu:latest 127.0.0.1:5000/ubuntu:latest
$ docker image ls
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
127.0.0.1:5000/ubuntu:latest      latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

- 使用`docker push`上传标记的镜像。
```
$ docker push 127.0.0.1:5000/ubuntu:latest
The push refers to repository [127.0.0.1:5000/ubuntu]
373a30c24545: Pushed
a9148f5200b0: Pushed
cdd3de0940ab: Pushedfc56279bbb33: Pushed
b38367233d37: Pushed
2aebd096e0e2: Pushed
latest: digest: sha256:fe4277621f10b5026266932ddf760f5a756d2facd505a94d2da12f4f52f71f5a size: 1568
```

- 用`curl`查看仓库中的镜像。
```
$ curl 127.0.0.1:5000/v2/_catalog
{"repositories":["ubuntu"]}
```

- 这里可以看到 {"repositories":["ubuntu"]}，表明镜像已经被成功上传了。
- 从私有仓库中下载这个镜像。
```
$ docker image rm 127.0.0.1:5000/ubuntu:latest
$ docker pull 127.0.0.1:5000/ubuntu:latest
Pulling repository 127.0.0.1:5000/ubuntu:latest
ba5877dc9bec: Download complete
511136ea3c5a: Download complete
9bad880da3d2: Download complete
25f11f5fb0cb: Download complete
ebc34468f71d: Download complete
2318d26665ef: Download complete
$ docker image ls
REPOSITORY                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
127.0.0.1:5000/ubuntu:latest       latest              ba5877dc9bec        6 weeks ago         192.7 MB
```
## 注意事项

- 如果你不想使用 127.0.0.1:5000 作为仓库地址，比如想让本网段的其他主机也能把镜像推送到私有仓库。你就得把例如 192.168.199.100:5000 这样的内网地址作为私有仓库地址，这时你会发现无法成功推送镜像。
- 这是因为 Docker 默认不允许非 HTTPS 方式推送镜像。我们可以通过 Docker 的配置选项来取消这个限制。
### Ubuntu 14.04, Debian 7 Wheezy

- 对于使用 upstart 的系统而言，编辑`/etc/default/docker`文件，在其中的`DOCKER_OPTS`中增加如下内容：
```
DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com --insecure-registries=192.168.199.100:5000"
```

- 重新启动服务:
```
$ sudo service docker restart
```
### Ubuntu 16.04+, Debian 8+, centos 7

- 对于使用 systemd 的系统，请在`/etc/docker/daemon.json`中写入如下内容（如果文件不存在请新建该文件）
```
{
  "registry-mirror": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [
    "192.168.199.100:5000"
  ]
}   
```
> 注意：该文件必须符合`json`规范，否则 Docker 将不能启动。

### 其他
对于 Docker for Windows、Docker for Mac 在设置中编辑`daemon.json`增加和上边一样的字符串即可。
