---
title: 生成器generator用法
date: 2019-05-09 21:49:20
tags: 
  - Python
---


## 生成器的理解
- 核心：

- - 生成器是一种特殊的迭代器，简单的理解为：生成器是懒人版本的迭代器
  - 在调用next()函数时，才会生成下一个变量
  - 通俗的说，只要函数中有yield关键字，就称为是生成器，而该函数不在是函数

- 特点：

- - 生成器占用内存小，只有在被使用时才会被调用

- 如何实现？

- - 使用yield关键字

- - - 本质是程序执行到yield，从这里暂停，跳出去执行next()函数，yield的值就是next()函数的返回值，

## 创建生成器的方法
### 方式一：将列表改为元组，即[ ] 改为（）

```python
a = [i for i in range(5)]
print(a, type(a))

b = (i for i in range(5))
print(b, type(b))

# output:
[0, 1, 2, 3, 4] <class 'list'>
<generator object <genexpr> at 0x7fd718146a40> <class 'generator'>


def generator(k):
    i = 1 
    while True:
        yield i ** k
        i += 1
```


### 方式二：使用yield关键字

- yield 的作用
    - 保存当前运行状态（断点），然后暂停执行，即将生成器（函数）挂起
    - 将yield关键字后面表达式的值作为返回值返回，此时可以理解为起到了return的作用
    - 用for循环调用generator时，拿不到generator的return语句的返回值。如果想要拿到返回值，必须捕获StopIteration错误，返回值包含在StopIteration的value中：


```python
def fib(n):
    current = 0
    num1, num2 = 0, 1
    while current < n:
        num = num1
        num1, num2 = num2, num1+num2
        current += 1
        yield num
    return 'done'

f = fib(5)
while True:
    try:
        ret = next(f)
        print(ret,end=' ')
    except StopIteration as e:
        print(e.value)
        break
```



## 唤醒生成器的方式
### next()方式
- next()函数让生成器从断点处继续执行
>Python3中的生成器可以使用return返回最终运行的返回值，而Python2中的生成器不允许使用return返回一个返回值（即可以使用return从生成器中退出，但return后不能有任何表达式）。

### send()方式
- send()函数的唤醒的一个好处是可以在唤醒的同时向断点处传入一个附加数据。

```python
def create_num(all_num):
    a, b = 0, 1
    current_num = 0
    while current_num < all_num:
        ret = yield a
        print(">>>ret>>>>", ret)
        a, b = b, a+b
        current_num += 1

obj = create_num(10)

# obj.send(None)  # send一般不会放到第一次启动生成器，如果非要这样做 那么传递None
ret = next(obj)
print(ret)
# send的结果是下一次调用yield时 yield后面的值
ret = obj.send("dddddd")
print(ret)

```



## 迭代器与生成器的比较

- 迭代器：

- - 在初始化时就完成了所有元素在内存中的产生，内存空间占用等于其大小，故占用内存大
  - 迭代器是一个有限集合

- 生成器：

- - 在初始化时只是完成了其被调用行为的定义，在被调用时才会返回由方法计算得到的元素
  - 只有在被调用时才会产生的可迭代对象，故占用内存小
  - 生成器通过被访问时调用CPU计算完成了对返回元素的即时生成，是一种能够有效节省内存和利用CPU算力的方法。
  - 生成器可以是一个无限集。

## return 与yield的区别

- return

- - 在程序函数中返回某个值，返回之后的函数就不在继续执行，彻底结束

- yield

- - 带有yield的函数是一个迭代器，函数返回某一个值时，会停留在某个位置，返回函数后，会在前面停留的位置继续执行，直到程序结束。
