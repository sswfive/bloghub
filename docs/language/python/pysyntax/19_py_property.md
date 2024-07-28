---
title: python中的property
date: 2019-05-29 20:42:28
tags: 
  - Python
---

# python中的property

##  常用dir内置函数
    在 标识符 / 数据 后输入一个 .，然后按下 TAB 键，iPython 会提示该对象能够调用的 方法列表
    使用内置函数 dir 传入 标识符 / 数据，可以查看对象内的 所有属性及方法

方法 | 功能
:---:|:---
 __ new __ |创建对象时，会被 自动 调用
__ init __|对象被初始化时，会被 自动 调用；[1]分配空间，[2]设置初始值
__ del __|对象被从内存中销毁前，会被 自动 调用
__ str __ |返回对象的描述信息，(用return返回结果，返回的结果是一个字符串)

### 初始化的同时设置初始值:

```
>在开发中，如果希望在 创建对象的同时，就设置对象的属性，可以对 __init__ 方法进行 改造

> 把希望设置的属性值，定义成 __init__ 方法的参数

> 在方法内部使用 self.属性 = 形参 接收外部传递的参数

> 在创建对象时，使用 类名(属性1, 属性2...) 调用

>在 对象的方法内部，是可以 直接访问对象的属性 的

>同一个类 创建的 多个对象 之间，属性 互不干扰！

>在定义属性时，如果不知道设置什么初始值，可以设置为 None,None关键字表示什么都没有,表示一个空对象，没有方法和属性，是一个特殊的常量;可以将None赋值给任何一个变量
```
# property属性
概念：  
    一种用起来像是使用的实例属性一样的特殊属性，可以对应于某个方法;

对概念的理解：  
想获得一个实例对象或类的属性值，但是这个属性却是其它属性进行一定运算后得到的结果，此时可以用property属性
    
property属性的定义和调用要注意一下几点：  
定义时，在实例方法的基础上添加 @property 装饰器；并且仅有一个self参数
调用时，无需括号

方法：foo_obj.func()

property属性：foo_obj.prop

###   实例如下：

```python
对于京东商城中显示电脑主机的列表页面，每次请求不可能把数据库中的所有内容都显示到页面上，而是通过分页的功能局部显示，所以在向数据库中请求数据时就要显示的指定获取从第m条到第n条的所有数据 这个分页的功能包括：
根据用户请求的当前页和总数据条数计算出 m 和 n
根据m 和 n 去数据库中请求数据
# ############### 定义 ###############
class Pager:
    def __init__(self, current_page):
        # 用户当前请求的页码（第一页、第二页...）
        self.current_page = current_page
        # 每页默认显示10条数据
        self.per_items = 10 
        
    @property
    def start(self):
        val = (self.current_page - 1) * self.per_items
        return val
        
    @property
    def end(self):
        val = self.current_page * self.per_items
        return val
        
# ############### 调用 ###############
p = Pager(1)
p.start  # 就是起始值，即：m
p.end  # 就是结束值，即：n
```
###  总结：
      Python的property属性的功能是：property属性内部进行一系列的逻辑计算，最终将计算结果返回。

###   property属性的有两种方式
    装饰器 即：在方法上应用装饰器
    类属性 即：在类中定义值为property对象的类属性

#### 经典类，具有一种@property装饰器
```python
# ############### 定义 ###############    
class Goods:
    @property
    def price(self):
        return "laowang"
        
# ############### 调用 ###############
obj = Goods()
result = obj.price  # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
print(result)
```

#### 新式类，具有三种@property装饰器
```python
#coding=utf-8
# ############### 定义 ###############
class Goods:
    """python3中默认继承object类
        以python2、3执行此程序的结果不同，因为只有在python3中才有@xxx.setter  @xxx.deleter
    """
    @property
    def price(self):
        print('@property')

    @price.setter
    def price(self, value):
        print('@price.setter')

    @price.deleter
    def price(self):
        print('@price.deleter')
# ############### 调用 ###############
obj = Goods()
obj.price          # 自动执行 @property 修饰的 price 方法，并获取方法的返回值
obj.price = 123    # 自动执行 @price.setter 修饰的 price 方法，并将  123 赋值给方法的参数
del obj.price      # 自动执行 @price.deleter 修饰的 price 方法
```
### 注意
    经典类中的属性只有一种访问方式，其对应被 @property 修饰的方法
    新式类中的属性有三种访问方式，并分别对应了三个被@property、@方法名.setter、@方法名.deleter修饰的方法


**由于新式类中具有三种访问方式，分别将三个方法定义为对同一个属性：获取、修改、删**除
```python
class Goods(object):
    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8
        
    @property
    def price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price
        
    @price.setter
    def price(self, value):
        self.original_price = value
        
    @price.deleter
    def price(self):
        del self.original_price
        
obj = Goods()
obj.price         # 获取商品价格
obj.price = 200   # 修改商品原价
del obj.price     # 删除商品原价
```
###  注意：

- property方法中有个四个参数:
  - 第一个参数是方法名，调用 对象.属性 时自动触发执行方法
  - 第二个参数是方法名，调用 对象.属性 ＝ XXX 时自动触发执行方法
  - 第三个参数是方法名，调用 del 对象.属性 时自动触发执行方法
  - 第四个参数是字符串，调用 对象.属性.__doc__ ，此参数是该属性的描述信息

```python
#coding=utf-8

class Foo(object):
    def get_bar(self):
        print("getter...")
        return 'laowang'
        
    def set_bar(self, value): 
        """必须两个参数"""
        print("setter...")
        return 'set value' + value
        
    def del_bar(self):
        print("deleter...")
        return 'laowang'
        
    BAR = property(get_bar, set_bar, del_bar, "description...")
    
obj = Foo()
obj.BAR  # 自动调用第一个参数中定义的方法：get_bar
obj.BAR = "alex"  # 自动调用第二个参数中定义的方法：set_bar方法，并将“alex”当作参数传入
desc = Foo.BAR.__doc__  # 自动获取第四个参数中设置的值：description...
print(desc)
del obj.BAR  # 自动调用第三个参数中定义的方法：del_bar方法
```
# 总结：
    定义property属性共有两种方式，分别是【装饰器】和【类属性】，而【装饰器】方式针对经典类和新式类又有所不同。
    通过使用property属性，能够简化调用者在获取数据的流程

# property属性-应用
##### 1. 私有属性添加getter和setter方法

```python
class Money(object):
    def __init__(self):
        self.__money = 0
        
    def getMoney(self):
        return self.__money
        
    def setMoney(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")
```
##### 2. 使用property升级getter和setter方法
```python
class Money(object):
    def __init__(self):
        self.__money = 0
        
    def getMoney(self):
        return self.__money
        
    def setMoney(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")
    # 定义一个属性，当对这个money设置值时调用setMoney,当获取值时调用getMoney
    money = property(getMoney, setMoney)  
a = Money()
a.money = 100  # 调用setMoney方法
print(a.money)  # 调用getMoney方法
```
##### 3. 使用property取代getter和setter方法

> 重新实现一个属性的设置和读取方法,可做边界判定
```python
class Money(object):
    def __init__(self):
        self.__money = 0
    # 使用装饰器对money进行装饰，那么会自动添加一个叫money的属性，当调用获取money的值时，调用装饰的方法
    
    @property
    def money(self):
        return self.__money
    # 使用装饰器对money进行装饰，当对money设置值时，调用装饰的方法
    
    @money.setter
    def money(self, value):
        if isinstance(value, int):
            self.__money = value
        else:
            print("error:不是整型数字")
            
a = Money()
a.money = 100
print(a.money)
```
