---
title: 多版本多工具部署K8S集群
date: 2024-08-16 23:50:14
tags: 
  - K8S
---

> 再出发，应业务需要，需要再次着手部署 kubernetes 集群，之前分别陆续用不同工具部署过不同的版本，在此再做一次全新的部署，并重新整理和归档以前部署的流程步骤和遇到的问题及解决办法。——2024年08月16日23:50:14。


## kubeadm 部署 v1.28.0 版本
> 本次目标是搭建一个一主两从的 k8s 集群。硬件要求建议不要低于2 核 CPU、2G 内存、20G 硬盘，操作系统需要是 Linux。
> 三台机器之间需要配置 SSH 免密。

### 前置准备说明
**以下步骤需要分别在所有机器上执行！！！**

- 硬件要求

| 机器配置 | 用途 | 服务器主机名 | 内网IP地址 |
| --- | --- | --- | --- |
| 2核4G内存，50G硬盘，CentOS 7.6操作系统 | 管理节点 | k8s-master | 192.168.1.11 |
| 2核2G内存，50G硬盘，CentOS 7.6操作系统 | 工作节点 | k8s-worker1 | 192.168.1.12 |
| 2核2G内存，50G硬盘，CentOS 7.6操作系统 | 工作节点 | k8s-worker2 | 192.168.1.13 |


1. 关闭防火墙及相关配置
```bash
# 关闭和禁用防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux，selinux 是 Linux 系统下的一个安全服务，需要关闭
setenforce 0  # 临时
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久

# 关闭 Linux 的 swap 分区，提升 Kubernetes 的性能
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久
```

2. 按照规划修改主机名
> K8s 使用主机名进行节点的通信，所以需要按照表格中的主机名修改 3 台服务器的主机名。注意：K8s 要求主机名使用 “-” 或者 “.” 连接，不能使用下划线 “_”。

```bash
# 在192.168.1.11 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-master

# 在192.168.1.12 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-worker1

# 在192.168.1.13 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-worker2
```

3. 主机名解析
```bash
# 添加各个节点的解析，IP 地址需要替换为你自己服务器的内网 IP 地址。
cat >> /etc/hosts << EOF
192.168.1.11 k8s-master
192.168.1.12 k8s-worker1
192.168.1.13 k8s-worker2
EOF
```

4. 时间同步
> K8s 要求集群中的所有服务器时间一致，使用 ntpdate（Network Time Protocol）从网络同步时间，执行以下命令安装 ntpdate，之后再执行同步命令。

```bash
# 安装ntp服务
yum install ntpdate -y

# 安装完成后使用阿里云的时间服务同步时间 
ntpdate ntp1.aliyun.com


###############PS#######################
# 如果提示没有匹配到 ntpdate（Unable to find a match: ntpdate），则可以使用 wlnmp 的软件源安装 wntp，
# 然后再执行时间同步，执行以下命令：

rpm -ivh http://mirrors.wlnmp.com/centos/wlnmp-release-centos.noarch.rpm
yum install wntp -y

# 安装完成后使用阿里云的时间服务同步时间 
ntpdate ntp1.aliyun.com
###############PS#######################
```

5. 配置网路
> 为了让 K8s 能够转发网络流量，需要修改 iptables 的配置。执行以下命令快速编辑 “etc/sysctl.d/k8s.conf” 配置文件，增加 iptables 的配置。

```bash
# 修改 Linux 内核参数，添加网桥过滤和地址转发功能
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 使配置生效
sudo sysctl --system
```

6. 重启服务器
```bash
# 运行 “reboot” 命令，重启服务器，让以上配置生效
reboot
```
### 安装相关软件
**以下步骤需要分别在所有机器上执行！！！**

1. 安装 Docker
> 可以使用 Docker 或 Podman 作为运行容器的运行时组件，再次使用 Docker

```bash
# 通过 wget 命令获取 docker 软件仓库信息
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

# 安装 docker-ce
yum -y install docker-ce

# 开启 docker 服务
systemctl enable docker && systemctl start docker
```
> Docker 安装完成之后，配置阿里云提供的镜像库来加速镜像下载。执行以下命令快速编辑 “/etc/docker/daemon.json” 文件，增加镜像库配置，最后再重启 Docker 服务。

```bash
# 配置镜像下载加速器，国内使用阿里云镜像库会更快
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

# 重启 docker 服务
systemctl restart docker
```

2. 安装 cri-dockered
> cri-dockerd 用于为 Docker 提供一个能够支持 K8s 容器运行时标准的工具，从而能够让 Docker 作为 K8s 容器引擎。通过 wget 下载 cri-dockerd 安装包，然后通过 rpm 命令进行安装。如果官方地址下载太慢，可以使用离线安装包进行安装

```bash
# 通过 wget 命令获取 cri-dockerd软件
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.12/cri-dockerd-0.3.12-3.el7.x86_64.rpm
# 通过 rpm 命令执行安装包
rpm -ivh cri-dockerd-0.3.12-3.el7.x86_64.rpm
```
> 安装完成后使用 vi 编辑器修改配置文件（/usr/lib/systemd/system/cri-docker.service），在 “ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint fd://” 这一行增加 “–pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9”。

```bash
# 打开 cri-docker.service 配置文件
vi /usr/lib/systemd/system/cri-docker.service

# 修改对应的配置项
ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint fd:// --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9
```
> 配置文件修改完成后，重新加载并开启 cri-dockered 服务

```bash
# 加载配置并开启服务
systemctl daemon-reload
systemctl enable cri-docker && systemctl start cri-docker
```

3. 添加阿里云 YUM 软件源
> 国内是无法访问 K8s 官方的镜像库，如果不增加这个配置，后续就无法安装相关工具。

```bash
# 执行以下命令快速编辑 “/etc/yum.repos.d/kubernetes.repo” 配置文件，添加阿里云的镜像库。
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
### 安装 kubeadm、kubelet、kubectl
**以下步骤需要分别在所有机器上执行！！！**
> kubeadm 用来初始化 K8s 集群；kubelet 是每个节点的 K8s 管理员；kubectl 是 K8s 的命令行交互工具

```bash
# 指定了安装的版本是 1.28.0
yum install -y kubelet-1.28.0 kubeadm-1.28.0 kubectl-1.28.0

# 安装成功后，配置开机启动服务
systemctl enable kubelet

```
### Master 节点初始化 k8s
**只需要在 Master 节点上执行以下初始化命令！！！**
```bash
# “apiserver-advertise-address” 参数需要替换成自己的 Master 节点服务器的内网 IP。

kubeadm init \
  --apiserver-advertise-address=192.168.1.11 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.28.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --cri-socket=unix:///var/run/cri-dockerd.sock \
  --ignore-preflight-errors=all


###############参数说明#######################
# apiserver-advertise-address：集群广播地址，用 master 节点的内网 IP。
# image-repository：由于默认拉取镜像地址 k8s.gcr.io 国内无法访问，这里指定阿里云镜像仓库地址。
# kubernetes-version： K8s 版本，与上面安装的软件版本一致。
# service-cidr：集群 Service 网段。pod-network-cidr：集群 Pod 网段。
# cri-socket：指定 cri-socket 接口，我们这里使用 unix:///var/run/cri-dockerd.sock。
###############参数说明#######################
```
> 执行成功后会出现以下内容

```bash
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.11:6443 --token xxxxxx \
        --discovery-token-ca-cert-hash sha256:xxxxxx
```
在以上返回结果中有 3 条命令需要立即执行，这是用来设置 kubectl 工具的管理员权限，执行之后就可以在 Master 节点上通过终端窗口使用 kubectl 命令了。
```bash
# 在 Master 节点上执行以下命令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
在返回结果的最后，还有 1 条 “kubeadm join” 命令，这个是用来加入工作节点的，需要在工作节点上执行。

### work 节点加入 k8s 集群
K8s 初始化之后，就可以在其他 2 个工作节点上执行 “kubeadm join” 命令，因为我们使用了 cri-dockerd ，需要在命令加上 “–cri-socket=unix:///var/run/cri-dockerd.sock” 参数（注：如果使用公有云的服务器，需要先打开 Master 节点所在服务器的 6443 端口，这个是 K8s 的 API server 端口。）
```bash
# 在两个工作节点上执行
kubeadm join 192.168.1.11:6443 --token xxxxxx \
        --discovery-token-ca-cert-hash sha256:xxxxxx \
        --cri-socket=unix:///var/run/cri-dockerd.sock


###############PS#######################
# 命令中 token 有效期为 24 小时，当 token 过期之后，执行 “kubeadm join” 命令就会报错。这时可以直接在 Master 节点上使用以下命令生成新的 token，然后再使用 “kubeadm join” 命令加入节点。
kubeadm token create --print-join-command
###############PS#######################
```
此时，我们的集群就部署成功了。你可以使用 “kubectl get node” 命令来查看集群节点状态。
> 你会注意到所有节点的状态都是 “NotReady”，这是由于集群还缺少网络插件，集群的内部网络还没有正常运作。

```bash
[root@k8s-master ~]# kubectl get node
NAME           STATUS     ROLES           AGE   VERSION
k8s-master     NotReady   control-plane   84m   v1.28.0
k8s-worker1    NotReady   <none>          82m   v1.28.0
k8s-worker2    NotReady   <none>          47s   v1.28.0
```
安装 k8s 网络插件
K8s 网络插件，也称为容器网络接口（CNI）插件，是实现 K8s 集群中容器间通信和网络连接的关键组件。在这里我使用 Calico 作为集群的网络插件。Calico 是通过执行一个 YAML 文件部署到 K8s 集群里的，所以我们首先需要通过 “wget” 命令下载这个 YAML 文件。
```bash
# 下载 Calico 插件部署文件
wget https://docs.projectcalico.org/manifests/calico.yaml
```
其次，通过 vi 编辑器修改 “calico.yaml” 文件中的 “CALICO_IPV4POOL_CIDR” 参数，需要与前面 “kubeadm init” 命令中的 “–pod-network-cidr” 参数一样（10.244.0.0/16）,修改位置如下所示
```bash
# 修改文件时注意格式对齐
4598 # The default IPv4 pool to create on startup if none exists. Pod IPs will be
4599 # chosen from this range. Changing this value after installation will have
4600 # no effect. This should fall within `--cluster-cidr`.
4601 - name: CALICO_IPV4POOL_CIDR
4602   value: "10.244.0.0/16"
4603 # Disable file logging so `kubectl logs` works.
4604 - name: CALICO_DISABLE_FILE_LOGGING
4605   value: "true"
```
最后，使用 “kubectl apply” 命令将 Calico 插件部署到集群里，部署 YAML 文件命令如下：
```bash
kubectl apply -f calico.yaml
```
Calico 部署会比较慢，大概等个几分钟，等待 Calico 部署完成后，再次通过命令 “kubectl get node” 查看节点状态，就可以看到所有节点已经准备就绪，恭喜你！此时集群正式搭建成功。
```bash
[root@k8s-master ~]# kubectl get node
NAME           STATUS   ROLES           AGE   VERSION
k8s-master     Ready    control-plane   12h   v1.28.0
k8s-worker1    Ready    <none>          12h   v1.28.0
k8s-worker2    Ready    <none>          10h   v1.28.0
```
PS: 离线安装方式（在线安装失败时使用此方法）
```bash
# 假定下载的文件名为：“calico-image-3.25.1.zip”， 解压后在每个节点上执行导入命令。
ls *.tar |xargs -i docker load -i {}

# “calico-yaml.zip” 中是部署文件，在 Master 节点上依次部署两个 YAML 文件。
# 先删除之前部署不成功的 Calico
kubectl delete -f calico.yaml

# 依次部署两个 yaml 文件
kubectl apply -f tigera-operator.yaml --server-sidekubectl apply -f custom-resources.yaml

# 然后等待部署完成，就可以看到所有节点准备就绪。
```

