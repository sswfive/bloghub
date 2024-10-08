---
title: Flink(On YARN)安装部署
date: 2023-08-15 21:46:44
tags:
  - SoftwareInstall
---
## Flink On YARN

  Flink是一个批处理和流处理结合的分布式计算框架，其核心是提供了数据分发以及并行化计算的流处理引擎，作为分布式流式计算引擎主要为上层服务提供更加实时性数据操作，该组件主要适用于数据低延时处理。在本系统中主要用于支撑Flink模型的运行和计算。

### 部署说明

| 事项       | 详情                                           |
| ---------- | ---------------------------------------------- |
| 部署节点   | 192.168.21.73-192.168.21.75                    |
| 安装包位置 | /data/software/flink-1.15.3-bin-scala_2.12.tgz |
| 运行目录   | /data/workspace/flink-1.15.3                   |

### Flink部署

#### step1: 解压安装包

```shell
sudo tar -zxvf /data/software/flink-1.15.3-bin-scala_2.12.tgz -C /data/workspace
sudo chown -R zhongche:zhongche /data/workspace/flink-1.15.3/
```

#### step2: 配置 Flink 环境变量

```shell
vim ~/.bashrc
```

​     配置内容如下：

```shell
# FLINK
export FLINK_HOME=/data/workspace/flink-1.15.3
export PATH=$PATH:$FLINK_HOME/bin
```

​     生效配置：

```shell
source ~/.bashrc
```

#### step3: 配置flink-conf.yaml文件

在${FLINK_HOME}/conf 路径下配置flink-conf.yaml文件，配置项如下：

```shell
jobmanager.rpc.address: node73

jobmanager.rpc.port: 6123

jobmanager.bind-host: 0.0.0.0

jobmanager.memory.process.size: 2g

taskmanager.bind-host: 0.0.0.0

taskmanager.host: node73

taskmanager.memory.process.size: 4g

taskmanager.numberOfTaskSlots: 4

parallelism.default: 4

# JVM Metaspace Memory --- Task Manager JVM的元空间内存大小，默认 256m
taskmanager.memory.jvm-metaspace.size: 512m

taskmanager.memory.jvm-overhead.fraction: 0.2
task.cancellation.timeout: 0

execution.target: yarn-per-job

high-availability: zookeeper

high-availability.storageDir: hdfs://ns/flink_1.15/yarn/ha

high-availability.zookeeper.quorum: node73:2181,node74:2181,node75:2181

high-availability.zookeeper.path.root: /flink_1.15_yarn

high-availability.cluster-id: /flink_1.15

state.backend: rocksdb

state.checkpoints.dir: hdfs://ns/flink_1.15/checkpoints

state.savepoints.dir: hdfs://ns/flink_1.15/savepoints

# 最多保留已完成的检查点实例的数量
state.checkpoints.num-retained: 100
state.backend.incremental: true

akka.ask.timeout: 100s
web.timeout: 300000

taskmanager.debug.memory.log: true
taskmanager.debug.memory.log-interval: 10000
taskmanager.network.netty.transport: "epoll"

jobmanager.execution.failover-strategy: region

rest.address: localhost

rest.bind-port: 50100-50200

rest.bind-address: 0.0.0.0

historyserver.web.address: node73
historyserver.archive.clean-expired-jobs: true

# JobManager归档的作业信息所存放的目录
jobmanager.archive.fs.dir: hdfs://ns/flink_1.15/completed-jobs/

# 指定History Server扫描哪些归档目录
historyserver.archive.fs.dir: hdfs://ns/flink_1.15/completed-jobs/

# 指定History Server间隔多少毫秒扫描一次归档目录
historyserver.archive.fs.refresh-interval: 60000

classloader.check-leaked-classloader: false
```



#### step4: 在$FLINK_HOME/bin/flink中添加如下配置

```shell
export HADOOP_CLASSPATH=`hadoop classpath`
```



#### step5: 将Flink 跟目录分发至各服务器节点

```shell
sudo scp -r /data/workspace/flink-1.15.3/ root@node74:/data/workspace
sudo scp -r /data/workspace/flink-1.15.3/ root@node75:/data/workspace
sudo scp -r /data/workspace/flink-1.15.3/ root@node76:/data/workspace

# 分别进入node74、node75、node76修改hadoop-3.2.4所属用户组和所属用户
sudo chown -R zhongche:zhongche /data/workspace/flink-1.15.3

# 分别进入node74、node75、node76添加环境变量

vim ~/.bashrc

配置内容如下：

export FLINK_HOME=/data/workspace/flink-1.15.3
export PATH=$PATH:$FLINK_HOME/bin

生效配置：

source ~/.bashrc
```



#### step6: 创建相关目录 

```shell
[zhongche@node73 flink-1.15.3]$ hdfs dfs -mkdir -p /flink_1.15/checkpoints
[zhongche@node73 flink-1.15.3]$ hdfs dfs -mkdir -p /flink_1.15/savepoints
[zhongche@node73 flink-1.15.3]$ hdfs dfs -mkdir -p /flink_1.15/completed-jobs
[zhongche@node73 flink-1.15.3]$ hdfs dfs -mkdir -p /flink_1.15/yarn/ha
```



测试代码：

```shell
$FLINK_HOME/bin/flink run -m yarn-cluster -yjm 1024 -ytm 1024 $FLINK_HOME/examples/batch/WordCount.jar
```

```shell
[zhongche@node73 flink-1.15.3]$ $FLINK_HOME/bin/flink run -m yarn-cluster -yjm 1024 -ytm 1024 /data/workspace/flink-1.15.3/examples/batch/WordCount.jar 
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/data/workspace/flink-1.15.3/lib/log4j-slf4j-impl-2.17.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/data/workspace/hadoop-3.2.4/share/hadoop/common/lib/slf4j-reload4j-1.7.35.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Executing WordCount example with default input data set.
Use --input to specify file input.
Printing result to stdout. Use --output to specify output path.
2023-07-17 10:46:25,896 WARN  org.apache.flink.yarn.configuration.YarnLogConfigUtil        [] - The configuration directory ('/data/workspace/flink-1.15.3/conf') already contains a LOG4J config file.If you want to use logback, then please delete or rename the log configuration file.
2023-07-17 10:46:26,153 INFO  org.apache.flink.yarn.YarnClusterDescriptor                  [] - No path for the flink jar passed. Using the location of class org.apache.flink.yarn.YarnClusterDescriptor to locate the jar
2023-07-17 10:46:26,166 WARN  org.apache.flink.yarn.YarnClusterDescriptor                  [] - Job Clusters are deprecated since Flink 1.15. Please use an Application Cluster/Application Mode instead.
2023-07-17 10:46:26,298 INFO  org.apache.hadoop.conf.Configuration                         [] - resource-types.xml not found
2023-07-17 10:46:26,299 INFO  org.apache.hadoop.yarn.util.resource.ResourceUtils           [] - Unable to find 'resource-types.xml'.
2023-07-17 10:46:26,346 INFO  org.apache.flink.yarn.YarnClusterDescriptor                  [] - Cluster specification: ClusterSpecification{masterMemoryMB=2048, taskManagerMemoryMB=4096, slotsPerTaskManager=4}
2023-07-17 10:46:31,295 INFO  org.apache.flink.yarn.YarnClusterDescriptor                  [] - Submitting application master application_1689415931790_0004
2023-07-17 10:46:31,537 INFO  org.apache.hadoop.yarn.client.api.impl.YarnClientImpl        [] - Submitted application application_1689415931790_0004
2023-07-17 10:46:31,537 INFO  org.apache.flink.yarn.YarnClusterDescriptor                  [] - Waiting for the cluster to be allocated
2023-07-17 10:46:31,539 INFO  org.apache.flink.yarn.YarnClusterDescriptor                  [] - Deploying cluster, current state ACCEPTED
2023-07-17 10:46:38,338 INFO  org.apache.flink.yarn.YarnClusterDescriptor                  [] - YARN application has been deployed successfully.
2023-07-17 10:46:38,339 INFO  org.apache.flink.yarn.YarnClusterDescriptor                  [] - Found Web Interface node74:50100 of application 'application_1689415931790_0004'.
Job has been submitted with JobID c0245edf54e4eb4a320535316cb1e030
Program execution finished
Job with JobID c0245edf54e4eb4a320535316cb1e030 has finished.
Job Runtime: 10739 ms
Accumulator Results: 
- 62999036f072a117ac8c6cf197e3bc4e (java.util.ArrayList) [170 elements]
```

