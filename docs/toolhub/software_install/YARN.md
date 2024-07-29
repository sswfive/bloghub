---
title: YARN集群安装部署
date: 2023-08-15 21:38:24
tags:
  - Installation
---
## YARN集群安装

### step1: 修改mapred-site.xml文件

在 ${HADOOP_HOME}/etc/hadoop 路径下配置 mapred-site.xml 文件，配置项如下：

```xml
<configuration>
  <!-- mr程序默认运行方式:yarn-集群模式,local-本地模式  -->
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>

  <!-- jobhistory 服务配置 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>node75:10020</value>
  </property>
  <!-- jobhistory web ui 访问端口 -->
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>node75:19888</value>
  </property>

  <property>
    <name>mapreduce.application.classpath</name>
    <value>$HADOOP_HOME/share/hadoop/mapreduce/*,$HADOOP_HOME/share/hadoop/mapreduce/lib/*</value>
  </property> 

  <!--MR App Master环境变量 -->
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
  </property>
  <!--MR MapTask环境变量 -->
  <property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
  </property>
  <!--MR ReduceTask环境变量 -->
  <property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
  </property>
```

### step2: 修改yarn-site.xml文件

在 ${HADOOP_HOME}/etc/hadoop 路径下配置 yarn-site.xml 文件，配置项如下：

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
  <property>
     <name>yarn.nodemanager.aux-services</name>
     <value>mapreduce_shuffle</value>
  </property>

  <!--HA模式配置-->
  <!-- 开启RM高可用 -->
  <property>
     <name>yarn.resourcemanager.ha.enabled</name>
     <value>true</value>
  </property>
  <!--HA模式配置-->
  <!-- 指定RM的cluster id -->
  <property>
    <name>yarn.resourcemanager.cluster-id</name>
    <value>cluster</value>
  </property>
  <!--HA模式配置-->
  <!-- 指定RM的名字 -->
  <property>
    <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
  </property>
  <!--HA模式配置-->
  <!-- 分别指定RM的地址 -->
  <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>node73</value>
  </property>
  <!--HA模式配置-->
  <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>node74</value>
  </property>
  <!-- rm1 WebUI地址 -->
  <property>
     <name>yarn.resourcemanager.webapp.address.rm1</name>
     <value>node73:8088</value>
  </property>
  <!-- rm2 WebUI地址 -->
  <property>
     <name>yarn.resourcemanager.webapp.address.rm2</name>
     <value>node74:8088</value>
  </property>
  <!-- 开启自动故障转移 -->
  <property>
     <name>yarn.resourcemanager.ha.automatic-failover.enabled</name>
     <value>true</value>
  </property>
  <!-- zk集群地址 -->
  <property>
     <name>hadoop.zk.address</name>
     <value>node73:2181,node74:2181,node75:2181</value>
  </property>

  <!-- 开启RM重启状态恢复机制 -->
  <property>
     <name>yarn.resourcemanager.recovery.enabled</name>
     <value>true</value>
  </property>
  <!-- 状态存储采用ZK -->
  <property>
     <name>yarn.resourcemanager.store.class</name>
     <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
  </property>
  <!--ZK保存的已完成任务的最大数量-->
  <property>
    <name>yarn.resourcemanager.state-store.max-completed-applications</name>
    <value>100</value>
  </property>
  <!--RM内存中保存的已完成任务的最大数量，调整该参数主要是为了RM内存与ZK中保存的任务信息和数量一致-->
  <property>
    <name>yarn.resourcemanager.max-completed-applications</name>
    <value>100</value>
  </property>
  <!-- 定义了ZK的ZNode节点所能存储的最大数据量，以字节为单位，默认是1048576字节，也就是1MB，这里改成4MB -->
  <property>
    <name>yarn.resourcemanager.zk-max-znode-size.bytes</name>
    <value>4194304</value>
  </property>

  <!-- 容器虚拟内存与物理内存之间的比率，默认：2.1 -->
  <property>
     <name>yarn.nodemanager.vmem-pmem-ratio</name>
     <value>2.1</value>
  </property>

  <!-- 开启yarn日志聚集功能，收集每个容器的日志集中存储在一个地方，默认：false -->
  <property>
     <name>yarn.log-aggregation-enable</name>
     <value>true</value>
  </property>
  <!-- 日志保留时间设置为一个月 -->
  <property>
     <name>yarn.log-aggregation.retain-seconds</name>
     <value>2592000</value>
  </property>
  <property>
     <name>yarn.log.server.url</name>
     <value>http://node75:19888/jobhistory/logs</value>
  </property>
 <!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
  <property>
     <name>yarn.nodemanager.pmem-check-enabled</name>
     <value>false</value>
  </property>
  <!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
  <property>
      <name>yarn.nodemanager.vmem-check-enabled</name>
      <value>false</value>
  </property>
</configuration>
```

### step3: 修改capacity-scheduler.xml文件

在${HADOOP_HOME}/etc/hadoop路径下配置 capacity-scheduler.xml 文件，配置项如下：

```xml
  <property>
    <name>yarn.scheduler.capacity.resource-calculator</name>
    <!-- <value>org.apache.hadoop.yarn.util.resource.DefaultResourceCalculator</value> -->
    <value>org.apache.hadoop.yarn.util.resource.DominantResourceCalculator</value>
    <description>
      The ResourceCalculator implementation to be used to compare
      Resources in the scheduler.
      The default i.e. DefaultResourceCalculator only uses Memory while
      DominantResourceCalculator uses dominant-resource to compare
      multi-dimensional resources such as Memory, CPU etc.
    </description>
  </property>
```



## YARN运维命令

（1）启动Yarn集群，检查RM和NM是否启动成功。

```shell
${HADOOP_HOME}/sbin/start-yarn.sh

${HADOOP_HOME}/sbin/stop-yarn.sh
```

（2）启动JobHistoryServer历史服务器。

```shell
${HADOOP_HOME}/sbin/mr-jobhistory-daemon.sh start historyserver

${HADOOP_HOME}/sbin/mr-jobhistory-daemon.sh stop historyserver
```

​        启动之后检查两台机器上是否都启动RM，同时可以通过如下命令查看这两机器的active和standby分别是哪个节点。

```shell
yarn rmadmin -getAllServiceState
```

​	     输出结果如下：

```shell
node73:8033                                        standby   
node74:8033                                        active   
```

​		这里注意的是，active和standby两个节点均可以通过默认的8088端口去访问Yarn WebUI控制台，只不过当访问standby节点的WebUI时会自动跳转active节点的WebUI。

（3）运行测试用例。

向Yarn集群提交一个蒙特卡略计算圆周率的MapReducer程序。

```shell
yarn jar ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.4.jar pi 2 2 
```

日志如下：

```shell
Number of Maps  = 2
Samples per Map = 2
Wrote input for Map #0
Wrote input for Map #1
Starting Job
22/02/22 16:11:14 INFO client.RMProxy: Connecting to ResourceManager at bigdata03/172.16.10.205:8032
22/02/22 16:11:14 INFO input.FileInputFormat: Total input files to process : 2
22/02/22 16:11:15 INFO mapreduce.JobSubmitter: number of splits:2
22/02/22 16:11:15 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1645515567477_0001
22/02/22 16:11:15 INFO conf.Configuration: resource-types.xml not found
22/02/22 16:11:15 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
22/02/22 16:11:15 INFO resource.ResourceUtils: Adding resource type - name = memory-mb, units = Mi, type = COUNTABLE
22/02/22 16:11:15 INFO resource.ResourceUtils: Adding resource type - name = vcores, units = , type = COUNTABLE
22/02/22 16:11:15 INFO impl.YarnClientImpl: Submitted application application_1645515567477_0001
22/02/22 16:11:15 INFO mapreduce.Job: The url to track the job: http://bigdata03:8088/proxy/application_1645515567477_0001/
22/02/22 16:11:15 INFO mapreduce.Job: Running job: job_1645515567477_0001
22/02/22 16:11:20 INFO mapreduce.Job: Job job_1645515567477_0001 running in uber mode : false
22/02/22 16:11:20 INFO mapreduce.Job:  map 0% reduce 0%
22/02/22 16:11:23 INFO mapreduce.Job:  map 100% reduce 0%
22/02/22 16:11:28 INFO mapreduce.Job:  map 100% reduce 100%
22/02/22 16:11:29 INFO mapreduce.Job: Job job_1645515567477_0001 completed successfully
22/02/22 16:11:29 INFO mapreduce.Job: Counters: 49
        File System Counters
                FILE: Number of bytes read=50
                FILE: Number of bytes written=635598
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=512
                HDFS: Number of bytes written=215
                HDFS: Number of read operations=11
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=3
        Job Counters
                Launched map tasks=2
                Launched reduce tasks=1
                Data-local map tasks=2
                Total time spent by all maps in occupied slots (ms)=2482
                Total time spent by all reduces in occupied slots (ms)=1971
                Total time spent by all map tasks (ms)=2482
                Total time spent by all reduce tasks (ms)=1971
                Total vcore-milliseconds taken by all map tasks=2482
                Total vcore-milliseconds taken by all reduce tasks=1971
                Total megabyte-milliseconds taken by all map tasks=2541568
                Total megabyte-milliseconds taken by all reduce tasks=2018304
        Map-Reduce Framework
                Map input records=2
                Map output records=4
                Map output bytes=36
                Map output materialized bytes=56
                Input split bytes=276
                Combine input records=0
                Combine output records=0
                Reduce input groups=2
                Reduce shuffle bytes=56
                Reduce input records=4
                Reduce output records=0
                Spilled Records=8
                Shuffled Maps =2
                Failed Shuffles=0
                Merged Map outputs=2
                GC time elapsed (ms)=99
                CPU time spent (ms)=1280
                Physical memory (bytes) snapshot=814084096
                Virtual memory (bytes) snapshot=7487442944
                Total committed heap usage (bytes)=548405248
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=236
        File Output Format Counters
                Bytes Written=97
Job Finished in 15.741 seconds
Estimated value of Pi is 4.00000000000000000000
```

​		运行成功！