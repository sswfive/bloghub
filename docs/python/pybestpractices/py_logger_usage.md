---
title: Python中不同场景中使用日志的方式
date: 2024-07-31 22:49:50
tags:
  - Python
---

## 日志专题手册

- 日志操作手册
    - [在多个模块中使用日志](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#using-logging-in-multiple-modules)
    - [在多线程中使用日志](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#logging-from-multiple-threads)
    - [使用多个日志处理器和多种格式化](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#multiple-handlers-and-formatters)
    - [在多个地方记录日志](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#logging-to-multiple-destinations)
    - [日志服务器配置示例](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#configuration-server-example)
    - [处理日志处理器的阻塞](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#dealing-with-handlers-that-block)
    - [通过网络发送和接收日志](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#sending-and-receiving-logging-events-across-a-network)


  - 在日志记录中添加上下文信息
      - 使用日志适配器传递上下文信息
      - [使用除字典之外的其它对象传递上下文信息](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#using-objects-other-than-dicts-to-pass-contextual-information)
      - [使用过滤器传递上下文信息](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#using-filters-to-impart-contextual-information)


  - 从多个进程记录至单个文件
    - [Using concurrent.futures.ProcessPoolExecutor](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#using-concurrent-futures-processpoolexecutor)
  - [轮换日志文件](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#using-file-rotation)
  - [使用其他日志格式化方式](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html#use-of-alternative-formatting-styles)

