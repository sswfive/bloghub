---
title: k8s中etcd挂载内存卷步骤
date: 2021-07-02 11:29:47
tags: 
  - K8S
---
## STEP1: 停掉API-server和 etcd
```bash
systemctl stop kubelet && systemctl stop containerd
```
## STEP2: 修改/etc/fstab 加上tmpfs挂载项
```bash
mkdir /data/etcd-data
mount -a
```
## STEP3： 拷贝数据到新的文件夹下， 然后创建软连接
```bash
cp -r /var/lib/etcd/* /data/etcd-data
rm -rf /var/lib/etcd
ln -s /data/etcd-data /var/lib/etcd

```
## STEP4: 重启API-server 和 etcd 观察集群是否正常
```bash
systemctl stop containerd && systemctl stop kubelet
```

## STEP5: 设置自动备份和恢复
```bash
mkdir /data/etcd-data-backup

# 备份
rsync  -a --delete --recursive --force /data/etcd-data /data/etcd-data-backup

# 还原
rsync -a --recursive --force /data/etcd-data-backup /data/etcd-data
```

