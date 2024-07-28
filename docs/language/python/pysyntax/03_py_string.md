---
title: python之string
date: 2019-05-02 21:33:35
tags: 
  - Python
---

# python之string

## 字符串 | String（immutable）

### 概述：

- 字符串是由独立字符组成的序列，即字符的集合，是一个不可变对象；
- 通常使用单引号（''）双引号（""）或者三引号之中（''' '''或""" """）表示
- 支持索引、切片、遍历等操作
```python
s1 = 'string1'
s2 = "string2"
s3 = '''this's a "String" '''
s4 = """select * from user where name='tom' """
```

### 常用API

> 下文中关于字符串变量示例都用tmpstr表示

```python
len(tmpstr)  # 获取字符串的长度  

tmpstr.count(tmpstr2)  # tmpstr2在tmpstr中出现的次数

tmpstr[索引]  # 从字符串中取出单个字符
```

#### 判断类型

```python
tmpstr.isspace()  # 如果 tmpstr 中只包含空格，则返回 True

tmpstr.isalpha()  # 如果tmpstr至少有一个字符并且所有字符都是字母返回 True

tmpstr.isalnum()  # 如果tmpstr至少有一个字符并且所有字符都是字母或数字则返回 True

tmpstr.isdecimal()  # 如果 tmpstr 只包含数字则返回 True，全角数字

tmpstr.isdigit()  # 如果tmpstr只包含数字则返回True，全角数字、⑴、\u00b2

tmpstr.isnumeric()  # 如果tmpstr只包含数字则返回True，全角数字，汉字数字

tmpstr.istitle()  # 如果tmpstr是标题化的(每个单词的首字母大写)则返回 True

tmpstr.islower()  # 如果tmpstr中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是小写，则返回 True

tmpstr.isupper()  # 如果tmpstr中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是大写，则返回 True
```

#### 查找和替换

```python
tmpstr.startswith(str)  # 检查字符串是否是以 str 开头，是则返回 True

tmpstr.endswith(str)  # 检查字符串是否是以 str 结束，是则返回 True

tmpstr.find(str, start=0, end=len(tmpstr))  # 检查字符串是否是以 str 结束，是则返回 True

tmpstr.rfind(str, start=0, end=len(tmpstr))  # 类似于 find()，不过是从右边开始查找

tmpstr.index(str,start=0,end=len(tmpstr))  # 跟find()方法类似，不过如果 str 不在 string 会报错

tmpstr.rindex(str,start=0,end=len(tmpstr))  # 类似于index()，不过是从右边开始

tmpstr.replace(old_str, new_str, num=string.count(old))  # 把 string 中的 old_str 替换成 new_str，如果 num 指定，则替换不超过 num 次
```

#### 大小写转换

```python
tmpstr.capitalize()  # 把tmpstr的第一个字符大写

tmpstr.title()  # 把tmpstr的每个单词首字母大写

tmpstr.lower()  # 转换 tmpstr 中所有大写字符为小写

tmpstr.upper()  # 转换 tmpstr 中的小写字母为大写

tmpstr.swapcase()  # 翻转 tmpstr 中的大小写
```

#### 文本对齐

```python
tmpstr.ljust(width)  # 返回一个原字符串左对齐，并使用空格填充至长度 width 的新字符串

tmpstr.rjust(width)  # 返回一个原字符串右对齐，并使用空格填充至长度 width 的新字符串

tmpstr.center(width)  # 返回一个原字符串居中，并使用空格填充至长度 width 的新字符串
```

#### 去除空白字符

```python
tmpstr.lstrip()  # 截掉 tmpstr 左边（开始）的空白字符

tmpstr.rstrip()  # 截掉 tmpstr 右边（末尾）的空白字符

tmpstr.strip()  # 截掉 tmpstr 左右两边的空白字符
```

#### 拆分和连接

```python
tmpstr.partition(str)  # 把字符串 tmpstr 分成一个 3 元素的元组 (str前面, str, str后面)

tmpstr.rpartition(str)  # 类似于partition()方法，不过是从右边开始查找

tmpstr.split(str="", num)  # 以 str 为分隔符拆分 string，如果 num 有指定值，则仅分隔 num + 1 个子字符串，str 默认包含 '\r', '\t', '\n' 和空格

tmpstr.splitlines()  # 按照行('\r', '\n', '\r\n')分隔，返回一个包含各行作为元素的列表

tmpstr.join(seq)  # 以 string 作为分隔符，将 seq 中所有的元素（的字符串表示）合并为一个新的字符串 【例如：",".join(["a","b","c"])  →“a,b,c” 】
```


### 转义字符

| **转义字符** | **说明**                                     |
| ------------ | -------------------------------------------- |
| \            | 在行尾的续行符，即一行未完，转到下一行继续写 |
| \'           | 单引号                                       |
| \"           | 双引号                                       |
| \0           | 空                                           |
| \n           | 换行符                                       |
| \r           | 回车符                                       |
| \t           | 横向制表符                                    |
| \v           | 纵向制表符                                    | 
| \a           | 响铃                                         |
| \b           | 退格（Backspace）                            |
| \\           | 反斜线 \                                     |
| \0dd         | 八进制数，dd 代表字符，如 \012 代表换行      |
| \xhh         | 十六进制数，hh 代表字符，如 \x0a 代表换行    |

### 工程实践：

- 常用API实践
```python
# 字符串join连接
"string".join(iterable) -> str
# 将可迭代对象连接起来，使用string作为分隔符
# 可迭代对象本身元素都是字符串
# 返回一个新字符串
lst = ['1','2','3']
print("\"".join(lst)) # 分隔符是双引号
print(" ".join(lst))
print("\n".join(lst))
lst = ['1',['a','b'],'3']
print(" ".join(lst))

# 字符串+连接
+ -> str
# 将2个字符串连接在一起
# 返回一个新字符串

# 字符串分割
split
split(sep=None, maxsplit=-1) -> list of strings

# 将字符串按照分隔符分割成若干字符串，并返回列表
partition
partition(sep) -> (head, sep, tail)
# 将字符串按照分隔符分割成2段，返回这2段和分隔符的元组
```

- 切片实践
```python
name = "abcdefg"

print(name[2:5:1])  # cde
print(name[2:5])  # cde
print(name[:5])  # abcde
print(name[1:])  # bcdefg
print(name[:])  # abcdefg
print(name[::2])  # aceg
print(name[:-1])  # abcdef, 负1表示倒数第一个数据
print(name[-4:-1])  # def
print(name[::-1])  # gfedcba
```

### 扩展：关于Bytes、Bytearray

> bytes：不可变字节序列
> bytearray：字节数组，可变

- 概述
   - 字符串与bytes
      - 字符串是字符组成的有序序列，字符可以使用编码来理解 
      - bytes是字节组成的有序的不可变序列
      - bytearray是字节组成的有序的可变序列 

   - 编码与解码
      - 字符串按照不同的字符集编码encode返回字节序列bytes 
         - encode(encoding='utf-8', errors='strict') -> bytes 
      - 字节序列按照不同的字符集解码decode返回字符串
         - bytes.decode(encoding="utf-8", errors="strict") -> str
         - bytearray.decode(encoding="utf-8", errors="strict") -> str 


- bytes的使用
```python
# 定义
bytes() 空bytes
bytes(int) 指定字节的bytes，被0填充
bytes(iterable_of_ints) -> bytes [0,255]的int组成的可迭代对象
bytes(string, encoding[, errors]) -> bytes 等价于string.encode()
bytes(bytes_or_buffer) -> immutable copy of bytes_or_buffer 从一个字节序列或者buffer复制出
一个新的不可变的bytes对象
# 使用b前缀定义
# 只允许基本ASCII使用字符形式b'abc9' 
# 使用16进制表示b"\x41\x61"


# 常用操作
#和str类型类似，都是不可变类型，所以方法很多都一样。只不过bytes的方法，输入是bytes，输出是 bytes
b'abcdef'.replace(b'f',b'k')
b'abc'.find(b'b')
# 类方法 bytes.fromhex(string)
# string必须是2个字符的16进制的形式，'6162 6a 6b'，空格将被忽略
bytes.fromhex('6162 09 6a 6b00') p hex()
# 返回16进制表示的字符串
'abc'.encode().hex() p索引
b'abcdef'[2] 返回该字节对应的数，int类型
```

-  bytearray的使用
```python
# 定义
bytearray() 空bytearray
bytearray(int) 指定字节的bytearray，被0填充
bytearray(iterable_of_ints) -> bytearray [0,255]的int组成的可迭代对象
bytearray(string, encoding[, errors]) -> bytearray 近似string.encode()，不过返回可变对象 pbytearray(bytes_or_buffer) 从一个字节序列或者buffer复制出一个新的可变的bytearray对象
# 注意，b前缀定义的类型是bytes类型

# 常用操作
append(int) 尾部追加一个元素
insert(index, int) 在指定索引位置插入元素
extend(iterable_of_ints) 将一个可迭代的整数集合追加到当前bytearray
pop(index=-1) 从指定索引上移除元素，默认从尾部移除 premove(value) 找到第一个value移除，找不到抛
ValueError异常
# 注意:上述方法若需要使用int类型，值在[0, 255]
clear() 清空bytearray
reverse() 翻转bytearray，就地修改
```
