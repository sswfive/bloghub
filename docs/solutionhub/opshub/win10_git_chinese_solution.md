---
title: win10的cmd中操作git中文乱码解决方法
date: 2023-04-06 07:41:55
---
# win10的cmd中操作git中文乱码解决方法


在win10的cmd终端中使用git时，出现中文乱码问题的解决方法

- 在终端中输入下列指令

```bash
git config --global core.quotepath false 
git config --global gui.encoding utf-8
git config --global i18n.commit.encoding utf-8 
git config --global i18n.logoutputencoding utf-8 
```

- 在系统环境变量中新增环境变量

```bash
# LESSCHARSET=utf-8
变量名：LESSCHARSET
值：utf-8
```

- 重启cmd，或者关闭新开一个cmd终端