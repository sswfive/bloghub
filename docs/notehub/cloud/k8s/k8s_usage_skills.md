---
title: k8s实用小技巧
date: 2023-02-19 21:30:58
tags: 
  - K8S
---
### 1.快速编写yaml文件
#### STEP1:使用命令生成yaml文件
```bash
# kubectl create 
kubectl create deployment web --image=nginx -o yaml --dry-run
# -o 使用yaml格式展示
# -dry-run 尝试运行，并不会真正运行
# 保存至myweb.yaml

kubectl create deployment web --image=nginx -o yaml --dry-run > myweb.yaml

```
#### STEP2:使用命令导出 yaml 文件
```bash
kubectl get
kubectl get deploy # 查看部署
kubectl get deploy nginx -o yaml --export > myweb.yaml
```
### 
### 
### 2.把compose文件转换成k8s声明式配置文件。

- 下载链接：[https://github.com/kubernetes/kompose/releases/](https://github.com/kubernetes/kompose/releases/)
```bash
# 下载安装
curl -L https://github.com/kubernetes/kompose/releases/download/v1.22.0/kompose-darwin-amd64 -o kompose

chmod +x compose && sudo mv kompose /usr/local/bin/

# 使用
#step1:
kompose convert     

step2:调整转换后的文件内容

step3:kubectl apply -f k8s  # 安装应用到k8s集群
```

- 效果
```bash
(base) ➜  bluewhale_docker ls
bk-docker-compose.yml bluewhale             compose               docker-compose.yml    frontend              mysql
(base) ➜  bluewhale_docker kompose convert
WARN Volume mount on the host "/Users/roninssw/ssWang/codehub/hccodes/bluewhale_docker/compose/mysql/init/init.sql" isn't supported - ignoring path on the host
WARN Volume mount on the host "/Users/roninssw/ssWang/codehub/hccodes/bluewhale_docker/compose/mysql/conf/my.cnf" isn't supported - ignoring path on the host
INFO Network db_network is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
INFO Network web_network is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
WARN Volume mount on the host "/Users/roninssw/ssWang/codehub/hccodes/bluewhale_docker/bluewhale" isn't supported - ignoring path on the host
WARN Volume mount on the host "/Users/roninssw/ssWang/codehub/hccodes/bluewhale_docker/compose/bw_uwsgi" isn't supported - ignoring path on the host
INFO Network web_network is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
INFO Network db_network is detected at Source, shall be converted to equivalent NetworkPolicy at Destination
INFO Kubernetes file "mysqldb-service.yaml" created
INFO Kubernetes file "nginx-service.yaml" created
INFO Kubernetes file "website-service.yaml" created
INFO Kubernetes file "mysqldb-pod.yaml" created
INFO Kubernetes file "mysqldb-claim0-persistentvolumeclaim.yaml" created
INFO Kubernetes file "bluewhale-db-vol-persistentvolumeclaim.yaml" created
INFO Kubernetes file "mysqldb-claim2-persistentvolumeclaim.yaml" created
INFO Kubernetes file "db_network-networkpolicy.yaml" created
INFO Kubernetes file "nginx-pod.yaml" created
INFO Kubernetes file "static-volume-persistentvolumeclaim.yaml" created
INFO Kubernetes file "web_network-networkpolicy.yaml" created
INFO Kubernetes file "website-pod.yaml" created
INFO Kubernetes file "website-claim0-persistentvolumeclaim.yaml" created
INFO Kubernetes file "website-claim2-persistentvolumeclaim.yaml" created
```

### 3.使用k8s部署django应用的流程：

1. 搭建k8s集群环境
2. 配置kube/config文件（`vim ~/.kube/config`）   
3. 配置完成以后可以通过`kubectl` 和集群进行交互
4. 使用Dockerfile文件编写Django服务并生成镜像，或者使用docker-compose生成，
5. 如果使用docker-compose生成的镜像，可以使用kompose 脚本工具对docker-compose文件转成k8s的yaml文件
6. 修改生成的yaml文件，
7. 使用`kubectl apply -f 文件/文件夹` ，运行 服务的容器œ

### 4. 部署django服务到阿里云的k8s集群的流程        

- step1：创建kubernetes集群，创建镜像仓库
- step2: 配置本地的kubeconfig
- step3: Docker login 到镜像仓库
- step4: Docker build & docker push 推送镜像到镜像仓库
- step5:  k8s 部署到集群：` kubectl apply -f k8s`
