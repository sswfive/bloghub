---
title: python之容器类型
date: 2019-05-02 23:29:07
tags: 
  - Python
---

# python之容器类型

## 列表 | List（mutable）

### 概述：
> 在其他主流编程语言中，集合的数据类型通常是一致的

- 概念：
  - 列表 [], 是一个可以放置任意数据类型（数字、字符串、对象、列表等）的可变有序集合，底层是一个线性表结构；

- 常用操作
   - 支持索引、复数索引、切片、嵌套、遍历 等操作。
- 应用场景
   - 例如想要存储的数据或数量是可变的，比如社交平台上的一个日志功能，是统计一个用户在一周之内看了哪些用户的帖子

### 常用API
> 下列列表示例均用tmplist表示

#### 创建
```python
# 定义或初始化
[]  # 方式一
list()  # 方式二
```

#### 查找：

```python
tmplist.index(value, 开始位置下标, 结束位置下标) # 返回指定数据所在位置的下标，如果查找的数据不存在则报错。

tmplist[index]  # index是索引或者下标

index(value,[start,[stop]])
# 通过值value，从指定区间查找列表内的元素是否匹配 
# 匹配第一个就立即返回索引
# 匹配不到，抛出异常ValueError

count(value)
# 返回列表中匹配value的次数
# 时间复杂度：index和count都是O(n)

len(tmplist)
# 返回列表元素的个数

name_list = ['Tom', 'Lily', 'Rose']
print(name_list[0])  # Tom
print(name_list[1])  # Lily
print(name_list[2])  # Rose
```

#### 增加 ： 
```python
tmplist.insert(index, obj)  
# 在指定位置插入数据 ,时间复杂度： O(1) 
# 索引超过上界： 尾部追加
# 索引超过下界： 头部追加

tmplist.append(obj)  
# 在末尾追加数据  
# 时间复杂度： O(1)

tmplist.extend(tmplist2)  # 将列表2 的数据追加到列表

extend(iterable)
# 将可迭代对象追加进来，返回None

+ -> list
# 连接操作，将两个列表连接起来
# 产生的新的列表，原列表不变
# 本质上是调用了 __add__()方法

* -> list
# 重复操作，将本列表元素重复n遍，返回新的列表
```

#### 修改：
```python
tmplist[index] = value  # 修改指定索引的数据，索引不能超界
```

#### 删除：
```python
del tmplist[index]   
# 删除指定索引的数据（从内存中删除）

tmplist.remove[obj] 
# 从左到右查找第一个匹配value值，移除该元素

pop([index]) -> item
# 从索引处弹出一个元素，索引超界抛出IndexError错误

pop()
# 从列表尾部弹出一个元素

clear()
# 清楚列表所有元素，剩下一个空列表
```

#### 统计：
```python
len(tmplist)  # 列表长度

tmplist.count(value)  # value在列表中出现的次数
```

#### 排序：
```python
tmplist.sort()  
sort(key=None, reverse=False)
# 升序

tmplist.sort(reverse=True)  # 降序
tmplist.sort(key=funcname) # key是一个函数，指定key如何排序

tmplist.reverse() # 反转列表
```

#### 复制
```python
tmplist.copy()  

copy()
# 复制，返回一个新的列表
# 浅拷贝，遇到引用类型，只是复制了一个引用而已

deepcopy()
# 深拷贝，递归拷贝

```
#### 成员操作
```python
in
[3, 4] in [1, 3, [3, 4]]
for 1 in [1,2,3,4]
```

---

## 元组 | tuple（immutable）

### 概述

- 概念：
  - tuple 元组，是一个可以放置任意数据类型（数字、字符串、对象、列表等）的不可变的有序集合；

> 元组与列表的比较：
>
> 对于相同的元素，元组的存储空间比列表占用的少（元组属于静态变量，python默认会在后台对静态资源进行缓存，在下次创建同样大小的元组时，python就不在向操作系统发出请求，去寻找内存，而是直接分配之前已经缓存的内存空间，这样能大大加快程序的运行速度）; 

- 常用操作
   - 支持索引、复数索引、切片、嵌套、遍历 等操作。
   - 如果元组里有里列表，可以修改列表里的数据

-  应用场景
   - 对于存储的数据和数量不变的场景，通常选择元组会更加合适。
```python
# 比如有一个函数，需要返回的是一个地点的经纬度，
def get_location():
    ..... 
    return (longitude, latitude)
```
### 常用API
#### 创建
```python
# 定义
tuple() # 空元组
tuple(iterable)
tuple(1, ) # 一个元素的定义，需要有逗号
```
#### 访问
```python
# 支持索引（下标）
# 正索引：从左至右，从0开始，为列表中每一个元素编号
# 负索引：从右至左，从-1开始
# 正负索引不可以超界，否则引发异常IndexError

# 元组通过索引访问
tmptuple[index]
tmptuple[1]
tmptuple[-2]
tmptuple[1] =5
```
#### 查询
```python
index(value,[start,[stop]])
# 通过值value，从指定区间查找列表内的元素是否匹配
# 匹配第一个就立即返回索引
# 匹配不到，抛出异常ValueError

count(value)
# 返回列表中匹配value的次数

# 时间复杂度
index和count都是O(n)

len(tuple)
# 返回元素的个数
```


---

## 字典 | Dict（mutable）
### 概述

- 概念：
  - 字典{}, 是一系列由键（key）和值（value）配对组成的元素的集合。

- 特点：
  - 在python3.7+中，字典是有序的
  - 相对于列表和元组，字典的性能更优，特别是对于查找、添加和删除操作，字典都能在常数时间复杂度内完成。
  - 字典和集合是进行性能高度优化的数据结构。特别是对于查找、添加和删除操作
  - 字典内部结构是一个Hash表，这张表里存了哈希值、键和值三个元素

- 常用操作
  - 支持增加、删除、更新、遍历等操作

- 应用场景
  - 通常应用在对元素的高效查询场景中

### 常用API
#### 创建
```python
# 定义与初始化
d = dict()
d = {}  # 创建空字典推荐写法
dict(**kwargs）# 使用name=value对初始化一个字典
dict(iterable, **kwargs)  # 使用可迭代对象和name=value对构造字典，不过可迭代对象的元素必须是一个二元结构
d = dict(((1,'a'),(2,'b')))  #  或者 d = dict(([1,'a'],[2,'b']))     

dict(mapping,**kwarg) # 使用一个字典构建另一个字典
d = {'a':10, 'b':20, 'c':None, 'd':[1,2,3]}

# demo
d1 = {'name': 'jason', 'age': 20, 'gender': 'male'}
d2 = dict({'name': 'jason', 'age': 20, 'gender': 'male'})
d3 = dict([('name', 'jason'), ('age', 20), ('gender', 'male')])
d4 = dict(name='jason', age=20, gender='male') 

d1 == d2 == d3 ==d4
True
```
#### 取值：
```python
tmpdict[key]  # 可以从字典中取值，key不存在会报错

tmpdict.get(key)  # 可以从字典中取值，key不存在不会报错

tmpdict.get(key, None)  # 可以从字典中取值，key不存在返回None
```
#### 删除 ：
```python
tmpdict.pop（key）  # 删除指定键值对，key不存在会报错

tmpdict.popitem()   # 随机删除一个键值对

tmpdict.clear() # 清空字典

del tmpdict[name]  #  删除字典或删除字典中指定键值对。
```
#### 字典转列表：
```python
tmpdict.keys()  # 所有key列表

tmpdict.values()  # 所有value列表

tmpdict.items() # 所有(key,value)元组列表
```
#### 修改、新增、合并：
```python
tmpdict[key]=value  # key存在则修改，不存在则新建

tmpdict.setdefault(key,value) # 如果key存在，不会修改数据，如果key不存在，新建键值对

tmpdict.updata（tmpdict2）  # 将字典2的数据合并到字典
```
#### 统计
```python
len(tmpdict)  # 统计字典长度
```
#### 其他
```python
# 类方法
dict.fromkeys(iterable, value)
d = dict.fromkeys(range(5))
d = dict.fromkeys(range(5),0)
```

---

## 集合 | Set（mutable）
### 概述

- 概念：

  - 集合{}, 是一系列无序的、唯一的元素集合;

    - set的元素必须要求可以hash（目前不可hash的类型有：list/set）

    - 使用set()，进行创建，不能使用{}创建


- 常用操作
  - 支持增加、删除、更新、遍历等操作
  - 不支持索引

- 应用场景
  - 通常应用在对元素的高效查询场景中

### 常用API
#### 创建
- 创建集合使用{}或set()， 但是如果要创建空集合只能使用set()，因为{}用来创建空字典。

```python
s1 = {10, 20, 30, 40, 50}
print(s1)

s2 = {10, 30, 20, 10, 30, 40, 30, 50}
print(s2)

s3 = set('abcdefg')
print(s3)

s4 = set()
print(type(s4))  # set

s5 = {}
print(type(s5))  # dict
```
#### 增加

- add()
```python
s1 = {10, 20}
s1.add(100)
s1.add(10)
print(s1)  # {100, 10, 20}
```

- update() 追加的数据是序列。
```python
s1 = {10, 20}
# s1.update(100)  # 报错
s1.update([100, 200])
s1.update('abc')
print(s1)
```
#### 删除

- remove()，删除集合中的指定数据，如果数据不存在则报错。
```python
s1 = {10, 20}

s1.remove(10)
print(s1)

s1.remove(10)  # 报错
print(s1)
```

- discard()，删除集合中的指定数据，如果数据不存在也不会报错。
```python
s1 = {10, 20}

s1.discard(10)
print(s1)

s1.discard(10)
print(s1)
```

- pop()，随机删除集合中的某个数据，并返回这个数据。
```python
s1 = {10, 20, 30, 40, 50}
del_num = s1.pop()
print(del_num)print(s1)
```
#### 查找

- in：判断数据在集合序列
- not in：判断数据不在集合序列
```python
s1 = {10, 20, 30, 40, 50}

print(10 in s1)
print(10 not in s1)
```
#### 其他运算

- 全集
   - 所有元素的集合。例如实数集，所有实数组成的集合就是全集
- 子集subset和超集superset
   - 一个集合A所有元素都在另一个集合B内，A是B的子集，B是A的超集
- 真子集和真超集
   - A是B的子集，且A不等于B，A就是B的真子集，B是A的真超集
- 并集:多个集合合并的结果
- 交集:多个集合的公共部分
- 差集:集合中除去和其他集合公共部分
### 工程实践

- 共同好友
   - 你的好友A、B、C，他的好友C、B、D，求共同好友
   - 交集问题:{'A', 'B', 'C'}.intersection({'B', 'C', 'D'})
- 微信群提醒
   - XXX与群里其他人都不是微信朋友关系
   - 并集:userid in (A | B | C | ...) == False，A、B、C等是微信好友的并集，用户ID不在这个并集中，说明他和任何人都不是朋友

- 权限判断
   - 有一个API，要求权限同时具备A、B、C才能访问，用户权限是B、C、D，判断用户是否能够访问该API
      - API集合A，权限集合P
      - A - P = {} ，A-P为空集，说明P包含A 
      - A.issubset(P) 也行，A是P的子集也行 
      - A & P = A 也行
   - 有一个API，要求权限具备A、B、C任意一项就可访问，用户权限是B、C、D，判断用户是否能够访问该API
      - API集合A，权限集合P
      - A & P != {} 就可以
      - A.isdisjoint(P) == False 表示有交集
- 一个总任务列表，存储所有任务。一个完成的任务列表。找出为未完成的任务
   - 业务中，任务ID一般不可以重复
   - 所有任务ID放到一个set中，假设为ALL
   - 所有已完成的任务ID放到一个set中，假设为COMPLETED，它是ALL的子集 
   - ALL - COMPLETED = UNCOMPLETED 
