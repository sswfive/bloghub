---
title: Spark(ON YARN）安装部署
date: 2023-08-15 21:41:28
tags:
  - SoftwareInstall
---
## Spark On YARN

  Spark是一种通用的分布式大数据计算引擎，其具有高可靠性、高扩展性、高效性以及高容错性等特点。Spark作为分布式计算引擎主要为上层服务提供更加便利数据操作，该组件主要适用于大多数批处理任务的场景。在本系统中主要用于支撑Spark模型的运行和计算。

### 部署说明

- 192.168.21.73 -> node73
- 192.168.21.74 -> node74
- 192.168.21.75 -> node75

| 事项       | 详情                                         |
| ---------- | -------------------------------------------- |
| 部署节点   | 192.168.21.73-192.168.21.75                  |
| 安装包位置 | /data/software/spark-3.2.3-bin-hadoop3.2.tgz |
| 运行目录   | /data/workspace/spark-3.2.3-bin-hadoop3.2    |

### Spark部署

#### step1: 解压安装包

```shell
sudo tar -zxvf /data/software/spark-3.2.3-bin-hadoop3.2.tgz -C /data/workspace
sudo chown -R demouser:demouser /data/workspace/spark-3.2.3-bin-hadoop3.2
```

#### step2: 配置环境变量

```shell
vim ~/.bashrc
```

配置内容如下：

```shell
# Spark
export SPARK_HOME=/data/workspace/spark-3.2.3-bin-hadoop3.2
export PATH=$PATH:$SPARK_HOME/bin
```

​		执行如下命令,使配置生效：

```bash
source ~/.bashrc
```



#### step3: 配置相关文件

​		【spark-defaults.conf】

```
cp $SPARK_HOME/conf/spark-defaults.conf.template $SPARK_HOME/conf/spark-defaults.conf
vim $SPARK_HOME/conf/spark-defaults.conf
```

```shell
spark.master.rest.enabled        true
spark.master.rest.port           16066
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://ns/spark_3.2.3/history
spark.history.fs.logDirectory    hdfs://ns/spark_3.2.3/history
spark.eventLog.compress         true
spark.history.ui.port           18888
spark.shuffle.service.enabled   true
spark.shuffle.service.port      7337

# 使用yarn的方式提交spark应用时,配置spark.yarn.jars并且上传了程序所有的依赖jar到HDFS /spark-3.2.3_jars/ 下的方式更能减少资源的上传
spark.yarn.jars        hdfs://ns/spark-3.2.3_jars/*
```

​		

​		【spark-env.sh】

```shell
cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
vim $SPARK_HOME/conf/spark-env.sh
```

```shell
export JAVA_HOME=/data/workspace/jdk1.8.0_201
export HADOOP_HOME=/data/workspace/hadoop-3.2.4
export HADOOP_CONF_DIR=/data/workspace/hadoop-3.2.4/etc/hadoop

# HA模式下配置
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=node73:2181,node74:2181,node75:2181 -Dspark.deploy.zookeeper.dir=/spark-3.2.3"

# worker进程工作目录
export SPARK_WORKER_DIR=/data/workspace/spark-3.2.3-bin-hadoop3.2/work
# 配置自动自动清理，清理周期以及保留最近多长时间的数据
export SPARK_WORKER_OPTS="-Dspark.worker.cleanup.enabled=true -Dspark.worker.cleanup.interval=86400 -Dspark.worker.cleanup.appDataTtl=2592000"

export SPARK_MASTER_WEBUI_PORT=18080
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18888 -Dspark.history.retainedApplications=30 -Dspark.history.fs.logDirectory=hdfs://ns/spark_3.2.3/history"

# job任务在运行过程中产生大量的临时目录位置
export SPARK_LOCAL_DIRS=/data/workspace/spark-3.2.3-bin-hadoop3.2/tmp

export YARN_CONF_DIR=/data/workspace/hadoop-3.2.4/etc/hadoop

# 分配给spark master/worker守护进程的内存
export SPARK_DAEMON_MEMORY=2g
```



#### step4: 修改log目录、pid目录

​		【spark-daemon.sh】

```shell
vim $SPARK_HOME/sbin/spark-daemon.sh
```

```shell
# get log directory
if [ "$SPARK_LOG_DIR" = "" ]; then
  export SPARK_LOG_DIR="/data/workspace/spark-3.2.3-bin-hadoop3.2/log"      # log目录
fi
mkdir -p "$SPARK_LOG_DIR"
touch "$SPARK_LOG_DIR"/.spark_test > /dev/null 2>&1
TEST_LOG_DIR=$?
if [ "${TEST_LOG_DIR}" = "0" ]; then
  rm -f "$SPARK_LOG_DIR"/.spark_test
else
  chown "$SPARK_IDENT_STRING" "$SPARK_LOG_DIR"
fi

if [ "$SPARK_PID_DIR" = "" ]; then
  SPARK_PID_DIR=/data/workspace/spark-3.2.3-bin-hadoop3.2/pid               # pid目录
fi
```

#### step5: 创建相关目录

```
mkdir /data/workspace/spark-3.2.3-bin-hadoop3.2/tmp
mkdir /data/workspace/spark-3.2.3-bin-hadoop3.2/log
mkdir /data/workspace/spark-3.2.3-bin-hadoop3.2/pid
mkdir /data/workspace/spark-3.2.3-bin-hadoop3.2/work
hdfs dfs -mkdir -p /spark_3.2.3/history
hdfs dfs -mkdir /spark-3.2.3_jars
hdfs dfs -put /data/workspace/spark-3.2.3-bin-hadoop3.2/jars/* /spark-3.2.3_jars
```

#### step6: 将Spark跟目录分发至对应的节点

```shell
sudo scp -r /data/workspace/spark-3.2.3-bin-hadoop3.2/ root@node74:/data/workspace/
sudo scp -r /data/workspace/spark-3.2.3-bin-hadoop3.2/ root@node75:/data/workspace/
sudo scp -r /data/workspace/spark-3.2.3-bin-hadoop3.2/ root@node76:/data/workspace/

# 分别进入node74、node75、node76修改hadoop-3.2.4所属用户组和所属用户
sudo chown -R zhongche:zhongche /data/workspace/spark-3.2.3-bin-hadoop3.2

# 分别进入node74、node75、node76添加环境变量

vim ~/.bashrc

配置内容如下：

export SPARK_HOME=/data/workspace/spark-3.2.3-bin-hadoop3.2
export PATH=$PATH:$SPARK_HOME/bin

生效配置：

source ~/.bashrc
```



### SPARK ON YARN环境配置

#### step1: 在yarn-site.xml中添加如下配置

```shell
vim $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

```xml
# 修改
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>spark_shuffle,mapreduce_shuffle</value>
</property>
# 添加
<property>
    <name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
    <value>org.apache.spark.network.yarn.YarnShuffleService</value>
</property>
```

检查一下**${HADOOP_HOME}/share/hadoop/yarn/lib**目录下是否有spark-\*-yarn-shuffle.jar，其中\*代表spark版本号，如果没有需要从spark的安装目录下拷贝过来。

**spark-\*-yarn-shuffle.jar在$SPARK_HOME的yarn目录下**

```shell
cp $SPARK_HOME/yarn/spark-3.2.3-yarn-shuffle.jar ${HADOOP_HOME}/share/hadoop/yarn/lib
```

将修改的文件分发至node74,node75

```shell
scp $HADOOP_HOME/etc/hadoop/yarn-site.xml zhongche@node74:$HADOOP_HOME/etc/hadoop/
scp $HADOOP_HOME/etc/hadoop/yarn-site.xml zhongche@node75:$HADOOP_HOME/etc/hadoop/
scp ${HADOOP_HOME}/share/hadoop/yarn/lib/spark-3.2.3-yarn-shuffle.jar zhongche@node74:${HADOOP_HOME}/share/hadoop/yarn/lib/
scp ${HADOOP_HOME}/share/hadoop/yarn/lib/spark-3.2.3-yarn-shuffle.jar zhongche@node75:${HADOOP_HOME}/share/hadoop/yarn/lib/
```

重启YARN服务

```
${HADOOP_HOME}/sbin/stop-yarn.sh
${HADOOP_HOME}/sbin/start-yarn.sh
```



测试代码：

```shell
$SPARK_HOME/bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
$SPARK_HOME/examples/jars/spark-examples_2.12-3.2.3.jar \
100
```

```shell
2023-07-17 14:10:38,643 INFO cluster.YarnScheduler: Removed TaskSet 0.0, whose tasks have all completed, from pool 
2023-07-17 14:10:38,644 INFO scheduler.DAGScheduler: ResultStage 0 (reduce at SparkPi.scala:38) finished in 1.605 s
2023-07-17 14:10:38,649 INFO scheduler.DAGScheduler: Job 0 is finished. Cancelling potential speculative or zombie tasks for this job
2023-07-17 14:10:38,650 INFO cluster.YarnScheduler: Killing all running tasks in stage 0: Stage finished
2023-07-17 14:10:38,652 INFO scheduler.DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 1.665838 s
Pi is roughly 3.1417683141768316
2023-07-17 14:10:38,671 INFO server.AbstractConnector: Stopped Spark@54d80039{HTTP/1.1, (http/1.1)}{0.0.0.0:4040}
2023-07-17 14:10:38,674 INFO ui.SparkUI: Stopped Spark web UI at http://node73:4040
2023-07-17 14:10:38,678 INFO cluster.YarnClientSchedulerBackend: Interrupting monitor thread
2023-07-17 14:10:38,696 INFO cluster.YarnClientSchedulerBackend: Shutting down all executors
2023-07-17 14:10:38,697 INFO cluster.YarnSchedulerBackend$YarnDriverEndpoint: Asking each executor to shut down
2023-07-17 14:10:38,703 INFO cluster.YarnClientSchedulerBackend: YARN client scheduler backend Stopped
2023-07-17 14:10:38,777 INFO spark.MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
2023-07-17 14:10:38,789 INFO memory.MemoryStore: MemoryStore cleared
2023-07-17 14:10:38,789 INFO storage.BlockManager: BlockManager stopped
2023-07-17 14:10:38,810 INFO storage.BlockManagerMaster: BlockManagerMaster stopped
2023-07-17 14:10:38,812 INFO scheduler.OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
2023-07-17 14:10:38,818 INFO spark.SparkContext: Successfully stopped SparkContext
2023-07-17 14:10:38,820 INFO util.ShutdownHookManager: Shutdown hook called
2023-07-17 14:10:38,821 INFO util.ShutdownHookManager: Deleting directory /tmp/spark-b2b4040b-04ce-403c-8a10-d1dffce37fa4
2023-07-17 14:10:38,824 INFO util.ShutdownHookManager: Deleting directory /data/workspace/spark-3.2.3-bin-hadoop3.2/tmp/spark-0f7c40fc-1111-4eed-8e2a-77b0d7598e7c
```

