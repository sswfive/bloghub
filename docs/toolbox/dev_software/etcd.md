---
title: Etcd集群部署
date: 2023-07-20 22:56:13
tags:
  - SoftwareInstall
---
## 部署说明
| 事项 | 详情 |
| --- | --- |
| 部署节点 | 192.168.21.71（Master）、192.168.21.77（salve）、192.168.21.78（salve） |
| 部署策略 | Centos7.9 系统  集群部署 |
| 服务端口号 | 2379 |
| 安装部署目录 | usr/local/bin/etcd  /usr/local/bin/etcdctl |

## 部署指南
### 部署（HTTP）方式：

- 下载、安装
```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.5.7/etcd-v3.5.7-linux-amd64.tar.gz

tar zxf etcd-v3.5.7-linux-amd64.tar.gz --strip-components=1 -C /usr/local/bin etcd-v3.5.7-linux-amd64/etcd{,ctl}

etcdctl version

scp /usr/local/bin/etcd* node77:/usr/local/bin/
scp /usr/local/bin/etcd* node78:/usr/local/bin/
```

- 编写配置文件(三个节点都需要)
```bash
vim /etc/etcd/etcd.conf

# node71节点： 
#[Member]
ETCD_NAME="node71"
ETCD_DATA_DIR="/data/workspace/etcd/data.etcd"
ETCD_LISTEN_PEER_URLS="http://192.168.21.71:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.21.71:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.21.71:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.21.71:2379"
ETCD_INITIAL_CLUSTER="node71=http://192.168.21.71:2380,node77=http://192.168.21.77:2380,node78=http://192.168.21.78:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"


# node77节点：
#[Member]
ETCD_NAME="node77"
ETCD_DATA_DIR="/data/workspace/etcd/data.etcd"
ETCD_LISTEN_PEER_URLS="http://192.168.21.77:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.21.77:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.21.77:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.21.77:2379"
ETCD_INITIAL_CLUSTER="node71=http://192.168.21.71:2380,node77=http://192.168.21.77:2380,node78=http://192.168.21.78:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"


# node78节点：
#[Member]
ETCD_NAME="node78"
ETCD_DATA_DIR="/data/workspace/etcd/data.etcd"
ETCD_LISTEN_PEER_URLS="http://192.168.21.78:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.21.78:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.21.78:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.21.78:2379"
ETCD_INITIAL_CLUSTER="node71=http://192.168.21.71:2380,node77=http://192.168.21.77:2380,node78=http://192.168.21.78:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```

- 编写启动脚本
```bash
# vim /data/workspace/etcd_node71.sh

# For each machine
TOKEN=qazwx123
CLUSTER_STATE=new
NAME_1=node71
NAME_2=node77
NAME_3=node78
HOST_1=192.168.21.71
HOST_2=192.168.21.77
HOST_3=192.168.21.78
CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_3}=http://${HOST_3}:2380

# For node71
THIS_NAME=${NAME_1}
THIS_IP=${HOST_1}
etcd --data-dir=/data/workspace/etcd/data.etcd --name ${THIS_NAME} \
	--initial-advertise-peer-urls http://${THIS_IP}:2380 \
	--listen-peer-urls http://${THIS_IP}:2380 \
	--advertise-client-urls http://${THIS_IP}:2379,http://127.0.0.1:2379 \
	--listen-client-urls http://${THIS_IP}:2379,http://127.0.0.1:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} \
	--initial-cluster-token ${TOKEN}  > /data/workspace/etcd/etcd.out 2>&1 &


# vim /data/workspace/etcd_node77.sh
# For each machine
TOKEN=qazwx123
CLUSTER_STATE=new
NAME_1=node71
NAME_2=node77
NAME_3=node78
HOST_1=192.168.21.71
HOST_2=192.168.21.77
HOST_3=192.168.21.78
CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_3}=http://${HOST_3}:2380

# For node77
THIS_NAME=${NAME_2}
THIS_IP=${HOST_2}
etcd --data-dir=/data/workspace/etcd/data.etcd --name ${THIS_NAME} \
	--initial-advertise-peer-urls http://${THIS_IP}:2380 \
	--listen-peer-urls http://${THIS_IP}:2380 \
	--advertise-client-urls http://${THIS_IP}:2379,http://127.0.0.1:2379 \
	--listen-client-urls http://${THIS_IP}:2379,http://127.0.0.1:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} \
	--initial-cluster-token ${TOKEN}  > /data/workspace/etcd/etcd.out 2>&1 &


# vim /data/workspace/etcd_node78.sh
# For each machine
TOKEN=qazwx123
CLUSTER_STATE=new
NAME_1=node71
NAME_2=node77
NAME_3=node78
HOST_1=192.168.21.71
HOST_2=192.168.21.77
HOST_3=192.168.21.78
CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_3}=http://${HOST_3}:2380

# For node78
THIS_NAME=${NAME_3}
THIS_IP=${HOST_3}
etcd --data-dir=/data/workspace/etcd/data.etcd --name ${THIS_NAME} \
	--initial-advertise-peer-urls http://${THIS_IP}:2380 \
	--listen-peer-urls http://${THIS_IP}:2380 \
	--advertise-client-urls http://${THIS_IP}:2379,http://127.0.0.1:2379 \
	--listen-client-urls http://${THIS_IP}:2379,http://127.0.0.1:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} \
	--initial-cluster-token ${TOKEN}  > /data/workspace/etcd/etcd.out 2>&1 &
```

- 执行启动命令
```bash
chmode +x etcd_node71.sh etcd_node77.sh etcd_node78.sh

scp /data/workspace/etcd/etcd_node77.sh node77:/data/workspace/etcd
scp /data/workspace/etcd/etcd_node78.sh node78:/data/workspace/etcd

# 三个节点同时执行
# node71
./etcd_node71.sh

# node77
./etcd_node77.sh

# node78
./etcd_node78.sh
```

- 配置开机启动（三个节点都需要）
```bash
vim /usr/lib/systemd/system/etcd.service

[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/etc/etcd/etcd.conf
ExecStart=etcd \
--name=${ETCD_NAME} \
--data-dir=${ETCD_DATA_DIR} \
--listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
--listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
--advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS},http://127.0.0.1:2379 \
--initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
--initial-cluster=${ETCD_INITIAL_CLUSTER} \
--initial-cluster-token=${ETCD_INITIAL_CLUSTER} \
--initial-cluster-state=${ETCD_INITIAL_CLUSTER_STATE}

Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

- 启动
```bash
systemctl daemon-reload

systemctl start etcd
```

- 查看集群启动状态
```bash
etcdctl --write-out=table member list
+------------------+---------+--------+---------------------------+-------------------------------------------------+------------+
|        ID        | STATUS  |  NAME  |        PEER ADDRS         |                  CLIENT ADDRS                   | IS LEARNER |
+------------------+---------+--------+---------------------------+-------------------------------------------------+------------+
| 6dcf7486a69f8697 | started | node71 | http://192.168.21.71:2380 | http://127.0.0.1:2379,http://192.168.21.71:2379 |      false |
| a813d75d75a2bb6a | started | node77 | http://192.168.21.77:2380 | http://127.0.0.1:2379,http://192.168.21.77:2379 |      false |
| c9fca2d62adf253a | started | node78 | http://192.168.21.78:2380 | http://127.0.0.1:2379,http://192.168.21.78:2379 |      false |
+------------------+---------+--------+---------------------------+-------------------------------------------------+------------+
```

---

### 部署（HTTPS）方式

- 下载、安装
```bash
# node71节点执行
wget https://github.com/etcd-io/etcd/releases/download/v3.5.7/etcd-v3.5.7-linux-amd64.tar.gz

tar zxf etcd-v3.5.7-linux-amd64.tar.gz --strip-components=1 -C /usr/local/bin etcd-v3.5.7-linux-amd64/etcd{,ctl}

etcdctl version

scp /usr/local/bin/etcd* node77:/usr/local/bin/
scp /usr/local/bin/etcd* node78:/usr/local/bin/
```

- 使用cfssl工具生成证书
```bash
wget "https://github.com/cloudflare/cfssl/releases/download/v1.6.2/cfssl_1.6.2_linux_amd64" -O /usr/local/bin/cfssl

wget "https://github.com/cloudflare/cfssl/releases/download/v1.6.2/cfssljson_1.6.2_linux_amd64" -O /usr/local/bin/cfssljson

chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson 

cfssl version





```

- 创建一个证书存放目录(三个节点都需要)
```bash
mkdir -p /etc/etcd/ssl
```

- 创建一个用于证书的json文件
```json
vim ca-config.json

{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "etcd": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ],
        "expiry": "876000h"
      }
    }
  }
}
```

- 创建一个用于生成CA证书和key的json文件
```json
vim etcd-ca-csr.json 
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}
```

- 生成CA证书和证书的key
```bash
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca
ls /etc/etcd/ssl/
etcd-ca.csr  etcd-ca-key.pem  etcd-ca.pem
```

- 创建用于etcd服务证书的json
```json
vim etcd-csr.json

{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ]
}
```

- 生成ETCD服务证书
```bash
cfssl gencert -ca=/etc/etcd/ssl/etcd-ca.pem -ca-key=/etc/etcd/ssl/etcd-ca-key.pem -config=ca-config.json -hostname=127.0.0.1,node71,node77,node78,192.168.21.71,192.168.21.77,192.168.21.78 -profile=etcd etcd-csr.json | cfssljson -bare /etc/etcd/ssl/etcd

ls /etc/etcd/ssl/
etcd-ca.csr  etcd-ca-key.pem  etcd-ca.pem  etcd.csr  etcd-key.pem  etcd.pem
```

- 将证书复制到其他节点
```bash
for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do        scp /etc/etcd/ssl/${FILE} node77:/etc/etcd/ssl/${FILE};      done

for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do        scp /etc/etcd/ssl/${FILE} node78:/etc/etcd/ssl/${FILE};      done
```

- ETCD配置
   - 创建etcd配置文件（三个节点都需要）
```bash
vim /etc/etcd/etcd.config.yml 

# node71节点
name: 'node71'
data-dir: /data/workspace/etcd/data
wal-dir: /data/workspace/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://192.168.21.71:2380'
listen-client-urls: 'https://192.168.21.71:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: 'https://192.168.21.71:2380'
advertise-client-urls: 'https://192.168.21.71:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'node71=https://192.168.21.71:2380,node77=https://192.168.21.77:2380,node78=https://192.168.21.78:2380'
initial-cluster-token: 'etcd-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
peer-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  peer-client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
debug: false
log-package-levels:
log-outputs: [default]
force-new-cluster: false

# node77节点
name: 'node77'
data-dir: /data/workspace/etcd/data
wal-dir: /data/workspace/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://192.168.21.77:2380'
listen-client-urls: 'https://192.168.21.77:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: 'https://192.168.21.77:2380'
advertise-client-urls: 'https://192.168.21.77:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'node71=https://192.168.21.71:2380,node77=https://192.168.21.77:2380,node78=https://192.168.21.78:2380'
initial-cluster-token: 'etcd-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
peer-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  peer-client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
debug: false
log-package-levels:
log-outputs: [default]
force-new-cluster: false

# node78节点
name: 'node78'
data-dir: /data/workspace/etcd/data
wal-dir: /data/workspace/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://192.168.21.78:2380'
listen-client-urls: 'https://192.168.21.78:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: 'https://192.168.21.78:2380'
advertise-client-urls: 'https://192.168.21.78:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'node71=https://192.168.21.71:2380,node77=https://192.168.21.77:2380,node78=https://192.168.21.78:2380'
initial-cluster-token: 'etcd-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
peer-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  peer-client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
debug: false
log-package-levels:
log-outputs: [default]
force-new-cluster: false
```

- 配置开机启动(三个节点都需要)
```bash
vim /usr/lib/systemd/system/etcd.service

[Unit]
Description=Etcd Service
Documentation=https://coreos.com/etcd/docs/latest/
After=network.target

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.config.yml
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
Alias=etcd3.service
```

- 启动etcd服务(三个节点都需要)
```bash
systemctl daemon-reload
systemctl enable --now etcd
```

- 查看etcd的状态
```bash
etcdctl --endpoints="192.168.21.71:2379,192.168.21.77:2379,192.168.21.78:2379" --cacert=/etc/etcd/ssl/etcd-ca.pem --cert=/etc/etcd/ssl/etcd.pem --key=/etc/etcd/ssl/etcd-key.pem  endpoint status --write-out=table

+------------------+---------+--------+---------------------------+-------------------------------------------------+------------+
|        ID        | STATUS  |  NAME  |        PEER ADDRS         |                  CLIENT ADDRS                   | IS LEARNER |
+------------------+---------+--------+---------------------------+-------------------------------------------------+------------+
| 6dcf7486a69f8697 | started | node71 | http://192.168.21.71:2380 | http://127.0.0.1:2379,http://192.168.21.71:2379 |      false |
| a813d75d75a2bb6a | started | node77 | http://192.168.21.77:2380 | http://127.0.0.1:2379,http://192.168.21.77:2379 |      false |
| c9fca2d62adf253a | started | node78 | http://192.168.21.78:2380 | http://127.0.0.1:2379,http://192.168.21.78:2379 |      false |
+------------------+---------+--------+---------------------------+-------------------------------------------------+------------+
```

## 安装客户端工具

- etcd-manager
> [https://github.com/gohutool/boot4go-etcdv3-browser](https://github.com/gohutool/boot4go-etcdv3-browser)
> [https://hub.docker.com/r/joinsunsoft/etcdv3-browser](https://hub.docker.com/r/joinsunsoft/etcdv3-browser)

```bash
docker container run --rm -p 9980:80 --name etcdv3-browser  joinsunsoft/etcdv3-browser:0.9.0
```
