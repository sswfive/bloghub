---
title: kafka安装部署
date: 2023-08-14 17:50:57
tags:
  - SoftwareInstall
---
## Kafka

Kafka作为分布式消息队列主要服务于实时流计算场景，在本系统中主要用于测试模拟生产环境运行模型，后续须对接生产环境的Kakfa。

### 部署说明

- 192.168.21.73 -> node73
- 192.168.21.74 -> node74
- 192.168.21.75 -> node75

| 事项             | 详情                                        |
| ---------------- | ------------------------------------------- |
| kafka代理节点    | 192.168.21.73                               |
| zookeeper节点    | 192.168.21.73-192.168.21.75                 |
| kafka安装包位置  | /data/software/kafka_2.12-2.1.1.tgz         |
| kafka运行目录    | /data/workspace/kafka_2.12-2.1.1            |
| kafka数据目录    | /data/workspace/kafka_2.12-2.1.1/kafka-logs |
| 启停状态维护脚本 | $KAFKA_HOME/bin/kafka-server.sh             |

## Kafka部署

### step1: 解压安装包

```shell
sudo tar -zxvf /data/software/kafka_2.12-2.1.1.tgz -C /data/workspace
sudo chown -R demouser:demouser /data/workspace/kafka_2.12-2.1.1/
```

### step2: 配置 Flink 环境变量

```shell
vim ~/.bashrc
```

​     配置内容如下：

```shell
# FLINK
export KAFKA_HOME=/data/workspace/kafka_2.12-2.1.1
export PATH=$PATH:$KAFKA_HOME/bin
```

​     生效配置：

```shell
source ~/.bashrc
```

### step3: 配置server.properties文件

在${KAFKA_HOME}/config路径下配置server.properties文件，配置项如下：

```properties
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://node73:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
#advertised.listeners=PLAINTEXT://your.host.name:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/data/workspace/kafka_2.12-2.1.1/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended for to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=node73:2181,node74:2181,node75:2181/kafka-2.1.1

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
```



### step4：运维管理

例如创建一个名称为t1的主题，同时指定1个分区和1个副本。

```shell
$KAFKA_HOME/bin/kafka-topics.sh --create \
--zookeeper node73:2181,node74:2181,node75:2181/kafka-2.1.1 \
--replication-factor 1 \
--partitions 1 \
--topic t1
```

​		【注意】这里的--zookeeper必须对应$KAFKA_HOME/config/server.properties配置文件中的zookeeper.connect参数所对应的值，容易出错的地方在于如果配置了kafka注册到节点znode，那么在通过kafka-topics.sh操作主题时--zookeeper参数也必须附带上这个znode，否则会有如下异常：

```shell
ERROR org.apache.kafka.common.errors.InvalidReplicationFactorException: Replication factor: 1 larger than available brokers: 0.
 (kafka.admin.TopicCommand$)
```

查看主题

【查看单个主题】

```shell
$KAFKA_HOME/bin/kafka-topics.sh --describe \
--zookeeper node73:2181,node74:2181,node75:2181/kafka-2.1.1 \
--topic t1
Topic:t1        PartitionCount:1        ReplicationFactor:1     Configs:
        Topic: t1       Partition: 0    Leader: 2       Replicas: 2     Isr: 2
```

【列出所有主题】

```shell
$KAFKA_HOME/bin/kafka-topics.sh --list \
--zookeeper node73:2181,node74:2181,node75:2181/kafka-2.1.1
t1
```

删除主题

【方式一】

```shell
$KAFKA_HOME/bin/kafka-run-class.sh kafka.admin.TopicCommand --delete \
--zookeeper node73:2181,node74:2181,node75:2181/kafka-2.1.1 \
--topic t1
Topic t1 is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
```

【方式二】

```shell
$KAFKA_HOME/bin/kafka-topics.sh --delete \
--zookeeper node73:2181,node74:2181,node75:2181/kafka-2.1.1 \
--topic t1
```

生产消息

【使用内部网卡】

```shell
$KAFKA_HOME/bin/kafka-console-producer.sh \
--broker-list node73:9092 \
--topic t1
```

【使用外部网卡】

```shell
$KAFKA_HOME/bin/kafka-console-producer.sh \
--broker-list node73:9092 \
--topic t1
```

消费消息

【使用内部网卡】

```shell
$KAFKA_HOME/bin/kafka-console-consumer.sh \
--bootstrap-server node73:9092 \
--topic t1 \
--from-beginning
```

【使用外部网卡】

```shell
$KAFKA_HOME/bin/kafka-console-consumer.sh \
--bootstrap-server node73:9092 \
--topic t1 \
--from-beginning
```

查看消费者组

```shell
$KAFKA_HOME/bin/kafka-consumer-groups.sh \
--bootstrap-server node73:9092 \
--list
```

查看消费者组详细消费情况

```shell
$KAFKA_HOME/bin/kafka-consumer-groups.sh \
--bootstrap-server node73:9092 \
--group console-consumer-51788 \
--describe
```