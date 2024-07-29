---
title: Zookeeper安装部署
date: 2023-08-15 21:48:17
tags:
  - Installation
---
## Zookeeper环境部署

  Zookeeper作为分布式协调组件主要承载相关大数据组件的高可用保障，当前分布式系统中某节点服务因故障退出时能够实现快速切换。本系统主要用于支撑Yarn、Kakfa、Spark以及Flink服务的运行。

### 部署说明

- 192.168.21.73 -> node73
- 192.168.21.74 -> node74
- 192.168.21.75 -> node75

| 事项             | 详情                                                        |
| ---------------- | ----------------------------------------------------------- |
| 部署节点         | 192.168.21.73-192.168.21.75                                 |
| 安装包位置       | /data/software/apache-zookeeper-3.6.3-bin.tar.gz            |
| 运行目录         | /data/workspace/apache-zookeeper-3.6.3-bin                  |
| 数据存储目录     | /data/workspace/apache-zookeeper-3.6.3-bin/snapshot         |
| 启停状态维护脚本 | /data/workspace/apache-zookeeper-3.6.3-bin/bin/zkServerX.sh |

### Zookeeper部署

#### step1: 解压文件到指定目录

```shell
sudo tar -zxvf /data/software/apache-zookeeper-3.6.3-bin.tar.gz -C /data/workspace
```

```
sudo chown -R demouser:demouser /data/workspace/apache-zookeeper-3.6.3-bin
```

#### step2: 配置环境变量

```shell
vim ~/.bashrc
```

配置内容如下：

```shell
# Zookeeper
export ZK_HOME=/data/workspace/apache-zookeeper-3.6.3-bin
export PATH=$PATH:$ZK_HOME/bin
```

生效配置：

```shell
source ~/.bashrc
```

#### step3: 配置 zoo.cfg文件

```shell
cp $ZK_HOME/conf/zoo_sample.cfg $ZK_HOME/conf/zoo.cfg
vim $ZK_HOME/conf/zoo.cfg
```

```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=60
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=10
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/data/workspace/apache-zookeeper-3.6.3-bin/snapshot
dataLogDir=/data/workspace/apache-zookeeper-3.6.3-bin/transaction
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
maxClientCnxns=120
maxSessionTimeout=120000
minSessionTimeout=4000
cnxTimeout=30000
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
autopurge.snapRetainCount=16
# Purge task interval in hours
# Set to "0" to disable auto purge feature
autopurge.purgeInterval=24
snapCount=409600
preAllocSize=65536

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

server.1=node73:2888:3888
server.2=node74:2888:3888
server.3=node75:2888:3888

4lw.commands.whitelist=*
```

#### step4: 优化JVM参数

```shell
> vim $ZK_HOME/conf/java.env

export JAVA_HOME=/data/workspace/jdk1.8.0_201
export JVMFLAGS="-Xms2048m -Xmx2048m $JVMFLAGS"
```

修改完成使用jmap -heap $pid来验证内存修改情况。

#### step5: 修改日志存储

```shell
> vim $ZK_HOME/bin/zkEnv.sh

if [ "x${ZOO_LOG_DIR}" = "x" ]
then
    # ZOO_LOG_DIR="$ZOOKEEPER_PREFIX/logs"
    ZOO_LOG_DIR="/data/workspace/apache-zookeeper-3.6.3-bin/log"
fi
```

#### step6: 锁定pid文件

```shell
> vim $ZK_HOME/bin/zkServer.sh

if [ -z "$ZOOPIDFILE" ]; then
    if [ ! -d "$ZOO_DATADIR" ]; then
        mkdir -p "$ZOO_DATADIR"
    fi
    # ZOOPIDFILE="$ZOO_DATADIR/zookeeper_server.pid"
    ZOOPIDFILE="/data/workspace/apache-zookeeper-3.6.3-bin/pid/zookeeper_server.pid"
else
    # ensure it exists, otw stop will fail
    mkdir -p "$(dirname "$ZOOPIDFILE")"
fi
```

#### step7: 多个节点分别创建数据存储路径

```shell
sudo mkdir -p /data/workspace/apache-zookeeper-3.6.3-bin/snapshot
sudo mkdir -p /data/workspace/apache-zookeeper-3.6.3-bin/transaction
sudo mkdir -p /data/workspace/apache-zookeeper-3.6.3-bin/log
sudo mkdir -p /data/workspace/apache-zookeeper-3.6.3-bin/pid
```

#### step8: 将zookeeper跟目录分发至各服务器节点

```shell
sudo scp -r /data/workspace/apache-zookeeper-3.6.3-bin root@node74:/data/workspace/
sudo scp -r /data/workspace/apache-zookeeper-3.6.3-bin root@node75:/data/workspace/

# 分别进入node74、node75修改apache-zookeeper-3.6.3-bin所属用户组和所属用户
sudo chown -R zhongche:zhongche /data/workspace/apache-zookeeper-3.6.3-bin

# 分别进入node74、node75添加环境变量

vim ~/.bashrc

配置内容如下：

export ZK_HOME=/data/workspace/apache-zookeeper-3.6.3-bin
export PATH=$PATH:$ZK_HOME/bin

生效配置：

source ~/.bashrc
```

#### step9: 在数据存储目录下配置myid文件并设置id，创建数据存储目录：

```shell
# 在node73下执行
echo 1 > /data/workspace/apache-zookeeper-3.6.3-bin/snapshot/myid
# 在node74下执行
echo 2 > /data/workspace/apache-zookeeper-3.6.3-bin/snapshot/myid
# 在node75下执行
echo 3 > /data/workspace/apache-zookeeper-3.6.3-bin/snapshot/myid
```

#### step10: 通过如下指令启动服务

```shell
$ZK_HOME/bin/zkServer.sh start
```

 完毕之后通过jps查看到zk进程为QuorumPeerMain。

通过如下命令验证zk端口是否存在：

```shell
netstat -ntlp | grep 2181
```

查看zk节点状态为leader或者follower代表成功。

```shell
$ZK_HOME/bin/zkServer.sh status
```

#### step11: 为了方便在一台节点上同时启停管理zk节点，可以扩展如下脚本：

编写节点列表配置文件：

```shell
> vim $ZK_HOME/conf/nodes

node73
node74
node75
```

编写扩展脚本：

```shell
> vim $ZK_HOME/bin/zkServerX.sh

#!/bin/bash

for node in `cat $ZK_HOME/conf/nodes`
do
  echo "********************************" [ $node ] "************************************"
  ssh $node $ZK_HOME/bin/zkServer.sh $1
  echo "*******************************************************************************"
done
```

```shell
chmod u+x $ZK_HOME/bin/zkServerX.sh
```

11.运维命令

（1）启动zookeeper服务。

```shell
${ZK_HOME}/bin/zkServerX.sh start
```

（2）查看zookeeper服务状态。

```shell
${ZK_HOME}/bin/zkServerX.sh status
```

（3）停止zookeeper服务。

```shell
${ZK_HOME}/bin/zkServerX.sh stop
```

（4）重启zookeeper服务。

```shell
${ZK_HOME}/bin/zkServerX.sh restart
```



