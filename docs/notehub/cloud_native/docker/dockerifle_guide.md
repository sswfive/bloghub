---
title: Dockerfile常用命令与编写指南
date: 2022-04-15 12:30:59
tags: 
  - Docker
---
> Dockerfile中的每一个指令都会建立一层镜像

## FROM：指定基础镜像

- 在dockerfile文件中必须是第一条命令
- 可以使用Docker store上官方镜像（nginx、redis、mongo、mysql、httpd、php、tomcat 等）
- 可以使用方便开发，构建，运行各种语言的应用镜像（node、openjdk、python、ruby、golang 等）
- 可以使用更为基础的操作系统镜像，这些操作系统的软件库提供了更广阔的扩展空间（ubuntu、debian、centos、fedora、alpine 等）
- 可以使用空白镜像（scratch）,一般go开发的使用比较多
```dockerfile
FROM nginx
```
## RUN：执行命令

- RUN 的格式有两种：
   - shell格式：RUN <命令>， 和在命令行直接输入命令的效果一样。
```dockerfile
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

   - exec格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式
```dockerfile
FROM debian:jessie
RUN apt-get update
RUN apt-get install -y gcc libc6-dev make
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
# 上述这种写法，创建了7层镜像。通常在实际应用中是没有必要的，可以优化为下列形式
FROM debian:jessie
RUN buildDeps='gcc libc6-dev make' \
&& apt-get update \
&& apt-get install -y $buildDeps \
&& wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
&& mkdir -p /usr/src/redis \
&& tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
&& make -C /usr/src/redis \
&& make -C /usr/src/redis install \
&& rm -rf /var/lib/apt/lists/* \
&& rm redis.tar.gz \
&& rm -r /usr/src/redis \
&& apt-get purge -y --auto-remove $buildDeps

# 此时只创建了一层镜像，节省了空间，加快了制作镜像的速度，
# 另外通常在执行完安装命令以后，一般需要去清理掉软件包和缓存。缩小镜像的大小。
```

## Build: 构建镜像
```bash
docker build [选项] <上下文路径/URL/->

$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM nginx
---> e43d811ce2f4
Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
---> Running in 9cdc27646c7b
---> 44aa4490ce2c
Removing intermediate container 9cdc27646c7b
Successfully built 44aa4490ce2c
```
## Context：镜像构建上下文
> 当我们进行镜像构建的时候，并非所有定制都会通过 RUN 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 COPY 指令、ADD 指令等。而 docker build 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？
> 这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

```dockerfile
COPY ./package.json /app/

# 这并不是要复制执行 docker build 命令所在的目录下的 package.json，也不是复制 Dockerfile 所在目录下的 package.json，
# 而是复制 上下文（context） 目录下的 package.json。
```
> `COPY`这类指令中的源文件的路径都是相对路径。这也是初学者经常会问的为什么 COPY ../package.json /app 或者 COPY /opt/xxxx /app 无法工作的原因，因为这些路径已经超出了上下文的范围，Docker 引擎无法获得这些位置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。

```bash
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
...
# 通过build 输出，可以看到发送上下文的过程
```

- 经验Note:
   - 一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一个`.dockerignore`，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。
   - 那么为什么会有人误以为 **.** 是指定 Dockerfile 所在目录呢？这是因为在默认情况下，如果不额外指定 Dockerfile 的话，会将上下文目录下的名为 Dockerfile 的文件作为 Dockerfile。
   - 这只是默认行为，实际上 Dockerfile 的文件名并不要求必须为 Dockerfile，而且并不要求必须位于上下文目录中，比如可以用`-f ../Dockerfile.php`参数指定某个文件作为 Dockerfile。
   - 一般大家习惯性的会使用默认的文件名 Dockerfile，以及会将其置于镜像构建上下文目录中。

## 迁移镜像
> Docker 还提供了docker load和docker save命令，用以将镜像保存为一个 tar 文件，然后传输到另一个位置上，再加载进来。这是在没有 Docker Registry 时的做法，现在已经不推荐，镜像迁移应该直接使用 Docker Registry，无论是直接使用 Docker Hub 还是使用内网私有 Registry 都可以。

使用docker save命令可以将镜像保存为归档文件。比如我们希望保存这个 alpine 镜像。
```bash
docker image ls alpine
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              baa5d63471ea        5 weeks ago         4.803 MB

# 保存镜像的命令为：
docker save alpine | gzip > alpine-latest.tar.gz

# 加载镜像
docker load -i alpine-latest.tar.gz
Loaded image: alpine:latest
```

- 上述两条命令可以合并为一条命令,并且带有进度条
```bash
docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
```

