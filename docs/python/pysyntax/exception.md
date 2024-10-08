---
title: python异常处理
date: 2019-05-03 23:35:08
tags: 
  - Python
---

# python异常处理

## 程序错误与异常

- 程序错误：通常指语法错误，即写了不符合编程规范，无法被识别与执行的代码；造成的后果程序崩溃；
- 程序异常：通常指程序的语法正确，可以被执行，但在执行过程中遇到了错误，抛出了异常；造成的后果是程序终止运行；

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230807/image.4ktr3t5ckwe0.png)

## 如何处理异常

> https://docs.python.org/zh-cn/3/tutorial/errors.html

- 异常处理的目的：当程序遇到异常时，经过异常处理后，使程序继续往下执行；

### **异常处理语法**

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230807/image.6m97s0ien180.webp)

```python
try:
    一段可能出错的代码    # 一般容易出错的场景：网络请求、读写写入、处理与外部系统的交互
except 异常类型1：
    针对错误类型1，对应的代码处理
except 异常类型2:       # 只处理所示的异常
    处理状况一
except (异常类型3, 异常类型4, …):
    处理状况二
except异常类型 as 名称:
    处理状况三
except :              # 处理所有异常情形
    处理状况四
else :
    未发生异常的处理
ﬁnally :
    最后一定会执行的代码，         # 无论前面代码是否出错，最后一定会执行的代码，
    例如 f.close()                # 常用场景是关闭网络、关闭文件等
```

- 案例Demo

```python
try:
    x = int(input("Please enter a number: "))
    y = 100 / x
except ValueError:
    print("Error: there was an error")
except ZeroDivisionError:
    print("Error: 0 is an invalid number")
except Exception:
    print("Error: another error occurred")
finally:
    print("Cleanup can go here")
```

### 抛出异常语法

```python
try:
    x = int(input("Please enter a number: "))
    y = 100 / x
except ValueError:
    raise("Error: there was an error")
except ZeroDivisionError:
    raise("Error: 0 is an invalid number")
except Exception:
    raise
```



## 如何自定义异常

```Python
class CustomError(Exception): 
    def __init__(self, value): 
        self.value = value
    def __str__(self): 
        return f"Error: {self.value}"
      
try:
    raise CustomError("something went wrong")
except CustomError as e:
    print(e)
# prints "Error: something went wrong"
```



## 杜绝滥用异常处理

- bad demo

```python
d = {'name': 'ssw', 'age': 25}
try:
 value = d['five']
 ...
except KeyError as err:
 print('KeyError: {}'.format(err))
```

