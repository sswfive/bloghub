---
title: Python基础知识梳理
date: 2019-05-01 12:43:35
tags: 
  - Python
---

# Python基础知识梳理


## 关键字清单
| 关键字 | 描述 |
| --- | --- |
| and | 逻辑与 |
| or | 逻辑或 |
| not | 逻辑非 |
| if | 条件语句，常与else、elif结合使用 |
| elif | 条件语句，常与if、else结合使用 |
| else | 条件语句，与if、elif结合使用；也可用于异常或循环语句 |
| for | for循环语句 |
| while | while循环语句 |
| True | 布尔值，表示真 |
| False | 布尔值，表示假 |
| continue | 跳出本次循环，断续执行下次循环 |
| break | 中断整个循环 |
| pass | 空的类、方法和函数的占位符 |
| try | 用于异常捕获，与except、finally、else结合使用 |
| except | 捕获异常后的操作代码块，与try、finally、else结合使用 |
| finally | 出现异常后，始终要执行finally包含的代码块，与try、except结合使用 |
| raise | 抛出异常 |
| from | 用于导入模块，与import结合使用 |
| import | 用于导入模块，与from结合使用 |
| def | 定义函数或方法 |
| return | 函数或方法的返回值 |
| class | 定义类 |
| lambda | 匿名函数 |
| del | 删除变量或某个序列中的值 |
| global | 定义全局变量的标识符 |
| nonlocal | 声明一个非全局变量，用于标识外部作用域的变量 |
| in | 判断某个变量是否在序列中 |
| is | 判断是否是同一个变量 |
| None | 空，其类型为：NoneType |
| assert | 用于调试、断言 |
| as | 创建别名 |
| with  | 上下文标识符，常与open使用，用于读写文件 |
| yield | 生成器标识符，结束一个函数 |

## 基本数据类型
| 类别名称 | 类别描述 | 类型分类 |
| --- | --- | --- |
| Number | 数字 | 不可变 |
| String | 字符串 | 不可变 |
| Tuple | 元组 | 不可变 |
| List | 列表 | 可变 |
| Set | 集合 | 可变 |
| Dict | 字典 | 可变 |

## 条件语句
```python
if 表达式1:
    语句
    if 表达式2:
        语句
    elif 表达式3:
        语句
    else:
        语句
elif 表达式4:
    语句
else:
    语句
```
## 循环语句

- while循环
```python
# 表示1：
while 判断条件(condition)：
    执行语句(statements)……

    
# 表示2：
while <expr>:
    <statement(s)>
else:
    <additional_statement(s)>
```

- for循环
```python
for <variable> in <sequence>:
    <statements>
else:
    <statements>
```
## 函数表示

- 实名函数
```python
def 函数名（参数列表）:
    函数体
```

- 匿名函数
```python
lambda [arg1 [,arg2,.....argn]]:expression
```
## 类表示
```python
class ClassName:
    <statement-1>
    .
    .
    .
    <statement-N>

    
class DerivedClassName(Base1, Base2, Base3):
    <statement-1>
    .
    .
    .
    <statement-N>
```
## 异常处理
```python
try:
    执行代码
except Exception as e:
    发生异常时执行的代码
else:
    没有异常时执行的代码
finally:
    不管有没有异常都会执行的代码
```
