---
title: def函数与lambda函数
date: 2019-05-10 13:38:35
tags: 
  - Python
---

# def函数与lambda函数

## 函数的概念

- 函数就是为了实现某一功能的代码段，只要写好后，就可以重复使用；
- 在python中，函数是一等公民（friist-class citizen）



## Python中的函数：

- def函数： 通常被称为常规函数，
- lambda函数： 通常被称为匿名函数



### def函数

- def函数使用def关键字修饰
  - 1.被def修饰的函数，在被调用前，该函数都是不存在的，当程序调用该函数时，def语句才会创建一个新的函数对象，并赋予其名一个函数名.
  - 在一个函数内部再定义一个函数，此时，不同函数间的声明顺序就无所谓了，其原因是，def是可执行语句，函数在调用之前都不存在，只需要保证调用时，所需函数都已经声明和定义。
- 特点：
  - 函数也是对象
  - 函数可以赋予变量
  - 函数可以当做参数传给另一个函数
  - 函数里可以在定义函数，也就是函数的嵌套
  - 函数的返回值也可以是函数对象（闭包）

```python
def func(message):
    print('Got a message: {}'.format(message))
    
send_message = func
send_message('hello world')

# 输出
Got a message: hello world


def my_func(message):
    my_sub_func(message) # 调用 my_sub_func() 在其声明之前不影响程序执行

def my_sub_func(message):
    print('Got a message: {}'.format(message))

my_func('hello world')
# 输出
# Got a message: hello world
```

- demo

```python
def find_largest_element(l):
    if not isinstance(l, list):
        print('input is not type of list')
        return
    if len(l) == 0:
        print('empty input')
        return
    largest_element = l[0]
    for item in l:
        if item > largest_element:
            largest_element = item
            print('largest element is: {}'.format(largest_element))


find_largest_element([10, 6,-5, 3, 0])
# 输出
# largest element is: 8
```

### 嵌套函数

- 顾名思义，函数里面又定义了函数，其主要作用是：

  ①，保证内部函数的隐私，内部函数只能被外部函数所调用和访问，不会暴露在全局作用域，通常用于定义隐私数据（如：数据库的连接信息）

  ②合理的使用函数嵌套，能够提高程序的运行效率。

```python
def connect_db():
    def get_db_config():
        ...
        return host, username, password
    conn = connector.connect(get_db_config())
    return conn
```



### lambda函数

- 匿名函数的格式

```python
lambda argument1, argument2,... argumentN : expression

# eg:
square = lambda x: x**2
```

- lambda是一个表达式(expression)，并不是一个语句（statement）

  > 表达式：就是用一系列公式去表达一个特定场景，如：x**2, x+2;
  >
  > 语句：一定是完成了某些功能，如赋值语句完成赋值，条件语句完成选择功能；

- lambda主体只有一行的简单表达式，不支持多行的代码块

- lambda专注简单任务，def函数专注复杂的多行逻辑。

```python
[(lambda x: x*x)(x) for x in range(10)]
# 输出
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]


l = [(1, 20), (3, 0), (9, 10), (2, -1)]
l.sort(key=lambda x: x[1]) # 按列表中元祖的第二个元素排序
print(l)
# 输出
[(2, -1), (3, 0), (9, 10), (1, 20)]
```

- lambda函数的目的
  - 简化代码的复杂度，提高代码的可读性

## lambda函数与函数式编程

- 函数式编程：是指代码中每一块都是不可变的，都是由纯函数（指函数本身相互独立，互不影响，对于相同的输入，总会有相同的输出）的形式组成；
- 函数式编程的特点：
  - 优点：主要在于纯函数的不可变特性使程序更加健壮，易于调试和测试；
  - 缺点：限制多，不好写。

```python
# 非纯函数写法
def multiply_2(l):
    for index in range(0, len(l)):
        l[index] *= 2
    return l
  
  
# 纯函数写法
def multiply_2_pure(l):
    new_list = []
    for item in l:
        new_list.append(item * 2)
    return new_list
```

## python中的函数式编程的代表

### map(function, iterable)

- 对 iterable 中的每个 元素，都运用 function 这个函数，最后返回一个新的可遍历的集合。

### filter(function, iterable)

- 对 iterable 中的每个元素，都使用 function 判断，并返回 True 或者 False，最后将返回 True 的元素组成一个新的可遍历的集合。

### reduce(function, iterable)

- 通常用来对一个集合做一些累积操作

```python 
l = [1, 2, 3, 4, 5]
new_list = map(lambda x: x * 2, l) # [2， 4， 6， 8， 10]

new_list = filter(lambda x: x % 2 == 0, l) # [2, 4]

product = reduce(lambda x, y: x * y, l) # 1*2*3*4*5 = 120
```

