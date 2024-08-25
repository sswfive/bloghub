---
title: kubeadm部署k8s集群指南(v1.19.4)
date: 2021-03-02 20:14:32
tags: 
  - K8S
---


## STEP1: 前置工作(准备/检查环境)
### 1.集群服务器要求：

- 每台机器 2 GB 或更多的内存 (如果少于这个数字将会影响您应用的运行内存)
- 2 CPU 或更多
- 集群中的所有机器的网络互通
- 可以访问外网
- 禁止 swap 分区
> eg:

|          IP	               | 主机名  |  系统版本  | CPU | 内存   | 备注 |
| --- | --- | --- | --- | --- | --- |
| 192.168.33.131 | master1 | centos8.2  | 2G | 2 | 

此配置为最低要求 |
| 192.168.33.132 | node1 | centos8.2 | 2G | 2 |  |
| 192.168.33.133 | node2 | centos8.2 | 2G | 2 |  |

### 2.设置服务器静态IP/DNS/hosts主机名
> eg：

```bash
# 查看网卡名字
nmcli d

# 修改配置文件，xxx为对应网卡的名字
vim /etc/sysconfig/network-scripts/ifcfg-xxx   

# 修改并保存网卡信息（修改demo如下：）
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static" #启用静态IP地址
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="5e51c653-26b7-4e87-bbe6-4a06df6ff1d4"
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.33.132" #设置IP地址
NETMASK="255.255.255.0"  #设置子网掩码
GATEWAY="192.168.189.2"  #设置网关
DNS1="8.8.8.8"			 #DNS
DNS2="114.114.114.114"

# 保存退出并重启网卡 xxx为网卡名
ifup xxx
（eg: ifup ens33）
```
### 3.设置主机名（hostname）
```bash
# xxx为主机名 
hostnamectl set-hostname xxx 
```
### 4.查看或设置hosts

- 节点的 hostname 必须使用标准的 DNS 命名，另外千万不用什么默认的 `localhost` 的 hostname，会导致各种错误出现的。在 Kubernetes 项目里，机器的名字以及一切存储在 Etcd 中的 API 对象，都必须使用标准的 DNS 命名（RFC 1123）。可以使用命令 `hostnamectl set-hostname ydzs-node1` 来修改 hostname。
```bash
vim /etc/hosts 添加如下信息
192.168.33.131 master1
192.168.33.132 node1
192.168.33.133 node2

或
cat >> /etc/hosts << EOF
192.168.33.131 master1
192.168.33.132 node1
192.168.33.133 node2
EOF
```
### 5.禁用防火墙：
```bash
systemctl stop firewalld && systemctl disable firewalld
```
### 6.禁用swap（交换分区）
```bash
swapoff -a  && sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab


# 备注参考（通常仅执行上述命令即可）：
# 关闭Swap，机器重启后不生效
swapoff -a

# 修改/etc/fstab永久关闭Swap
cp -p /etc/fstab /etc/fstab.bak$(date '+%Y%m%d%H%M%S')

# CentOS
sed -i "s/\/dev\/mapper\/centos-swap/\#\/dev\/mapper\/centos-swap/g" /etc/fstab

# 修改后重新挂载全部挂载点
mount -a

# 查看Swap
free -m
cat /proc/swaps

# 或
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
### 7.禁用selinux
```bash
setenforce 0    # 临时
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config
```
### 8.加载br_netfilter模块

- 由于开启内核 ipv4 转发需要加载 br_netfilter 模块
```bash
 modprobe br_netfilter
```
### 9.将桥接的IPv4流量传递到iptables的链
```bash
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# 重启生效
sysctl --system 
或
sysctl -p /etc/sysctl.d/k8s.conf
```
> Explain:
> bridge-nf 使得 netfilter 可以对 Linux 网桥上的 IPv4/ARP/IPv6 包过滤。比如，设置`net.bridge.bridge-nf-call-iptables＝1`后，二层的网桥在转发包时也会被 iptables的 FORWARD 规则所过滤。常用的选项包括：
>    - net.bridge.bridge-nf-call-arptables：是否在 arptables 的 FORWARD 中过滤网桥的 ARP 包
>    - net.bridge.bridge-nf-call-ip6tables：是否在 ip6tables 链中过滤 IPv6 包
>    - net.bridge.bridge-nf-call-iptables：是否在 iptables 链中过滤 IPv4 包
>    - net.bridge.bridge-nf-filter-vlan-tagged：是否在 iptables/arptables 中过滤打了 vlan 标签的包。

### 10.安装 ipvs：

- 保证在节点重启后能自动加载所需模块。使用`lsmod | grep -e ip_vs -e nf_conntrack_ipv4`命令查看是否已经正确加载所需的内核模块
```bash
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

- 接下来还需要确保各个节点上已经安装了 ipset 软件包：
```bash
yum install ipset
```

- 为了便于查看 ipvs 的代理规则，最好安装一下管理工具 ipvsadm：
```bash
yum install ipvsadm -y
```
### 11.同步服务器时间
```bash
#yum和dnf都可以安装，推荐用dnf
yum install chrony -y
systemctl enable chronyd && systemctl start chronyd

#备份配置
cp /etc/chrony.conf  /etc/chrony.conf.orig
#注解掉pool
sed -i '/^pool/s/^/#/' /etc/chrony.conf
grep '#pool' /etc/chrony.conf
sed -i '/#pool/a\server cn.pool.ntp.org iburst' /etc/chrony.conf
sed -i '/#pool/a\server ntp.ntsc.ac.cn iburst' /etc/chrony.conf
sed -i '/#pool/a\server ntp1.aliyun.com iburst' /etc/chrony.conf
grep -A 3 '#pool' /etc/chrony.conf

# 重启
systemctl restart chronyd.service

chronyc sources
# 输出如下内容
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^- 0.cl.ntp.edgeuno.com          3   6     7     1  -3631us[+4657us] +/-  286ms
^? ntp8.flashdance.cx            2   6     5     1  -6585us[+1703us] +/-  172ms
^* time.neu.edu.cn               1   6     7     1   -104us[+8184us] +/-   29ms
^+ ntp6.flashdance.cx            2   6     7     0    -16ms[  -16ms] +/-  188ms
(base) [root@automlgpu3 ~]# date
Wed Jun 30 13:53:22 CST 2021
```
### 12.安装常用软件
```bash
yum install -y vim  lrzsz wget curl man tree rsync gcc gcc-c++ cmake telnet
```

---

## STEP2:安装依赖——Docker
详情查阅：[【docker-安装-卸载指南】](https://www.yuque.com/ronin-ssw/emrye0/hkiq83)

- Note:
   - 由于默认情况下 kubelet 使用的 cgroupdriver 是 systemd，所以需要保持 docker 和kubelet 的 cgroupdriver 一致，我们这里修改 docker 的 cgroupdriver=systemd。如果不修改 docker 则需要修改 kubelet 的启动配置，需要保证两者一致。

---

## STEP3:安装kubeadm/kubelet/kubectl
> Note: 
> - 执行此步骤前确保STEP1和STEP2都成功执行
> - 下列过程使用执行yum源的方式进行安装
> - 此步骤所有的结点都需要执行

### 1.设置yum源

- 能使用科学上网设置源的方式
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

- 不能科学上网设置源的方式
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
### 2.重新构建yum缓存
```bash
yum -y install epel-release
yum clean all
yum makecache
```
### 3.安装kubeadm/kubelet/kubectl
```bash
yum -y install kubelet-1.19.4 kubeadm-1.19.4 kubectl-1.19.4

# 查看 k8s 的版本
kubelet --version

# 查看 kubeadm 的版本
kubeadm version

# 查看 kubectl 版本
kubectl version --client
```
### 4.设置kubelet服务开机启动
```bash
systemctl enable kubelet && systemctl start kubelet
```

---

## STEP4: 初始化集群
> Note: master结点配置kubeadm 初始化文件
> - 执行的初始化的方式有多种
>    - 配置yaml的方式
>    - 使用命令行指定配置参数方式

### 方式一：配置yaml的方式
#### 1.导出默认的初始化配置：
```bash
$ kubeadm config print init-defaults > kubeadm.yaml
```
#### 2.根据需求修改配置
> - kube-proxy 的模式为 ipvs
> - 安装 flannel 网络插件,需要将 networking.podSubnet 设置为`10.244.0.0/16`：

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.33.131  # apiserver 节点内网IP
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: ydzs-master  # 默认读取当前master节点的hostname
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers  # 修改成阿里云镜像源
kind: ClusterConfiguration
kubernetesVersion: v1.16.2
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16  # Pod 网段，flannel插件需要使用这个网段
  serviceSubnet: 10.96.0.0/12
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs  # kube-proxy 模式
```
#### 3.执行初始化
```bash
kubeadm init --config kubeadm.yaml

# 执行成功会显示如下内容
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.33.131:6443 --token j7t5rh.7ao1kxy4ngj3ua2u \
    --discovery-token-ca-cert-hash sha256:22a3006ef3935419f13f3da59592f2450da4fd33035f81bd0d8048efd0cb38aa 
```
### 方式二：使用命令行指定配置参数方式
```bash
kubeadm init \
--apiserver-advertise-address=192.168.33.131 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.18.2 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 
```

- 参数说明：
```bash
--apiserver-advertise-address：本机IP（也是 Master的IP），使用内网IP
--image-repository registry.aliyuncs.com/google_containers ：拉取镜像的地址，这里使用阿里云的镜像仓库
--kubernetes-version： k8s 版本号
--service-cidr：暴露服务的IP网段，IP不与现有的冲突就可以
--pod-network-cidr： Pod的IP网段，IP不与现有的冲突就可以
```
### 拷贝 kubeconfig 文件
```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 查看节点信息

- 状态为NoteReady的原因是没有安装网卡
```bash
kubectl get nodes

# 输出如下信息：
NAME        STATUS     ROLES    AGE     VERSION
centos801   NotReady   master   4h55m   v1.19.4
```

---

## STEP5:添加节点
> Note: 
> - 安装kubeadm 、 kubelet 、kubectl 
> - 如果忘了token值，可以使用下列命令重新生成
>    - kubeadm token create --print-join-command

### 1.安装kubeadm 、 kubelet 、kubectl 

- 可以参考上面master节点的安装方式
- 注意:node节点不需要执行init的操作
### 2.执行join操作
```bash
kubeadm join 192.168.33.131:6443 --token j7t5rh.7ao1kxy4ngj3ua2u \
    --discovery-token-ca-cert-hash sha256:22a3006ef3935419f13f3da59592f2450da4fd33035f81bd0d8048efd0cb38aa 
    
# 执行成功后会显示如下内容：
[root@centos803 ~]# kubeadm join 192.168.33.131:6443 --token whrgqy.u4qqzag60frlk4vw     --discovery-token-ca-cert-hash sha256:5846f57ce7e8843f5adc272a3261a8b5b158867328134ee745622eef2185e532
W0304 14:10:56.883083   26739 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "centos803" could not be reached
        [WARNING Hostname]: hostname "centos803": lookup centos803 on 8.8.8.8:53: no such host
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
### 3.在master查看节点信息

- 状态为NoteReady的原因是没有安装网卡
```bash
kubectl get nodes

# 输出如下信息：
NAME        STATUS     ROLES    AGE     VERSION
centos801   NotReady   master   4h55m   v1.19.4
centos802   NotReady   <none>   4h16m   v1.19.4
centos803   NotReady   <none>   3h46m   v1.19.4
```
## STEP6: 安装网络插件
Note: 在master节点安装就好
> 常用的kubernetes网络插件有：
> - flannel（此处选择安装）
> - calico

[k8s官方文档中网络插件说明](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
### 1.安装flannel
> 当我们部署完网络插件后执行 ifconfig 命令，正常会看到新增的`cni0`与`flannel1`这两个虚拟设备，但是如果没有看到`cni0`这个设备也不用太担心，我们可以观察`/var/lib/cni`目录是否存在，如果不存在并不是说部署有问题，而是该节点上暂时还没有应用运行，我们只需要在该节点上运行一个 Pod 就可以看到该目录会被创建，并且`cni0`设备也会被创建出来。

```bash
$ wget https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
# 因为有节点是多网卡，所以需要在资源清单文件中指定内网网卡
# 搜索到名为 kube-flannel-ds-amd64 的 DaemonSet，在kube-flannel容器下面
$ vi kube-flannel.yml
......
containers:
- name: kube-flannel
  image: quay.io/coreos/flannel:v0.11.0-amd64
  command:
  - /opt/bin/flanneld
  args:
  - --ip-masq
  - --kube-subnet-mgr
  - --iface=eth0  # 如果是多网卡的话，指定内网网卡的名称
......
$ kubectl apply -f kube-flannel.yml  # 安装 flannel 网络插件
```
### 2.再次在master查看节点信息

- 状态为NoteReady的原因是没有安装网卡
```bash
kubectl get nodes

# 输出如下信息：
NAME        STATUS     ROLES    AGE     VERSION
centos801   Ready      master   4h55m   v1.19.4
centos802   Ready      <none>   4h16m   v1.19.4
centos803   Ready      <none>   3h46m   v1.19.4
```

---

## 备注内容：
### 重置的方法：
如果你的集群安装过程中遇到了其他问题，我们可以使用下面的命令来进行重置：
```bash
$ kubeadm reset
$ ifconfig cni0 down && ip link delete cni0
$ ifconfig flannel.1 down && ip link delete flannel.1
$ rm -rf /var/lib/cni/
```

---

### 彻底卸载的方法
```bash
#!/bin/bash
kubeadm reset -f
modprobe -r ipip
lsmod
rm -rf ~/.kube/
rm -rf /etc/kubernetes/
rm -rf /etc/systemd/system/kubelet.service.d
rm -rf /etc/systemd/system/kubelet.service
rm -rf /usr/bin/kube*
rm -rf /etc/cni
rm -rf /opt/cni
rm -rf /var/lib/etcd
rm -rf /var/etcd
yum -y remove kubeadm kubectl kubelet
```
## 常用插件安装

> 所有的插件都在master主机上安装，它会自动推送到node节点上安装，如需要会连接上master


### 1.安装kubernetes-dashboard

- 下载安装
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
```

- 查看是否安装成功
```bash
kubectl get po --all-namespaces|grep dashboard
```

- 修改node为NodePort模式
```bash
kubectl patch svc -n kubernetes-dashboard kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}'
```

- 查看服务 svc是service是缩写
```bash
kubectl get svc -n kubernetes-dashboard
```

- 访问浏览器,使用https的方式访问
```
https://192.168.33.130:32005/
```

- 输入刚刚初始化生成的token: tsmqw9.wbp9qzvd2owjgj6s
```
kubeadm join 192.168.33.132:6443 --token  \
    --discovery-token-ca-cert-hash sha256:5ca47847041f5f152d818e38355a8972174413cdb9f0f515393ffce773a5a2db
```

- 创建dashboard管理用户及token
```
# 创建dashboard管理用户
kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard

# 绑定用户为集群管理用户
kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin \
--serviceaccount=kubernetes-dashboard:dashboard-admin

# 查看管理用户的token, 并复制到浏览器中登录
kubectl describe secret -n kubernetes-dashboard dashboard-admin-token
```

- 下列为安装dashboard时执行步骤的log，供参考
```
[root@centos801 ~]# kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
namespace/kubernetes-dashboard configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
serviceaccount/kubernetes-dashboard configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
service/kubernetes-dashboard configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
secret/kubernetes-dashboard-certs configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
secret/kubernetes-dashboard-csrf configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
secret/kubernetes-dashboard-key-holder configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
configmap/kubernetes-dashboard-settings configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
role.rbac.authorization.k8s.io/kubernetes-dashboard configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/kubernetes-dashboard configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
service/dashboard-metrics-scraper configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/dashboard-metrics-scraper configured
[root@centos801 ~]# kubectl get po --all-namespaces|grep dashboard
kubernetes-dashboard   dashboard-metrics-scraper-6b4884c9d5-kmr82   1/1     Running            0          132m
kubernetes-dashboard   dashboard-metrics-scraper-6b4884c9d5-nq7ns   1/1     Terminating        0          5h8m
kubernetes-dashboard   kubernetes-dashboard-7bfbb48676-phc8t        1/1     Terminating        0          5h8m
kubernetes-dashboard   kubernetes-dashboard-7bfbb48676-pjfgb        1/1     Running            0          132m
kubernetes-dashboard   kubernetes-dashboard-7f99b75bf4-765h8        0/1     Pending            0          16s
[root@centos801 ~]# kubectl patch svc -n kubernetes-dashboard kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}'

service/kubernetes-dashboard patched
[root@centos801 ~]# service/kubernetes-dashboard patched
-bash: service/kubernetes-dashboard: 没有那个文件或目录
[root@centos801 ~]# kubectl patch svc -n kubernetes-dashboard kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}'
service/kubernetes-dashboard patched (no change)
[root@centos801 ~]# kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.106.23.98    <none>        8000/TCP        5h10m
kubernetes-dashboard        NodePort    10.108.155.79   <none>        443:32005/TCP   5h10m
[root@centos801 ~]# kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
serviceaccount/dashboard-admin created
[root@centos801 ~]# kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin \
> --serviceaccount=kubernetes-dashboard:dashboard-admin
clusterrolebinding.rbac.authorization.k8s.io/dashboard-cluster-admin created
[root@centos801 ~]# kubectl describe secret -n kubernetes-dashboard dashboard-admin-token
Name:         dashboard-admin-token-qflhd
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: 8a81530d-8abf-4af1-aa66-f026bbee16f7

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IldWZTB2aWVxWE0tVUhad3pQdnRhM1Z2a01PSi1rMS1EemdvQk44bVNMVkUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tcWZsaGQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiOGE4MTUzMGQtOGFiZi00YWYxLWFhNjYtZjAyNmJiZWUxNmY3Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmRhc2hib2FyZC1hZG1pbiJ9.GdJoeFOZB_p0qvc1RmU0o2MfyCZl_ElWQzrieILwKCkPZpJlfDCPdXGBPkrtGlCIBfBvufaLwyFBeer9USb-3AcfwYBfACI5ZUUE6Wu6svTJwsMFdIIgCtWBDldcqKU1TRcGPaOaEIPkne_OZX4sn2hSny04wc4ewfv99iUakTBi8etBx4cBBs5V55lZ47aTb0DCsit8cKRsj-7nEqWX3yzlREe_UBvAa7nRXQaM4dcgGXolWphYYxQQAdLi3Iq2qzqxu3jdodWKgOa2pvMM0Opd_zbtVKsABZA0TihxtWs6w5clHpScsRt-urBNTJVz7tuYHkkV6AyO6CZ-8ZOCYg
ca.crt:     1025 bytes
namespace:  20 bytes

[root@centos801 ~]# kubectl get nodes
NAME        STATUS     ROLES    AGE     VERSION
centos801   Ready      master   6h47m   v1.18.2
centos802   NotReady   <none>   6h8m    v1.18.2
centos803   Ready      <none>   5h38m   v1.18.2
```

- 集群成功部署



### 扩展

#### 1.获取ca证书的方法

> discovery-token-ca-cert-hash 的值是不会变的，想要查看则执行：


```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

#### 2.配置自动补全的命令

```
# 安装bash自动补全插件
yum install bash-completion -y

# 设置kubectl与kubeadm命令补全，下次login生效
kubectl completion bash >/etc/bash_completion.d/kubectl
kubeadm completion bash > /etc/bash_completion.d/kubeadm

------------------------------------或者下面的方式----------------------------------------------------------------
dnf install bash-completion -y
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
```
