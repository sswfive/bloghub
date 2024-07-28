---
title: python条件与循环语句
date: 2019-05-04 16:00:19
tags: 
  - Python
---

# python条件与循环语句

## 条件语句

### 常规使用

```python
# 单条件判断
if condition_1:
    statement_1
else:
    statement_n

# 多条件判断
if condition_1:
    statement_1
elif condition_2:
    statement_2
...
elif condition_i:
    statement_i
else:
    statement_n
```

- if 语句是可以单独使用，但elif 、else都必须和if成对使用

### 省略条件场景

- 最贱实践：除boolean类型的数据外，条件判断最好采用显性写法

```python
if s:   # s is a string
    ...
if l:   # l is a list
    ...
if i:   # i is an int
    ...
... 

```



## 循环语句

### for循环

```python
for item in <iterable>:
    ...
```



### while循环

```python
while condition:
    ....
```

- while常与continue及break一起使用，
- continue： 跳出当前这层循环
- Break： 终止当前整个循环体。



### for循环和while的使用场景

- for循环通常用于遍历一个已知的集合，找出满足条件的元素，并进行相关操作
- while循环一般用于需要再满足某个条件前，不停地重复某个操作，并没有特定的集合需要去遍历。



### for循环和while循环的效率

- 通常情况下，在同等数量级的循环中，for循环的效率高于while循环。

```python
i = 0
while i < 1000000:
    i += 1

for i in range(0, 1000000):
    pass
```

- range()函数是由C语言编写，直接调用，速度非常快，
- `i += 1`,则由python解释器间接调用底层的C语言，并且此操作还涉及到对象的创建和删除



### 多元表达式

```python
expression1 if condition else expression2 for item in iterable

# 等价于

for item in iterable:
    if condition:
        expression1
    else:
        expression2

# 若没有else
expression for item in iterable if condition
```

- demo

```python
[(xx, yy) for xx in x for yy in y if xx != yy]

# 等价于

l = []
for xx in x:
    for yy in y:
        if xx != yy:
            l.append((xx, yy))
```

