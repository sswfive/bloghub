---
title: Docker常用命令速查
date: 2022-04-04 23:10:59
tags: 
  - Docker
---

> docker 目录：
> - /etc/docker/   -----------> docker的认证目录
> - /var/lib/docker/ ----------> docker 的应用安装目录


## 镜像管理命令
### 搜索镜像
```bash
docker search [iamge_name]
# eg:
docker search centos   
```
### 获取镜像
```bash
docker pull [iamge_name]
# eg:
docker pull centos   

# 备注：获取的镜像存在在/var/lib/docekr 目录里
```
### 查看镜像
```bash
# 查看指定镜像
docker images <image_name>

# 查看所有镜像
docker images
docker image ls

# 查看所有本地的iamges,包括已删除的镜像记录
docker images -a
```
### 查看镜像历史
```bash
docker history [image_name]
# 可以查看此镜像默认启动了哪些命令，或者封账了哪些系统层
```
### 重命名镜像
```bash
docker tag [old_image]:[old_version] [new_image]:[new_version]
# eg:
docker tag nginx:latest sswang-nginx:v1.0
```
### 删除镜像
```bash
docker rmi [image_id/image_name:image_version]
# eg:
docker rmi 3fa822599e10 

# 如果一个image_id存在多个名称，那么应该使用name:tag的格式删除镜像清除状态为dangling的镜像
docker image prune 

# 移除所有未被使用的镜像
docker image prune -a

# 删除部分镜像
docker image prune -a --filter "until=24h"

# 删除全部的images，
docker rmi $(docker images -q)
docker rmi -f $(docker images -q)  # 强制删除加-f

# 删除所有悬挂状态的镜像
docker image rm $(docker image ls -f dangling=true -q)
docker image 

# 删除untagged images，也就是那些id为的image的话可以用
docker rmi $(docker images | grep "^" | awk "{print $3}") 

# 删除所有不使用的镜像
docker image prune --force --all或者docker image prune -f -a
```
### 定制镜像

- 当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个`docker commit`命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。
```bash
docker run --name webserver -d -p 80:80 nginx  # 运行一个容器

docker commit \
 --author "wss" \
 --message "修改了内容" \
 webserver \
 nginx:v2 
 
 # webserver 启动的容器
```

- 可以使用docker diff 查看修改的内容
### 导入导出镜像方式一：
```bash
docker save -o [包文件] [镜像]   # docker save 会保存镜像的所有历史记录和元数据信息
docker save [镜像 1] ... [镜像 n] > [包文件]
# eg:
docker save -o nginx.tar sswang-nginx

docker load < [image.tar_name]  # docker load 不能指定镜像的名称
docker load --input [image.tar_name]
# eg:
docker load < nginx.tar
```

### 导入导出镜像方式二
```bash
# cat 镜像压缩包 | docker import 镜像名：版本   
cat op_docker_img.tar | docker import - op_docker:v1

# 镜像导入到本地  
docker export 镜像id > 本地保存路径.tar
 
# 本地导入镜像
docker import - 镜像名 < 本地保存路径.tar
```

上传镜像到仓库
```bash
docker login   
docker pushdocker push 镜像名：tags
```

---

## 容器管理命令

### 查看容器
```bash
docker ps  # ps 显示正在运行的容器
docker ps -a  # 显示所有运行过的容器，包括已经不运行的容器
docker container ls  # 查看正在运行的容器
docker ps -aq  # 列出所有的容器id
```
### 启动（运行）容器
```bash
docker run <参数，可选> [docker_image] [执行的命令]
docker run -d nginx  # -d 表示以守护进程运行
-i 表示以“交互模式”运行容器
-t 表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即 分配一个伪终端。
--name 为创建的容器命名
-v 表示目录映射关系(前者是宿主机目录，后者是映射到宿主机上的目录，即 宿主机目录:容器中目录)，可以使 用多个-v 做多个目录或文件映射。注意:最好做目录映射，在宿主机上做修改，然后 共享到容器上。
-d 在run后面加上-d参数,则会创建一个守护式容器在后台运行(这样创建容器后不 会自动登录容器，如果只加-i -t 两个参数，创建后就会自动进去容器)。
-p 表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p 做多个端口映射
-e 为容器设置环境变量
--network=host 表示将主机的网络环境映射到容器中，容器的网络与主机相同



# 启动已经终止的容器
docker start [container_id]

```
### 关闭容器       
```bash
docker stop [container_id]
# eg:
docker stop 8005c40a1d16

docker stop $(docker ps -aq)   # 停止所有的container
```

### 删除容器
```bash
# 正常删除容器
docker rm [container_id]
docker container prune  # 删除所有停止的容器
# eg:
docker rm 1a5f6a0c9443

# 强制删除
docker rm -f [container_id]
# eg:
docker rm -f 8005c40a1d16

# 删除部分容器
docker container prune --filter "until=24h"

# 批量关闭容器
docker rm -f $(docker ps -aq)

# 一键删除所有已经停止的容器
docker container prune
```
### 清除容器
```bash
docker system prune  # 可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及 dangling 镜像（即无 tag 的镜像）。
docker system prune -a # 清理得更加彻底，可以将没有容器使用 Docker镜像都删掉。

docker container prune : 仅删除停止运行的容器。
docker ps -aq 同上

docker rm -f $(docker ps -aq) : 删除所有容器（包括停止的、正在运行的）。
docker container rm -f $(docker container ls -aq) : 同上。

docker builder prune  # 删除 build cache。
```
### 查看磁盘使用情况
```bash
docker system df 

[root@centos801 bluewhale]# docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          14        10        2.671GB   1.267GB (47%)
Containers      19        17        125.2kB   0B (0%)
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B
```

### 进入容器
> docker 容器启动命令参数详解：
> --name:给容器定义一个名称
> -i:则让容器的标准输入保持打开。
> -t:让docker分配一个伪终端,并绑定到容器的标准输入上
> /bin/bash:执行一个命令

```bash
# 创建并进入容器
docker run --name [container_name] -it [docker_image] /bin/bash

# 手工方式进入容器
docker exec -it 容器id /bin/bash
```
### 退出容器
```bash
exit
Ctrl + D
```

### 基于容器创建镜像
```bash
docker commit -m '改动信息' -a "作者信息" [container_id] [new_image:tag]
# eg:
docker commit -m 'mkdir /sswang' -a "sswang" d74fff341687 sswang-nginx:v0.2
```

### 查看日志
```bash
# 查看容器运行日志
docker logs [容器id]
# eg:
docker logs 7c5a24a68f96
```

### 查看信息
```bash
# 查看容器全部信息
docker inspect [容器id]
# eg:
docker inspect 930f29ccdf8a

# 查看容器网络信息
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 930f29ccdf8a
```

---

## 数据管理命令
`数据卷`是一个可供一个或多个容器使用的特殊目录，它绕过`UFS`，可以提供很多有用的特性：

- 数据卷 可以在容器之间共享和重用
- 对 数据卷 的修改会立马生效
- 对 数据卷 的更新，不会影响镜像
- 数据卷 默认会一直存在，即使容器被删除
> 数据卷 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的 数据卷。

- 选择 -v 还是 -–mount 参数： Docker 新用户应该选择`--mount`参数，经验丰富的 Docker 使用者对`-v`或者 `--volume`已经很熟悉了，但是推荐使用`--mount`参数。

### 创建一个数据卷：
```bash
docker volume create my-vol
```

### 查看所有的数据卷：
```bash
$ docker volume ls
local               my-vol
```

### 在主机里查看指定数据卷的信息
```bash
$ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

### 在运行run命令时挂载数据卷

- 使用`--mount`标记来将 数据卷 挂载到容器里
- 在一次`docker run`中可以挂载多个 数据卷。
```bash
$ docker run -d -P \
    --name web \
    # -v my-vol:/wepapp \
    --mount source=my-vol,target=/webapp \
    training/webapp \
    python app.py
```

### 查看数据卷的具体信息
```bash
$ docker inspect web
...
"Mounts": [
    {
        "Type": "volume",
        "Name": "my-vol",
        "Source": "/var/lib/docker/volumes/my-vol/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
...
```

### 删除数据卷：

- 数据卷作用用来数据持久化，独立于容器之外，不会因为容器被删除而数据卷被删除的
- 如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用`docker rm -v`这个命令。
```bash
docker volume rm my-vol
```

- 无主的数据卷可能会占据很多空间，要清理请使用以下命令
```bash
docker volume prune
```

### 挂载主机目录
-  Docker 新用户应该选择 --mount 参数
- 经验丰富的 Docker 使用者对 -v 或者 --volume 已经很熟悉了
- 推荐使用 --mount 参数。
- 备注：
   - -v:如果本地目录不存在 Docker 会自动为你创建一个文件夹
   - --mount 参数时如果本地目录不存在，Docker 会报错。
   - Docker 挂载主机目录的默认权限是 读写，用户也可以通过增加`readonly`指定为 只读。
```bash
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/opt/webapp \
    --mount type=bind,source=/src/webapp,target=/opt/webapp, readonly \
    training/webapp \
    python app.py
# 上述命令的意思是加载主机的 /src/webapp 目录到容器的 /opt/webapp目录
```

- 加了`readonly`之后，就挂载为 只读 了。如果你在容器内 /opt/webapp 目录新建文件，会显示如下错误:
```bash
/opt/webapp # touch new.txt
touch: new.txt: Read-only file system

# 查看数据卷的具体内容
$ docker inspect web
...
"Mounts": [
    {
        "Type": "bind",
        "Source": "/src/webapp",
        "Destination": "/opt/webapp",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

- `--mount`标记也可以从主机挂载单个文件到容器中
   - 小技巧：记录在容器输入过的命令。
```bash
$ docker run --rm -it \
   # -v $HOME/.bash_history:/root/.bash_history \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:17.10 \
   bash

root@2affd44b4667:/# history
1  ls
2  diskutil list
```

---

## 网络管理命令
### Bridge模式
当`Docker`进程启动时，会在主机上创建一个名为`docker0`的虚拟网桥，此主机上启动的`Docker`容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。从`docker0`子网中分配一个 IP 给容器使用，并设置 docker0 的 IP 地址为容器的**默认网关**。在主机上创建一对虚拟网卡`veth pair`设备，Docker 将 veth pair 设备的一端放在新创建的容器中，并命名为`eth0`（容器的网卡），另一端放在主机中，以`vethxxx`这样类似的名字命名，并将这个网络设备加入到 docker0 网桥中。可以通过`brctl show`命令查看。
`bridge`模式是 docker 的默认网络模式，不写`–net`参数，就是`bridge`模式。使用`docker run -p`时，docker 实际是在`iptables`做了`DNAT`规则，实现端口转发功能。可以使用`iptables -t nat -vnL`查看。`bridge`模式如下图所示：![](https://cdn.nlark.com/yuque/0/2021/jpeg/2130981/1616337102707-7371b91a-5186-47da-92d5-1bbe09f4fe55.jpeg#averageHue=%23f9f8f8&height=656&id=JjWhW&originHeight=656&originWidth=584&originalType=binary&ratio=1&rotation=0&showTitle=false&size=0&status=done&style=none&title=&width=584)
演示：
```bash
$ docker run -tid --net=bridge --name docker_bri1 \
             ubuntu-base:v3
             docker run -tid --net=bridge --name docker_bri2 \
             ubuntu-base:v3 
$ brctl show
$ docker exec -ti docker_bri1 /bin/bash
$ ifconfig –a
$ route –n
```
如果你之前有 Docker 使用经验，你可能已经习惯了使用`--link`参数来使容器互联。
随着 Docker 网络的完善，强烈建议大家将容器加入自定义的 Docker 网络来连接多个容器，而不是使用 --link 参数。
下面先创建一个新的 Docker 网络。
```bash
$ docker network create -d bridge my-net
```
`-d`参数指定 Docker 网络类型，有 `bridge overlay`。其中 overlay 网络类型用于 Swarm mode，在本小节中你可以忽略它。
运行一个容器并连接到新建的 my-net 网络
```bash
$ docker run -it --rm --name busybox1 --network my-net busybox sh
```
打开新的终端，再运行一个容器并加入到 my-net 网络
```bash
$ docker run -it --rm --name busybox2 --network my-net busybox sh
```
再打开一个新的终端查看容器信息
```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b47060aca56b        busybox             "sh"                11 minutes ago      Up 11 minutes                           busybox2
8720575823ec        busybox             "sh"                16 minutes ago      Up 16 minutes                           busybox1
```
下面通过 ping 来证明 busybox1 容器和 busybox2 容器建立了互联关系。 在 busybox1 容器输入以下命令
```bash
/ # ping busybox2
PING busybox2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.072 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.118 ms
```
用 ping 来测试连接 busybox2 容器，它会解析成 172.19.0.3。 同理在 busybox2 容器执行 ping busybox1，也会成功连接到。
```bash
/ # ping busybox1
PING busybox1 (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.064 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.143 ms
```
这样，busybox1 容器和 busybox2 容器建立了互联关系。
如果你有多个容器之间需要互相连接，推荐使用`Docker Compose`。

### Host 模式
如果启动容器的时候使用`host`模式，那么这个容器将不会获得一个独立的`Network Namespace`，而是和宿主机共用一个 Network Namespace。容器将不会虚拟出自己的网卡，配置自己的 IP 等，而是使用宿主机的 IP 和端口。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。 Host模式如下图所示：
![](https://cdn.nlark.com/yuque/0/2021/jpeg/2130981/1616337102857-3fa24961-4f7d-4db6-8cb2-a40336cf7b25.jpeg#averageHue=%23fafafa&height=692&id=J4KrF&originHeight=692&originWidth=614&originalType=binary&ratio=1&rotation=0&showTitle=false&size=0&status=done&style=none&title=&width=614)
演示：
```bash
$ docker run -tid --net=host --name docker_host1 ubuntu-base:v3
$ docker run -tid --net=host --name docker_host2 ubuntu-base:v3
$ docker exec -ti docker_host1 /bin/bash
$ docker exec -ti docker_host1 /bin/bash
$ ifconfig –a
$ route –n
```

### Container 模式
这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡设备通信。 Container模式示意图： ![](https://cdn.nlark.com/yuque/0/2021/jpeg/2130981/1616337102544-c3f084d9-64d1-4092-8422-155dad7bf1e8.jpeg#averageHue=%23e2e3e0&height=694&id=Xhikr&originHeight=694&originWidth=616&originalType=binary&ratio=1&rotation=0&showTitle=false&size=0&status=done&style=none&title=&width=616)演示：
```bash
$ docker run -tid --net=container:docker_bri1 \
              --name docker_con1 ubuntu-base:v3
$ docker exec -ti docker_con1 /bin/bash
$ docker exec -ti docker_bri1 /bin/bash
$ ifconfig –a
$ route -n
```

### None模式
使用`none`模式，Docker 容器拥有自己的 Network Namespace，但是，并不为Docker 容器进行任何网络配置。也就是说，这个 Docker 容器没有网卡、IP、路由等信息。需要我们自己为 Docker 容器添加网卡、配置 IP 等。 None模式示意图: ![](https://cdn.nlark.com/yuque/0/2021/jpeg/2130981/1616337102792-52989aca-725b-4d7b-8615-e3f662945466.jpeg#averageHue=%23fbfbfb&height=693&id=STRpL&originHeight=693&originWidth=619&originalType=binary&ratio=1&rotation=0&showTitle=false&size=0&status=done&style=none&title=&width=619)演示：
```bash
$ docker run -tid --net=none --name \
                docker_non1 ubuntu-base:v3
$ docker exec -ti docker_non1 /bin/bash
$ ifconfig –a
$ route -n
```

---

## 常用清理命令

- 一键删除所有已经停止的容器
```
docker container prune
```

- 删除所有容器（包含停止的 正在运行的）
```
docker rm -f $(docker ps -aq)
docker container rm -f $(docker container ls -aq)
```

- 所有悬挂状态的镜像
```
docker image ls -f dangling=true
```

- 删除所有悬挂状态的镜像
```
docker image rm $(docker image ls -f dangling=true -q)
docker image prune
```

- 删除不在使用的数据集
```
docker volume rm $(docker volume ls -q)
docker volume prune
```

- 删除build cache
```
docker builder prune
```

- 一键清理
```
docker system prune
```

---

## compose命令
### docker-compose.yml
```bash
version: "3"
services:
 website:
   build: ./bluewhale
   hostname: website
   command: bash -c "python3 manage.py collectstatic --no-input &&
                     python manage.py makemigrations &&
                     python3 manage.py migrate &&
                     gunicorn bluewhale.wsgi:application --bind 0.0.0.0:8001"
   expose:
      - "8001"
   volumes:
      - ./bluewhale:/data/projects/bluewhale_docker/bluewhale:rw # 挂载项目代码
      - static-volume:/data/projects/bluewhale_docker/bluewhale/static
      - ./compose/bw_uwsgi:/tmp # 挂载uwsgi日志
   links:
      - mysqldb
   depends_on: # 依赖关系
      - mysqldb
   networks:
      - web_network
      - db_network
   environment:
      - DEBUG=False
   restart: "on-failure"
   tty: true
   stdin_open: true

networks:
  web_network:
    driver: bridge
  db_network:
    driver: bridge
volumes:
  static-volume:
  bluewhale_db_vol:

```

### 启动、停止、删除
```bash
docker-compose up -d  # -d表示守护进程启动
docker-compose stop  
docker-compose rm 

#停止和删除可以合并执行
docker-compose stop && docker-compose rm
```

