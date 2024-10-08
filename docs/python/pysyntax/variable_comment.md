---
title: python变量与注释
date: 2019-05-01 15:33:35
tags: 
  - Python
---

# python变量与注释

## 变量
> 程序中，数据都是临时存储在内存中，为了更快速查找或使用这个数据，通常把这个数据在内存中存储之后定义一个名称，这个名称就是变量
- 变量就是用来标记数据在内存中的存储位置。

### 命名规范

规范一：
- 由数字、字母、下划线组成
- 不能数字开头
- 不能使用内置关键字
- 严格区分大小写

规范二：
- 见名知义。
- 大驼峰：即每个单词首字母都大写，例如：`MyName`。
- 小驼峰：第二个（含）以后的单词首字母大写，例如：`myName`。
- 下划线：例如：`my_name`。

### 基本使用

- 定义变量
```python
# 变量名 = 值
author = 'sswang'
print('Hello, {}'.format(author))  # 输出打印方式一
print("Hello, %s" % author)  # 输出打印方式二 
print(f"author is {author}")  # 输出打印方式三【目前最常用的方式】
```

- 交换变量
```python
author, reader = 'ssw', 'sswang'
author, reader = reader, author
print(f"author: {author} reader: {reader}")
```

- 变量解包
   - 普通解包
> 解包是python里一种特殊的赋值操作，允许把一个可迭代对象的所有成员，一次性赋值给多个变量
```python
usernames = ['sswang1', 'sswang2']
author, reader = usernames
print(f"author: {author} reader: {reader}")

attrs = [1, ['sswang',100]]
user_id, (username, score)= attrs
print(f"user_id:{user_id}", f"username:{username}", f"score:{score}")
```

-  - 动态解包
> 只要使用*号表达式（*variables）作为变量名，它便贪婪地捕获多个值对象，并将捕获到的内容作为列表赋值给variables
```python
data = ['sswang','apple', 'orange', 'banana', 100]
username, *fruits, score = data
print(f"username:{username}", f"fruits:{fruits}", f"score:{score}")
```

- 切片解包
> **相对于切片，动态解包看上去更优雅**；
```python
username, fruits, score = data[0], data[1:-1], data[-1]
print(f"username:{username}", f"fruits:{fruits}", f"score:{score}")
```

- 在循环语句中使用
```python
for username, score in [("sswang1",'99'), ("sswang2",'98')]:
    print(f"username:{username}", f"score:{score}")
```

### 变量技巧
#### 技巧一：善用单下划线变量名 _

- 常被用作一个无意义的占位符，若在解包赋值时忽略某些变量，可以使用 _；
- 在python的交互式环境中，单下划线变量还有另外一个含义：默认保存我们输入的上个表达式的返回值
```python
usernames = ['sswang1', 'sswang2']
data = ['sswang','apple', 'orange', 'banana', 100]
author, _ = usernames  # 忽略展开时的第二个变量；
username, *_, score = data  # 忽略第一个和最后一个变量之间的所有变量

## 在python的交互式环境中，_ 变量还有另外一个含义：默认保存我们输入的上个表达式的返回值
In [1]: "data".upper()
Out[1]: 'DATA'
In [2]: print(_)
DATA
```

#### 技巧二：编码时多给变量注明类型

- 动态语言在使用变量不需要做任何的类型声明，对于开发者来说是优势，但对于读者来说，其实是一个劣势，因为让代码可读性变差了
- 在编码时建议尽可能多的使用类型注解，具体的做法就是：在变量名后添加类型，使用冒号分隔；

## 注释
> 代码体现技能，注释体现素养

- 注释是代码非常重要的组成部分，通常来说，注释泛指那些不影响代码实际行为的文字，它们主要起额外说明作用。

### 注释的分类
1. 代码内注释（通过在首行输入#号来表示）
2. 函数文档，也称接口注释
   1. 常用的接口注释风格
      1. Sphinx文档风格
      2. Google风格
```python
# Sphinx风格
class Person:
    """人
    
    param name: 姓名
    param age: 年龄
    param favorite_color: 最喜欢的颜色
    """
    def __init__(self, name, age, favorite_color) -> None:
        self.name = name
        self.age = age
        self.favorite_color = favorite_color

```

### 坏的注释方式
1. 用注释屏蔽代码
   - 好的做法是，将不需要的代码直接删除，而不是注释，如果未来真的需要用到这些久代码，可以去git历史里找
2. 用注释复述代
   - 好的做法1：应该描述代码之外的说明性文字，应该尽量提供读者无法直接从代码里读出来的信息，描述代码为什么要这么做，而不是简单的叙述代码本身
   - 好的做法2： 指引性注释，这种注释并不是复述代码，而是简明扼要的概括代码功能，起到代码导读的作用。
```python
# bad
# 调用strip()去掉空格
input_string = input_string.strip()

# good
# 如果直接把空格的输入传递到后端处理，可能会造成后端服务崩溃
# 因此使用strip()去掉首尾空格
input_string = input_string.strip()
```

3. 弄错接口注释的受众
   - 好的做法：应该站在函数设计者的角度，着重描述函数的功能和参数说明
```python
# bad
def resize_image(image, size):
    """将图片缩放到指定尺寸，并返回新的图片
    
    该函数将使用Pilot模块读取对象，然后调用.resize()方法将其缩放到指定尺寸。
    但由于Pilot模块自身限制，这个函数不能很好的处理过大的文件，当文件大小超过5MB时，
    resize()方法的性能会因为内存分配问题急剧下降，对于超过5MB的图片文件，请使用resize_big_iamge()替代
    
    :param iamge: 图片文件对象
    :param size: 包含宽高的元组：（width, height）
    :return : 新图片对象
    """
    ...

# good
def resize_image(image, size):
    """将图片缩放到指定尺寸，并返回新的图片
    
    注意： 当文件超过5MB时，请使用resize_big_image()
    
    :param iamge: 图片文件对象
    :param size: 包含宽高的元组：（width, height）
    :return : 新图片对象
    """
```

## 最佳实践：

1. 按照代码的职责来组织代码：让变量的定义靠近使用
2. 保持变量在两个方面的一致性：名字一致性和类型一致性
3. 适当的定义临时变量可以提升代码可读性
4. 同一作用域内不要有太多变量，解决方法：提炼数据类、拆分函数
5. 能不定义变量就别定义，不必要的变量会让代码显得冗长、啰嗦
6. 显式优于隐式：不要使用locals()批量获取变量
7. 空行也红茶一种“注释”，适当的空行让代码更易读
8. 把接口注释当成一种函数设计工具：先写注释，后写代码
