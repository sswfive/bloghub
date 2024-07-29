---
title: spark调度系统的探索
date: 2021-12-08 07:21:54
tags: 
  - Spark
  - BigData
---


## 调度系统的核心组件

### DAGScheduler

- 核心职责：

  - 把计算流图DAG拆分成执行阶段的Stages(不同的运行阶段),同时还要负责把Stages转化为任务集合TaskSets

- DAG到Stages的思路

  - 以Actions算子为起点，从后向前回溯DAG，以Shuffle操作为边界去划分Stages

- 对分布式任务Task的理解：

  | 属性名   | stageId       | stageAttempId | taskBinary | partition         | Locs       |
  | -------- | ------------- | ------------- | ---------- | ----------------- | ---------- |
  | 属性含义 | Task所在Stage | 失败重试编号  | 任务代码   | task对应的RDD分区 | 本地倾向性 |

  - stageId、stageAttemptId 标记了 Task 与执行阶段 Stage 的所属关系；
  - taskBinary 则封装了隶属于这个执行阶段的用户代码；
  - partition 就是我们刚刚说的 RDD 数据分区；
  - locs 属性以字符串的形式记录了该任务倾向的计算节点或是 Executor ID。

- 对应到wordcount案例中的流程如下：

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230314/image.5ctts62m3c40.webp)

- **DAGScheduler的职责总结：**

  1. 根据用户代码构建DAG

  2. 以Shuffle为边界切割Stages

  3. 基于Stages创建TaskSets，并将TaskSets提交给TaskScheduler请求调度

     

### SchedulerBackend

- 核心职责
  - 与资源管理器（Standalone、YARN、Mesos等）强绑定，是资源管理器在spark中的代理。

- 具体实现

  > ExecutorDataMap 是一种 HashMap，它的 Key 是标记 Executor 的字符串，Value 是一种叫做 ExecutorData 的数据结构。ExecutorData 用于封装 Executor 的资源状态，如 RPC 地址、主机地址、可用 CPU 核数和满配 CPU 核数等等，它相当于是对 Executor 做的“资源画像”。

  - 对于集群中可用的计算资源，SchedulerBackend用一个叫做ExecutorDataMap的数据结构，来记录每一个计算节点中的资源状态

  - 对内：使用Executor做资源画像

  - 对外：以WorkerOffer（封装了 Executor ID、主机地址和 CPU 核数，它用来表示一份可用于调度任务的空闲资源）为粒度提供计算资源。

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230314/image.5eidrx2htko0.webp)

- 在集群中的通信机制
  - SchedulerBackend 与集群内所有 Executors 中的 ExecutorBackend 保持周期性通信，双方通过 LaunchedExecutor、RemoveExecutor、StatusUpdate 等消息来互通有无、变更可用计算资源



### TaskScheduler

- 核心职责：
  - 接收Scheduler提供的WorkerOffer，TaskScheduler是按照任务的本地倾向性，来选出TaskSet中合适调度的Tasks。
- 具体实现：
  - Task与RDD的partitions是一一对应的，在创建Task的过程中，DAGScheduler会根据数据分区的物理地址，来为Task设置locs属性，locs属性记录了数据分片所在的计算节点，Executor进程ID。

- Task的特点：
  - 每个任务都是自带本地倾向性，通俗的理解就是，每个任务都有自己的调度意愿【具体可见下文的调度系统的调度策略部分内容】
- 任务流程：
  - Scheduler—>WorkerOffer—>TaskScheduler—>LaunchTask—>SchedulerBackend—>LauchTask—>ExecutorBackend



### ExecutorsBackend

- 核心职责：

  1. 接收SchedulerBackend发来的任务，调用Executors线程池中的CPU线程执行Task,每个线程负责处理一个Task；
  2. 每个Task处理完毕，这些线程便会通过ExecutorBackend向Driver端的SchedulerBackend发送StatusUpdate事件，告知Task执行状态；
  3. TaskScheduler与SchedulerBackend通过接力的方式，最终将状态汇报给DAGScheduler；

- 任务分发和执行的示意图

  ![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230314/image.3rbx0fadovc0.webp)

---



## 调度系统的核心思想

- 数据不动、代码动
  - 具体理解：在任务调度过程中，为了完成分布式计算， spark倾向于让数据待在原地，保持不动，而把计算任务（代码）调度、分发到数据所在的地方，从而消除数据分发引入的性能隐患。毕竟相比于分发数据，分发代码要轻量很多。

---



## 调度系统的调度策略

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230314/image.3yjykpsnpru0.webp)

- TaskScheduler接收到WorkerOffer之后，按照本性倾向性的从强到弱的关系来遍历TaskSets中Tasks

### 本地倾向性的强弱关系

- PROCESS_LOCAL（强）
  - JVM进程中
- NODE_LOCAL（较强）
  - 节点内
- RACK_LOCAL（一般）
  - 不超出物理机架的范围
- ANY（随意）
  - 任意、无所谓、不重要

### 本地倾向性的初衷

- 本质上是用来区分计算（代码）与数据之间的关系。

---



## 调度系统的调度流程

1. DAGScheduler以Shuffle为边界，将开发者设计的计算流图DAG拆分成多个执行阶段Stages,然后每个Stage创建任务集TaskSet；
2. SchedulerBackend通过与Executors中的ExecutorBackend的交互来实时地获取集群中可用的计算资源，并将这些信息记录到ExecutorDataMap数据结构中；
3. 与此同时，SchedulerBackend根据ExecutorDataMap中可用资源创建WorkerOffer，以WorkerOffer为粒度提供计算资源；
4. 对于给定WorkerOffer，TaskScheduler结合TaskSet中任务的本地倾向，按照强弱关系的顺序，一次对TaskSet中的任务进行遍历，有限调度本地性倾向要求苛刻的Task;
5. 被选中的Task由TaskScheduler传递SchedulerBackend，再由SchedulerBAckend分发到Executors中的ExecutorBackend。Executors接收到Task之后，即调用本地线程池执行分布式任务。