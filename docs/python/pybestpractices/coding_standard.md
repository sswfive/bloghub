---
title: Python编码规范
date: 2020-11-23 20:39:09
tags:
  - Python
---


>我有一个梦想，我写的代码，可以像诗歌一样优美。我有一个梦想，我做的设计，能恰到好处，既不过度，也无不足。这种带有一点洁癖的完美主义就像一把达摩克利斯之剑，时刻提醒我，不能将就、不能妥协。   ---摘自《代码精进之路：从码农到工匠》

## Python之禅

```python
In [1]: import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```


我们也许都知道，好的变量命名和注释并不是为计算机而写，而是为阅读者或维护者而写。那么每种编程语言都有其特定的编程规范和最佳实践，那么下面开始介绍一下再Python编程应用中个人认为比较好的实践规范：

## 变量命名：

关于变量的命名，在热门的编程语言中，主要分为驼峰命名派（CameCase）和蛇形命名派（snake_case），而在Python中，这两种方式其实在不同的场景中应用都比较频繁，驼峰命名主要应用在类名的定义上，其他场景绝大部分都是使用蛇形命名，如：

- **普通变量名**：变量名，要全部小写，多个词通过下划线连接,即蛇形命名；
- **常量名**：要全部大写，多个词通过下划线连接；
- **函数(或方法)名**：要全部小写，多个词通过下划线连接；
- **类名**：要以驼峰形式命名，即单词首字母大写，多个词的话，每个词首字母大写然后直接拼接;
- **类的私有变量名**: 变量名前需要加2个下划线；

**最佳实践：**

- 命名需要做到名如其意，不要吝啬名字的长度;
- 当命名与关键字冲突时，在变量末尾加下划线，比如：class_;
- 遵循PEP 8原则（Python官方制定的编码风格指南和建议）；
- 变量的描述性要强，比如：冬天的梅花比花的描述性更强；
- 变量名要尽量短，最好不要超过4个单词；
- 命名时建议要考虑匹配类型；	

| Type                       | Public             | Internal                                                     |
| -------------------------- | ------------------ | ------------------------------------------------------------ |
| Modules                    | lower_with_under   | _lower_with_under                                            |
| Packages                   | lower_with_under   |                                                              |
| Classes                    | CapWords           | _CapWords                                                    |
| Exceptions                 | CapWords           |                                                              |
| Functions                  | lower_with_under() | _lower_with_under()                                          |
| Global/Class Constants     | CAPS_WITH_UNDER    | _CAPS_WITH_UNDER                                             |
| Global/Class Variables     | lower_with_under   | _lower_with_under                                            |
| Instance Variables         | lower_with_under   | _lower_with_under (protected) or __lower_with_under (private) |
| Method Names               | lower_with_under() | _lower_with_under() (protected) or __lower_with_under() (private) |
| Function/Method Parameters | lower_with_under   |                                                              |
| Local Variables            | lower_with_under   |                                                              |

- 实践

```python
# bad
# 描述性弱的名字，看不懂在做什么
value = process(s.strip())

# good
# 描述性强的名字，尝试从用户输入里解析一个用户名
username = extract_username(input_string.strip())


# 由于描述性强的建议，往往会让开发者认为只有变量名越长，描述性才会更强，其实不然
# bad 
def upgrade_to_level3(user):
    ""“如果积分满足要求，将用户升级到级别3"""
    how_many_points_needed_for_user_level3 = get_level_points(3)
    if user.points >= how_many_points_needed_for_user_level3:
        upgrade(user)
    else:
        raise Error(f"积分不够，必须要{how_xxxxxxx}分")

# good
def upgrade_to_level3(user):
    ""“如果积分满足要求，将用户升级到级别3"""
    level3_points = get_level_points(3)
    if user.points >= level3_points:
        upgrade(user)
else:
    raise Error(f"积分不够，必须要{level3_points}分")


# 说明：我们可以通过函数名和文档已经知道函数的作用和功能；
#     文档表明了其目的，那么在函数内部，完全可以将变量改为level3_points
# 诀窍：结合代码情景和上下文为变量进行命名；


# 配合布尔值类型的变量名
is_superuser
has_errors
allow_enmpy

# 匹配int/float类型的变量名
## 释义为数字的所有单词：port(端口号)、age(年龄)
## 使用_id结尾的单词：user_id, host_id,
## 使用以length/count开头或结尾的单词：length_of_usernmae、max_length、users_count

## 不建议拿一个名词的复数形式作为int类型的变量名；
### apples、trips,因为这类名字容易装着普通容器对象List[Apple]、List[Trip],建议使用number_of_apples或 trips_count进行表示
```


## 导包规范

在导包时，导入的内容总应该放在文件顶部, 位于模块注释和文档字符串之后,模块全局变量和常量之前. 导入应该按照从最通用到最不通用的顺序分组: 

1. 标准库导入
2. 第三方库导入
3. 应用程序指定导入

最佳实践：

- 当使用 import xxx时， 通常建议每行import只导入一个对象；
- 当使用from xxx import xxx时，import后面跟着的对象要是一个package(包对应代码目录)或者module(模块对应代码文件)，不要是一个类或者一个函数，
- 当使用from xxx import xxx时，原则上建议一行导入一个包，若实在想要在一行导入多个包，建议使用（）括号括起来；
- 当导入的包名较长时，建议使用as 进行局部重命名；
- 在工程开发上，建议引入格式化工具进行优化，如isort、black;

```python
import foo

from foo import bar
from foo.bar import baz
from foo.bar import Quux

from foo.bar import (baz,Quux)
from Foob import ar

import pandas as pd
```

---

## 注释规范：

在Python中,单行注释主要以#开头，多行注释或文档注释通常使用三连字符串进行注释，

- 代码块注释，在代码块上一行的相同缩进处以 # 开始书写注释；代码块注释最需要写注释的地方是代码中那些技巧性的部分；对于复杂的操作，应该在其操作开始之前写上若干行注释，对于不是一目了然的代码，应该在其行尾添加注释；
- 代码行注释，在代码行的尾部跟2个空格，然后以 # 开始书写注释，行注释尽量少写；
- TODO注释应该在所有开头处包含“TODO”字符串，紧跟着用括号括起来你的名字，email地址或其他标识符，然后是一个可选的冒号；
- 对于TODO注释的目的是用来表示“将来做某件事”，建议添加指定的日期；
- 英文注释开头要大写，结尾要写标点符号，避免语法错误和逻辑错误，中文注释也是相同要求；
- 改动代码逻辑时，一定要及时更新相关的注释；

**实践：**

```python
# TODO(ssw@gmail.com): Use a "*" here for string repetition.
# TODO(Luke) Change this to use relations.
```

---

## 缩进规范：

在python中，通常使用四个空格、或1个tab键实现缩进。但非常不建议Tab和空格混合使用；

## 空行规范：

1. 全局的(文件级别的)类和全局的函数上方要有两个空行
2. 类中的函数上方要有一个空行
3. 函数内部不同意群的代码块之前要有一个空行
4. 不要把多行语句合并为一行(即不要使用分号分隔多条语句)
5. 当使用控制语句if/while/for时，即使执行语句只有一行命令，也需要另起一行
6. 代码文件尾部有且只有一个空行

---

## 空格规范：

1. 函数的参数之间要有一个空格
2. 列表、元组、字典的元素之间要有一个空格
3. 字典的冒号之后要有一个空格
4. 使用#注释的话，#后要有一个空格
5. 操作符(例如+，-，*，/，&，|，=，==，!=)两边都要有一个空格，不过括号(包括小括号、中括号和大括号)内的两端不需要空格

---

## 换行规范：

1. 一般通过代码逻辑拆分等手段来控制每行的最大长度不超过79个字符，但有些时候单行代码还是不可避免的会超过这个限制，这个时候就需要通过换行来解决问题了。
2. 两种实现换行的方法：
   第一种，通过小括号进行封装，此时虽然跨行，但是仍处于一个逻辑引用之下。比如函数参数列表的场景、过长的运算语句场景
   第二种，直接通过换行符()实现

---

## 文档描述规范

通常使用三个双引号开始，三个双引号结尾来实现文档注释；

1. 首先用一句话简单说明这个函数是做什么，然后跟一段话来详细解释；
2. 再往后就是参数列表、参数格式、返回值格式的描述；

**示例：**

```python
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
        if i & (i-1) == 0:        # true iff i is a power of 2
```





