---
title: vim常用命令总结
date: 2022-03-03 23:06:42
tags: 
  - vim
---
## 编辑(插入)模式
### 插入

   - i: insert  ;                           I: insert before line 
   - a: append                         A: append after line
   - o: open a line below       O: append a line above
### 删除
```bash
ctrl + h   # 删除上一个字符
ctrl + w   # 删除上个单词
ctrl + u   # 删除当前行
```
### Esc键的替代键
```bash
ctrl + c 
ctrl + [
```
### 跳转
```bash
gi    # 快速跳转到最后一次编辑的地方并进入插入模式
```
### 移动
#### 单词间移动
```bash
hjkl   # 等价于方向键

w/W   # 移动到下一个word/WORD开头

e/E   # 移动到下一个word/WORD尾部

b/B   # 回到上一个word/WORD开头，可以理解为backword

# word指的是以非空白符分割的单词，
# WORD指的是以空白分割的单词
```
#### 行间搜索移动
```bash
f{char}   # 移动到char字符上，
t{char}   # t移动到char的前一个字符

# 如果第一次没搜到，使用分号(;)/逗号(,),  继搜该行下一个/上一个
# 大写的F表示发过来搜前面的字符
```
#### 水平移动
```bash
0   # 移动行首第一个字符，
^   # 移动到第一个非空白字符
$   # 移动到行尾
g_  # 移动到行尾非空白字符
```
#### 垂直移动
```bash
()  # 使用括号在句子间移动，可以用：help来查看帮助
{}  # 使用{}在段落间移动
```
#### 页面移动
```bash
gg/G       # 移动到文件开头或结尾， 使用ctrl + o 快速返回
H/M/L      # 跳转到屏幕的开头（Head）, 中间（Middle）结尾（Lower）
ctrl + u   # 上翻页（upword）
ctrl + f   # 下翻页(forward)
zz         # 把屏幕置为中间
```
## 命令模式
### 保存
```bash
:wq
:x
```
### 分屏
```bash
:vs (vertical split)
:sp  (split)
```
### 全局替换
```bash
:% s/foo/bar/g    # foo ->bar
```


## Visual(可视化）模式
### 选择行:

   - V
### 方块选择：

   - ctrl + v

`