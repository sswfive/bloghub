---
title: 机器学习的常见术语
date: 2023-08-16 22:14:32
tags:
  - 机器学习
---

## 数据

- 一行数据称为一个样本，一列数据称为一个特征
- 有些数据有目标值（标签值），有些数据没有目标值

## 数据集

类别划分：
- 根据的输入的数据的特点进行划分：
  - 特征值 + 目标值（目标值是连续的和离散的）
  - 只有特征值， 没有目标值
- 根据输出的数据进行划分：
  - 连续的值（回归）、有限个离散值（分类）

数据集分割
- 训练集：用户训练、构建模型。（划分比例：70%，80%，75%）
- 测试集：在模型检验时使用，用于评估模型是否有效。（划分比例：30%，20%，25%）

sklearn API:

- 数据集划分

```python
sklearn.model_selecction.train_test_split
# x: 数据集的特征值
# y: 数据集的标签值
# train_size: 测试集的大小，一般为float
# random_state: 随机种子，不同的种子会造成不同的随机采样结果，相同的种子采样结果相同；
# return:训练集的特征值、测试集的特征值、训练标签、
```

- 数据集API

```python
sklearn.datasets("加载获取流行数据集")
datasets.load_*()
# 获取小规模数据集，数据包含在datasets里；

# 分类数据集：
load_iris():  # 加载并返回鸢尾花数据集
load_digts()：# 加载并返回

# 回归数据集：
load_boston():     # 加载并返回波士顿房价数据集
load_diablesets(): # 加载并返回糖尿病数据集
datasets.fetch_*(data_home=None)  # 获取大规模数据集，需要从网络上下载，函数的第一个参数是data_home,表示数据集下载目录，默认是~/scikit_learn_data/
fetch_20newsgroups(data_home=None, subset='train'):  # 用于分类的大数据集
    
# 通过sklearn.datasets获取数据集的返回的类型
# load*和fetch*返回的数据类型datasets.base.Bunch(字典格式)
data:   # 特征数据数组，是[n_samples*n_features]的二维numpy.ndarray数组
target:   # 标签数组，是n_samples的一维numpy.ndarray数组
DESCR:   # 数据描述
feature_names：   # 特征名，新闻数  # 据，手写数字，（没有回归数据集）
target_names:   # 标签名
```

## 监督学习
> 有数据、有标签的是监督学习

**定义：**

- （supervised learning）,可以由输入数据中学到或建立一个模型，并以此模式推断新的结果；

**特点：**

- 输入数据是由输入特征值和目标值所组成；

- 有特征值 、有目标值
   - 目标值连续---> 回归
   - 目标值离散--->分类
- 函数的输出可以是一个连续的值（称为回归）
- 或是输出是有限个离散值（称为为分类）

**类别：**

- 分类（目标值离散型）
  - 分类是监督学习的一个核心问题，在监督学习中，当输出变量取有限个离散值时，预测问题变成分类问题，最基础的便是二分类问题，及判断是非，从两个类别中选择一个作为预测结果

- - 常见算法：k-近邻算法、贝叶斯分类、决策树、随机森林、逻辑回归、神经网络

- 回归（目标值连续型）

- - 回归是监督学习的另一个重要问题，回归用于预测输入变量和输出变量之间的关系，输出是连续型的值；
  - 常见算法：线性回归、岭回归

- 标注：隐马尔可夫模型

**应用场景：**

- 分类：

- - 在银行业务中，构建一个客户分类模型，按客户贷款风险的大小进行分类
  - 图像处理中，分类可以用来监测图像是否有人脸出现，动物类别等；
  - 手写识别中，分类可以用于识别手写的数字
  - 文本分类

- 回归：

- - 房价预测，根据某地历史房价数据，进行一个预测；
  - 金融信息，每日股票值；

## 无监督学习
> 只有数据没有标签的是无监督学习（也称：非监督学习）
定义：

- （unSupervised learning）,可以由输入数中学到或建立一个模型，并以此模式推断新的结果。

特点：

- 输入数据是由输入特征值组成输入数据没有被标记，也没有确定的结果样本数据类别位未知，需要根据样本件的相对性对样本集进行分类（聚类），试图使类内差距最小化，类间差距最大化

- 常见算法：
  - 聚类（k-means）

## 半监督学习
> 结合了监督学习和非监督学习的是半监督学习

- 有特征值，但是一部分数据有目标值，一部分没有

## 强化学习
> 从经验中总结提升，或称从环境中自学习的是强化学习

- 动态过程（上一步输出是下一步输入）
- 四要素：agent、action、environment、reward



## 转换器（Transformer）

>  实现了特征工程的API

- fit_transform(): 输入数据直接转换
- fit(): 输入数据、但不做事情（），计算平均值，方差等

- - +transform(): 进行数据的转换

## 估计器（Estimator）

> 实现了一类算法的API

- 常用估计器

- - 分类估计器

- - - sklearn.neighbors: k-近邻算法
    - sklearn.naive_bayes: 贝叶斯
    - sklearn.linear_model.LogisticRegression:  逻辑回归
    - sklearn.tree: 决策树与随机森林

- - 回归估计器

- - - sklearn.linear_model.LinearRegression：线性回归
    - sklearn.linear_model.Ridge: 岭回归

- - 聚类估计器

- 估计器的使用流程

  ![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230810/image.3a93vtf4gha0.webp)

1. 输入训练数据集，调用fit(x_train, y_train)
2. 输入测试数据集、调用估计器（estimator）

1. 1. y_predict = predict(x_test)
   2. 预测的准确率：score(x_test, y_test)
