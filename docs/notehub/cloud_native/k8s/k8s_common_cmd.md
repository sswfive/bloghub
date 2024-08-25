---
title: k8s常用命令总结
date: 2021-03-09 18:14:31
tags: 
  - K8S
---
## 1. 创建/删除资源
一般会有两种方式：

- 通过yaml文件
- 命令创建。

```bash
# 通过文件创建一个Deployment
kubectl create -f /path/to/deployment.yaml
cat /path/to/deployment.yaml | kubectl create -f -

# 不过一般可能更常用下面的命令来创建资源
kubectl apply -f /path/to/deployment.yaml

# 删除资源
kubectl delete -f /path/to/deployment.yaml
或 
kubectl delete pod pod名称


# 通过kubectl命令直接创建
kubectl run nginx_app --image=nginx:1.9.1 --replicas=3
```

kubectl还提供了一些更新资源的命令，如

- kubectl edit
- kubectl patch
- kubectl replace

```bash
# kubectl edit：相当于先用get去获取资源，然后进行更新，最后对更新后的资源进行apply
kubectl edit deployment/nginx_app

# kubectl patch：使用补丁修改、更新某个资源的字段，比如更新某个node
kubectl patch node/node-0 -p '{"spec":{"unschedulable":true}}'
kubectl patch -f node-0.json -p '{"spec": {"unschedulable": "true"}}'

# kubectl replace：使用配置文件来替换资源
kubectl replace -f /path/to/new_nginx_app.yaml
```

## 2. 查看资源

获取不同种类资源的信息。
### 2.1 nodes相关命令
```shell
kubectl get nodes  # 查看集群的信息
```
### 2.2 pod相关命令
```shell
kubectl get pods  # 查看pods 

kubectl get pod -o wide  # 查看指定pod的详细信息

kubectl get pods --all-namespaces  # 查看所有命名空间的pods
或者
kubectl get pods -A

kubectl get pods -n 命名空间名 pod名称

# kubectl describe 资源类型 资源名称  
kubectl describe pod nginx-XXXXXX  # 查看名称为nginx-XXXXXX的Pod的信息

kubectl delete pod pod名 --grace-period=0 --force  # 强制删除一个pod(适用于普通delete无法删除的场景)




```
### 2.3 deployments相关命令
```shell
kubectl get deployments-A  # 查看所有空间的deployments
或者
kubectl get deployments --all-namespaces

kubectl get deployments -n kube-system  # 查看指定命名空间的deployments

kubectl describe deployment nginx  #查看名称为nginx的Deployment的信息
```
### 2.4  logs相关命令
```shell
kubectl logs - 查看pod中的容器的打印日志

(和命令docker logs 类似)

# kubectl logs Pod名称

#查看名称为nginx-pod-XXXXXXX的Pod内的容器打印的日志

kubectl logs -f nginx-pod-XXXXXXX
```
### 2.5  exec 相关命令
```shell
kubectl exec
- 在pod中的容器环境内执行命令(和命令docker exec 类似)

# kubectl exec Pod名称 操作命令

# 在名称为nginx-pod-xxxxxx的Pod中运行bash
kubectl exec-it nginx-pod-xxxxxx /bin/bash
```
### 2.6 namespace 操作
```shell
kubectl delete namespace 名称  # 删除指定namespace 
```
### 2.6 其他命令
```bash

# 指定资源的信息，格式：kubectl get <resource_type>/<resource_name>，比如获取deployment nginx_app的信息
kubectl get deployment/nginx_app -o wide

# 也可以对指定的资源进行格式化输出，比如输出格式为json、yaml等
kubectl get deployment/nginx_app -o json

# 还可以对输出结果进行自定义，比如对pod只输出容器名称和镜像名称
kubectl get pod httpd-app-5bc589d9f7-rnhj7 -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image

# 获取某个特定key的值还可以输入如下命令得到，此目录参照go template的用法，且命令结尾'\n'是为了输出结果换行
kubectl get pod httpd-app-5bc589d9f7-rnhj7 -o template --template='{{(index spec.containers 0).name}}{{"\n"}}'

#其他更多命令使用kubectl get --help查看
```

## 3. 部署命令集

部署命令包括资源的运行管理命令、扩容和缩容命令和自动扩缩容命令。

### 3.1 rollout命令

管理资源的运行，比如eployment、Daemonet、StatefulSet等资源。

- 查看部署状态：比如更新deployment/nginx_app中容器的镜像后查看其更新的状态。

```bash
kubectl set image deployment/nginx_app nginx=nginx:1.9.1
kubectl rollout status deployment/nginx_app
```

- 资源的暂停及恢复：发出一次或多次更新前暂停一个 Deployment，然后再恢复它，这样就能在Deployment暂停期间进行多次修复工作，而不会发出不必要的 rollout。

```bash
# 暂停
kubectl rollout pause deployment/nginx_app

# 完成所有的更新操作命令后进行恢复
kubectl rollout resume deployment/nginx_app
```

- 回滚：如上对一个Deployment的image做了更新，但是如果遇到更新失败或误更新等情况时可以对其进行回滚。

```bash
# 回滚之前先查看历史版本信息
kubectl rollout history deployment/nginx_app

# 回滚
kubectl rollout undo deployment/nginx_app

# 当然也可以指定版本号回滚至指定版本
kubectl rollout undo deployment/nginx_app --to-revision=<version_index>
```

### 3.2 scale命令

对一个Deployment、RS、StatefulSet进行扩/缩容。

```bash
# 扩容
kubectl scale deployment/nginx_app --replicas=5
# 如果是缩容，把对应的副本数设置的比当前的副本数小即可

# 另外，还可以针对当前的副本数目做条件限制，比如当前副本数是5则进行缩容至副本数目为3
kubectl scale --current-replicas=5 --replicas=3 deployment/nginx_app
```

### 3.3 autoscale命令

通过创建一个autoscaler，可以自动选择和设置在K8s集群中Pod的数量。

```bash
# 基于CPU的使用率创建3-10个pod
kubectl autoscale deployment/nginx_app --min=3 --max=10 --cpu_percent=80
```

## 4. 集群管理命令

### 4.1 cordon & uncordon命令

设置是否能够将pod调度到该节点上。

```bash
# 不可调度
kubectl cordon node-0

# 当某个节点需要维护时，可以驱逐该节点上的所有pods(会删除节点上的pod，并且自动通过上面命令设置
# 该节点不可调度，然后在其他可用节点重新启动pods)
kubectl drain node-0

# 待其维护完成后，可再设置该节点为可调度
kubectl uncordon node-0
```

### 4.2 taint命令

目前仅能作用于节点资源，一般这个命令通常会结合pod的tolerations字段结合使用，对于没有设置对应toleration的pod是不会调度到有该taint的节点上的，这样就可以避免pod被调度到不合适的节点上。一个节点的taint一般会包括key、value和effect(effect只能在NoSchedule, PreferNoSchedule, NoExecute中取值)。

```bash
# 设置taint
kubecl taint nodes node-0 key1=value1:NoSchedule

# 移除taint
kubecl taint nodes node-0 key1:NoSchedule-
```

如果pod想要被调度到上述设置了taint的节点node-0上，则需要在该pod的spec的tolerations字段设置：

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"

# 或者
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```

## 5. 其它

```bash
# 映射端口允许外部访问
kubectl expose deployment/nginx_app --type='NodePort' --port=80
# 然后通过kubectl get services -o wide来查看被随机映射的端口
# 如此就可以通过node的外部IP和端口来访问nginx服务了

# 转发本地端口访问Pod的应用服务程序
kubectl port-forward nginx_app_pod_0 8090:80
# 如此，本地可以访问：curl -i localhost:8090

# 在创建或启动某些资源的时候没有达到预期结果，可以使用如下命令先简单进行故障定位
kubectl describe deployment/nginx_app
kubectl logs nginx_pods
kubectl exec nginx_pod -c nginx-app <command>

# 集群内部调用接口(比如用curl命令)，可以采用代理的方式，根据返回的ip及端口作为baseurl
kubectl proxy &

# 查看K8s支持的完整资源列表
kubectl api-resources

# 查看K8s支持的api版本
kubectl api-versions
```

## 未归档命令大综合

```bash
# 查看具体pods，记得后边跟namespace名字哦
kubectl get pods  kubernetes-dashboard-76479d66bb-nj8wr --namespace=kube-system
# 查看pods具体信息
kubectl get pods -o wide kubernetes-dashboard-76479d66bb-nj8wr --namespace=kube-system
# 查看集群健康状态
kubectl get cs
# 获取所有deployment
kubectl get deployment --all-namespaces
# 列出该 namespace 中的所有 pod 包括未初始化的
kubectl get pods --include-uninitialized
# 查看deployment()
kubectl get deployment nginx-app
# 查看rc和servers
kubectl get rc,services
# 查看pods结构信息（重点，通过这个看日志分析错误）
# 对控制器和服务，node同样有效
kubectl describe pods xxxxpodsname --namespace=xxxnamespace
# 其他控制器类似吧，就是kubectl get 控制器 控制器具体名称
# 查看pod日志
kubectl logs $POD_NAME
# 查看pod变量
kubectl exec my-nginx-5j8ok -- printenv | grep SERVICE
# 集群
kubectl get cs           # 集群健康情况
kubectl cluster-info     # 集群核心组件运行情况
kubectl get namespaces    # 表空间名
kubectl version           # 版本
kubectl api-versions      # API
kubectl get events       # 查看事件
kubectl get nodes      //获取全部节点
kubectl delete node k8s2  //删除节点
kubectl rollout status deploy nginx-test
# 创建
kubectl create -f ./nginx.yaml           # 创建资源
kubectl create -f .                            # 创建当前目录下的所有yaml资源
kubectl create -f ./nginx1.yaml -f ./mysql2.yaml     # 使用多个文件创建资源
kubectl create -f ./dir                        # 使用目录下的所有清单文件来创建资源
kubectl create -f https://git.io/vPieo         # 使用 url 来创建资源
kubectl run -i --tty busybox --image=busybox    ----创建带有终端的pod
kubectl run nginx --image=nginx                # 启动一个 nginx 实例
kubectl run mybusybox --image=busybox --replicas=5    ----启动多个pod
kubectl explain pods,svc                       # 获取 pod 和 svc 的文档
# 更新
kubectl rolling-update python-v1 -f python-v2.json           # 滚动更新 pod frontend-v1
kubectl rolling-update python-v1 python-v2 --image=image:v2  # 更新资源名称并更新镜像
kubectl rolling-update python --image=image:v2                 # 更新 frontend pod 中的镜像
kubectl rolling-update python-v1 python-v2 --rollback        # 退出已存在的进行中的滚动更新
cat pod.json | kubectl replace -f -                              # 基于 stdin 输入的 JSON 替换 pod
强制替换，删除后重新创建资源。会导致服务中断。
kubectl replace --force -f ./pod.json
为 nginx RC 创建服务，启用本地 80 端口连接到容器上的 8000 端口
kubectl expose rc nginx --port=80 --target-port=8000
 
更新单容器 pod 的镜像版本（tag）到 v4
kubectl get pod nginx-pod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -
kubectl label pods nginx-pod new-label=awesome                      # 添加标签
kubectl annotate pods nginx-pod icon-url=http://goo.gl/XXBTWq       # 添加注解
kubectl autoscale deployment foo --min=2 --max=10                # 自动扩展 deployment “foo”
# 编辑资源
kubectl edit svc/docker-registry                      # 编辑名为 docker-registry 的 service
KUBE_EDITOR="nano" kubectl edit svc/docker-registry   # 使用其它编辑器
# 动态伸缩pod
kubectl scale --replicas=3 rs/foo                                 # 将foo副本集变成3个
kubectl scale --replicas=3 -f foo.yaml                            # 缩放“foo”中指定的资源。
kubectl scale --current-replicas=2 --replicas=3 deployment/mysql  # 将deployment/mysql从2个变成3个
kubectl scale --replicas=5 rc/foo rc/bar rc/baz                   # 变更多个控制器的数量
kubectl rollout status deploy deployment/mysql                         # 查看变更进度
# 删除
kubectl delete -f ./pod.json                                              # 删除 pod.json 文件中定义的类型和名称的 pod
kubectl delete pod,service baz foo                                        # 删除名为“baz”的 pod 和名为“foo”的 service
kubectl delete pods,services -l name=myLabel                              # 删除具有 name=myLabel 标签的 pod 和 serivce
kubectl delete pods,services -l name=myLabel --include-uninitialized      # 删除具有 name=myLabel 标签的 pod 和 service，包括尚未初始化的
kubectl -n my-ns delete po,svc --all # 删除 my-ns namespace下的所有 pod 和 serivce，包括尚未初始化的
kubectl delete pods prometheus-7fcfcb9f89-qkkf7 --grace-period=0 --force 强制删除

# 交互
kubectl logs nginx-pod                                 # dump 输出 pod 的日志（stdout）
kubectl logs nginx-pod -c my-container                 # dump 输出 pod 中容器的日志（stdout，pod 中有多个容器的情况下使用）
kubectl logs -f nginx-pod                              # 流式输出 pod 的日志（stdout）
kubectl logs -f nginx-pod -c my-container              # 流式输出 pod 中容器的日志（stdout，pod 中有多个容器的情况下使用）
kubectl run -i --tty busybox --image=busybox -- sh  # 交互式 shell 的方式运行 pod
kubectl attach nginx-pod -i                            # 连接到运行中的容器
kubectl port-forward nginx-pod 5000:6000               # 转发 pod 中的 6000 端口到本地的 5000 端口
kubectl exec nginx-pod -- ls /                         # 在已存在的容器中执行命令（只有一个容器的情况下）
kubectl exec nginx-pod -c my-container -- ls /         # 在已存在的容器中执行命令（pod 中有多个容器的情况下）
kubectl top pod POD_NAME --containers               # 显示指定 pod和容器的指标度量
# 调度配置
$ kubectl cordon k8s-node                                                # 标记 my-node 不可调度
$ kubectl drain k8s-node                                                 # 清空 my-node 以待维护
$ kubectl uncordon k8s-node                                              # 标记 my-node 可调度
$ kubectl top node k8s-node                                              # 显示 my-node 的指标度量
$ kubectl cluster-info dump                                             # 将当前集群状态输出到 stdout                                    
$ kubectl cluster-info dump --output-directory=/path/to/cluster-state   # 将当前集群状态输出到 /path/to/cluster-state
#如果该键和影响的污点（taint）已存在，则使用指定的值替换
$ kubectl taint nodes foo dedicated=special-user:NoSchedule
```

