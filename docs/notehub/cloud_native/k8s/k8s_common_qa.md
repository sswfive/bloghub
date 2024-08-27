---
title: k8s部署常见问题汇总
date: 2021-03-22 21:31:17
tags: 
  - K8S
---
## 1.系统重启以后6643 was refused

```bash
# 报错内容：
[root@centos801 ~]# kubectl get nodes
The connection to the server 192.168.33.130:6443 was refused - did you specify the right host or port?

# 解决方案
sudo -i  # 为了频繁的执行某些只有超级用户才能执行的权限，而不用每次输入密码
swapoff -a  # 关闭系统的交换区。
exit
strace -eopenat kubectl version
```

## 2.node节点重新join时，需要执行下列命令
```bash
kubeadm reset

# 如果有报错信息如下：
The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
[root@centos803 ~]# iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

# 则执行命令，然后再执行join操作
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X


```
## 3.kube-flannel的状态异常

- 具体异常信息
```bash
[root@centos801 ~]# kubectl get pods -n kube-system
NAME                                READY   STATUS              RESTARTS   AGE
coredns-6d56c8448f-hnjbz            0/1     ContainerCreating   0          14m
coredns-6d56c8448f-tc4j2            0/1     ContainerCreating   0          14m
etcd-centos801                      1/1     Running             0          14m
kube-apiserver-centos801            1/1     Running             0          14m
kube-controller-manager-centos801   1/1     Running             0          14m
kube-flannel-ds-8bvql               0/1     CrashLoopBackOff    6          9m41s
kube-flannel-ds-lj5vp               0/1     CrashLoopBackOff    6          8m52s
kube-flannel-ds-qxgq9               0/1     CrashLoopBackOff    7          11m
kube-proxy-jsssg                    1/1     Running             0          14m
kube-proxy-pq9rf                    1/1     Running             0          8m52s
kube-proxy-z77tv                    1/1     Running             0          9m41s
kube-scheduler-centos801            1/1     Running             0          14m
```

- 错误原因是在进行kubeadm init时没有添加如下参数
```bash
--pod-network-cidr=10.244.0.0/16
```

- 但是检查初始化命令有进行设置此参数
```bash
kubeadm init \
--apiserver-advertise-address=192.168.33.131 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.19.4 \
--service-cidr=10.96.0.0/12 
--pod-network-cidr=10.244.0.0/16
```

- 寻思千遍，花了近三个小时在chrome、kubeadm init、kube join 之间辗转反侧
```bash
[root@centos801 ~]# kubeadm init \
> --apiserver-advertise-address=192.168.33.131 \
> --image-repository registry.aliyuncs.com/google_containers \
> --kubernetes-version v1.19.4 
W0312 14:43:44.370062  280711 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.19.4
[preflight] Running pre-flight checks
        [WARNING FileExisting-tc]: tc not found in system path
        [WARNING Hostname]: hostname "centos801" could not be reached
        [WARNING Hostname]: hostname "centos801": lookup centos801 on 114.114.114.114:53: no such host
```

- 发现最后一个参数丢失，最终人傻掉了，--pod-network-cidr 前面添加一个 "\",再重新执行init命令，即可解决问题
```bash
kubeadm init \
--apiserver-advertise-address=192.168.33.131 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.19.4 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16

[root@centos801 ~]# kubectl get pods -n kube-system
NAME                                READY   STATUS    RESTARTS   AGE
coredns-6d56c8448f-qhftp            1/1     Running   0          3m42s
coredns-6d56c8448f-w8h9n            1/1     Running   0          3m42s
etcd-centos801                      1/1     Running   0          3m58s
kube-apiserver-centos801            1/1     Running   0          3m59s
kube-controller-manager-centos801   1/1     Running   0          3m58s
kube-flannel-ds-5fljb               1/1     Running   0          52s
kube-flannel-ds-cxk58               1/1     Running   0          22s
kube-flannel-ds-z9zpr               1/1     Running   0          2m2s
kube-proxy-ftvsz                    1/1     Running   0          3m42s
kube-proxy-j6x7f                    1/1     Running   0          22s
kube-proxy-l6pxl                    1/1     Running   0          52s
kube-scheduler-centos801            1/1     Running   0          3m58s
```
## 4.重新进行集群init和join操作的正确流程
```bash
# master和node节点都需要执行
kubeadm reset --force
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

# 只在master节点执行
kubeadm init \
--apiserver-advertise-address=192.168.33.131 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.19.4 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 

# 只在node节点执行
kubeadm join 192.168.33.131:6443 --token hxxxre.qbaa224113efwjs1 \
  --discovery-token-ca-cert-hash sha256:208d7b79e19f460c5311eeeddfc9102316932d877f717b8bb1bc86dbb85133a7
```

**1/ 各组件启动失败时，如何方便的查看错误？**
可使用命令 `systemctl status <组件名>` 或 `journalctl -xefu <组件名>`查看日志。
**2/ 注意cgroup启动问题**
```
failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"
```
kubelet 默认使用 `cgroupfs` 启动，可修改为docker 的`systemd`， 或修改docker 启动为 `cgroupfs`。
docker 修改的地方为 /etc/docker/daemon.json
```
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```
kuberlet 可修改systemd 的service中启动命令增加`--cgroup-driver`参数：
```
# 可指定启动参数 
DOCKER_CGROUPS=$(docker info | grep 'Cgroup' | cut -d' ' -f3)
--cgroup-driver=$DOCKER_CGROUPS
```
**3/ 节点删除后如何重新授权？**
节点删除后，重启 kubelet。在Master 端是看不到申请信息的，需要将上次申请的证书删除，再重启。
```
# master
kubectl delete node node01
# node01 
rm -f /opt/kubernetes/ssl/kubelet*
systemctl restart kubelet
```
**4/ 排查pod无法启动的思路**
```
# 1. 先查看pod的最近的操作
kubectl describe pod <pod-name> -n kube-system 
# 2. 查看pod 的日志
kubectl logs pod <pod-name>
```
**5/ 解决Dashboard 2.0 chrome浏览器不能访问的问题**
```
mkdir key && cd key
# 1 生成证书
openssl genrsa -out dashboard.key 2048
openssl req -new -out dashboard.csr -key dashboard.key -subj '/CN=kubernetes-dashboard-certs'
openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt
# 2 删除原有的证书secret
kubectl delete secret kubernetes-dashboard-certs -n kube-system
# 3 创建新的证书secret
kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard.key --from-file=dashboard.crt -n kube-system
# 4 查看dashboard pod，v2.0是 -n kubernetes-dashboard
kubectl get pod -n kube-system
# 5 重启dashboard pod，v2.0是 -n kubernetes-dashboard
kubectl delete pod <pod name> -n kube-system
```
**6/ 如何彻底的删除一个pod?**
K8S中pod 是有`deployments`来发布的，所以直接删除pod后，k8s无法识别是人为还是故障，它会根据`deployments` 描述的信息，重新启动一个pod。所以要删除pod 时，直接删除对应的 `deployments `，pod会自动删除。
```
kubectl delete deployment <deployment_name> -n <namespace>
```
**7/ 节点维护时，如何手动驱赶pod?**
```
# 1、标记节点不可调度
kubectl cordon <node name>
# 2、手动迁移pod 
kubectl drain <node name> --delete-local-data --ignore-daemonsets --force
# 3、带节点维护完成之后，恢复可调度
kubectl uncordon <node name>
```

## 5. kubeadm 安装k8s1.19.4需要的镜像说明：
> 执行kubeadm init 卡在pull image ，由于默认源是谷歌，镜像通常拉取不下来，所以需要使用国内源进行必要镜像的拉取

### step1:查看当前版本的kubeadm 需要的镜像包和版本
```bash
kubeadm  config images list  
```

- 日志
```bash
(base) [root@automlgpu4 k8s]# kubeadm config images list
W0608 17:33:50.021855   83264 version.go:102] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get "https://dl.k8s.io/release/stable-1.txt": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
W0608 17:33:50.022758   83264 version.go:103] falling back to the local client version: v1.19.4
W0608 17:33:50.023105   83264 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
k8s.gcr.io/kube-apiserver:v1.19.4
k8s.gcr.io/kube-controller-manager:v1.19.4
k8s.gcr.io/kube-scheduler:v1.19.4
k8s.gcr.io/kube-proxy:v1.19.4
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/coredns:1.7.0
```
### step2:将默认的配置文件导出并编辑
```bash
kubeadm config print init-defaults > kubeadm.conf 
vim kubeadm.conf  # 打开文件并修改
```
```bash
(base) [root@automlgpu4 k8s]# vim kubeadm.conf 

 1 kubeadm.conf                                                                                                                                                                            X 
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
   advertiseAddress: 172.16.2.117  # 本机的ip
   bindPort: 6443
 nodeRegistration:
   criSocket: /var/run/dockershim.sock
   name: automlgpu4
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
 imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers  # 更更换为阿里镜像源
 kind: ClusterConfiguration
 kubernetesVersion: v1.19.4
 networking:
   dnsDomain: cluster.local
   serviceSubnet: 10.96.0.0/12
 scheduler: {}
 ~                                                                                                                                                                                           
 ~                                                                                                                                                                                           
 ~             
```
### step3:拉取image
```bash
kubeadm config images pull --config kubeadm.conf 
```

- 日志
```bash
(base) [root@automlgpu4 k8s]# kubeadm config images pull --config kubeadm.conf
W0608 17:09:52.812134   60183 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.19.4
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.19.4
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.19.4
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.19.4
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.13-0
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.7.0
```
### step4: 修改镜像标签
```bash
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.19.4 k8s.gcr.io/kube-apiserver:v1.19.4
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.19.4 k8s.gcr.io/kube-controller-manager:v1.19.4
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.19.4 k8s.gcr.io/kube-scheduler:v1.19.4
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.19.4 k8s.gcr.io/kube-proxy:v1.19.4
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2 k8s.gcr.io/pause:3.2
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.13-0 k8s.gcr.io/etcd:3.4.13-0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.7.0 k8s.gcr.io/coredns:1.7.0
```
### step5:执行kubeadm init 操作
```bash
kubeadm init --config kubeadm.conf
```

- 日志：
```bash
(base) [root@automlgpu4 k8s]# kubeadm init --config kubeadm.conf 
W0608 17:24:49.913798   72683 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.19.4
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [automlgpu4 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.16.2.117]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [automlgpu4 localhost] and IPs [172.16.2.117 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [automlgpu4 localhost] and IPs [172.16.2.117 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 17.004681 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.19" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node automlgpu4 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node automlgpu4 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: abcdef.0123456789abcdef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.2.117:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:ede5cb0692ec5bf94eb0b2305ac3bea32e25bc10e87dce5a9c9be25bbcedc9cc 
(base) [root@automlgpu4 k8s]# mkdir -p $HOME/.kube
(base) [root@automlgpu4 k8s]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
(base) [root@automlgpu4 k8s]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
(base) [root@automlgpu4 k8s]# kubectl get node 
NAME         STATUS     ROLES    AGE   VERSION
automlgpu4   NotReady   master   41s   v1.19.4

```

