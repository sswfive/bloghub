---
title: Java环境安装
date: 2023-08-15 21:52:48
tags:
  - Installation
---
## Java环境部署

  使用JDK为平台的运行提供Java服务运行环境，JDK全称Java SE Development kit(JDK)，即Java标准版开发包，是Oracle提供的一套用于开发Java应用程序的开发包，它提供编译，运行Java程序所需要的各种工具和资源，包括java编译器，Java运行时环境，以及常用的Java类库等。在本平台中网关服务、用户管理服务、权限认证服务以及数据管理服务是基于Java语言开发的。

### 部署说明

- 192.168.21.71->node71
- 192.168.21.72->node72
- 192.168.21.73->node73
- 192.168.21.74->node74
- 192.168.21.75->node75

| 事项          | 详情                               |
| ------------- | ---------------------------------- |
| 部署节点      | 192.168.21.71~192.168.21.75        |
| JDK安装包位置 | /data/software/jdk1.8.0_201.tar.gz |
| JDK安装目录   | /data/workspace/jdk1.8.0_201       |

### 安装部署

#### step1: 解压文件到指定目录

```shell
sudo tar -zxvf /data/software/jdk1.8.0_201.tar.gz -C /data/workspace
```

#### step2: 配置环境变量

环境变量配置于/etc/profile

```shell
vim /etc/profile
```

配置如下内容:

```shell
# JDK1.8
export JAVA_HOME=/data/workspace/jdk1.8.0_201
export PATH=$PATH:$JAVA_HOME/bin
```

#### step3: 配置生效

```shell
source /etc/profile
```

#### step4: 验证

```shell
java -version

java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
```

#### step5: 将JDK跟目录分发至各服务器节点

```shell
sudo scp -r /data/workspace/jdk1.8.0_201 root@node71:/data/workspace/
sudo scp -r /data/workspace/jdk1.8.0_201 root@node72:/data/workspace/
sudo scp -r /data/workspace/jdk1.8.0_201 root@node73:/data/workspace/
sudo scp -r /data/workspace/jdk1.8.0_201 root@node74:/data/workspace/
sudo scp -r /data/workspace/jdk1.8.0_201 root@node75:/data/workspace/

# 分别进入各服务器节点添加环境变量

vim /etc/profile

配置内容如下：

export JAVA_HOME=/data/workspace/jdk1.8.0_201
export PATH=$PATH:$JAVA_HOME/bin

生效配置：

source /etc/profile
```



