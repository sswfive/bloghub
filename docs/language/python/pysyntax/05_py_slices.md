---
title: python切片与推导式
date: 2019-05-03 12:29:56
tags: 
  - Python
---
# python切片与推导式

## 切片操作

- 概念：
  - 通过索引区间访问线性结构的一段数据
  - sequence[start:stop:step] 表示返回[start,stop)区间的子序列，step表示步长，默认为1 
- 常用操作
  - 支持负索引（start为0可以省略，stop为0可以省略）
  - 超过上界，就取到末尾，超过下界，取到开头
  - [:]表示从头至尾，全部元素被取出，等效于copy()方法
  
- 支持切片的数据结构
  - 字符串
  - 列表
  - 元组

---
## 推导式操作

- 概念：
  - 推导式别名为生成式
  - 用一个表达式创建和控制一个有规律的容器。
  - 支持推导式的容器类型有：列表推导式、字典推导式、集合推导式

- 目的：
  - 简化代码

- 分类：
  - 列表推导式
  - 字典推导式
  - 集合推导式

### 列表推导式

- 场景应用：
  - 用于快速创建列表

- 创建列表的方式
  - while循环实现
  - for 循环实现
  - 列表推导式实现

```python
# 语法
[xx for xx in range()]

# 常规实现
list1 = [i for i in range(10)]

# range()步长
list1 = [i for i in range(0, 10, 2)]

# 带if的列表推导式
list2 = [i for i in range(10) if i % 2 == 0]

# 多个for循环实现列表推导式
list3 = [(i, j) for i in range(1, 3) for j in range(3)]
```

---

### 字典推导式

- 场景应用：
  - 创建字典
  - 快速合并两个字典
  - 快速提取字典中目标数据

```python
# 语法：
dict0 = {xx1: xx2 for ... in ...}

dict1 = {i: i**2 for i in range(1, 5)}

list1 = ['name', 'age', 'gender', 'id']
list2 = ['Tom', 20, 'man']
dict2 = {list1[i]: list2[i] for i in range(len(list2))}
# 备注：如果两个列表数据个数不同，len统计数据多的列表数据个数会报错；len统计数据少的列表数据个数不会报错

counts = {'MBP': 268, 'HP': 125, 'DELL': 201, 'Lenovo': 199, 'acer': 99}
dict3 = {key: value for key, value in counts.items() if value >= 200}
```

### 集合推导式

- 场景应用：
  - 创建集合

```python
# 语法
{xx for xx in ...}

tmplist = [1, 1, 2]
set1 = {i ** 2 for i in tmplist}
```