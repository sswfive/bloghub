---
title: Docker安装卸载指南
date: 2022-04-01 20:09:50
tags: 
  - Docker
---


## Install for CentOS
### step1: 卸载旧版本

- 旧版本的 Docker 叫 `docker` 或者 `docker-engine`，现在的版本被称为`docker-ce`，如果之前安装过，则先卸载掉之前的版本：
```bash
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
### step2:安装Docker-CE
#### 离线安装
```bash
https://download.docker.com/linux/centos/7/x86_64/stable/Packages/ 

# 在上述网址中下载自己想要安装的版本（.rpm文件）
```
#### 在线安装

- 安装环境依赖（yum-utils提供了yum-config-manager的一些实用的功能）
```bash
yum install -y yum-utils  device-mapper-persistent-data lvm2
```

- 添加stable版本的docker仓库
```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

- 安装最新版本
```bash
yum install docker-ce docker-ce-cli containerd.io
```

- 安装指定版本
```bash
# 查看可安装的版本命令
yum list docker-ce --showduplicates | sort -r

# 上述命令输出如下：
 * updates: mirrors.tuna.tsinghua.edu.cn
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror, langpacks
 * extras: mirrors.huaweicloud.com
docker-ce.x86_64            3:19.03.4-3.el7                     docker-ce-stable
......
docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
......
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable

# eg: 安装docker-ce-18.09.9版本
sudo yum -y install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io 
```
#### 设置开机自动启动
```bash
systemctl daemon-reload 

systemctl enable docker  && systemctl start docker
```
#### 验证是否安装成功

- 如果出现docker的版本信息则表示成功安装
```bash
docker info 
```

#### 配置扩展（非必须但是建议）

- 新建docker.json的docker配置文件
> 获取阿里云提供的镜像加速服务的链接：
> [https://cr.console.aliyun.com/cn-beijing/instances/mirrors。](https://cr.console.aliyun.com/cn-beijing/instances/mirrors。)

```bash
mkdir -p /etc/docker/

vim /etc/docker/daemon.json
{
  "data-root": "/data/software/docker",
  "registry-mirrors": ["https://p0rish3o.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

- daemon.json内容说明：
   - data-root： 修改安装目录（默认安装目录在/var/lib/docker下）
      - 参考链接:  [迁移 Docker 的默认安装(存储)目录](https://strikefreedom.top/migrate-docker-installation-directory)
   - registry-mirrors： 配置阿里云作为镜像加速服务
   - exec-opts： 此配置为安装k8s所必须

---

## Install for Ubuntu
```bash
# step1. 更新ubuntu的apt源索引
sudo apt-get update

# step2: 安装包允许apt通过HTTPS使用仓库
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common gnupg lsb-release

# step3: 添加Docker官方GPG key
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# step4: 设置Docker稳定版仓库
# sudo add-apt-repository \
#    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
#    $(lsb_release -cs) \
#    stable"
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# step5: 添加仓库后，更新apt源索引
sudo apt-get update

# step7: 安装最新版Docker CE（社区版）
sudo apt-get install docker-ce  docker-ce-cli containerd.io docker-compose-plugin

# step8: 验证是否安装成功,拉取成功则表示安装完成
sudo docker run hello-world
```
> 备注：为了避免每次命令都输入sudo，可以设置用户权限，**注意执行后须注销重新登录**

```bash
sudo usermod -a -G docker $USER
```

---

# 卸载指南：
```bash
yum list installed | grep docker 

yum -y remove  docker.x86_64   docker-client.x86_64  docker-common.x86_64

rm -rf  /var/lib/docker
```

