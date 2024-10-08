---
title: k8s核心概念
date: 2021-03-18 21:19:32
tags: 
  - K8S
---

## 声明式管理 

- 通过yml声明期望的状态，k8s自动根据定义的状态进行调度（相对命令式管理而言，k8s也提供命令式管理的方式
- 声明式：`kubectl apply -f <directory>`
- 命令式：`kubectl run xxx,  kubectl expose...`
## 名词及解释

- Master node: 集群主控节点，上面运行调度容器，管理容器；
- Worker node:工作节点，上面运行应用的容器；
- Pods: 容器，k8s对容器进行管理， 自动创建、销毁容器； 
- Service: 使用标签选择算符（selectors) 标识一组pod,在集群内有固定IP,可以用于集群内部容器/应用提供稳定的访问入口，可以通过service名称访问到服务
- Deployment:一套部署容器的集合；
- ReplicationController:可以复制的容器，功能集比ReplicaSet少，已不推荐使用；
- ReplicaSet:可以扩容、缩容的容器副本集，在容器集不需要状态，也不需要被其他容器访问的时候，可以使用ReplicaSet.其他容器无法直接访问RS， 可以通过service来访问；
- StatefulSet:有状态的服务，比如zookeeper,集群被外部使用的时候，需要指定多个zookeeper服务的ip或者hostname.由于是有状态的，比如3个节点的zookeeper， 不论如何缩容（包括宕机恢复），3个节点容器名称/主机名总是zk-0, zk-1, zk-2
- Ingress: 对外暴露可以访问的入口，为服务提供外网（从集群外）的访问入口；
- Namespace:资源可以放在namespace下面， 不同namespace之间相互隔离；
- Controller:包括有节点控制器、路由控制器、服务控制器；
