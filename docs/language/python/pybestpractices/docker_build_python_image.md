---
title: Docker构建Python镜像实践指南
date: 2023-06-18 16:44:37
tags: 
  - Python
  - Docker
---

# Docker构建Python镜像实践指南


## 实践指南
### 推荐设置的环境变量
```dockerfile
# 设置环境变量 

# 建议构建 Docker 镜像时一直为 `1`, 防止 python 将 pyc 文件写入硬盘
ENV PYTHONDONTWRITEBYTECODE 1
# 建议构建 Docker 镜像时一直为 `1`, 防止 python 缓冲 (buffering) stdout 和 stderr, 以便更容易地进行容器日志记录
ENV PYTHONUNBUFFERED 1
```

- 不建议使用 ENV DEBUG 0 环境变量，没必要。
### 使用非root用户运行容器进程

- 出于安全考虑，推荐运行 Python 程序前，创建 非 root 用户并切换到该用户。
```dockerfile
# 创建一个具有明确 UID 的非 root 用户，并增加访问 /app 文件夹的权限。
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /appUSER appuser
```
### 使用 `.dockerignore` 排除无关文件
```dockerfile
**/__pycache__
**/*venv
**/.classpath
**/.dockerignore
**/.env
**/.git
**/.gitignore
**/.project
**/.settings
**/.toolstarget
**/.vs
**/.vscode
**/*.*proj.user
**/*.dbmdl
**/*.jfm
**/bin
**/charts
**/docker-compose*
**/compose*
**/Dockerfile*
**/node_modules
**/npm-debug.log
**/obj
**/secrets.dev.yaml
**/values.dev.yaml
*.db
.python-version
LICENSE
README.md
```
### 不建议使用 Alpine 作为 Python 的基础镜像

- 大多数 Linux 发行版使用 GNU 版本（glibc）的标准 C 库，几乎每个 C 程序都需要这个库，包括 Python。但是 Alpine Linux 使用 musl, Alpine 禁用了 Linux wheel 支持。
- 缺少大量依赖
   - CPython 语言运行时的相关依赖
   - openssl 相关依赖
   - libffi 相关依赖
   - gcc 相关依赖
   - 数据库驱动相关依赖
   - pip 相关依赖
   - 构建可能更耗时
- Alpine Linux 使用 musl，一些二进制 wheel 是针对 glibc 编译的，但是 Alpine 禁用了 Linux wheel 支持。现在大多数 Python 包都包括 PyPI 上的二进制 wheel，大大加快了安装时间。但是如果你使用 Alpine Linux，你可能需要编译你使用的每个 Python 包中的所有 C 代码。
- 基于 Alpine 构建的 Python 镜像反而可能更大
```dockerfile
# 因为缺少依赖，所以在pip安装前需要安装全依赖。
RUN set -eux \    
&& apk add --no-cache --virtual .build-deps build-base \    openssl-dev libffi-dev gcc musl-dev python3-dev \    
&& pip install --upgrade pip setuptools wheel \    
&& pip install --upgrade -r /app/requirements.txt \    
&& rm -rf /root/.cache/pip
```
### 建议使用官方的 python slim 镜像作为基础镜像

- 镜像库是这个：[https://hub.docker.com/](https://hub.docker.com/)_/python， 并且使用 python:<version>-slim 作为基础镜像，能用 python:<version>-slim-bullseye 作为基础镜像更好（因为更新，相对就更安全一些）.
- 这个镜像不包含默认标签中的常用包，只包含运行 python 所需的最小包。这个镜像是基于 Debian 的。
- 使用官方 python slim 的理由还包括：
   - 稳定性
   - 安全升级更及时
   - 依赖更新更及时
   - 依赖更全
   - Python 版本升级更及时
   - 镜像更小
### 一般情况下，Python 镜像构建不需要使用"多阶段构建"

- Python 没有像 Golang 一样，可以把所有依赖打成一个单一的二进制包
- Python 也没有像 Java 一样，可以在 JDK 上构建，在 JRE 上运行
- Python 复杂而散落的依赖关系，在"多阶段构建"时会增加复杂度

如果有一些特殊情况，可以尝试使用"多阶段构建"压缩镜像体积：

- 构建阶段需要安装编译器
- Python 项目复杂，用到了其他语言代码（如 C/C++/Rust)
### Dockerfile 中 pip 编写建议

- 使用 pip 安装依赖时，可以添加 `--no-cache-dir` 减少镜像体积：
```dockerfile
# 安装 pip 依赖
COPY requirements.txt .
RUN python -m pip install --no-cache-dir --upgrade -r requirements.txt
```
## Dockerfile 编写参考
### slim版本
```dockerfile
FROM python:3.10-slim

LABEL maintainer="ssw"

EXPOSE 8000

# Keeps Python from generating .pyc files in the container
ENV PYTHONDONTWRITEBYTECODE=1

# Turns off buffering for easier container logging
ENV PYTHONUNBUFFERED=1

# Install pip requirements
COPY requirements.txt .
RUN python -m pip install --no-cache-dir --upgrade -r requirements.txt

WORKDIR /app
COPY . /app

# Creates a non-root user with an explicit UID and adds permission to access the /app folder
# For more info, please refer to https://aka.ms/vscode-docker-python-configure-containers
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /app
USER appuser

# During debugging, this entry point will be overridden. For more information, please refer to https://aka.ms/vscode-docker-python-debug
CMD ["uvicorn", "shortener_app.main:app", "--host", "0.0.0.0"]
```
### Alpine版本(不建议)
```dockerfile
FROM python:3.10-alpine

LABEL maintainer="ssw"

EXPOSE 8000

# Keeps Python from generating .pyc files in the container
ENV PYTHONDONTWRITEBYTECODE=1

# Turns off buffering for easier container logging
ENV PYTHONUNBUFFERED=1

# Install pip requirements
COPY requirements.txt .
RUN set -eux \
    && apk add --no-cache --virtual .build-deps build-base \
    openssl-dev libffi-dev gcc musl-dev python3-dev \
    && pip install --upgrade pip setuptools wheel \
    && pip install --upgrade -r requirements.txt \
    && rm -rf /root/.cache/pip

WORKDIR /app
COPY . /app

# Creates a non-root user with an explicit UID and adds permission to access the /app folder
# For more info, please refer to https://aka.ms/vscode-docker-python-configure-containers
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /app
USER appuser

# During debugging, this entry point will be overridden. For more information, please refer to https://aka.ms/vscode-docker-python-debug
CMD ["uvicorn", "shortener_app.main:app", "--host", "0.0.0.0"]
```
