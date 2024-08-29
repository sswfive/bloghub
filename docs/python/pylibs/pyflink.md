---
title: PyFlink之分布式计算
date: 2023-08-09 16:08:00
tags:
  - Python
---

# PyFlink之分布式计算


> 此文内容基于apache-flink-1.17.1版本，如有偏差详见官方文档

## 文档说明：
官方文档：

- [Flink官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.17/)
- [Python API 文档](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/dev/python/overview/)


开发文档：

- [pyflink官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.17/api/python/)

部署文档：

- [命令行部署](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/cli/)
## 安装pyflink

- 环境准备：
   - Java环境：Java8+
   - python环境： python3.7+， conda(miniconda)或其他,
- 基础环境安装：
   - java环境安装步骤略
   - conda环境安装步骤略
- pyflink安装
```bash
# 创建虚拟环境
conda create -n pyflink-py38env python=3.8 -y

conda activate pyflink-py38env

pip install apache-flink==1.17.1  -i https://pypi.tuna.tsinghua.edu.cn/simple
```

- 环境验证
```bash
conda activate pyflink-py38env

cd -/pyflink-py38env/lib/python3.8/site-packages/pyflink/ 

python examples/table/word_count.py 
-----------<log>----------------
(pyflink-py38env) [root@node80 pyflink]# python examples/table/word_count.py 
Using Any for unsupported type: typing.Sequence[~T]
No module named google.cloud.bigquery_storage_v1. As a result, the ReadFromBigQuery transform *CANNOT* be used with `method=DIRECT_READ`.
Executing word_count example with default input data set.
Use --input to specify file input.
Printing result to stdout. Use --output to specify output path.
+I[To, 1]
+I[be,, 1]
+I[or, 1]
+I[not, 1]
+I[to, 1]
+I[be,--that, 1]
+I[is, 1]
+I[the, 1]
+I[question:--, 1]
+I[Whether, 1]
...

# pyflink安装成功
```

- **重要提示：**
   - 如果需要将pyflink程序提交到远程的YARN集群，需要从集群拷贝如下配置文件，并到到该虚拟环境的pyflink/conf目录下
      - core-site.xml
      - hdfs-site.xml
      - flink-conf.yaml

## 开发pyflink程序
### Kafka源的使用

- [官方使用说明](https://nightlies.apache.org/flink/flink-docs-release-1.17/zh/docs/connectors/datastream/kafka/)
### 第三方依赖的使用
在使用pyflink进行程序开发时，用户在开发UDF算子时常常会引入第三方依赖库,此时对依赖包引入和运行的管理就会成为刚需；当在本地执行pyflink时，用户可以将第三方python库下载安装到本地后在进行执行，但当pyflink程序需要提交到远程执行时，此方法就行不通。于是pyflink官方提供了对依赖包的管理方法，其中在DataStream API 和 Table API实现方式是不同的。

- 目前官方提供了对Jar依赖和Python依赖管理的说明和示例，详见[官方链接](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/dev/python/dependency_management/)，
- **需要注意是，在pyflink代码中添加依赖时，目前只支持本地文件的引入（即：file:///)，不支持远程（如 hdfs:///）方式的引入**。

## 提交pyflink程序（部署）
### 提交模式概述
当通过flink run 提交pyflink程序时，flink将执行python命令，解析编译pyflink程序，生成一个jar程序，此过程也被称为JobGraph对象生成，其本质是python vm 与java vm 通过RPC的方式进行通信，当然，不同的提交模式，其实现逻辑会不同；目前，提交模式分为以下几种：

- Session模式
- Standalone模式
- Application模式（此模式目前支持YARN 或 K8S，也是官方推荐的模式）
- Per-job 模式（在1.17版本中已被弃用）
### 提交参数说明

- [官方参数使用说明](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/cli/)

- |               选项**                | **描述**                                                     |
  | :---------------------------------: | ------------------------------------------------------------ |
  |            -py、--python            | 带有程序入口点的 Python 脚本。可以用 --pyFiles 选项配置依赖资源。 |
  |          -pym、--pyModule           | 带有程序入口点的 Python 模块。该选项必须与 --pyFiles 一起使用。 |
  |          -pyfs、--pyFiles           | 为作业附加自定义文件。支持 .py/.egg/.zip/.whl 之类的标准资源文件后缀或目录。这些文件将被添加到本地客户端和远程 Python UDF Worker 的 PYTHONPATH 中。以 .zip 为后缀的文件将被解压缩，并且被添加到 PYTHONPATH。逗号（“,”）可以用作指定多个文件的分隔符（比如，--pyFiles file:///tmp/myresource.zip,hdfs:///$namenode_address/myresource2.zip）。 |
  |        -pyarch、--pyArchives        | 为作业添加 Python 归档文件。归档文件将被解压缩到 Python UDF Worker 的工作目录。可以为每个归档文件指定目标目录。如果指定目标目录名称，那么归档文件将被解压缩到具有指定名称的目录。否则，归档文件将被解压缩到与归档文件同名的目录中。通过该选项上传的文件可以通过相对路径访问。可以使用 “#” 作为归档文件路径和目标目录名称的分隔符。可以使用 “,” 作为指定多个归档文件的分隔符。可以使用该选项上传 Python UDF 中使用的虚拟环境和数据文件（比如，--pyArchives file:///tmp/py37.zip,file:///tmp/data.zip#data --pyExecutable py37.zip/py37/bin/python）。在 Python UDF 中可以访问数据文件，比如：f = open('data/data.txt', 'r')。 |
  | -pyclientexec、--pyClientExecutable | 当通过 “flink run” 提交 Python 作业或编译包含 Python UDF 的 Java/Scala 作业时，用于发起 Python 进程的 Python 解释器的路径（比如，--pyArchives file:///tmp/py37.zip --pyClientExecutable py37.zip/py37/python）。 |
  |       -pyexec、--pyExecutable       | 指定用于执行 Python UDF Worker 的 Python 解释器的路径（比如，--pyExecutable /usr/local/bin/python3）。Python UDF Worker 依赖 Python 3.7+、Apache Beam（版本 == 2.43.0）、Pip（版本 >= 20.3）和 SetupTools（版本 >= 37.0.0）。请确保指定的环境满足上述要求。 |
  |      -pyreq、--pyRequirements       | 指定定义第三方依赖的 requirements.txt 文件。这些依赖将被安装，并且被添加到 Python UDF Worker 的 PYTHONPATH。可选地指定包含这些依赖的安装包的目录。如果可选参数存在，那么使用“#”作为分隔符（--pyRequirements file:///tmp/requirements.txt#file:///tmp/cached_dir）。 |

- Note: 通过 -pyarch 指定的归档文件将通过 Blob 服务被分发到 TaskManager，文件大小限制是 2GB。如果归档文件的大小超过 2GB，那么可以将它上传到分布式文件系统，然后在命令行选项 -pyarch 中使用路径

- 打包pyflink运行环境
```bash
# 找到 minconda(安装路径 envs目录下) 或者对应虚拟环境安装目录
# 打包 pyflink-py38env 虚拟环境
cd -/conda/envs/pyflink-py38env
zip -r pyflink-py38env.zip pyflink-py38env  #
```

- Note： 需要注意打包后的zip是单层（pyflink-py38env.zip/bin/python）的还是双层的(pyflink-py38env.zip/pyflink-py38env/bin/python)
### 本地运行

- 无依赖的运行
```bash
./bin/flink run --python examples/python/table/word_count.py   # pyflink/bin/flink
```

- 引入外部资源时运行
```bash
./bin/flink run \
      --python examples/python/table/word_count.py \
      --pyFiles file:///user.txt,hdfs:///$namenode_address/username.txt
```

- 运行引用 Java UDF 或外部连接器的 PyFlink 作业。在 --jarfile 指定的文件将被上传到集群。
```bash
./bin/flink run \
      --python examples/python/table/word_count.py \
      --jarfile <jarFile>
```
### 提交到JobManager

- Tips：
   - 如果提交的pyflink程序及其依赖在本地：路径参数采用：file:///
   - 如果提交的pyflink程序在文件系统（如hdfs）,路径参数采用：hdfs:///
- 单文件提交
```bash
./flink run \
--jobmanager localhost:8081 \
-pyarch file:///path/pyflink-py38env.zip \
-pyexec pyflink-py38env.zip/bin/python3 \
-pyclientexec pyflink-py38env.zip/bin/python3 \
-py /pyflink-py38env/lib/python3.8/site-packages/pyflink/examples/table/word_count.py
```

- 以工程（文件夹）的方式提交
```bash
./flink run \
--jobmanager localhost:8081 \
-pyarch file:///path/pyflink-py38env.zip \
-pyexec pyflink-py38env.zip/bin/python3 \
-pyfs /pyflink-py38env/lib/python3.8/site-packages/pyflink/examples/table \
-pym word_count  # 入口程序
```
### 提交到YARN（Flink On YARN）
> 此处使用的run-application模式，其他模式类似

- 提交程序在本地
```bash
./bin/flink run-application \
-t yarn-application \
-Dyarn.ship-files=/data/bluewhale/modelhub/model-pyflink-stream \
-Dyarn.application.name=pyflink-battery-calculate01 \
-pyarch model-pyflink-stream/pyflink-py38env.zip \
-pyclientexec pyflink-py38env/bin/python \
-pyexec pyflink-py38env/bin/python \
-pyfs model-pyflink-stream \
-pym evcrrc_calculate_code_by_kafka \
--jarfile /data/workspace/flink-1.17.1/lib/flink-sql-connector-kafka-1.17.1.jar
```

- 提交程序在hdfs
   - 在提交程序前，先将程序和相关依赖上传到hdfs
```bash
./bin/flink run-application 
-t yarn-application \
-pyarch hdfs:///data/mmodelhub/model-pyflink-stream/pyflink-py38env.zip \
-Dyarn.application.name=pyflink-stream-model-1 \
-pyclientexec pyflink-py38env.zip/bin/python \
-pyexec pyflink-py38env.zip/bin/python \
-py hdfs:///data/mmodelhub/model-pyflink-stream/pyflink-stream-kafka.py \
--jarfile /data/workspace/flink-1.17.1/lib/flink-sql-connector-kafka-1.17.1.jar
```

提交的是pyflink-datastream模型，需要注意一下参数

- Dyarn.ship-files： 该参数需要指定了pyflink流模型的绝对路径 ，路径指到模型文件夹  (本地提交时才会指定)
- -pyfs和-pym需要一起使用
## 实践总结

- 使用Application模式提交pyflink程序时，建议优先将pyflink程序使用到的依赖包，执行环境提前上传到HDFS文件系统（或其他文件系统）上，再在Shell 命令中以hdfs:///的方式指定需要用到的资源路径，以提高作业提交的效率。
- 在开发pyflink程序时，若需要引入第三方Jar，建议在提交任务时通过--jarfile 指定Jar包，不建议在代码中通过add_jar方式，因为此方式目前仅支持指定本地Jar路径，不支持指定hdfs路径。
- 在Flink On Yarn高可用架构下，目前遇到提交多个任务，在集群中只有一个任务正常执行，其他任务提交成功但未执行的问题，目前解决方案是：在flink-conf.yaml配置中，将cluster.id配置内容注释掉，此配置主要用于zookeeper管理多个flink集群，至于高可用部署场景该配置是否为必配置项，暂未深入分析；[参考](https://blog.csdn.net/weixin_40308031/article/details/123207933)
## 拓展：
### pyflink相关开源项目：

- [Provide docker environment and examples for PyFlink](https://github.com/pyflink/playgrounds/tree/master)
- [快速入门pyflink](https://github.com/uncleguanghui/pyflink_learn/tree/master)
- [Gathers Python deployment, infrastructure and practices.](https://github.com/huseinzol05/Gather-Deployment/tree/master)

参考博文：

- [https://devpress.csdn.net/big-data/647a9c5d762a09416a07f59b.html](https://devpress.csdn.net/big-data/647a9c5d762a09416a07f59b.html)
- [https://developer.aliyun.com/article/743088](https://developer.aliyun.com/article/743088)
- [https://www.modb.pro/db/128620](https://www.modb.pro/db/128620)：PyFlink核心技术揭秘
