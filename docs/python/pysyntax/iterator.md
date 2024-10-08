---
title: 迭代器iterator用法
date: 2019-05-09 23:48:20
tags: 
  - Python
---


## 名词解释：

- 迭代：访问集合元素的一种方式
- 迭代器：是一个可以记住遍历位置的对象
- 可迭代对象：把可以通过for...in...这类语句迭代读取一条数据供我们使用的对象


## 判断一个对象是否可以迭代的方法
- 判断一个对象是否是Iterabe对象即可
```python
from collections import Iterable

isinstance([], Iterable)  # True
isinstance({}, Iterable)  # True
isinstance('abc', Iterable)  # True
isinstance(55, Iterable)  # False
```

## 可迭代对象的本质
> 总结来说：一个对象实现了__iter__方法，就表明是一个可迭代对象

- 可迭代对象的本质是提供了一个迭代器帮助使用者对其进行遍历操作
- 实际上在进行迭代操作时，先获取该对象提供的迭代器，然后通过这个迭代器来一次获取对象中的每一个数据
- 迭代器的产生
    - 可迭代对象通过__iter__方法实现了迭代器，    


## iter()函数与next()函数

- 通过iter()函数获取可迭代对象的迭代器
    - iter()实际调用的是__iter__方法 
- 通过next()函数来获取遍历中的下一条数据
    - 当迭代到最后一个元素后，再次调用next()会抛出异常StopIteration

```python
>>> li = [1, 2, 3]
>>> li_iter = iter(li)
>>> next(li_iter)
1
>>> next(li_iter)
2
>>> next(li_iter)
3
>>> next(li_iter)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

## 判断一个对象是否是迭代器的方法
- 判断一个对象是否是Iterator对象即可

```python
from collections import Iterator

isinstance([], Iterator)  # False
isinstance(iter([]), Iterator)  # True
isinstance(iter("123"), Iterator)  # True
```

## 迭代器Iterator
>迭代器本身也是可迭代对象

- 核心：

  - 一个实现了__iter__方法和__next__方法的对象，就是迭代器。

  - 迭代器提供了一个next方法，调用此方法后，要么得到这个容器的下一个对象，要么得到一个Stoplteration的错误
  - 迭代器占用内存大

- 可迭代对象

  - 核心	
    - 通过iter()函数返回一个迭代器，再通过next()函数就可以实现遍历(for in语句将这个过程隐式化)。

  - 如何判断一个对象是否可迭代

    - 使用iter()函数进行判断

    - 使用isinstance(obj, Iterable)

### 实现原理demo

```python
class MyList(object):
    """自定义的可迭代对象"""
    def __init__(self):
        self.items = []

    def add(self, val):
        self.items.append(val)

    def __iter__(self):
        myiterator = MyIterator(self)
        return myiterator


class MyIterator(object):
    """自定义的迭代器"""
    def __init__(self, mylist):
        self.mylist = mylist
        # current用来记录当前访问到的位置
        self.current = 0

    def __next__(self):
        if self.current < len(self.mylist.items):
            item = self.mylist.items[self.current]
            self.current += 1
            return item
        else:
            raise StopIteration

    def __iter__(self):
        return self # 返回自身即可


if __name__ == '__main__':
    mylist = MyList()
    mylist.add(1)
    mylist.add(2)
    mylist.add(3)
    for num in mylist:
        print(num)
```

```python
def is_iterable(param):
    try: 
        iter(param) 
        return True
    except TypeError:
        return False

params = [
    1234,
    '1234',
    [1, 2, 3, 4],
    set([1, 2, 3, 4]),
    {1:1, 2:2, 3:3, 4:4},
    (1, 2, 3, 4)
]
    
for param in params:
    print('{} is iterable? {}'.format(param, is_iterable(param)))

########## 输出 ##########

1234 is iterable? False
1234 is iterable? True
[1, 2, 3, 4] is iterable? True
{1, 2, 3, 4} is iterable? True
{1: 1, 2: 2, 3: 3, 4: 4} is iterable? True
(1, 2, 3, 4) is iterable? True
```

