---
title: spark中shuffle的工作原理
date: 2021-12-08 22:41:35
tags: 
  - Spark
  - BigData
---

## 什么Shuffle？

> Shuffle的本意是扑克的洗牌

- 在分布式计算场景中，Shuffle被引申为集群范围内跨节点、跨进程的数据分发



## Shuffle有什么特点？

- Shuffle的计算会消耗所有类型的硬件资源，比如CPU、内存、磁盘与网络等；
  - 消耗硬件资源的具体表现为：Shuffle中的哈希与排序操作会大量消耗CPU，而Shuffle Write生成中间文件的过程，会消耗内存资源与磁盘I/O，最后，Shuffle Read阶段的数据拉取会引入大量的网络I/O。
- Shuffle是资源密集型计算



## 为什么需要shuffle?

- 在DAG的计算链条中，shuffle环节的执行性能是最差的。（性能差为啥还存在呢？）

  - 计算过程之所以需要shuffle，往往是有计算逻辑、或业务逻辑决定的

    > 在 Word Count 的例子中，我们的“业务逻辑”是对单词做统计计数，那么对单词“Spark”来说，在做“加和”之前，我们就是得把原本分散在不同 Executors 中的“Spark”，拉取到某一个 Executor，才能完成统计计数的操作。



## shuffle的工作原理

### Word Count案例分析

> reduceByKey算子中引入Shuffle的操作

```scala
import org.apache.spark.rdd.RDD
 
// 这里的下划线"_"是占位符，代表数据文件的根目录
val rootPath: String = _
val file: String = s"${rootPath}/wikiOfSpark.txt"
 
// 读取文件内容
val lineRDD: RDD[String] = spark.sparkContext.textFile(file)
 
// 以行为单位做分词
val wordRDD: RDD[String] = lineRDD.flatMap(line => line.split(" "))
val cleanWordRDD: RDD[String] = wordRDD.filter(word => !word.equals(""))
 
// 把RDD元素转换为（Key，Value）的形式
val kvRDD: RDD[(String, Int)] = cleanWordRDD.map(word => (word, 1))
// 按照单词做分组计数
val wordCounts: RDD[(String, Int)] = kvRDD.reduceByKey((x, y) => x + y)
```

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230314/image.61jqtgvrqgk0.webp)

- reduceByKey的计算被分成切割为两个执行阶段。

  > Map阶段：shuffle之前的Stage
  >
  > Reduce阶段： shuffle之后的Stage

  - 在Map阶段，每个Executors先把自己负责的数据分区做初步聚合（局部聚合）；
  - 在shuffle环节，不同的单词被分发到不同节点的Executors中；
  - 在Reduce阶段，Executors以单词为Key做第二次聚合（全局聚合），从而完成统计计算的任务。



### Shuffle中间文件

- Map阶段与Reduce阶段，通过生产与消费shuffle中间文件的方式，来完成集群范围内的数据交换。
  - Map阶段生产shuffle中间文件，Reduce阶段消费shuffle中间文件，二者以中间文件为媒介，完成数据交换。

- 什么是shuffle中间文件？

  - shuffle中间文件时统称、泛指，它包含两类实体文件，一个是记录（Key，Value）键值对的data文件，另外一个是记录键值对所属Reduce Task的index文件。（index文件标记了data文件中的哪些记录，应该油下游Reduce阶段的哪些Task消费）

  - 结构：

    ![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230314/image.58th8v1o6vk0.webp)

- shuffle的数据交换

  - 数据交换规则又叫分区规则，因为它定义了分布式数据集在Reduce阶段如何划分数据分区

  - 计算公式

    ```scala
    //假设 Reduce 阶段有 N 个 Task，这 N 个 Task 对应着 N 个数据分区，那么在 Map 阶段，每条记录应该分发到哪个 Reduce Task，是由下面的公式来决定的
    
    P = Hash(Record Key) % N
    ```



### Shuffle Write

> shuffle中间文件的生成
>
> shuffle中间文件，是以Map Task为粒度生成的

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230314/image.hbl7flwv3i8.webp)

- Shuffle Write的流程
  1. 对于数据分区中的数据记录，逐一计算其目标分区，然后填充内存数据结构；
  2. 当数据结构填满后，如果分区中还有未处理的数据记录，就对结构中的数据记录按（目标分区ID，Key）排序，将所有数据溢出到临时文件，同时清空数据结构；
  3. 重复前两步，指到分区中所有的数据记录都被处理为止；
  4. 对所有临时文件和内存数据结构中剩余的数据记录做归并排序，生成数据文件和索引文件。



### Shuffle Read

> 对于每一个Map Task生成的中间文件，其中的目标分区数量是由Reduce阶段的任务数量（并行度）决定的。

- 对于所有Map Task生成的中间文件，Reduce Task需要通过网络从不同节点的硬盘中下载并拉取属于自己的数据内容，
- 不同的Reduce Task正是根据index文件中的起始索引来确定哪些数据内容是“属于自己的”
- Reduce 阶段不同于 Reduce Task 拉取数据的过程，往往也被叫做 Shuffle Read。