---
title: python中的面向对象编程OOP
date: 2019-05-15 17:22:49
tags: 
  - Python
---
# python中的面向对象编程OOP

## 类属性
- 类属性就是 **类对象** 所拥有的属性，它被 **该类的所有实例对象 所共有**。
- 类属性可以使用 **类对象** 或 **实例对象** 访问。
- 类属性只能通过类对象修改，不能通过实例对象修改，如果通过实例对象修改类属性，表示的是创建了一个实例属性。

类属性的优点

- **记录的某项数据 始终保持一致时**，则定义类属性。
- **实例属性** 要求 **每个对象** 为其 **单独开辟一份内存空间** 来记录数据，而 **类属性** 为全类所共有 ，**仅占用一份内存**，**更加节省内存空间**。
## 实例属性
```python
class Dog(object):
    def __init__(self):
        self.age = 5

    def info_print(self):
        print(self.age)


wangcai = Dog()
print(wangcai.age)  # 5
# print(Dog.age)  # 报错：实例属性不能通过类访问
wangcai.info_print()  # 5
```
## 类方法

- 需要用装饰器@classmethod来标识其为类方法，对于类方法，**第一个参数必须是类对象**，一般以cls作为第一个参数。
- 类方法使用场景
   - 当方法中 **需要使用类对象** (如访问私有类属性等)时，定义类方法
   - 类方法一般和类属性配合使用
```python
class Dog(object):
    __tooth = 10

    @classmethod
    def get_tooth(cls):
        return cls.__tooth


wangcai = Dog()
result = wangcai.get_tooth()
print(result)  # 10
```
## 静态方法

- 静态方法特点
   - 需要通过装饰器@staticmethod来进行修饰，**静态方法既不需要传递类对象也不需要传递实例对象（形参没有self/cls）**。
   - 静态方法 也能够通过 **实例对象** 和 **类对象** 去访问。
- 静态方法使用场景
   - 当方法中 **既不需要使用实例对象**(如实例对象，实例属性)，**也不需要使用类对象** (如类属性、类方法、创建实例等)时，定义静态方法
   - **取消不需要的参数传递**，有利于 **减少不必要的内存占用和性能消耗**
```
class Dog(object):
    @staticmethod
    def info_print():
        print('这是一个狗类，用于创建狗实例....')


wangcai = Dog()
# 静态方法既可以使用对象访问又可以使用类访问
wangcai.info_print()
Dog.info_print()
```


```python
class A(object):

    a = 'A'  # 类属性

    def __init__(self):
        self.b = "B"  # 实例属性

    class Meta:  # 对象属性(在一个类中定义另外一个类，当做另外一个类的属性)
        c = 'C'  # 这些属性控制类本身，即A的属性

    def a_print(self):  # 实例方法
        print('a_print')

    @classmethod
    def class_print(cls):  # 类方法
        print(cls.__name__)
        print('class_print')


    @staticmethod
    def static_print():  # 静态方法
        print('static_print')

    @property
    def data(self):  # 该方法当做属性来使用
        print('data')

    def __str__(self):
        return '__str__'  # 魔法方法，返回类名


print(A.a)  # 类调用类属性，成功
"""
    A
"""

print("---**********-----")

# print(A.b)  # 类调用实例属性失败
"""
    AttributeError: type object 'A' has no attribute 'b'
"""

print(A().a)  # A()——-->实例对象
print(A().b)
"""
    A
    B
"""

print(A().Meta.c)

# A.a_print() # 错误
A.a_print(A()) # 正确

"""
A.class_print()
A.static_print()
类名可以直接调用：
    类方法
    静态方法

"""

"""
A().a_print()
A().class_print()
A().static_print()
A().data

类对象可以调用所有的方法
"""

class B(object):
    A = A() # 把另一个类的对象当做另外一个类的属性来进行调用，（另外一种方式就是继承）
B().A.a_print()


```
