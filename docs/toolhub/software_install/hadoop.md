---
title: Hadoop安装部署
date: 2023-08-15 21:31:41
tags:
  - Installation
---
### Hadoop集群安装

  HDFS是一个分布式文件系统，用于存储和管理文件，另外，HDFS也是Hadoop生态系统一个重要的组成部分，是Hadoop的存储组件，能够提供高吞吐量的数据访问，非常适合大规模数据集上的应用。在本系统中作为分布式存储组件，主要承载平台所有文件类型数据的存储。

**部署说明**

- 192.168.21.73 -> node73
- 192.168.21.74 -> node74
- 192.168.21.75 -> node75

| 事项                  | 详情                                                         |
| --------------------- | ------------------------------------------------------------ |
| HDFS集群部署节点      | 192.168.21.73（NameNode/DataNode）192.168.21.74（NameNode/DataNode）192.168.21.75（DataNode/SecondaryNameNode） |
| YARN集群部署节点      | 192.168.21.73（ResourceManager/NodeManager）192.168.21.74（ResourceManager/NodeManager）192.168.21.75（NodeManager） |
| Hadoop安装包目录      | /data/software/hadoop-3.2.4.tar.gz                           |
| Hadoop运行目录        | /data/workspace/hadoop-3.2.4                                 |
| HDFS NameNode数据路径 | /data/workspace/hadoop-3.2.4/data/hdfs/name                  |
| HDFS DataNode数据路径 | /data/workspace/hadoop-3.2.4/data/hdfs/data                  |
| HADOOP临时数据路径    | /data/workspace/hadoop-3.2.4/data/tmp                        |
| Hadoop日志路径        | /data/workspace/hadoop-3.2.4/logs                            |
| HDFS进程路径          | /data/workspace/hadoop-3.2.4/pid                             |
| YARN进程路径          | /data/workspace/hadoop-3.2.4/pid                             |
| MAPRED进程路径        | /data/workspace/hadoop-3.2.4/pid                             |
| HDFS WebUI            | http://192.168.21.73:50070/http://192.168.21.74:50070        |
| YARN WebUI            | http://192.168.21.73:8088/http://192.168.21.74:8088          |
| 历史服务器WebUI       | http://192.168.21.75:19888                                   |

##### Hadoop部署文档

###### step1: 解压安装包

```shell
sudo tar -zxvf /data/software/hadoop-3.2.4.tar.gz -C /data/workspace/hadoop-3.2.4
sudo chown -R demouser:demouser /data/workspace
```

###### step2: 配置Hadoop环境变量

```shell
vim ~/.bashrc
```

​     配置内容如下：

```shell
# Hadoop
export HADOOP_HOME=/data/workspace/hadoop-3.2.4
export HADOOP_CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath)
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
```

​     生效配置：

```shell
source ~/.bashrc
```

###### step3: 配置core-site.xml文件

 在${HADOOP_HOME}/etc/hadoop路径下配置core-site.xml文件，配置项如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <!-- 如果是HA模式则指定hdfs的nameservice为ns 如果是非HA模式，则为hdfs://<host>:<ip>-->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://ns</value>
  </property>

  <!--指定hadoop数据存放目录-->
  <property>
     <name>hadoop.tmp.dir</name>
     <value>/data/workspace/hadoop-3.2.4/data/tmp</value>
  </property>

  <!-- Hadoop的缓冲区大小。默认为4KB，增加为128KB。 -->
  <property>
    <name>io.file.buffer.size</name>
    <value>131072</value>
  </property>

  <!-- 和系统内核中的backlog相关参数联动，但要小于net.core.somaxconn -->
  <property>
    <name>ipc.server.listen.queue.size</name>
    <value>20480</value>
  </property>

  <property>
    <name>hadoop.proxyuser.zhongtai.hosts</name>
    <value>*</value>
  </property>
  
  <property>
    <name>hadoop.proxyuser.zhongtai.groups</name>
    <value>*</value>
  </property>
    
  <!--HA模式配置-->  
  <!--指定zookeeper地址-->
  <property>
        <name>ha.zookeeper.quorum</name>
        <value>node73:2181,node74:2181,node75:2181</value>
  </property>
    
      <!-- 
        检查点被删除后的分钟数。如果为零，垃圾桶功能将被禁用。该选项可以在服务器和客户端上配置。[单位(minute)]
        如果垃圾箱被禁用服务器端，则检查客户端配置。
        如果在服务器端启用垃圾箱，则会使用服务器上配置的值，并忽略客户端配置值。 
        https://www.jianshu.com/p/1827431d4eb5
      -->
    <property>
      <name>fs.trash.interval</name>
      <value>4320</value>
    </property>

<!-- 
    垃圾检查点之间的分钟数。应该小于或等于fs.trash.interval。[单位(minute)]
    如果为零，则将该值设置为fs.trash.interval的值。每次检查指针运行时，它都会从当前创建一个新的检查点，并删除比fs.trash.interval更早创建的检查点。
    注意：通过回收站恢复误删的数据，要求时间不能超过fs.trash.interval配置的时间。生产中为了防止误删数据，建议开启HDFS的回收站机制。
  -->
    <property>
      <name>fs.trash.checkpoint.interval</name>
      <value>60</value>
    </property>

</configuration>
```

###### step4: 配置hdfs-site.xml文件

在${HADOOP_HOME}/etc/hadoop路径下配置hdfs-site.xml文件，配置项如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <!--HA模式配置-->  
  <!--指定hdfs的nameservice为ns，需要和core-site.xml中的保持一致 -->
  <property>
     <name>dfs.nameservices</name>
     <value>ns</value>
  </property>

  <!-- NameNode 同时和 DataNode 通信的线程数，默认为10，将其增大为40。-->
  <property>
    <name>dfs.namenode.handler.count</name>
    <value>40</value>
  </property>

  <!-- 当 DataNode上面的连接数超过配置中的设置时， DataNode就会拒绝连接。-->
  <property>
    <name>dfs.datanode.max.xcievers</name>
    <value>65536</value>
  </property>

  <!-- 执行 start-balancer.sh 的带宽，默认为1048576(1MB/s)，将其増大到20MB/s -->
  <property>
    <name>dfs.datanode.balance.bandwidthPerSe</name>
    <value>20485760</value>
  </property>
    
  <!-- 
     DataNode 在进行文件传输时最大线程数，最高设置为8192，如果集群中有某台 DataNode 主机的这个值比其他主机的大，
     那么出现的问题是，这台主机上存储的数据相对别的主机比较多，导致数据分布不均匀的问题，即使 balance 仍然会不均匀。 
   -->
  <property>
    <name>dfs.datanode.max.transfer.threads</name>
    <value>8192</value>
  </property>
    
  <!--HA模式配置-->    
  <!-- ns下面有两个NameNode，分别是nn1，nn2 -->
  <property>
     <name>dfs.ha.namenodes.ns</name>
     <value>nn1,nn2</value>
  </property>
    
  <!--HA模式配置-->  
  <!-- nn1的RPC通信地址 -->
  <property>
     <name>dfs.namenode.rpc-address.ns.nn1</name>
     <value>node73:9000</value>
  </property>
   
  <!--HA模式配置-->    
  <!-- nn1的http通信地址 -->
  <property>
     <name>dfs.namenode.http-address.ns.nn1</name>
     <value>node73:50070</value>
  </property>
  <!--HA模式配置-->  
  <!-- nn2的RPC通信地址 -->
  <property>
     <name>dfs.namenode.rpc-address.ns.nn2</name>
     <value>node74:9000</value>
  </property>
    
  <!-- nn2的http通信地址 -->
  <property>
     <name>dfs.namenode.http-address.ns.nn2</name>
     <value>node74:50070</value>
  </property>
  <!--HA模式配置-->  
  <!-- 指定NameNode的元数据在JournalNode上的存放位置 -->
  <property>
     <name>dfs.namenode.shared.edits.dir</name>
     <value>qjournal://node73:8485;node74:8485;node75:8485;node76:8485/ns</value>
  </property>
  <!--HA模式配置-->  
  <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
  <property>
     <name>dfs.journalnode.edits.dir</name>
     <value>/data/workspace/hadoop-3.2.4/data/journal</value>
  </property>

  <!-- 指定NameNode在本地磁盘存放数据的位置 -->
  <property>
     <name>dfs.namenode.name.dir</name>
     <value>file:///data/workspace/hadoop-3.2.4/data/hdfs/name</value>
  </property>
    
  <!-- 指定DataNode在本地磁盘存放数据的位置 -->
  <property>
     <name>dfs.datanode.data.dir</name>
     <value>file:///data/workspace/hadoop-3.2.4/data/hdfs/data</value>
  </property>

  <property>
    <name>dfs.datanode.fsdataset.volume.choosing.policy</name>
    <value>org.apache.hadoop.hdfs.server.datanode.fsdataset.AvailableSpaceVolumeChoosingPolicy</value>
  </property>
  <!--HA模式配置-->  
  <!-- 开启NameNode故障时自动切换 -->
  <property>
     <name>dfs.ha.automatic-failover.enabled</name>
     <value>true</value>
  </property>
  <!--HA模式配置-->  
  <!-- 配置失败自动切换实现方式 -->
  <property>
     <name>dfs.client.failover.proxy.provider.ns</name>
     <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
    
  <property>
     <name>dfs.permissions.enabled</name>
     <value>false</value>
  </property>

  <!-- 注意：如果是伪分布式只能配置为1 -->
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>

  <property>
    <name>dfs.ha.fencing.methods</name>
    <value>
      sshfence
      shell(/bin/true)
    </value>
  </property>
</configuration>
```

###### step5: 配置hadoop-env.sh文件

在${HADOOP_HOME}/etc/hadoop路径下配置hadoop-env.sh文件，配置项如下：

```shell
export JAVA_HOME=/data/workspace/jdk1.8.0_201

# 修改hadooplog、pid、jvm的配置
export HADOOP_PID_DIR=/data/workspace/hadoop-3.2.4/pid
export HADOOP_MAPRED_PID_DIR=/data/workspace/hadoop-3.2.4/pid
export HADOOP_LOG_DIR=/data/workspace/hadoop-3.2.4/log

# JVM
export HDFS_NAMENODE_OPTS="-Dhadoop.security.logger=INFO,RFAS -Xmx4096m"
export HDFS_DATANODE_OPTS="-Dhadoop.security.logger=ERROR,RFAS -Xmx4096m"
export HDFS_JOURNALNODE_OPTS="-Xmx4096m"
```

###### step6: 修改hadoop pid、jvm的配置

 在${HADOOP_HOME}/etc/hadoop路径下配置 yarn-env.sh 文件，配置项如下：

```shell
export YARN_PID_DIR=/data/workspace/hadoop-3.2.4/pid

# JVM
export YARN_RESOURCEMANAGER_OPTS="-Drm.audit.logger=INFO,RMAUDIT -Xmx4096m"
export YARN_NODEMANAGER_OPTS="-Dnm.audit.logger=INFO,NMAUDIT -Xmx4096m"
```

在${HADOOP_HOME}/etc/hadoop路径下配置 mapred-env.sh 文件，配置项如下：

```shell
export HADOOP_MAPRED_PID_DIR=/data/workspace/hadoop-3.2.4/pid
# JVM
export MAPRED_HISTORYSERVER_OPTS="-Xmx4096m"
```

###### step7: 配置DataNode节点列表

在${HADOOP_HOME}/etc/hadoop/workers 文件中配置DataNode节点列表，内容如下：

```shell
node73
node74
node75
node76
```

###### step8: 创建目录

```
mkdir /data/workspace/hadoop-3.2.4/pid
mkdir /data/workspace/hadoop-3.2.4/log
mkdir -p /data/workspace/hadoop-3.2.4/data/journal
mkdir -p /data/workspace/hadoop-3.2.4/data/tmp
mkdir -p /data/workspace/hadoop-3.2.4/data/hdfs/data
mkdir -p /data/workspace/hadoop-3.2.4/data/hdfs/name
```



###### step9: 将Hadoop跟目录分发至各服务器节点

```shell
sudo scp -r /data/workspace/hadoop-3.2.4/ root@node74:/data/workspace/
sudo scp -r /data/workspace/hadoop-3.2.4/ root@node75:/data/workspace/
sudo scp -r /data/workspace/hadoop-3.2.4/ root@node76:/data/workspace/

# 分别进入node74、node75、node76修改hadoop-3.2.4所属用户组和所属用户
sudo chown -R demouser:demouser /data/workspace/hadoop-3.2.4

# 分别进入node74、node75、node76添加环境变量

vim ~/.bashrc

配置内容如下：

export HADOOP_HOME=/data/workspace/hadoop-3.2.4
export HADOOP_CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath)
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin

生效配置：

source ~/.bashrc
```

###### step10: 启动服务

Hadoop HA模式在首次启动时要保证zk是启动状态，然后一定需要注意顺序，主要步骤如下：

①在某一个namenode节点执行如下命令，创建命名空间，比如这里我们对 node73 节点进行如下操作。

```shell
hdfs zkfc -formatZK
```

②在每个journalnode节点用如下命令启动journalnode（进程名为JournalNode），我们在node73,node74,node75这三个节点进行如下操作。

```shell
${HADOOP_HOME}/sbin/hadoop-daemon.sh start journalnode
```

③在主namenode节点格式化namenode和journalnode目录，这里我们对node73做操作。

```shell
hdfs namenode -format ns
```

④在主namenode节点启动namenode进程，这里对node73做操作。

```shell
${HADOOP_HOME}/sbin/hadoop-daemon.sh start namenode
```

⑤在备namenode节点执行如下命令，在node74做操作。

```shell
# 把备namenode节点的目录格式化并把元数据从主namenode节点copy过来，并且这个命令不会把journalnode目录再格式化了！
hdfs namenode -bootstrapStandby
# 启动备namenode进程
${HADOOP_HOME}/sbin/hadoop-daemon.sh start namenode
```

⑥在两个namenode节点都执行以下命令，启动zkfc（进程名为DFSZKFailoverController），在node73,node74两个节点做操作。

```shell
${HADOOP_HOME}/sbin/hadoop-daemon.sh start zkfc
```

⑦在所有datanode节点都执行以下命令启动datanode。

```shell
${HADOOP_HOME}/sbin/hadoop-daemon.sh start datanode
```

 ⑧配置完HA之后可以下查看下两台NameNode的状态是否正常，命令如下：

```shell
[demouser@node73 flink-1.15.3]$ hdfs haadmin -getServiceState nn1
standby
[demouser@node73 flink-1.15.3]$ hdfs haadmin -getServiceState nn2
active
```



##### HDFS运维命令

（1）启停存储管理进程。

```shell
${HADOOP_HOME}/sbin/hadoop-daemon.sh stop datanode
${HADOOP_HOME}/sbin/hadoop-daemon.sh start datanode
```

（2）启停元数据管理进程。

```shell
${HADOOP_HOME}/sbin/hadoop-daemon.sh stop namenode
${HADOOP_HOME}/sbin/hadoop-daemon.sh start namenode
```

（3）启停HDFS集群。

```shell
${HADOOP_HOME}/sbin/stop-dfs.sh
${HADOOP_HOME}/sbin/start-dfs.sh
```

（4）查看HDFS HA NameNode状态。

```shell
hdfs haadmin -getServiceState nn1
hdfs haadmin -getServiceState nn2
```

（5）查看NameNode日志。

```shell
tail -1000f ${HADOOP_HOME}/logs/hadoop-xxx-namenode-xxx.log
```

（6）查看DataNode日志。

```shell
tail -1000f ${HADOOP_HOME}/logs/hadoop-xxx-datanode-xxx.log
```

