---
title: python中的内存管理
date: 2019-06-07 17:22:11
tags: 
  - Python
---

- 变量无需事先声明，也不需要指定类型
   - 动态语言的特性
- 编程中一般无须关心变量的存亡，也不用关心内存的管理
- python使用引用计数记录所有对象的引用数；
   - 当对象引用数变为0时，它就可以被垃圾回收GC
   - 技术增加：赋值给其他变量就增加引用计数，例如：x = 3; y = x;
   - 计数减少：
      - 函数运行结束时，局部变量就会被自动销毁，对象引用计数减少；
      - 变量赋值给其他对象，例如：x =3;y=x,x=4;
- 有关性能的时候，就需要考虑变量的引用问题，但是该释放内存，还是尽量不释放内存，看需求；
