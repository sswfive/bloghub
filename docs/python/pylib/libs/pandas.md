---
title: Pandas -> 数据分析工具
tags:
  - Python
---

## 写在前面
> - 官网：[http://pandas.pydata.org](http://pandas.pydata.org)
> - github地址：[https://github.com/pandas-dev/pandas](https://github.com/pandas-dev/pandas)
> - 实现语言：Python
> - 当前版本：version 1.5.1
> - 当前star数：35.7k

## 什么是Pandas？
> 官网介绍:
> pandas is open source, BSD-licensed library providing high-performance, easy-to-use data structures and data analysis tools for the Python programming language.

### 名称由来

- 来自panel data(面板数据) 和 data analysis(数据分析)的合成
### 概述

- 是一个强大的分析结构化数据的工具集，基于Numpy构建，提供了高级数据结构和数据操作工具
- 是Python成为强大而高效的数据分析环境的重要因素之一
### 功能特性

- 一个强大的分析和操作大型结构化数据集所需要的工具集
- 基于Numpy，提供了高性能矩阵运算
- 提供了大量能够快速便捷地处理数据的函数和方法
### 目标问题域

- Numpy通常用于处理数值，Pandas既能处理数值，还能处理其他类型的数据，如字符串、时间序列等
### 应用场景

- 应用于数据挖掘、数据分析场景
- 提供数据清洗功能
## Pandas的安装

- 安装
```bash
pip install pandas -i https://pypi.tuna.tsinghua.edu.cn/simple
或
conda install pandas
```

- 使用
```python
import pandas as pd
```
## Pandas的常用结构
> 1. Series: 一维，带标签数组
> 2. DataFrame: 二维， Series容器

### Series | 介绍
概述

- 概念：
   - 是一种类似于一维数组的对象，由一组数据（各种Numpy数据类型）以及一组与之对应的索引（数据标签）组成
- 概念理解：
   - Series对象本质是由两个数组组成，一个数组构成对象的建（index，索引）,一个数组构成对象的值（values）, 键->值
- 图例：
   ![series](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20221111/series.7gq089dwpzk0.webp)
- 类比：
   - numpy中的ndarray的很多方法都可以用于与series类型，比如argmax，clip，
   - series具有where方法，但是结果和ndarray不同
### DataFrame | 介绍
概述

- 概念：
   - 是一个表格型数据结构，它有一组有序的列，每列可以是不同类型的值，DataFrame既有行索引又有列索引，通常被看做由Series组成的字典（同一个索引），数据是以二维结构存放的。
- 概念理解：
   - 类似多维数组/表格数据（excel）
   - 每列数据都可以是不同类型的
   - 索引包括行索引、列索引
- 图例：
   ![dataframe](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20221111/dataframe.ch3zj2xnjqw.webp)

---

## Pandas的基本用法
### Series的基本用法
#### 创建
```python
import pandas as pd

# 方式一：通过list创建Series
ser1 = pd.Series(range(10, 20))
# ---------------方式一--------------------------
In [1]: import pandas as pd
In [2]: ser1 = pd.Series(range(10, 20))
In [3]: ser1
Out[3]:
0    10
1    11
...  ...
8    18
9    19
dtype: int64

In [4]: print(type(ser1))
<class 'pandas.core.series.Series'>

# 方式二：通过dict创建Series
year_data = {2021: 18.8, 2022: 28.8, 2023: 8.8}
ser2 = pd.Series(year_data)
# ---------------方式二--------------------------
In [2]: year_data = {2021: 18.8, 2022: 28.8, 2023: 8.8}
   ...: ser2 = pd.Series(year_data)

In [3]: ser2
Out[3]:
2021    18.8
2022    28.8
2023     8.8
dtype: float64
```
#### 索引

- 索引：
   - Series对象本质上有两个数组构成，一个数组构成对象的键（index, 索引），一个数据构成对象的值（values）
   - 索引操作时， 一个的时候直接传入序号或index, 多个的时候传入序号或index的列表
   - 索引对象不可变，保证了数据的安全性
- 常见的Index种类
   - Index：索引
   - Int64Index： 整数索引
   - MutiIndex: 层级索引
   - DatatimeIndex: 时间戳索引
- 常见索引操作：
```python
ser[‘label’], ser[pos]  # 行索引

ser[2:4], ser[‘label1’: ’label3’]  # 切片索引，（按索引名切片操作时，是包含终止索引的）

ser[[‘label1’, ’label2’, ‘label3’]]  # 不连续索引

ser_bool = ser_bool > 2  # 布尔索引
ser_obj[ser_bool]
ser_obj[ser_obj > 2]
```

- 实操：
```python
In [15]: import numpy as np
In [16]: ser3 = pd.Series(np.arange(5), index=list("ABCDE"))

In [17]: ser3
Out[17]:
A    0
B    1
C    2
D    3
E    4
dtype: int64

In [18]: ser3[1:4:2]  # 切片，步长为2 
Out[18]:
B    1
D    3
dtype: int64

In [19]: ser3[2]  # 取索引
Out[19]: 2

In [20]: ser3[1:4]  # 切片，步长为默认值1
Out[20]:
B    1
C    2
D    3
dtype: int64

In [21]: ser3[ser3>3]  # 值大于3的所有数据
Out[21]:
E    4
dtype: int64

In [22]: ser3["B"]  # 索引为B的数据
Out[22]: 1

In [24]: ser3[["A", "E"]]  # 索引为A、B的数据
Out[24]:
A    0
E    4
dtype: int64

In [25]: ser3[[0,1,2]]  # 取索引为0、1、2的数据（不连续索引）
Out[25]:
A    0
B    1
C    2
dtype: int64
```
##### 索引和值
```python
In [26]: ser2
Out[26]:
ABC
2021    18.8
2022    28.8
2023     8.8
Name: AAA, dtype: float64

In [27]: ser2.index  # 取索引值
Out[27]: Int64Index([2021, 2022, 2023], dtype='int64', name='ABC')

In [28]: ser2.values  # 取值
Out[28]: array([18.8, 28.8,  8.8])

In [29]: type(ser2.index)  
Out[29]: pandas.core.indexes.numeric.Int64Index

In [30]: type(ser2.values)
Out[30]: numpy.ndarray

In [31]: ser2.name = "Bobo"  # 设置Series对象名，没有就新增，有就修改
 
In [32]: ser2.index.name = "YEAR"  # 设置Series对象索引名，没有就新增，有就修改

In [33]: ser2
Out[33]:
YEAR
2021    18.8
2022    28.8
2023     8.8
Name: Bobo, dtype: float64
```
##### loc标签索引
```python
n [8]: ser4 = pd.Series(np.arange(5), index=list("abcde"))

In [9]: ser4['a':'e']
Out[9]:
a    0
b    1
c    2
d    3
e    4
dtype: int64

In [10]: ser4.loc['a':'e']
Out[10]:
a    0
b    1
c    2
d    3
e    4
dtype: int64
```
##### iloc位置索引

- 基于索引编号索引
```python
In [15]: ser5 = pd.Series(np.arange(5), index=list("abcde"))

In [16]: ser5
Out[16]:
a    0
b    1
c    2
d    3
e    4
dtype: int64

In [17]: ser5[1:4]
Out[17]:
b    1
c    2
d    3
dtype: int64

In [18]: ser5.iloc[1:4]
Out[18]:
b    1
c    2
d    3
dtype: int64
```
##### ix标签与位置混合索引(新版本中移除此属性)

- ix是以上二者的综合，既可以使用索引编号，又可以使用自定义索引，要视情况不同来使用，
- 如果索引既有数字又有英文，那么这种方式是不建议使用的，容易导致定位的混乱。
```python
In [28]: ser5.ix['a':'c']
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
Input In [28], in <cell line: 1>()
----> 1 ser5.ix['a':'c']

File /usr/local/Caskroom/miniforge/base/lib/python3.9/site-packages/pandas/core/generic.py:5907, in NDFrame.__getattr__(self, name)
   5900 if (
   5901     name not in self._internal_names_set
   5902     and name not in self._metadata
   5903     and name not in self._accessors
   5904     and self._info_axis._can_hold_identifiers_and_holds_name(name)
   5905 ):
   5906     return self[name]
-> 5907 return object.__getattribute__(self, name)

AttributeError: 'Series' object has no attribute 'ix'
```
#### 对齐运算
> 是数据清洗的重要过程，可以按索引对齐进行运算，如果没有对齐位置则补NaN

- Series 按行、索引对齐
```python
In [3]: s1 = pd.Series(range(10,20), index=range(10))

In [5]: s1
Out[5]:
0    10
1    11
2    12
3    13
4    14
5    15
6    16
7    17
8    18
9    19
dtype: int64

In [6]: s2 = pd.Series(range(20,25), index=range(5))

In [7]: s2
Out[7]:
0    20
1    21
2    22
3    23
4    24
dtype: int64

In [8]: s1 + s2
Out[8]:
0    30.0
1    32.0
2    34.0
3    36.0
4    38.0
5     NaN
6     NaN
7     NaN
8     NaN
9     NaN
dtype: float64
```

- 填充未对齐的数据进行运算
   - fill_value
      - 使用add、sub、div、mul 的同时，通过fill_value指定填充值，未对齐的数据将和填充值做运算
```python
In [27]: s1.add(s2, fill_value=5)
Out[27]:
0    30.0
1    32.0
2    34.0
3    36.0
4    38.0
5    20.0
6    21.0
7    22.0
8    23.0
9    24.0
dtype: float64

In [28]: s1.add(s2, fill_value=-2)
Out[28]:
0    30.0
1    32.0
2    34.0
3    36.0
4    38.0
5    13.0
6    14.0
7    15.0
8    16.0
9    17.0
dtype: float64
```
#### tolist()方法
```python
In [35]: ser4 = pd.Series([1,2,4,1,4,3,2])

In [36]: ser4.tolist()
Out[36]: [1, 2, 4, 1, 4, 3, 2]

In [37]: ser4.to_list()
Out[37]: [1, 2, 4, 1, 4, 3, 2]

```
#### unique()方法
```python
In [38]: ser4.unique()
Out[38]: array([1, 2, 4, 3])
```
#### 其他方法
```python
n [39]: ser4.unique()
  abs()                   convert_dtypes()        first()                 last()
  add()                   copy()                  first_valid_index()     last_valid_index()
  add_prefix()            corr()                  flags                   le()
  add_suffix()            count()                 floordiv()              loc
  agg()                   cov()                   ge()                    lt()
  aggregate()             cummax()                get()                   mad()
  align()                 cummin()                groupby()               map()
  all()                   cumprod()               gt()                    mask()
  any()                   cumsum()                hasnans                 max()
  append()                describe()              head()                  mean()
  apply()                 diff()                  hist()                  median()
  argmax()                div()                   iat                     memory_usage()
  argmin()                divide()                idxmax()                min()
  argsort()               divmod()                idxmin()                mod()
  array                   dot()                   iloc                    mode()
  asfreq()                drop()                  index                   mul()
  asof()                  drop_duplicates()       infer_objects()         multiply()              >
  astype()                droplevel()             info()                  name
  at                      dropna()                interpolate()           nbytes
  at_time()               dtype                   is_monotonic            ndim
  attrs                   dtypes                  is_monotonic_decreasing ne()
  autocorr()              duplicated()            is_monotonic_increasing nlargest()
  axes                    empty                   is_unique               notna()
  backfill()              eq()                    isin()                  notnull()
  between()               equals()                isna()                  nsmallest()
  between_time()          ewm()                   isnull()                nunique()
  bfill()                 expanding()             item()                  pad()
  bool()                  explode()               items()                 pct_change()
  clip()                  factorize()             iteritems()             pipe()
  combine()               ffill()                 keys()                  plot
  combine_first()         fillna()                kurt()                  pop()
  compare()               filter()                kurtosis()              pow()
```
### DataFrame的基本用法
```python
import pandas as pd
import numpy as np
```
#### 创建
```python
# 方式一：通过ndarray创建
In [3]: array = np.random.randn(5, 4)

In [4]: array
Out[4]:
array([[ 0.92360777, -0.40278525, -0.66873708,  1.80493655],
       [-2.35582906,  0.18222298, -0.54358948,  0.10477355],
       [-0.01014204, -0.29127566,  0.53816636, -0.39215923],
       [ 2.02733253, -0.79347232, -0.39637719,  0.98461896],
       [ 0.61317253, -1.95664987,  0.99766931, -0.43095178]])

In [5]: df1 = pd.DataFrame(array)

In [6]: df1
Out[6]:
          0         1         2         3
0  0.923608 -0.402785 -0.668737  1.804937
1 -2.355829  0.182223 -0.543589  0.104774
2 -0.010142 -0.291276  0.538166 -0.392159
3  2.027333 -0.793472 -0.396377  0.984619
4  0.613173 -1.956650  0.997669 -0.430952


# 方式二：通过dict创建
In [7]: dict_data = {'A': 1,
   ...:              'B': pd.Timestamp('20221109'),
   ...:              'C': pd.Series(1, index=list(range(4)),dtype='float32'),
   ...:              'D': np.array([3] * 4,dtype='int32'),
   ...:              'E': ["Python","Go","C++","C"],
   ...:              'F': 'sswang' }

In [8]: dict_data
Out[8]:
{'A': 1,
 'B': Timestamp('2022-11-09 00:00:00'),
 'C': 0    1.0
 1    1.0
 2    1.0
 3    1.0
 dtype: float32,
 'D': array([3, 3, 3, 3], dtype=int32),
 'E': ['Python', 'Go', 'C++', 'C'],
 'F': 'sswang'}

In [9]: df2 = pd.DataFrame(dict_data)

In [10]: df2
Out[10]:
   A          B    C  D       E       F
0  1 2022-11-09  1.0  3  Python  sswang
1  1 2022-11-09  1.0  3      Go  sswang
2  1 2022-11-09  1.0  3     C++  sswang
3  1 2022-11-09  1.0  3       C  sswang

```
#### 通过索引获取列数据
> Series类型

- df[col_idx] 或 df.col_idx
```python
In [10]: df2
Out[10]:
   A          B    C  D       E       F
0  1 2022-11-09  1.0  3  Python  sswang
1  1 2022-11-09  1.0  3      Go  sswang
2  1 2022-11-09  1.0  3     C++  sswang
3  1 2022-11-09  1.0  3       C  sswang

In [11]: df2['A']
Out[11]:
0    1
1    1
2    1
3    1
Name: A, dtype: int64

In [12]: df2['E']
Out[12]:
0    Python
1        Go
2       C++
3         C
Name: E, dtype: object

In [13]: df2.B
Out[13]:
0   2022-11-09
1   2022-11-09
2   2022-11-09
3   2022-11-09
Name: B, dtype: datetime64[ns]

```
#### 增加列数据

- df[new_col_idx] = data
   - 类似于dict的增加key-value
```python
In [14]: df2['H'] = df2['D'] + 5

In [15]: df2
Out[15]:
   A          B    C  D       E       F  H
0  1 2022-11-09  1.0  3  Python  sswang  8
1  1 2022-11-09  1.0  3      Go  sswang  8
2  1 2022-11-09  1.0  3     C++  sswang  8
3  1 2022-11-09  1.0  3       C  sswang  8
```
#### 删除列

- del df[col_idx]
```python
In [15]: df2
Out[15]:
   A          B    C  D       E       F  H
0  1 2022-11-09  1.0  3  Python  sswang  8
1  1 2022-11-09  1.0  3      Go  sswang  8
2  1 2022-11-09  1.0  3     C++  sswang  8
3  1 2022-11-09  1.0  3       C  sswang  8

In [16]: del(df2['H'])

In [17]: df2
Out[17]:
   A          B    C  D       E       F
0  1 2022-11-09  1.0  3  Python  sswang
1  1 2022-11-09  1.0  3      Go  sswang
2  1 2022-11-09  1.0  3     C++  sswang
3  1 2022-11-09  1.0  3       C  sswang
```
#### 索引操作
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2130981/1667948262825-31ed0411-f151-4044-bbf1-bd582830ca5c.png#averageHue=%23e5c7b7&clientId=u3e6955e9-d4c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=223&id=ufe059e5e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=892&originWidth=1790&originalType=binary&ratio=1&rotation=0&showTitle=false&size=711086&status=done&style=stroke&taskId=u4f8b6b1b-fc86-4cd1-8538-ca588a145f3&title=&width=448)

- columns指定列索引名
```python
In [19]: pd.DataFrame(np.random.randn(5,4), columns=['A','B','C','D'])
Out[19]:
          A         B         C         D
0  0.901589 -0.973230  0.257040 -0.518793
1  0.247386  1.722486  0.227000 -0.233353
2  1.219715 -1.395651  0.899632  1.013356
3 -0.083246  1.924391 -0.251804 -0.995503
4  0.782573 -1.076885 -0.014058 -0.197052
```

- 列索引
   - df[['label']]
```python
In [24]: df3
Out[24]:
          A         B         C         D
0  1.123462  1.105408  0.039270 -0.424612
1 -1.451999  0.002271 -0.449790 -0.840314
2 -1.323392 -1.312659 -0.477671 -0.475040
3  0.852456 -0.725810  0.343415 -0.888764
4 -0.634588  0.214208 -0.363505 -0.890744

In [25]: df3['A']
Out[25]:
0    1.123462
1   -1.451999
2   -1.323392
3    0.852456
4   -0.634588
Name: A, dtype: float64

In [28]: df3[['A']]
Out[28]:
          A
0  1.123462
1 -1.451999
2 -1.323392
3  0.852456
4 -0.634588
```

- 不连续索引
   - df[['label1','label2']]
```python
In [32]: df3 = pd.DataFrame(np.random.randn(5,4), columns=['A','B','C','D'])

In [33]: df3
Out[33]:
          A         B         C         D
0  0.467779 -0.447015 -0.610554  0.718384
1  0.680987  1.542523  0.828433  1.221965
2  0.154291 -0.015539  0.234407 -1.376114
3  2.084046  0.200158 -0.136955  1.275513
4  0.779262 -0.220597 -2.309179  0.470001

In [34]: df3[['A','C']]
Out[34]:
          A         C
0  0.467779 -0.610554
1  0.680987  0.828433
2  0.154291  0.234407
3  2.084046 -0.136955
4  0.779262 -2.309179
```
#### 高级索引
##### loc标签索引

   - DataFrame不能直接切片，可以通过loc来做切片
   - loc是基于标签名索引
```python
In [11]: df3 = pd.DataFrame(np.random.randn(5,4), columns=['A','B','C','D'])

In [12]: df3
Out[12]:
          A         B         C         D
0 -0.705794  0.491632  0.460443 -0.217467
1 -2.118379  0.839388  2.345748 -0.287588
2 -2.052069 -0.389537 -0.531405 -0.580226
3  1.368354 -0.104560  0.416091  2.155097
4  1.499490 -0.189129  1.554696 -0.544609

In [13]: df3['A']
Out[13]:
0   -0.705794
1   -2.118379
2   -2.052069
3    1.368354
4    1.499490
Name: A, dtype: float64

In [14]: df3.loc[0:2, 'A']
Out[14]:
0   -0.705794
1   -2.118379
2   -2.052069
Name: A, dtype: float64
```
##### iloc位置索引

   - 基于索引编号来索引
```python
In [19]: df4 = pd.DataFrame(np.random.randn(5,4), columns=['A','B','C','D'])

In [20]: df4
Out[20]:
          A         B         C         D
0  0.663622  0.157452 -0.789668  0.024412
1 -0.468456  0.078273 -0.440615 -0.604088
2  1.269526  0.430453  0.524557 -1.735621
3 -0.242327  1.308116 -0.266340  0.795823
4  1.806010 -0.328351 -0.622317  0.736839

In [21]: df4.iloc[0:3, 0]
Out[21]:
0    0.663622
1   -0.468456
2    1.269526
Name: A, dtype: float64
```
##### ix标签与位置混合索引(新版本中移除此属性)

   - ix是以上二者的综合，既可以使用索引编号，又可以使用自定义索引，要视情况不同来使用，
   - 如果索引既有数字又有英文，那么这种方式是不建议使用的，容易导致定位的混乱。
```python
In [29]: df4
Out[29]:
          A         B         C         D
0  0.663622  0.157452 -0.789668  0.024412
1 -0.468456  0.078273 -0.440615 -0.604088
2  1.269526  0.430453  0.524557 -1.735621
3 -0.242327  1.308116 -0.266340  0.795823
4  1.806010 -0.328351 -0.622317  0.736839

In [30]: df4.ix[0:3, 'A']
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
Input In [30], in <cell line: 1>()
----> 1 df4.ix[0:3, 'A']

File /usr/local/Caskroom/miniforge/base/lib/python3.9/site-packages/pandas/core/generic.py:5907, in NDFrame.__getattr__(self, name)
   5900 if (
   5901     name not in self._internal_names_set
   5902     and name not in self._metadata
   5903     and name not in self._accessors
   5904     and self._info_axis._can_hold_identifiers_and_holds_name(name)
   5905 ):
   5906     return self[name]
-> 5907 return object.__getattribute__(self, name)

AttributeError: 'DataFrame' object has no attribute 'ix'
```
#### 对齐运算
> 是数据清洗的重要过程，可以按索引对齐进行运算，如果没有对齐位置则补NaN

- DataFrame按行、列索引对齐
```python
In [9]: df1 = pd.DataFrame(np.ones((2,2)), columns=['a', 'b'])

In [10]: df1
Out[10]:
     a    b
0  1.0  1.0
1  1.0  1.0

In [11]: df2 = pd.DataFrame(np.ones((3,3)), columns=['a', 'b','c'])

In [12]: df2
Out[12]:
     a    b    c
0  1.0  1.0  1.0
1  1.0  1.0  1.0
2  1.0  1.0  1.0

In [13]: df1 + df2
Out[13]:
     a    b   c
0  2.0  2.0 NaN
1  2.0  2.0 NaN
2  NaN  NaN NaN
```

- 填充未对齐的数据进行运算
   - fill_value
      - 使用add、sub、div、mul 的同时，通过fill_value指定填充值，未对齐的数据将和填充值做运算
```python
In [19]: df1.sub(df2, fill_value=5.)
Out[19]:
     a    b    c
0  0.0  0.0  4.0
1  0.0  0.0  4.0
2  4.0  4.0  4.0

In [21]: df1.sub(df2, fill_value=3)
Out[21]:
     a    b    c
0  0.0  0.0  2.0
1  0.0  0.0  2.0
2  2.0  2.0  2.0

In [22]: df1.add(df2, fill_value=5)
Out[22]:
     a    b    c
0  2.0  2.0  6.0
1  2.0  2.0  6.0
2  6.0  6.0  6.0
```

## Pandas的函数应用
### 使用numpy函数
```python
In [3]: df = pd.DataFrame(np.random.randn(5,4) -1)

In [4]: df
Out[4]:
          0         1         2         3
0  0.190219 -0.548120 -1.059652  0.480065
1 -1.514674 -1.404162  0.373566 -0.930732
2 -1.277141 -1.426694 -1.560795 -0.486730
3 -0.868642 -1.076728 -1.100897 -0.389249
4 -0.712614 -2.140464 -2.005097 -0.588321

In [5]: np.abs(df)
Out[5]:
          0         1         2         3
0  0.190219  0.548120  1.059652  0.480065
1  1.514674  1.404162  0.373566  0.930732
2  1.277141  1.426694  1.560795  0.486730
3  0.868642  1.076728  1.100897  0.389249
4  0.712614  2.140464  2.005097  0.588321
```
### apply()

- apply将函数应用到列或者行上
   - 默认axis=0, 列方向
```python
In [6]: f = lambda x: x.max()

In [7]: df.apply(f)
Out[7]:
0    0.190219
1   -0.548120
2    0.373566
3    0.480065
dtype: float64

In [8]: df.apply(f, axis=1)
Out[8]:
0    0.480065
1    0.373566
2   -0.486730
3   -0.389249
4   -0.588321
dtype: float64
```
### applymap()

- applymap将函数应用到每个数据上
```python
In [9]: f2 = lambda x: '%.2f' % x

In [10]: df.applymap(f2)
Out[10]:
       0      1      2      3
0   0.19  -0.55  -1.06   0.48
1  -1.51  -1.40   0.37  -0.93
2  -1.28  -1.43  -1.56  -0.49
3  -0.87  -1.08  -1.10  -0.39
4  -0.71  -2.14  -2.01  -0.59
```
### 排序
#### 索引排序

- sort_index()
   - axis=0,按行，axis=1按列
   - 排序默认使用升序排序，ascending=False 为降序排序
```python
In [11]: s1 = pd.Series(range(10,15), index=np.random.randint(5, size=5))

In [12]: s1
Out[12]:
3    10
4    11
2    12
4    13
1    14
dtype: int64

In [13]: s1.sort_index()
Out[13]:
1    14
2    12
3    10
4    11
4    13
dtype: int64
```

- DataFrame
```python
# DataFrame排序注意轴向
In [16]: df1 = pd.DataFrame(np.random.randn(3,5),index=np.random.randint(3, size=3),columns=np.random
    ...: .randint(5, size=5))

In [17]: df1
Out[17]:
          2         2         0         4         4
0  0.891979 -0.640110 -0.402756 -0.377881  0.080344
1  0.617798  0.574241 -0.026625 -0.547804  0.191055
1 -0.227055  0.225085 -0.064116 -0.649598 -0.209348

In [18]: df1_isort = df1.sort_index(axis=1, ascending=False)

In [19]: df1_isort
Out[19]:
          4         4         2         2         0
0 -0.377881  0.080344  0.891979 -0.640110 -0.402756
1 -0.547804  0.191055  0.617798  0.574241 -0.026625
1 -0.649598 -0.209348 -0.227055  0.225085 -0.064116
```
#### 按值排序

- sort_values(by="columns name")
   - 根据某个唯一的列名进行排序，如果出现重复的列名则报错
```python
In [21]: df1_vsort = df1.sort_values(by=0, ascending=False)

In [22]: df1_vsort
Out[22]:
          2         2         0         4         4
1  0.617798  0.574241 -0.026625 -0.547804  0.191055
1 -0.227055  0.225085 -0.064116 -0.649598 -0.209348
0  0.891979 -0.640110 -0.402756 -0.377881  0.080344

In [23]: df2_vsort = df1.sort_values(by=4, ascending=False)
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
Input In [23], in <cell line: 1>()
----> 1 df2_vsort = df1.sort_values(by=4, ascending=False)
ValueError: The column label '4' is not unique.
```
### 处理缺失数据
```python
In [25]: df3 = pd.DataFrame([np.random.randn(3), [1., 2., np.nan], [np.nan, 4., np.nan], [1.,2.,3.]])
    ...:

In [26]: df3
Out[26]:
          0         1         2
0  0.722605  0.537739  0.340062
1  1.000000  2.000000       NaN
2       NaN  4.000000       NaN
3  1.000000  2.000000  3.000000
```
#### 判断是否存在缺失值isnull()
```python
In [27]: df3.isnull()
Out[27]:
       0      1      2
0  False  False  False
1  False  False   True
2   True  False   True
3  False  False  False
```
#### 丢弃缺失值dropna()
```python
In [28]: df3.dropna()
Out[28]:
          0         1         2
0  0.722605  0.537739  0.340062
3  1.000000  2.000000  3.000000

In [29]: df3.dropna(axis=1)
Out[29]:
          1
0  0.537739
1  2.000000
2  4.000000
3  2.000000
```
#### 填充缺失值 fillna()
```python
In [32]: df3.fillna(-100.)
Out[32]:
            0         1           2
0   -0.735379  0.985099   -0.733799
1    1.000000  2.000000 -100.000000
2 -100.000000  4.000000 -100.000000
3    1.000000  2.000000    3.000000
```
### 统计计算

- sum
- mean
- max
- min
- describe 产生多个统计数据
> skipna 排除缺失值，默认为True，
> axis=0,按列统计，axis=1按行统计

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2130981/1668032745138-069c91d6-0f29-462c-95e6-d9a6a6ddbca8.png#averageHue=%23f8f8f8&clientId=uf1c271e2-935a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=595&id=u654ed3c6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=595&originWidth=618&originalType=binary&ratio=1&rotation=0&showTitle=false&size=172540&status=done&style=stroke&taskId=uad8df69f-e947-4e34-a247-3efe24221a1&title=&width=618)
```python
In [12]: df1 = pd.DataFrame(np.random.randn(5,4), columns=['a','b','c','d'])

In [13]: df1
Out[13]:
          a         b         c         d
0 -1.052080 -0.307690 -0.359616 -0.412333
1  0.501699 -1.184435 -1.156710 -0.496644
2 -0.591768  0.922715  0.600647 -0.838571
3  0.863097 -0.048983  1.110432 -0.059342
4 -0.353466  1.057952  0.093335  0.528852

In [14]: df1.sum()
Out[14]:
a   -0.632518
b    0.439558
c    0.288090
d   -1.278038
dtype: float64

In [15]: df1.sum(axis=1)
Out[15]:
0   -2.131719
1   -2.336090
2    0.093023
3    1.865204
4    1.326673
dtype: float64

In [16]: df1.max(axis=1)
Out[16]:
0   -0.307690
1    0.501699
2    0.922715
3    1.110432
4    1.057952
dtype: float64

In [17]: df1.max()
Out[17]:
a    0.863097
b    1.057952
c    1.110432
d    0.528852
dtype: float64

In [18]: df1.describe()
Out[18]:
              a         b         c         d
count  5.000000  5.000000  5.000000  5.000000
mean  -0.126504  0.087912  0.057618 -0.255608
std    0.790352  0.926280  0.873678  0.518750
min   -1.052080 -1.184435 -1.156710 -0.838571
25%   -0.591768 -0.307690 -0.359616 -0.496644
50%   -0.353466 -0.048983  0.093335 -0.412333
75%    0.501699  0.922715  0.600647 -0.059342
max    0.863097  1.057952  1.110432  0.528852

In [19]: df1.var()
Out[19]:
a    0.624656
b    0.857995
c    0.763313
d    0.269101
dtype: float64
```

###  分组与聚合

- groupby:分组
   - 对数据集进行分组，然后对每组进行统计分析
   - SQL能够对数据进行过滤，分组聚合
   - pandas能利用groupby进行更复杂的分组运算
      - 分组运算过程：split->apply-combine
         - 拆分：进行分组的根据
         - 应用：每个分组运行的计算规则
         - 合并，把每个分组的计算合并起来

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2130981/1668033014913-41d0813d-afc1-4009-a773-eca9cad37e56.png#averageHue=%23eeeeee&clientId=uf1c271e2-935a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=410&id=ua2d6833a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=410&originWidth=630&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81572&status=done&style=stroke&taskId=u631ceaa4-bb41-411f-9b1d-65b3bc11fa0&title=&width=630)

```python
In [1]: import pandas as pd

In [2]: import numpy as np

In [3]: dict_obj = {'key1' : ['a', 'b', 'a', 'b',
   ...:                       'a', 'b', 'a', 'a'],
   ...:             'key2' : ['one', 'one', 'two', 'three',
   ...:                       'two', 'two', 'one', 'three'],
   ...:             'data1': np.random.randn(8),
   ...:             'data2': np.random.randn(8)}

In [4]: pd.DataFrame(dict_obj)
Out[4]:
  key1   key2     data1     data2
0    a    one  0.584048  0.659140
1    b    one  0.560518  0.694774
2    a    two  1.278102 -0.698619
3    b  three  0.507953 -1.096678
4    a    two -0.425591 -1.575006
5    b    two  1.125378  0.239921
6    a    one  1.399763 -0.357698
7    a  three -0.505960  0.850917
```
#### GroupBy对象
> groupby()进行分组，GroupBy对象没有进行实际运算，只是包含分组的中间数据
> 按列名分组：obj.groupby('label')

- DataFrameGroupBy
- SeriesGroupBy
```python
In [5]: df = pd.DataFrame(dict_obj)

In [6]: df
Out[6]:
  key1   key2     data1     data2
0    a    one  0.584048  0.659140
1    b    one  0.560518  0.694774
2    a    two  1.278102 -0.698619
3    b  three  0.507953 -1.096678
4    a    two -0.425591 -1.575006
5    b    two  1.125378  0.239921
6    a    one  1.399763 -0.357698
7    a  three -0.505960  0.850917

In [7]: g1 = df.groupby('key1')

In [8]: g1
Out[8]: <pandas.core.groupby.generic.DataFrameGroupBy object at 0x10bc33c40>

In [9]: g2 = df['data1'].groupby(df['key1'])

In [10]: g2
Out[10]: <pandas.core.groupby.generic.SeriesGroupBy object at 0x10ba21880>
```
#### 分组运算

- 对GroupBy对象进行分组运算、多重分组运算，如mean()
- 非数值不进行分组运算
- size()返回每个分组元素的个数
```python
In [11]: g1.mean()
<ipython-input-11-71b5d8821406>:1: FutureWarning: The default value of numeric_only in DataFrameGroupBy.mean is deprecated. In a future version, numeric_only will default to False. Either specify numeric_only or select only columns which should be valid for the function.
  g1.mean()
Out[11]:
         data1     data2
key1
a     0.466072 -0.224253
b     0.731283 -0.053994

In [12]: g2.mean()
Out[12]:
key1
a    0.466072
b    0.731283
Name: data1, dtype: float64

In [13]: g1.size()
Out[13]:
key1
a    5
b    3
dtype: int64

In [14]: g2.size()
Out[14]:
key1
a    5
b    3
Name: data1, dtype: int64
```
#### 按自定义的key分组

- 自定义的key可以是列表或多层列表
- obj.groupby(self_def_key)
- obj.groupby(['label1', 'label2'])  ->多层DataFrame
```python
In [15]: self_def_key = [0, 1, 2, 3, 3, 4, 5, 7]  # 自定义key列表

In [16]: df.groupby(self_def_key)
Out[16]: <pandas.core.groupby.generic.DataFrameGroupBy object at 0x10bc57e50>

In [17]: custome_group = df.groupby(self_def_key)

In [18]: custome_group.size()
Out[18]:
0    1
1    1
2    1
3    2
4    1
5    1
7    1
dtype: int64

In [19]: custome_group
Out[19]: <pandas.core.groupby.generic.DataFrameGroupBy object at 0x10bc33b50>

In [20]: custom_gourp2 = df.groupby([df['key1'],df['key2']])  # 按自定义key分组，多层列表

In [21]: custom_gourp2.size()
Out[21]:
key1  key2
a     one      2
      three    1
      two      2
b     one      1
      three    1
      two      1
dtype: int64

In [22]: custom_gourp3 = df.groupby(['key1', 'key2'])  # 按多个列多层分组

In [23]: custom_gourp3.size()
Out[23]:
key1  key2
a     one      2
      three    1
      two      2
b     one      1
      three    1
      two      1
dtype: int64

In [24]: custom_gourp3.mean()  # 多层分组按key的顺序进行
Out[24]:
               data1     data2
key1 key2
a    one    0.991905  0.150721
     three -0.505960  0.850917
     two    0.426256 -1.136813
b    one    0.560518  0.694774
     three  0.507953 -1.096678
     two    1.125378  0.239921

In [25]: custom_gourp3.mean().unstack()  # unstack可以将多层索引的结果转换成单层的DataFrame
Out[25]:
         data1                         data2
key2       one     three       two       one     three       two
key1
a     0.991905 -0.505960  0.426256  0.150721  0.850917 -1.136813
b     0.560518  0.507953  1.125378  0.694774 -1.096678  0.239921
```
#### GroupBy对象的迭代操作

- 每次迭代返回一个元组（group_name, group_data）
- 可用于分组数据的具体计算

- 单层分组
```python
# 单层分组，根据key1
for group_name, group_data in groupd1:
    print(group_name)
    print(group_data)
```

- 多层分组
```python
# 多层分组，根据key1 和 key2
for group_name, group_data in groupd2:
    print(group_name)
    print(group_data)
```
```python
In [27]: dict_obj = {'key1' : ['a', 'b', 'a', 'b',
    ...:                       'a', 'b', 'a', 'a'],
    ...:             'key2' : ['one', 'one', 'two', 'three',
    ...:                       'two', 'two', 'one', 'three'],
    ...:             'data1': np.random.randn(8),
    ...:             'data2': np.random.randn(8)}

In [28]: df = pd.DataFrame(dict_obj)

In [29]: df
Out[29]:
  key1   key2     data1     data2
0    a    one -0.547337  0.723809
1    b    one  0.143368 -1.455575
2    a    two  1.400180  0.091538
3    b  three -0.827495 -1.104382
4    a    two  0.991163  0.135557
5    b    two -0.003753 -0.834571
6    a    one -0.719334 -1.852736
7    a  three  0.107079 -0.110314

In [30]: gd1 = df.groupby('key1')

In [31]: for gname, gdata in gd1:
    ...:     print(gname)
    ...:     print(gdata)
    ...:
a
  key1   key2     data1     data2
0    a    one -0.547337  0.723809
2    a    two  1.400180  0.091538
4    a    two  0.991163  0.135557
6    a    one -0.719334 -1.852736
7    a  three  0.107079 -0.110314
b
  key1   key2     data1     data2
1    b    one  0.143368 -1.455575
3    b  three -0.827495 -1.104382
5    b    two -0.003753 -0.834571

In [36]: gd2 = df.groupby(['key1','key2'])

In [37]: gd2
Out[37]: <pandas.core.groupby.generic.DataFrameGroupBy object at 0x1222883a0>

In [38]: for gname, gdata in gd2:
    ...:     print(gname)
    ...:     print(gdata)
    ...:
('a', 'one')
  key1 key2     data1     data2
0    a  one -0.547337  0.723809
6    a  one -0.719334 -1.852736
('a', 'three')
  key1   key2     data1     data2
7    a  three  0.107079 -0.110314
('a', 'two')
  key1 key2     data1     data2
2    a  two  1.400180  0.091538
4    a  two  0.991163  0.135557
('b', 'one')
  key1 key2     data1     data2
1    b  one  0.143368 -1.455575
('b', 'three')
  key1   key2     data1     data2
3    b  three -0.827495 -1.104382
('b', 'two')
  key1 key2     data1     data2
5    b  two -0.003753 -0.834571
```
GroupBy对象转换成列表或字典
```python
# GroupBy对象转换list
list(groupd1)

# GroupBy对象转换dict
dict(list(groupd1))
```
```python
In [39]: list(gd1)
Out[39]:
[('a',
    key1   key2     data1     data2
  0    a    one -0.547337  0.723809
  2    a    two  1.400180  0.091538
  4    a    two  0.991163  0.135557
  6    a    one -0.719334 -1.852736
  7    a  three  0.107079 -0.110314),
 ('b',
    key1   key2     data1     data2
  1    b    one  0.143368 -1.455575
  3    b  three -0.827495 -1.104382
  5    b    two -0.003753 -0.834571)]


In [41]: dict(list(gd1))
Out[41]:
{'a':   key1   key2     data1     data2
 0    a    one -0.547337  0.723809
 2    a    two  1.400180  0.091538
 4    a    two  0.991163  0.135557
 6    a    one -0.719334 -1.852736
 7    a  three  0.107079 -0.110314,
 'b':   key1   key2     data1     data2
 1    b    one  0.143368 -1.455575
 3    b  three -0.827495 -1.104382
 5    b    two -0.003753 -0.834571}
```
#### 按列分组、按数据类型分组
```python
In [42]: df.dtypes   # 按列分组
Out[42]:
key1      object
key2      object
data1    float64
data2    float64
dtype: object

In [43]: df.groupby(df.dtypes, axis=1).size()  # 按数据类型分组
Out[43]:
   float64  object
0        2       2
1        2       2
2        2       2
3        2       2
4        2       2
5        2       2
6        2       2
7        2       2

In [44]: df.groupby(df.dtypes, axis=1).sum()  # 按数据类型分组
Out[44]:
    float64  object
0  0.176472    aone
1 -1.312207    bone
2  1.491718    atwo
3 -1.931876  bthree
4  1.126720    atwo
5 -0.838324    btwo
6 -2.572070    aone
7 -0.003235  athree
```
#### 通过字典分组
```python
In [52]: dict1 = {'a':'Python', 'b':'Python', 'c':'Go', 'd':'C', 'e':'Java'}

In [53]: df1.groupby(dict1, axis=1).size()
Out[53]:
   C  Go  Java  Python
A  1   1     1       2
B  1   1     1       2
C  1   1     1       2
D  1   1     1       2
E  1   1     1       2

In [54]: df.groupby(dict1, axis=1).count()
Out[54]:
Empty DataFrame
Columns: []
Index: [0, 1, 2, 3, 4, 5, 6, 7]
```
#### 通过函数分组，函数传入的参数为行索引或列索引
```python
In [57]: df3 = pd.DataFrame(np.random.randint(1, 10, (5,5)),
    ...:                        columns=['a', 'b', 'c', 'd', 'e'],
    ...:                        index=['AA', 'BBB', 'CC', 'D', 'EE'])

In [58]: df3
Out[58]:
     a  b  c  d  e
AA   8  4  1  4  5
BBB  2  5  6  9  1
CC   1  2  1  4  1
D    2  1  6  5  7
EE   7  8  4  8  5

In [59]: def group_key(idx):  # idx未列 索引或行索引
    ...:     return len(idx)
    ...:

In [60]: df3.groupby(group_key).size()
Out[60]:
1    1
2    3
3    1
dtype: int64
```
#### 通过索引级别分组
```python
In [61]: columns = pd.MultiIndex.from_arrays([['Python', 'Java', 'Python', 'Java', 'Python'],
    ...:                                      ['A', 'A', 'B', 'C', 'B']], names=['language', 'index'])
    ...: df_obj4 = pd.DataFrame(np.random.randint(1, 10, (5, 5)), columns=columns)

In [62]: df_obj4
Out[62]:
language Python Java Python Java Python
index         A    A      B    C      B
0             3    2      8    6      7
1             2    3      7    8      3
2             2    6      5    6      2
3             6    8      3    2      3
4             7    1      4    1      6

In [63]: df_obj4.groupby(level='language', axis=1).sum()
Out[63]:
language  Java  Python
0            8      18
1           11      12
2           12       9
3           10      12
4            2      17

In [64]: df_obj4.groupby(level='index', axis=1).sum()
Out[64]:
index   A   B  C
0       5  15  6
1       5  10  8
2       8   7  6
3      14   6  2
4       8  10  1
```
### 聚合（aggregation）

- 数据产生标量的过程，如mean(),count()
- 常用于对分组后的数据进行计算
```python
In [66]: dict1 = {'key1' : ['a', 'b', 'a', 'b',
    ...:                       'a', 'b', 'a', 'a'],
    ...:             'key2' : ['one', 'one', 'two', 'three',
    ...:                       'two', 'two', 'one', 'three'],
    ...:             'data1': np.random.randint(1,10, 8),
    ...:             'data2': np.random.randint(1,10, 8)}

In [68]: df1 = pd.DataFrame(dict1)

In [69]: df1
Out[69]:
  key1   key2  data1  data2
0    a    one      9      8
1    b    one      7      2
2    a    two      1      6
3    b  three      2      5
4    a    two      4      9
5    b    two      5      7
6    a    one      5      7
7    a  three      2      2
```
内置聚合函数

- sum()
- mean()
- max()
- min()
- count()
- size()
- describe()
```python
In [69]: df1
Out[69]:
  key1   key2  data1  data2
0    a    one      9      8
1    b    one      7      2
2    a    two      1      6
3    b  three      2      5
4    a    two      4      9
5    b    two      5      7
6    a    one      5      7
7    a  three      2      2

In [70]: df1.groupby('key1').sum()
<ipython-input-70-5b0424c20cbd>:1: FutureWarning: The default value of numeric_only in DataFrameGroupBy.sum is deprecated. In a future version, numeric_only will default to False. Either specify numeric_only or select only columns which should be valid for the function.
  df1.groupby('key1').sum()
Out[70]:
      data1  data2
key1
a        21     32
b        14     14

In [71]: df1.groupby('key1').max()
Out[71]:
     key2  data1  data2
key1
a     two      9      9
b     two      7      7

In [72]: df1.groupby('key1').min()
Out[72]:
     key2  data1  data2
key1
a     one      1      2
b     one      2      2

In [73]: df1.groupby('key1').mean()
<ipython-input-73-061796447ee4>:1: FutureWarning: The default value of numeric_only in DataFrameGroupBy.mean is deprecated. In a future version, numeric_only will default to False. Either specify numeric_only or select only columns which should be valid for the function.
  df1.groupby('key1').mean()
Out[73]:
         data1     data2
key1
a     4.200000  6.400000
b     4.666667  4.666667

In [74]: df1.groupby('key1').size()
Out[74]:
key1
a    5
b    3
dtype: int64

In [75]: df1.groupby('key1').count()
Out[75]:
      key2  data1  data2
key1
a        5      5      5
b        3      3      3

In [76]: df1.groupby('key1').describe()
Out[76]:
     data1                                     ...     data2
     count      mean       std  min  25%  50%  ...       std  min  25%  50%  75%  max
key1                                           ...
a      5.0  4.200000  3.114482  1.0  2.0  4.0  ...  2.701851  2.0  6.0  7.0  8.0  9.0
b      3.0  4.666667  2.516611  2.0  3.5  5.0  ...  2.516611  2.0  3.5  5.0  6.0  7.0

[2 rows x 16 columns]
```
#### 定义函数，传入agg方法中

- groupd.agg(func)
- func的参数为groupby索引对应的记录
```python
In [78]: def peak_range(df):
    ...:     return df.max() - df.min()
    ...:

In [79]: df1.groupby('key1').agg(peak_range)
<ipython-input-79-06de1b0dad4b>:1: FutureWarning: ['key2'] did not aggregate successfully. If any error is raised this will raise in a future version of pandas. Drop these columns/ops to avoid this warning.
  df1.groupby('key1').agg(peak_range)
Out[79]:
      data1  data2
key1
a         8      7
b         5      5

In [80]: df1.groupby('key1').agg(lambda df: df.max() - df.min())
<ipython-input-80-1c659816d76c>:1: FutureWarning: ['key2'] did not aggregate successfully. If any error is raised this will raise in a future version of pandas. Drop these columns/ops to avoid this warning.
  df1.groupby('key1').agg(lambda df: df.max() - df.min())
Out[80]:
      data1  data2
key1
a         8      7
b         5      5
```
#### 应用多个聚合函数

- 同时应用多个函数进行聚合操作，使用函数列表
```python
# 默认列名为函数名
In [81]: df1.groupby('key1').agg(['mean', 'std','count', peak_range])
<ipython-input-81-1cb38207e095>:1: FutureWarning: ['key2'] did not aggregate successfully. If any error is raised this will raise in a future version of pandas. Drop these columns/ops to avoid this warning.
  df1.groupby('key1').agg(['mean', 'std','count', peak_range])
Out[81]:
         data1                                data2
          mean       std count peak_range      mean       std count peak_range
key1
a     4.200000  3.114482     5          8  6.400000  2.701851     5          7
b     4.666667  2.516611     3          5  4.666667  2.516611     3          5

# 通过元组提供新的列名
In [83]: df1.groupby('key1').agg(['mean', 'std', 'count', ('range', peak_range)])
<ipython-input-83-987a71c7f6d8>:1: FutureWarning: ['key2'] did not aggregate successfully. If any error is raised this will raise in a future version of pandas. Drop these columns/ops to avoid this warning.
  df1.groupby('key1').agg(['mean', 'std', 'count', ('range', peak_range)])
Out[83]:
         data1                           data2
          mean       std count range      mean       std count range
key1
a     4.200000  3.114482     5     8  6.400000  2.701851     5     7
b     4.666667  2.516611     3     5  4.666667  2.516611     3     5
```
#### 使用dict对不同的列分别作用不同的聚合函数
```python
# 每列作用不同的聚合函数
In [85]: dict_mapping = {'data1':'mean',
    ...:                 'data2':'sum'}

In [86]: df1.groupby('key1').agg(dict_mapping)
Out[86]:
         data1  data2
key1
a     4.200000     32
b     4.666667     14

In [87]: dict_mapping = {'data1':['mean','max'],
    ...:                 'data2':'sum'}

In [88]: df1.groupby('key1').agg(dict_mapping)
Out[88]:
         data1     data2
          mean max   sum
key1
a     4.200000   9    32
b     4.666667   7    14
```
#### 常用的内置聚合函数
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2130981/1668037150540-ea871fdb-f977-42dd-bd35-14f2f2b2e08b.png#averageHue=%23f3f3f3&clientId=uf1c271e2-935a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=403&id=u09bfbb92&margin=%5Bobject%20Object%5D&name=image.png&originHeight=403&originWidth=593&originalType=binary&ratio=1&rotation=0&showTitle=false&size=129427&status=done&style=stroke&taskId=u5c205eed-bc6f-4298-9069-e5d91161b6b&title=&width=593)
### 数据的分组运算

- sum()
   - 统计
- merge()
   - 外连接
- transform
   - transform的计算结果和原始数据的形状保持一致，
   - grouped.transform(np.sum)
- groupby.apply(func)
   - func函数也可以在各分组上分别调用，最后结果通过pd.concat组装到一起（数据合并）
   - apply可以用来处理不同分组内的缺失数据填充，填充该分组的均值。

---

### 数据清洗
#### merge | 数据连接

- pd.merge： 根据单个或多个键将不同的DataFrame的行连接起来
```python
In [1]: import numpy as np

In [2]: import pandas as pd

In [3]: df1 = pd.DataFrame({'key': ['b', 'b', 'a', 'c', 'a', 'a', 'b'], 'data1' : np.random.randint(0,1
   ...: 0,7)})

In [4]: df1
Out[4]:
  key  data1
0   b      4
1   b      3
2   a      3
3   c      4
4   a      8
5   a      3
6   b      2

In [5]: df2 = pd.DataFrame({'key': ['a', 'b', 'd'],  'data2' : np.random.randint(0,10,3)})

In [6]: df2
Out[6]:
  key  data2
0   a      1
1   b      3
2   d      2

```

- 默认将重叠的列名作为外键进行连接
```python
In [7]: pd.merge(df1, df2)
Out[7]:
  key  data1  data2
0   b      4      3
1   b      3      3
2   b      2      3
3   a      3      1
4   a      8      1
5   a      3      1


```

- on:显示指定外键
- left_on: 左侧数据的外键，
- right_on：右侧数据的外键
```python
In [8]: pd.merge(df1, df2, on='key')  # 显示指定外键
Out[8]:
  key  data1  data2
0   b      4      3
1   b      3      3
2   b      2      3
3   a      3      1
4   a      8      1
5   a      3      1

In [9]: df1 = df1.rename(columns={'key':'key1'})

In [10]: df1
Out[10]:
  key1  data1
0    b      4
1    b      3
2    a      3
3    c      4
4    a      8
5    a      3
6    b      2

In [11]: df2 = df2.rename(columns={'key':'key2'})

In [12]: df2
Out[12]:
  key2  data2
0    a      1
1    b      3
2    d      2

In [13]: pd.merge(df1,df2, left_on='key1', right_on='key2')
Out[13]:
  key1  data1 key2  data2
0    b      4    b      3
1    b      3    b      3
2    b      2    b      3
3    a      3    a      1
4    a      8    a      1
5    a      3    a      1
```

- 连接：默认是内连接，结果中的键是交集
- how： 指定连接方式
- outer: 外连接
- left： 左连接
- right: 右连接
```python
In [14]: pd.merge(df1, df2,left_on='key1', right_on='key2', how='outer')
Out[14]:
  key1  data1 key2  data2
0    b    4.0    b    3.0
1    b    3.0    b    3.0
2    b    2.0    b    3.0
3    a    3.0    a    1.0
4    a    8.0    a    1.0
5    a    3.0    a    1.0
6    c    4.0  NaN    NaN
7  NaN    NaN    d    2.0

In [15]: pd.merge(df1, df2,left_on='key1', right_on='key2', how='left')
Out[15]:
  key1  data1 key2  data2
0    b      4    b    3.0
1    b      3    b    3.0
2    a      3    a    1.0
3    c      4  NaN    NaN
4    a      8    a    1.0
5    a      3    a    1.0
6    b      2    b    3.0

In [16]: pd.merge(df1, df2,left_on='key1', right_on='key2', how='right')
Out[16]:
  key1  data1 key2  data2
0    a    3.0    a      1
1    a    8.0    a      1
2    a    3.0    a      1
3    b    4.0    b      3
4    b    3.0    b      3
5    b    2.0    b      3
6  NaN    NaN    d      2

```

- 处理重复列名
   - suffixes:  默认是_x, _y
```python
In [17]: df1 = pd.DataFrame({'key': ['b', 'b', 'a', 'c', 'a', 'a', 'b'], 'data' : np.random.randint(0,1, 0,7)})

In [18]: df2 = pd.DataFrame({'key': ['a', 'b', 'd'],  'data' : np.random.randint(0,10,3)})

In [19]: pd.merge(df1, df2, on='key', suffixes=('_left', '_right'))
Out[19]:
  key  data_left  data_right
0   b          1           3
1   b          0           3
2   b          2           3
3   a          2           4
4   a          9           4
5   a          2           4
```

- 按索引连接
   - left_index=True
   - right_index=True
```python
In [20]: df3 = pd.DataFrame({'key': ['b', 'b', 'a', 'c', 'a', 'a', 'b'], 'data1' : np.random.randint(0,10,7)})

In [21]: df4 = pd.DataFrame({'key': ['a', 'b', 'd'],  'data2' : np.random.randint(0,10,3)},index=['a',' b','d'])

In [24]: pd.merge(df3,df4,left_on='key', right_index=True)
Out[24]:
  key key_x  data1 key_y  data2
0   b     b      7     b      8
1   b     b      7     b      8
6   b     b      0     b      8
2   a     a      1     a      5
4   a     a      3     a      5
5   a     a      1     a      5

```
#### concat | 数据合并

- 沿轴方向将多个对象合并到一起

   - numpy的concat
      - np.concatenate
```python
In [3]: arr1 = np.random.randint(0, 10, (3,4))

In [4]: arr2 = np.random.randint(0, 10, (3,4))

In [5]: arr1
Out[5]:
array([[2, 4, 2, 7],
       [1, 0, 8, 5],
       [0, 9, 6, 5]])

In [6]: arr2
Out[6]:
array([[4, 2, 9, 1],
       [7, 7, 6, 1],
       [2, 9, 1, 9]])

In [7]: np.concatenate([arr1, arr2])
Out[7]:
array([[2, 4, 2, 7],
       [1, 0, 8, 5],
       [0, 9, 6, 5],
       [4, 2, 9, 1],
       [7, 7, 6, 1],
       [2, 9, 1, 9]])

In [8]: np.concatenate([arr1, arr2], axis=1)
Out[8]:
array([[2, 4, 2, 7, 4, 2, 9, 1],
       [1, 0, 8, 5, 7, 7, 6, 1],
       [0, 9, 6, 5, 2, 9, 1, 9]])
```

- pd.concat
   - 默认轴向axis=0, 列
   - join指定合并方向，默认outer
   - Series合并时查看行索引有无重复
```python
# index 不重复时
In [12]: ser1= pd.Series(np.random.randint(0, 10, 5),index=range(0,5))

In [13]: ser2= pd.Series(np.random.randint(0, 10, 4),index=range(5,9))

In [14]: ser3= pd.Series(np.random.randint(0, 10, 3),index=range(9,12))

In [15]: ser1
Out[15]:
0    9
1    7
2    4
3    0
4    3
dtype: int64

In [16]: ser2
Out[16]:
5    0
6    6
7    7
8    0
dtype: int64

In [17]: ser3
Out[17]:
9     4
10    3
11    4
dtype: int64

In [18]: pd.concat([ser1, ser2, ser3])
Out[18]:
0     9
1     7
2     4
3     0
4     3
5     0
6     6
7     7
8     0
9     4
10    3
11    4
dtype: int64

In [19]: pd.concat([ser1, ser2, ser3], axis=1)
Out[19]:
      0    1    2
0   9.0  NaN  NaN
1   7.0  NaN  NaN
2   4.0  NaN  NaN
3   0.0  NaN  NaN
4   3.0  NaN  NaN
5   NaN  0.0  NaN
6   NaN  6.0  NaN
7   NaN  7.0  NaN
8   NaN  0.0  NaN
9   NaN  NaN  4.0
10  NaN  NaN  3.0
11  NaN  NaN  4.0
```

- index重复时
```python
In [22]: ser5 = pd.Series(np.random.randint(0, 10,4), index=range(4))

In [23]: ser6 = pd.Series(np.random.randint(0, 10,3), index=range(3))

In [24]: ser4
Out[24]:
0    3
1    4
2    8
3    7
4    1
dtype: int64

In [25]: ser5
Out[25]:
0    7
1    0
2    6
3    9
dtype: int64

In [26]: ser6
Out[26]:
0    3
1    3
2    1
dtype: int64

In [27]: pd.concat([ser4, ser5,ser6])
Out[27]:
0    3
1    4
2    8
3    7
4    1
0    7
1    0
2    6
3    9
0    3
1    3
2    1
dtype: int64

In [28]: pd.concat([ser4, ser5,ser6], axis=1)
Out[28]:
   0    1    2
0  3  7.0  3.0
1  4  0.0  3.0
2  8  6.0  1.0
3  7  9.0  NaN
4  1  NaN  NaN

In [29]: pd.concat([ser4, ser5,ser6], axis=1, join='inner')
Out[29]:
   0  1  2
0  3  7  3
1  4  0  3
2  8  6  1
```

- DataFrame合并时同时查看行索引和列索引有无重复
```python
In [30]: df5 = pd.DataFrame(np.random.randint(0, 10, (3,2)), index=['a','b','c'], columns=['A','B'])

In [31]: df6 = pd.DataFrame(np.random.randint(0, 10, (2, 2)), index=['a','b'], columns=['C','D'])

In [32]: df5
Out[32]:
   A  B
a  3  3
b  9  9
c  7  8

In [33]: df6
Out[33]:
   C  D
a  1  4
b  0  8

In [34]: pd.concat([df5, df6])
Out[34]:
     A    B    C    D
a  3.0  3.0  NaN  NaN
b  9.0  9.0  NaN  NaN
c  7.0  8.0  NaN  NaN
a  NaN  NaN  1.0  4.0
b  NaN  NaN  0.0  8.0

In [35]: pd.concat([df5, df6], axis=1, join='inner')
Out[35]:
   A  B  C  D
a  3  3  1  4
b  9  9  0  8
```
#### stack | 数据重构

- 将列索引旋转为行索引，完成层级索引
- DataFrame -> Series
```python
In [36]: df6 = pd.DataFrame(np.random.randint(0, 10, (5, 2)), columns=['data1','data2'])

In [37]: df6
Out[37]:
   data1  data2
0      5      9
1      2      5
2      2      0
3      9      6
4      2      4

In [38]: stacked = df6.stack()

In [39]: stacked
Out[39]:
0  data1    5
   data2    9
1  data1    2
   data2    5
2  data1    2
   data2    0
3  data1    9
   data2    6
4  data1    2
   data2    4
dtype: int64
```

- unstack | 将层级索引展开
- Series -> DataFrame
- 默认操作内层索引，即level=1
```python
In [39]: stacked
Out[39]:
0  data1    5
   data2    9
1  data1    2
   data2    5
2  data1    2
   data2    0
3  data1    9
   data2    6
4  data1    2
   data2    4
dtype: int64

In [40]: stacked.unstack()
Out[40]:
   data1  data2
0      5      9
1      2      5
2      2      0
3      9      6
4      2      4

In [41]: stacked.unstack(level=0)
Out[41]:
       0  1  2  3  4
data1  5  2  2  9  2
data2  9  5  0  6  4
```
#### 数据转换

- 处理重复数据
- duplicated()： 返回布尔型Series表示每行是否为重复行
- drop_duplicates(): 过滤重复行
   - 默认是全部列， 可指定按某个列判断
```python
In [4]: df1 = pd.DataFrame({'data1': ['a'] * 4 + ['b'] * 4, 'data2': np.random.randint(0,4,8)})

In [5]: df1
Out[5]:
  data1  data2
0     a      3
1     a      1
2     a      3
3     a      2
4     b      2
5     b      0
6     b      3
7     b      0

In [6]: df1.duplicated()
Out[6]:
0    False
1    False
2     True
3    False
4    False
5    False
6    False
7     True
dtype: bool

In [8]: df1.drop_duplicates()
Out[8]:
  data1  data2
0     a      3
1     a      1
3     a      2
4     b      2
5     b      0
6     b      3

In [10]: df1.drop_duplicates('data2')
Out[10]:
  data1  data2
0     a      2
1     a      1
4     b      0
```

- map
   - 根据map传入的函数对每行或每列进行转换
   - Series根据map传入的函数对每行或每列进行转换
```python
In [11]: ser1 = pd.Series(np.random.randint(0, 10, 10))

In [12]: ser1
Out[12]:
0    9
1    7
2    2
3    2
4    2
5    5
6    1
7    6
8    5
9    1
dtype: int64

In [13]: ser1.map(lambda x: x ** 2)
Out[13]:
0    81
1    49
2     4
3     4
4     4
5    25
6     1
7    36
8    25
9     1
dtype: int64
```
#### 数据替换

- replace ： 根据值的内容进行替换
```python
In [3]: ser_obj = pd.Series(np.random.randint(0,10,10))

In [4]: ser_obj
Out[4]:
0    6
1    5
2    2
3    2
4    6
5    4
6    3
7    1
8    8
9    8
dtype: int64

In [5]: ser_obj.replace(1, -100)
Out[5]:
0      6
1      5
2      2
3      2
4      6
5      4
6      3
7   -100
8      8
9      8
dtype: int64

In [6]: ser_obj.replace([6,8], -100)
Out[6]:
0   -100
1      5
2      2
3      2
4   -100
5      4
6      3
7      1
8   -100
9   -100
dtype: int64

In [10]: ser_obj.replace([4, 2], [-100, -200])
Out[10]:
0      6
1      5
2   -200
3   -200
4      6
5   -100
6      3
7      1
8      8
9      8
dtype: int64
```
