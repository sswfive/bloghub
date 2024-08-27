---
title: 查看Docker容器启动的详细命令及参数
date: 2023-07-19 23:32:04
tags: 
  - Docker
---
> 由于历史原因，前人在使用docker部署的服务时并没有留下详细的部署文档，因服务意外挂掉后，需要对其重启，带着这样的需求，尝试了以下几种方案，故此记录总结一下；


通过`docker ps -a`找到需要启动的容器名称

## 方式一：使用 docker原生命令

- `docker inspect <container name>` 查看
- 发现输出的信息比较详细，需要有一定的上下文背景，才可能知道当时该服务容器的启动命令，体验不是很友好
```json
# 以下日志仅展示了部分输出字段

[root@automlgpu4 bluewhale_dev]# docker inspect product-pg-1
[
    {
        "Id": "19ec2701ca0df73da5d193daaf2ef72b2bb55d144d7ea7ccd9136a5f7f62c17c",
        "Created": "2023-07-04T06:56:17.349741798Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "postgres"
        ],
        "Image": "sha256:1149d285a5f5c4340cefa2211869c3a6b1128ac78974545e0b4fe62d3d0e66a8",
        "ResolvConfPath": "/data/docker/containers/19ec2701ca0df73da5d193daaf2ef72b2bb55d144d7ea7ccd9136a5f7f62c17c/resolv.conf",
        "HostnamePath": "/data/docker/containers/19ec2701ca0df73da5d193daaf2ef72b2bb55d144d7ea7ccd9136a5f7f62c17c/hostname",
        "HostsPath": "/data/docker/containers/19ec2701ca0df73da5d193daaf2ef72b2bb55d144d7ea7ccd9136a5f7f62c17c/hosts",
        "LogPath": "/data/docker/containers/19ec2701ca0df73da5d193daaf2ef72b2bb55d144d7ea7ccd9136a5f7f62c17c/19ec2701ca0df73da5d193daaf2ef72b2bb55d144d7ea7ccd9136a5f7f62c17c-json.log",
        "Name": "/product-pg-1",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux"，
        "ExecIDs": null,
        "GraphDriver": {
            "Data": {},
            "Name": "overlay2"
        },
        "Config": {
            "Hostname": "pg",
            "Domainname": "",
            "User": ""
            },
            "Env": [
                "POSTGRES_USER=root",
                "POSTGRES_PASSWORD=2023",
                "TZ=Asia/Shanghai",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=en_US.utf8",
                "PG_MAJOR=14",
                "PG_VERSION=14.1",
                "PG_SHA256=4d3c101ea7ae38982f06bdc73758b53727fb6402ecd9382006fa5ecc7c2ca41f",
                "PGDATA=/var/lib/postgresql/data"
            ],
            "Cmd": [
                "postgres"
            ],
            "Image": "postgres:14-alpine",
            "Volumes": {
                "/var/lib/postgresql/data": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "com.docker.compose.config-hash": "f7d46590f5568ccc12b47ec4e104686e98fa22e081b8f1b2f57af3670affaef1",
                "com.docker.compose.container-number": "1",
                "com.docker.compose.depends_on": "",
                "com.docker.compose.image": "sha256:1149d285a5f5c4340cefa2211869c3a6b1128ac78974545e0b4fe62d3d0e66a8",
                "com.docker.compose.oneoff": "False",
                "com.docker.compose.project": "product"，
                "com.docker.compose.service": "pg",
                "com.docker.compose.version": "2.13.0"
            },
            "StopSignal": "SIGINT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "e637937788ef9cdb7b6b72b3fff145520cc990ce342bab28fdf27b8da9a87987",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "5432/tcp": null
            },
            "Networks": {
                "product_back": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "product-pg-1",
                        "pg",
                        "19ec2701ca0d"
                    ],
                    "NetworkID": "1bf3395d104c6b062e42d96dc58d7fbd32c8c25615a749be04820fb9eb5701ae",
                    "EndpointID": "ef940c3a5b9b6ee928c30ac48ece25551e63b3ed840c64eb02ab972c15e692e4",
                    "Gateway": "172.25.0.1",
                    "IPAddress": "172.25.0.9",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:19:00:09",
                    "DriverOpts": null
                }
            }
        }
    }

```

## 方式二：使用第三方工具

- `get_command_4_run_container` 
- 通过搜索引擎找到该工具，于是便开始尝试验证，具体使用如下；
```bash
# 拉取镜像
docker pull cucker/get_command_4_run_container

# 启动
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock cucker/get_command_4_run_container [容器名称]/[容器ID]

# 查看输出结果
```

- 对于启动命令封装成一个bash命令，后续可以直接使用 `get_docker_command [容器名称]/[容器ID]`
```bash
echo "alias get_docker_command='docker run --rm -v /var/run/docker.sock:/var/run/docker.sock cucker/get_command_4_run_container'" >> ~/.bashrc \
&& \
. ~/.bashrc
```

- 操作日志如下：
```bash
[root@automlgpu4 bluewhale_dev]# docker pull cucker/get_command_4_run_container
Using default tag: latest
latest: Pulling from cucker/get_command_4_run_container
8a49fdb3b6a5: Pull complete 
0357922e53aa: Pull complete 
9676b6d4b964: Pull complete 
ddbd03ee1059: Pull complete 
877a053836a3: Pull complete 
a68de691a1fb: Pull complete 
ac3fd788d9e3: Pull complete 
51fb85f07a5a: Pull complete 
Digest: sha256:d3fba2642b3bd30dc8bd7379b18e889c6ad9d76c138ffe870b47b7fb6b9c9e94
Status: Downloaded newer image for cucker/get_command_4_run_container:latest
docker.io/cucker/get_command_4_run_container:latest
[root@automlgpu4 bluewhale_dev]# docker run --rm -v /var/run/docker.sock:/var/run/docker.sock cucker/get_command_4_run_container bluewhale_dev-pg-1
docker run -d \
 --name bluewhale_dev-pg-1 \
 --cgroupns host \
 --env POSTGRES_PASSWORD=lanjing.2022 \
 --env TZ=Asia/Shanghai \
 --env POSTGRES_USER=root \
 --hostname pg \
 --label com.docker.compose.config-hash=5a31f16dd78cccc6759eba8ae3183d729013898d96a7b364124fec1efcabf280 \
 --label com.docker.compose.container-number=1 \
 --label com.docker.compose.depends_on="" \
 --label com.docker.compose.image=sha256:1149d285a5f5c4340cefa2211869c3a6b1128ac78974545e0b4fe62d3d0e66a8 \
 --label com.docker.compose.oneoff=False \
 --label com.docker.compose.project=bluewhale_dev \
 --label com.docker.compose.project.config_files=/data/bluewhale/bluewhale_dev/docker-compose.yml \
 --label com.docker.compose.project.working_dir=/data/bluewhale/bluewhale_dev \
 --label com.docker.compose.service=pg \
 --label com.docker.compose.version=2.13.0 \
 --log-opt max-size=100m \
 --network=bluewhale_dev_looklook_net \
 -p 5532:5432/tcp \
 --stop-signal SIGINT \
 -v /data/bluewhale/bluewhale_dev/data/pg:/var/lib/postgresql/data:rw \
 postgres:14-alpine
```
