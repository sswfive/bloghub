# Python常用开发工具汇总

## 工具1: Pycharm

- [Pycharm下载链接](https://www.jetbrains.com/zh-cn/pycharm/)
- 是一款python 集成开发环境（IDE）
- 分为专业版（professional-需破解）和社区版（community-免费)

### Pycharm设置py文件默认模版

- 设置方法
  - **Preferences -> Editor -> File and Code Template -> Python Script**

- 变量含义：

```
${PROJECT_NAME} - 当前Project名称;
${NAME} - 在创建文件的对话框中指定的文件名;
${USER} - 当前用户名;
${DATE} - 当前系统日期;
${TIME} - 当前系统时间;
${YEAR} - 年;
${MONTH} - 月;
${DAY} - 日;
${HOUR} - 小时;
${MINUTE} - 分钟；
${PRODUCT_NAME} - 创建文件的IDE名称;
${MONTH_NAME_SHORT} - 英文月份缩写, 如: Jan, Feb, etc;
${MONTH_NAME_FULL} - 英文月份全称, 如: January, February, etc；
```

- 模板

```python
# -*- coding: utf-8 -*-
"""
Author: ssw
File: ${NAME}.py
Time: ${DATE} ${TIME}
--------------------------------
Change Activity: ${DATE} ${TIME}
--------------------------------
Desc: 
"""

def main():
    pass

if __name__ == '__main__':
    main()
```



## 工具2: vscode

- [vscode下载链接](https://code.visualstudio.com/)
- 需要安装Python Plugin



## 工具3：Jupyter notebok

- [Jupyter安装说明](https://jupyter.org/install)
- 通常机器学习工程师使用较多



## 工具4：Ipython

- [Ipython安装说明](https://ipython.readthedocs.io/en/stable/install/index.html)
- 在交互场景下使用，可以快速编写demo和验证

- 常用命令

```python
In [2]: ?  # ipython的概述和简介

In [3]: help(name)   # 查询指定名称的帮助   

In [4]: obj?  # 列出obj对象的详细信息

In [5]: obj??  # 列出更详细的信息
```
- 特殊变量

  - _表示前一次输出
  - __表示倒数第二次输出
  - ___表示到处第三次输出
  - _dh 目录历史
  - _oh 输出历史

```python
In [18]: name = "ssw"

In [19]: name
Out[19]: 'ssw'

In [20]: age = 18

In [21]: age
Out[21]: 18

In [22]: gender = "male"

In [23]: gender
Out[23]: 'male'

In [24]: _   # 表示前一次输出
Out[24]: 'male'

In [25]: __  
Out[25]: 'male'

In [26]: ___
Out[26]: 'male'

In [27]: _
Out[27]: 'male'

In [28]: _dh
Out[28]: [PosixPath('/Users/roninsswang/Codehub/ronincode/hub-python')]

In [29]: _oh
Out[29]:
{6: '',
 8: '',
 9: 'sss',
 10: 'sss',
 12: 'sss',
 13: 14,
 14: 'sss',
 15: 'sss',
 16: [PosixPath('/Users/roninsswang/Codehub/ronincode/hub-python')],
 19: 'ssw',
 21: 18,
 23: 'male',
 24: 'male',
 25: 'male',
 26: 'male',
 27: 'male',
 28: [PosixPath('/Users/roninsswang/Codehub/ronincode/hub-python')]}

In [30]:
```
#### shell命令

- !ls -l
- !touch test.txt
- files= !ls -l | grep py
```python
In [42]: !ls -l
total 8
-rw-r--r--   1 roninsswang  staff   33  7 27 06:17 README.md
drwxr-xr-x   3 roninsswang  staff   96  7 27 21:10 case_demo
drwxr-xr-x  22 roninsswang  staff  704  7 27 06:17 common_utils
drwxr-xr-x   4 roninsswang  staff  128  7 27 06:17 db
drwxr-xr-x   8 roninsswang  staff  256  7 27 06:17 fastapi_cores
drwxr-xr-x   4 roninsswang  staff  128  7 27 06:34 jupyter-practice
drwxr-xr-x   3 roninsswang  staff   96  7 27 21:19 python工匠
drwxr-xr-x   5 roninsswang  staff  160  7 27 06:17 thirdlibs

In [43]: !touch test.txt

In [44]: !ls -l
total 8
-rw-r--r--   1 roninsswang  staff   33  7 27 06:17 README.md
drwxr-xr-x   3 roninsswang  staff   96  7 27 21:10 case_demo
drwxr-xr-x  22 roninsswang  staff  704  7 27 06:17 common_utils
drwxr-xr-x   4 roninsswang  staff  128  7 27 06:17 db
drwxr-xr-x   8 roninsswang  staff  256  7 27 06:17 fastapi_cores
drwxr-xr-x   4 roninsswang  staff  128  7 27 06:34 jupyter-practice
drwxr-xr-x   3 roninsswang  staff   96  7 27 21:19 python工匠
-rw-r--r--   1 roninsswang  staff    0  7 28 06:35 test.txt
drwxr-xr-x   5 roninsswang  staff  160  7 27 06:17 thirdlibs

In [45]: files = !ls -l| grep md

In [46]: files = !ls -l| grep py

In [47]: files
Out[47]:
['drwxr-xr-x   4 roninsswang  staff  128  7 27 06:34 jupyter-practice',
```
#### 魔法方法

- %cd 改变当前工作目录，cd可以认为是%cd的链接。路径历史在_dh中查看
- %pwd、pwd 显示当前工作目录
- %ls 、ls 返回文件列表
- 注意:%pwd这种是魔术方法，是IPython的内部实现，和操作系统无关。而!pwd 就要依赖当前操

作系统的shell提供的命令执行，默认windows不支持pwd命令

- %%js、%%javascript 在cell中运行js脚本

```python
In [48]: %cd Desktop
[Errno 2] No such file or directory: 'Desktop'
/Users/roninsswang/Codehub/ronincode/hub-python

In [49]: %pwd
Out[49]: '/Users/roninsswang/Codehub/ronincode/hub-python'

In [50]: %ls
README.md         common_utils/     fastapi_cores/       thirdlibs/
case_demo/        db/               jupyter-practice/ test.txt
```


