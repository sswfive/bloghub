---
title: docker-compose使用总结
date: 2022-04-21 19:21:32
tags: 
  - DockerCompose
  - Docker
---
## compose简述
- `Docker Compose`是`Docker`官方编排（Orchestration）项目之一，负责快速的部署分布式应用
- 它允许用户通过一个单独的 `docker-compose.yml`模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目
- 适用于开发环境或者单机环境下的容器编排
- Compose 中有两个重要的概念：
   - 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
   - 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

## 安装
### 二进制安装

- 从 官方 GitHub Release 处直接下载编译好的二进制文件即可。例如，在 Linux 64 位系统上直接下载对应的二进制包。
```
sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose version
```
### PIP 安装
> 注： x86_64 架构的 Linux 建议按照上边的方法下载二进制包进行安装，如果您计算机的架构是 ARM (例如，树莓派)，再使用 pip 安装。

这种方式是将 Compose 当作一个 Python 应用来从 pip 源中安装。执行安装命令：
```bash
$ sudo pip install -U docker-compose
Collecting docker-compose
  Downloading docker-compose-1.17.1.tar.gz (149kB): 149kB downloaded
...
Successfully installed docker-compose cached-property requests texttable websocket-client docker-py dockerpty six enum34 backports.ssl-match-hostname ipaddress
```

- bash 补全命令：
```bash
$ curl -L https://raw.githubusercontent.com/docker/compose/1.8.0/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

### 容器中执行
Compose 既然是一个 Python 应用，自然也可以直接用容器来执行它。
```
$ curl -L https://github.com/docker/compose/releases/download/1.8.0/run.sh > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```
实际上，查看下载的`run.sh`脚本内容，如下:
```
set -e
VERSION="1.8.0"
IMAGE="docker/compose:$VERSION"
# Setup options for connecting to docker hostif [ -z "$DOCKER_HOST" ]; then
    DOCKER_HOST="/var/run/docker.sock"fiif [ -S "$DOCKER_HOST" ]; then
    DOCKER_ADDR="-v $DOCKER_HOST:$DOCKER_HOST -e DOCKER_HOST"else
    DOCKER_ADDR="-e DOCKER_HOST -e DOCKER_TLS_VERIFY -e DOCKER_CERT_PATH"fi
# Setup volume mounts for compose config and contextif [ "$(pwd)" != '/' ]; then
    VOLUMES="-v $(pwd):$(pwd)"fiif [ -n "$COMPOSE_FILE" ]; then
    compose_dir=$(dirname $COMPOSE_FILE)fi# TODO: also check --file argumentif [ -n "$compose_dir" ]; then
    VOLUMES="$VOLUMES -v $compose_dir:$compose_dir"fiif [ -n "$HOME" ]; then
    VOLUMES="$VOLUMES -v $HOME:$HOME -v $HOME:/root" # mount $HOME in /root to share docker.configfi
# Only allocate tty if we detect oneif [ -t 1 ]; then
    DOCKER_RUN_OPTIONS="-t"fiif [ -t 0 ]; then
    DOCKER_RUN_OPTIONS="$DOCKER_RUN_OPTIONS -i"fi
exec docker run --rm $DOCKER_RUN_OPTIONS $DOCKER_ADDR $COMPOSE_OPTIONS $VOLUMES -w "$(pwd)" $IMAGE "$@"
```

- 可以看到，它其实是下载了`docker/compose`镜像并运行。

### 卸载

- 如果是二进制包方式安装的，删除二进制文件即可。
```
$ sudo rm /usr/local/bin/docker-compose
```

- 如果是通过 pip 安装的，则执行如下命令即可删除。
```
$ sudo pip uninstall docker-compose
```

## 使用

- 编写`Dockerfile`文件
```
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]
```

- 编写`docker-compose.yml`文件
```
version: '3'
services:
  web:    
  build: .    
  ports:    
  - "5000:5000"
  volumes:
       - .:/code
  redis:    
  image: "redis:alpine"
```

- 运行 compose 项目:
```
$ docker-compose up
```


## Compose 命令
### 常用命令总结
```bash
docker-compose up -d	以守护进程模式运行加-d选项
docker-compose ps	列出所有运行中的容器,类似docker ps
docker-compose logs web	查看web服务日志
docker-compose stop	停止已运行的服务
docker-compose down	停止并移除容器、网络、镜像和数据卷.比stop更彻底
docker-compose restart	重启YAML文件中定义的服务
docker-compose kill	停止服务
docker-compose rm	删除指定已经停止服务的容器
docker-compose build	构建或重建服务
docker-compose images	列出所有镜像
docker-compose pull	拉去并下载指定服务镜像
docker-compose push	push服务镜像
docker-compose top	显示各个容器内运行的进程
```

### 查看命令使用格式

- **docker-compose [COMMAND] --help**
- **docker-compose help [COMMAND]**
```
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```
命令选项：

- -f, --file FILE 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。
- -p, --project-name NAME 指定项目名称，默认将使用所在目录名称作为项目名。
- --x-networking 使用 Docker 的可拔插网络后端特性
- --x-network-driver DRIVER 指定网络后端的驱动，默认为 bridge
- --verbose 输出更多调试信息。
- -v, --version 打印版本并退出。
### build 格式

- **docker-compose build [options] [SERVICE...]， **构建（重新构建）项目中的服务容器
- 可以随时在项目目录下运行`docker-compose build`来重新构建服务。选项包括：
   - --force-rm 删除构建过程中的临时容器。
   - --no-cache 构建镜像过程中不使用 cache（这将加长构建过程）。
   - --pull 始终尝试通过 pull 来获取更新版本的镜像。
### config: 

- 验证 Compose 文件格式是否正确，若正确则显示配置，若格式错误显示错误原因。
### down：

- 此命令将会停止 up 命令所启动的容器，并移除网络
### exec：

- 进入指定的容器。
### help：

- 获得一个命令的帮助。
### images：

- 列出 Compose 文件中包含的镜像。
### kill：

- 格式为**docker-compose kill [options] [SERVICE...]**。
- 通过发送`SIGKILL`信号来强制停止服务容器。支持通过`-s`参数来指定发送的信号，例如通过如下指令发送`SIGINT`信号。
```
$ docker-compose kill -s SIGINT
```
### logs：

- 格式为**docker-compose logs [options] [SERVICE...]**，
- 查看服务容器的输出。默认情况下，docker-compose 将对不同的服务输出使用不同的颜色来区分。可以通过 `--no-color`来关闭颜色。该命令在调试问题的时候十分有用。
### pause：

- 格式为**docker-compose pause [SERVICE...]**，暂停一个服务容器。
### port：

- 格式为**docker-compose port [options] SERVICE PRIVATE_PORT**，打印某个容器端口所映射的公共端口。选项：
- --protocol=proto 指定端口协议，tcp（默认值）或者 udp。
- --index=index 如果同一服务存在多个容器，指定命令对象容器的序号（默认为 1）。
### ps：

- 格式为**docker-compose ps [options] [SERVICE...]**，列出项目中目前的所有容器。选项：
- `-q`只打印容器的 ID 信息。
### pull：

- 格式为`docker-compose pull [options] [SERVICE...]`，拉取服务依赖的镜像。选项：
- --ignore-pull-failures 忽略拉取镜像过程中的错误。
### push：

- 推送服务依赖的镜像到 Docker 镜像仓库。
### restart：

- 格式为**docker-compose restart [options] [SERVICE...]**，重启项目中的服务。选项：
- -t, --timeout TIMEOUT 指定重启前停止容器的超时（默认为 10 秒）。
### rm：

- 格式为**docker-compose rm [options] [SERVICE...]**,删除所有（停止状态的）服务容器。推荐先执行 `docker-compose stop`命令来停止容器。选项：
- -f, --force 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。
- -v 删除容器所挂载的数据卷。
### run：

- 格式为**docker-compose run [options] [-p PORT...] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]**，在指定服务上执行一个命令。例如：
```bash
$ docker-compose run ubuntu ping docker.com
```
> 将会启动一个 ubuntu 服务容器，并执行 ping docker.com 命令。默认情况下，如果存在关联，则所有关联的服务将会自动被启动，除非这些服务已经在运行中。
> 该命令类似启动容器后运行指定的命令，相关卷、链接等等都将会按照配置自动创建。

> 给定命令将会覆盖原有的自动运行命令； 不会自动创建端口，以避免冲突。

- 如果不希望自动启动关联的容器，可以使用`--no-deps`选项，例如:
```bash
$ docker-compose run --no-deps web python manage.py shell
```
将不会启动 web 容器所关联的其它容器,选项：

- -d 后台运行容器。
- --name NAME 为容器指定一个名字。
- --entrypoint CMD 覆盖默认的容器启动指令。
- -e KEY=VAL 设置环境变量值，可多次使用选项来设置多个环境变量。
- -u, --user="" 指定运行容器的用户名或者 uid。
- --no-deps 不自动启动关联的服务容器。
- --rm 运行命令后自动删除容器，d 模式下将忽略。
- -p, --publish=[] 映射容器端口到本地主机。
- --service-ports 配置服务端口并映射到本地主机。
- -T 不分配伪 tty，意味着依赖 tty 的指令将无法运行。
### scale：

- 格式为**docker-compose scale [options] [SERVICE=NUM...]**，设置指定服务运行的容器个数。 
- 通过 service=num 的参数来设置数量。例如：
```bash
$ docker-compose scale web=3 db=2
```
将启动 3 个容器运行 web 服务，2 个容器运行 db 服务。
一般的，当指定数目多于该服务当前实际运行容器，将新创建并启动容器；反之，将停止容器。选项：

- -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。
### start：

- 格式为**docker-compose start [SERVICE...]**，启动已经存在的服务容器。
### stop：

- 格式为**docker-compose stop [options] [SERVICE...]**, 停止已经处于运行状态的容器，但不删除它。通过`docker-compose start`可以再次启动这些容器。选项：
- -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。
### top：

- 查看各个服务容器内运行的进程。
### unpause：

- 格式为**docker-compose unpause [SERVICE...]**，恢复处于暂停状态中的服务。
### up：

- 格式为**docker-compose up [options] [SERVICE...]**，该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。链接的服务都将会被自动启动，除非已经处于运行状态。 可以说，大部分时候都可以直接通过该命令来启动一个项目。
- 默认情况，`docker-compose up`启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。 当通过 Ctrl-C 停止命令时，所有容器将会停止。
- 如果使用`docker-compose up -d`，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。
- 默认情况，如果服务容器已经存在，`docker-compose up`将会尝试停止容器，然后重新创建（保持使用 volumes-from 挂载的卷），以保证新启动的服务匹配 docker-compose.yml 文件的最新内容。如果用户不希望容器被停止并重新创建，可以使用 `docker-compose up --no-recreate`。这样将只会启动处于停止状态的容器，而忽略已经运行的服务。
- 如果用户只想重新部署某个服务，可以使用`docker-compose up --no-deps -d <SERVICE_NAME>`来重新创建服务并后台停止旧服务，启动新服务，并不会影响到其所依赖的服务。选项：
   - -d 在后台运行服务容器。
   - --no-color 不使用颜色来区分不同的服务的控制台输出。
   - --no-deps 不启动服务所链接的容器。
   - --force-recreate 强制重新创建容器，不能与`--no-recreate`同时使用。 `--no-recreate`如果容器已经存在了，则不重新创建，不能与 `--force-recreate`同时使用。 `--no-build`不自动构建缺失的服务镜像。
   - -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。
