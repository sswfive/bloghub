---
title: k8s的storageclass-nfs安装和使用
date: 2023-02-19 21:24:55
tags: 
  - K8S
---
# 安装：
> 共享数据目录设置为：/data/k8s/share/

## k8s-master节点安装（nfs-server）
### 1.关闭防火墙
```bash
systemctl stop firewalld.service && systemctl disable firewalld.service
```

### 2.安装nfs及依赖
```bash
 yum -y install nfs-utils rpcbind
```

### 3.设置共享目录权限
```bash
chmod 755 /data/k8s/share/
```

### 4.配置nfs
> 默认的配置文件路径：/etc/exports

```bash
$ vi /etc/exports
/data/k8s/share  *(rw,sync,no_root_squash)
```

- 配置参数说明：
   - *：表示任何人都有权限连接，当然也可以是一个网段，一个 IP，也可以是域名
   - rw：读写的权限
   - sync：表示文件同时写入硬盘和内存
   - no_root_squash：当登录 NFS 主机使用共享目录的使用者是 root 时，其权限将被转换成为匿名使用者，通常它的 UID 与 GID，都会变成 nobody 身份

### 5.启动nfs依赖-rpcbind
```bash
systemctl start rpcbind.service && systemctl enable rpcbind

# 查看状态是否为active
systemctl status rpcbind
```
### 6.启动nfs
```bash
systemctl start nfs.service && systemctl enable nfs

# 查看状态是否为active
systemctl status nfs
```

- 还可以通过一下命令确认是否安装启动成功
```bash
$ rpcinfo -p|grep nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    3   udp   2049  nfs_acl

```
### 7.查看挂载目录权限
```bash
$ cat /var/lib/nfs/etab 
/data/k8s/share	*(rw,sync,wdelay,hide,nocrossmnt,secure,no_root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,rw,secure,no_root_squash,no_all_squash)
```
## k8s-node节点安装（nfs-client）
### 1.关闭防火墙（同server端）
### 2.安装nfs及依赖（同server端）
### 3.启动nfs和依赖（同server端）
### 4.检查nfs收是否有共享目录
```bash
$ showmount -e 172.16.2.117
Export list for 172.16.2.117:
/data/k8s/share *
```
### 5.在client新建目录
```bash
$ mkdir -p /data/kubeadm/data
```
### 6.将nfs共享目录挂载到上述新建目录中
```bash
$ mount -t nfs 172.16.2.117:/data/k8s/share /data/kubeadm/data
```
### 7.测试

- 在client端新建一个文件
- 在server查看该文件是否存在

# 使用：
## 1.创建nfs-client-class.yaml
```yaml
apiVersion: storage.k8s.io/v1
 kind: StorageClass
 metadata:
   name: kubeflow-nfs-storage
   annotations:
     storageclass.kubernetes.io/is-default-class: "true"
 provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
```
## 2.创建nfs-client-deployment.yaml
```yaml
apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: nfs-client-provisioner
   labels:
     app: nfs-client-provisioner
   namespace: default
 spec:
   replicas: 1
   strategy:
     type: Recreate
   selector:
     matchLabels:
       app: nfs-client-provisioner
   template:
     metadata:
       labels:
         app: nfs-client-provisioner
     spec:
       serviceAccountName: nfs-client-provisioner
       containers:
         - name: nfs-client-provisioner
           image: quay.io/external_storage/nfs-client-provisioner:latest
           volumeMounts:
             - name: nfs-client-root
               mountPath: /persistentvolumes
           env:
             - name: PROVISIONER_NAME
               value: fuseim.pri/ifs
             - name: NFS_SERVER
               value: 172.16.2.117
             - name: NFS_PATH
               value: /data/k8s/share
       volumes:
         - name: nfs-client-root
           nfs:
             server: 172.16.2.117
             path: /data/k8s/share
```
## 3.创建nfs-client-sa.yaml
```yaml
apiVersion: v1
 kind: ServiceAccount
 metadata:
   name: nfs-client-provisioner
 
 ---
 kind: ClusterRole
 apiVersion: rbac.authorization.k8s.io/v1
 metadata:
   name: nfs-client-provisioner-runner
 rules:
   - apiGroups: [""]
     resources: ["persistentvolumes"]
     verbs: ["get", "list", "watch", "create", "delete"]
   - apiGroups: [""]
     resources: ["persistentvolumeclaims"]
     verbs: ["get", "list", "watch", "update"]
   - apiGroups: ["storage.k8s.io"]
     resources: ["storageclasses"]
     verbs: ["get", "list", "watch"]
   - apiGroups: [""]
     resources: ["events"]
     verbs: ["list", "watch", "create", "update", "patch"]
   - apiGroups: [""]
     resources: ["endpoints"]
     verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
 
 ---
 kind: ClusterRoleBinding
 apiVersion: rbac.authorization.k8s.io/v1
 metadata:
   name: run-nfs-client-provisioner
 subjects:
   - kind: ServiceAccount
     name: nfs-client-provisioner
     namespace: default
 roleRef:
   kind: ClusterRole
   name: nfs-client-provisioner-runner
   apiGroup: rbac.authorization.k8s.io
```
## 4.执行：
```bash
kubectl apply -f nfs-client-class.yaml
kubectl apply -f nfs-client-deployment.yaml
kubectl apply -f nfs-client-sa.yaml
```
## 5.查看：
```bash
$ kubectl get pods 
NAME                                      READY   STATUS             RESTARTS   AGE
nfs-client-provisioner-5bb4f74bfb-v9sqz   1/1     running            13         5d20h
```
