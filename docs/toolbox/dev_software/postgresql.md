---
title: PostgreSQL主从部署
date: 2023-07-20 23:00:00
tags:
  - SoftwareInstall
---

## 部署说明

| 事项         | 详情                                     |
| ------------ | ---------------------------------------- |
| 部署节点     | 192.168.21.77（主）, 192.168.21.78（从） |
| 服务端口号   | 5432                                     |
| 运行目录     | /data/workspace/postgres                 |
| 数据文件位置 | /data/workspace/postgres/data            |

## 部署指南

```shell
# step1.安装依赖包
#安装前必须要保证gcc编译器已经安装好。可以使用rpm -qa gcc检查。
yum install gcc make openssl zlib-devel -y

# step2.解压安装包
#解压到/data/software/postgres
tar -xzvf postgresql-12.2.tar.gz -C /data/software

# step3.编译并安装
cd /data/software/postgresql-12.2
./configure --prefix=/data/workspace/postgres/data --without-readline

# step4.安装
make && make install

# step5.创建postgres用户
useradd postgres
passwd postgres

# step6.创建目录和log
mkdir -p /data/workspace/postgres/data
mkdir -p /data/workspace/postgres/log
chown -R postgres:postgres /data/workspace/postgres/

#登陆一下，让系统创建家目录
su - postgres

# 配置环境变量
vim .bash_profile
export PGHOME=/data/workspace/postgres
export PATH=$PATH:$PGHOME/bin

source .bash_profile

# step7.初始化数据库
postgres初始化编码指定中文
在postgres用户下使用
initdb -D /data/workspace/postgres -E UTF8 --locale=zh_CN.UTF-8 -U postgres -W

Data page checksums are disabled.

Enter new superuser password: postgres
Enter it again: postgres

# setp8.启动
pg_ctl -D /data/workspace/postgres/data -l logfile start


# set9.主从设置

################################# 主库配置 ######################################
# 进入psql
psql -U postgres postgres
# 创建用户
ceate user replica with replication login password 'replication'; 
alter user replica with password 'replication';
# 修改pg_hba.conf
vim /data/workspace/postgres/data/pg_hba.conf
# ------------------------- pg_hba.conf修改内容 -------------------------
host  all          all      0.0.0.0/0    trust
host  replication  replica 0.0.0.0/0  md5 
host    replication     replica         0.0.0.0/0            trust 
# ------------------------- 结束 -------------------------
# 修改配置文件
vim /data/workspace/postgres/data/postgresql.conf
#-------------------------- 配置文件修改内容 --------------------------
listen_addresses = '*'
# 设置主pgsql为生成wal的主机，9.6开始没有hot_standby（热备模式） 
wal_level = replica 
 
# 开启连续归档 
archive_mode = on 
#归档命令。-o "StrictHostKeyChecking no" 作用是取消第一次连接输入yes或者no 
archive_command = 'scp -o "StrictHostKeyChecking no" %p postgres@192.168.21.78:/data/workspace/postgres/data/archive/%f' 
# archive_command = 'test ! -f /data/postgresql-12/archive/%f && scp %p pgslave.ayunw.cn:/data/postgresql-12/archive/%f' 
archive_cleanup_command = '/opt/postgresql-12.12/bin/pg_archivecleanup -d /data/postgresql-12.12/data/pg_wal %r >> /data/postgresql-12.12/data/pg_log/archive_cleanup.log 2>&1' 
# 最多有16个流复制连接。 
max_wal_senders = 20 
# 设置流服务保留的最多wal(老版本叫xlog)文件个数 
wal_keep_segments = 256 
# 数据堆清理的最大进程 
autovacuum_max_workers = 2 
max_worker_processes = 20 
max_logical_replication_workers = 10 
# 日志设置 
log_destination = 'stderr' 
logging_collector = on 
log_directory = 'pg_log' 
log_filename = 'postgresql-%w.log' 
log_file_mode = 0600 
log_truncate_on_rotation = on 
log_rotation_age = 1d 
log_rotation_size = 1GB 
 
log_min_messages = error 
# 执行超过300ms的sql语句会记录到pgsql的日志文件,类似于慢日志 
# 一般设置300ms就好，慢日志会打到pgsql日志文件，方便查问题 
log_min_duration_statement = 300 
log_checkpoints = on 
log_connections = on 
log_disconnections = on 
log_error_verbosity = verbose 
log_hostname = on 
log_line_prefix = '%m [%p] ' 
log_lock_waits = on  
log_statement = 'ddl' 
#-------------------------- 结束 --------------------------
# 创建archive目录，重启
cd /data/workspace/postgres/data
mkdir acrchive
pg_ctl -D /data/workspace/postgres/data start

################################# 从库配置 ######################################
# 备份数据目录
mkdir -p /data/workspace/pgsqldata_backup 
mv /data/workspace/postgres/data/* /data/workspace/pgsqldata_backup

# 从库做基础备份
pg_basebackup -h 192.168.21.71 -p 5432 -U replica -W -R -Fp -Xs -Pv -D /data/workspace/postgres/data/
# 配置从库的配置文件
cd /data/workspace/postgres/data
mv postgresql.conf postgresql.conf_master.bak
cp /data/workspace/pgsqldata_backup/postgresql.conf /data/workspace/postgres/data
# 修改配置文件
vim /data/workspace/postgres/data/postgresql.conf 
#-------------------------- 配置文件修改内容 --------------------------
restore_command = 'cp /data/workspace/postgres/data/archive/%f %p' 
archive_cleanup_command = '/data/workspace/postgres/bin/pg_archivecleanup -d /data/workspace/postgres/data/pg_wal %r && /data/workspace/postgres/bin/pg_archivecleanup -d /data/workspace/postgres/data/archive %r >> /data/workspace/postgres/data/pg_log/archive_cleanup.log 2>&1' 
 
# 9.6开始没有hot_standby（热备模式） 
wal_level = replica 
# 最多有16个流复制连接。 
max_wal_senders = 20 
 
# 设置比主库大，可以设置为2倍的数值 
wal_keep_segments = 512 
max_logical_replication_workers = 10 
 
autovacuum_max_workers = 2 
# 和主的值保持一致即可 
max_worker_processes = 20 
 
# 说明这台机器不仅用于数据归档，还可以用于数据查询 
hot_standby = on 
#流备份的最大延迟时间 
max_standby_streaming_delay = 30s  
# 向主机汇报本机状态的间隔时间 
wal_receiver_status_interval = 10s  
# 出现错误复制，向主机反馈 
hot_standby_feedback = on 
 
 
# 日志设置 
log_destination = 'stderr' 
logging_collector = on 
log_directory = 'pg_log' 
log_filename = 'postgresql-%w.log' 
log_file_mode = 0600 
log_truncate_on_rotation = on 
log_rotation_age = 1d 
log_rotation_size = 1GB 
 
log_min_messages = error 
# 执行超过300ms的sql语句会被记录到pgsql的日志文件中 
log_min_duration_statement = 300 
log_checkpoints = on 
log_connections = on 
log_disconnections = on 
log_error_verbosity = verbose 
log_hostname = on 
log_line_prefix = '%m [%p] ' 
log_lock_waits = on  
log_statement = 'ddl' 
#-------------------------- 结束 --------------------------
# 创建archive目录
mkdir archive
# 重启服务
pg_ctl -D /data/workspace/postgres/data restart
```

## 常用命令

```shell
# 启动数据库服务
pg_ctl start
# 停止数据库服务
pg_ctl stop
# 重启数据库服务
pg_ctl restart
# 进入postgres
psql -U postgres
# 退出
\q
```


