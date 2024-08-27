---
title: Redis主从-哨兵部署
date: 2023-07-20 22:01:01
tags:
  - SoftwareInstall
---
## 部署说明：
| 事项 | 详情 |
| --- | --- |
| 部署节点 | 192.168.21.71（Master）、192.168.21.77（salve）、192.168.21.78（salve） |
| 部署策略 | 一主两从，主从复制、哨兵机制、无密码访问策略 |
| 服务端口号 | 6379 |
| 安装包目录 | /data/software/redis-6.2.12.tar.gz |
| 安装部署目录 | /data/workspace/redis-6.2.12 |
| 启停脚本目录 | /data/workspace/redis-6.2.12/bin/ |
| 配置文件目录 | /data/workspace/redis-6.2.12/etc/redis.conf； sentinel.conf |
| 日志文件目录 | /data/workspace/redis-6.2.12/bin/redis.log ；sentinel.log |

## 部署详情：
### 通用步骤
```bash
# step1 下载安装包
wget https://download.redis.io/releases/redis-6.2.12.tar.gz

#step2： 所有节点执行
tar xvf redis-6.2.12.tar.gz
cd redis-6.2.12 

# step3: 编译
make

# step4: 安装
cd src/
make install 

# step5: 新建配置文件目录和执行文件目录
cd /data/workspace/redis-6.2.12/
mkdir etc 
mkdir bin 

# step6: 调整配置文件目录，便于统一管理
cp redis.conf etc/
cd src/
cp mkreleasehdr.sh redis-benchmark redis-check-aof redis-cli redis-server redis-sentinel ../bin/

# step6: 启动服务，测试是否安装成功
cd /data/workspace/redis-6.2.12/bin
./redis-server /workspace/redis/redis-6.2.12/etc/redis.conf

# step7: 查看是否启动，输出redis相关进程则表示安装成功
ps aux | grep redis
```
### 主从配置

- 注意replicaof参数：只有在当前节点为salve节点时，设置主节点的IP和PORT, 若当前节点为Master节点时则无需设置。
```bash
#step1: 修改redis.conf,以下仅展示修改后的内容

bind：0.0.0.0 
port：6379 
protected-mode：no 
daemonize：yes 
logfile：./redis.log 
replicaof 172.16.10.71 6379
replica-read-only no

# 查看修改后的配置
(base) [root@node71 etc]#  egrep -v "#|^$" redis.conf 
bind 0.0.0.0
protected-mode no
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ./redis.log
databases 16
always-show-logo no
set-proc-title yes
proc-title-template "{title} {listen-addr} {server-mode}"
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
rdb-del-sync-files no
dir ./
replica-serve-stale-data yes
replica-read-only no
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-diskless-load disabled
repl-disable-tcp-nodelay no
replica-priority 100
acllog-max-len 128
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
lazyfree-lazy-user-del no
lazyfree-lazy-user-flush no
oom-score-adj no
oom-score-adj-values 0 200 800
disable-thp yes
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
jemalloc-bg-thread yes
```

- 验证主从
```bash
# node71节点
(base) [root@node71 bin]# redis-cli
127.0.0.1:6379> select 6
OK
127.0.0.1:6379[6]> set zhongche wulian
OK
127.0.0.1:6379[6]> get zhongche
"wulian"
127.0.0.1:6379[6]> 

# node77节点
(base) [root@node77 bin]# redis-cli
127.0.0.1:6379> select 6
OK
127.0.0.1:6379[6]> get zhongche
"wulian"

(base) [root@node78 bin]# redis-cli
127.0.0.1:6379> select 6
OK
127.0.0.1:6379[6]> get zhongche
"wulian"
127.0.0.1:6379[6]> 
```
### 哨兵配置
```bash
#step1: 修改sentinel.conf,以下仅展示修改后的内容
//端口默认为26379。 
port:26379 
//关闭保护模式，可以外部访问。 
protected-mode:no 
//设置为后台启动。 
daemonize:yes 
//日志文件。 
logfile:./sentinel.log 
//指定主机IP地址和端口，并且指定当有2台哨兵认为主机挂了，则对主机进行容灾切换。 
sentinel monitor mymaster 192.168.231.130 6379 2 
//当在Redis实例中开启了requirepass，这里就需要提供密码。 
sentinel auth-pass mymaster pwdtest@2019 
//这里设置了主机多少秒无响应，则认为挂了。 
sentinel down-after-milliseconds mymaster 3000 
//主备切换时，最多有多少个slave同时对新的master进行同步，这里设置为默认的1。 
snetinel parallel-syncs mymaster 1 
//故障转移的超时时间，这里设置为三分钟。 
sentinel failover-timeout mymaster 180000 

# step2:启动三个哨兵
cd /data/workspace/redis-6.2.12/bin
redis-sentinel /data/workspace/redis-6.2.12/sentinel.conf

# step3: 查看哨兵信息
redis-cli -p 26379 
info sentinel 
#>>>log
(base) [root@node71 bin]# redis-cli -p 26379
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.21.71:6379,slaves=2,sentinels=3
```
## 常用命令
```bash
# 进入工作目录
cd /data/workspace/redis-6.2.12/bin

# 启动redis服务
redis-server /data/workspace/redis-6.2.12/etc/redis.conf

# 启动哨兵服务
redis-sentinel /data/workspace/redis-6.2.12/sentinel.conf

# 关闭服务端
./redis-cli shutdown
```
