---
title: Python中的Asyncio
date: 2019-05-20 17:17:58
tags: 
  - Python
---
# Python中的Asyncio

## 概念：
> 并发编程模型：
> - 多线程/多进程模型
> - 协程模型


> 单线程+异步I/O：现代操作系统对I/O操作的改进中最为重要的就是支持异步I/O。
> 如果充分利用操作系统提供的异步I/o支持，就可以利用单线程模型来执行多任务，这种全新的模型称为事件驱动模型。
> Nginx：就是支持异步I/O的web服务器，它在单核CPU上采用单进程模型就可以高效支持多任务，在多核CPU上，可以运行多个进程（数量与CPU核数相同），充分利用多核CPU。


> 在python中：
> 单线程+异步I/O的编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效多任务程序。


> 事件循环：启动一个统一的调度器，让调度器来决定一个时刻去运行哪个任务，于是省却了多线程中启动线程、管理线程、同步锁等各种开销。


核心：

- 是一种单线程+异步I/O的编程模型。
- 协程是实现编发编程的一种方式。

优点：

- 协程最大的优势是提高了执行效率
   - 没有线程切换的开销：子程序的切换不是线程切换，而是由程序自身控制，
   - 不需要多线程的锁机制：因为只有一个线程，也就不存在同时写变量冲突，在协程中控制共享资源不用加锁，只想要判断状态就好。
- 如果想充分利用CPU多核特性
   - 最简单的方式：多进程+ 协程，既充分利用了多核，又充分发挥了协程的高效率

## 协程实现方法
### 方式一：生成器：

- 是python2时代实现协程的方法
### 方法二：asyncio和async/await方法

- python3.7以后才支持

## asyncio实现原理分析
### 核心：

- async
   - 修饰词，声明异步函数
   - 调用异步函数，会得到一个协程对象（coroutime object）,但不会真正执行这个函数。

### 协程的执行方法

- 方式一：await调用
   - 本质是同步调用
   - 调用await，执行效果和python正常执行一样，程序会阻塞在这里，具体细节如下：
      - 调用await,当前步骤阻塞，进入被调用的协程函数，执行完毕返回到刚刚阻塞的地方再继续执行。
- 方式二：create_task()来创建任务
   - 异步调用，python3.7后才有的特性
   - 使用create_task()创建任务后，会直接开始进行调度执行，而不是在后面的await xxx以后开始执行
      - 后面的await仅仅是等候某个人任务执行完毕。
   - asyncio.gather返回的是所有已完成Task的result,
      - 不需要再进行调用或其他操作，就可以得到全部结果
- 方式三：asyncio.run()触发协程
   - asyncio.run()函数在python3.7之后才有的特性。
   - 可以让协程接口变得非常简单（可以不用去理会事件循环怎么定义和怎么使用的问题）
   - 一个非常好的编程规范：
      - 使用asyncio.run()作为主程序的入口函数，在程序运行周期内，只调用一次asyncio.run()

### 协程运行时剖析
```python

import asyncio

async def worker_1():
    print('worker_1 start')
    await asyncio.sleep(1)
    print('worker_1 done')

async def worker_2():
    print('worker_2 start')
    await asyncio.sleep(2)
    print('worker_2 done')

async def main():
    task1 = asyncio.create_task(worker_1())
    task2 = asyncio.create_task(worker_2())
    print('before await')
    await task1
    print('awaited worker_1')
    await task2
    print('awaited worker_2')

%time asyncio.run(main())

########## 输出 ##########

before await
worker_1 start
worker_2 start
worker_1 done
awaited worker_1
worker_2 done
awaited worker_2
Wall time: 2.01 s
```
过程分析

1. asyncio.run(main())，程序进入 main() 函数，事件循环开启；
2. task1 和 task2 任务被创建，并进入事件循环等待运行；运行到 print，输出 'before await'；
3. await task1 执行，用户选择从当前的主任务中切出，事件调度器开始调度 worker_1；
4. worker_1 开始运行，运行 print 输出'worker_1 start'，然后运行到 await asyncio.sleep(1)， 从当前任务切出，事件调度器开始调度 worker_2；
5. worker_2 开始运行，运行 print 输出 'worker_2 start'，然后运行 await asyncio.sleep(2) 从当前任务切出；
6. 以上所有事件的运行时间，都应该在 1ms 到 10ms 之间，甚至可能更短，事件调度器从这个时候开始暂停调度；
7. 一秒钟后，worker_1 的 sleep 完成，事件调度器将控制权重新传给 task_1，输出 'worker_1 done'，task_1 完成任务，从事件循环中退出；
8. await task1 完成，事件调度器将控制器传给主任务，输出 'awaited worker_1'，·然后在 await task2 处继续等待；
9. 两秒钟后，worker_2 的 sleep 完成，事件调度器将控制权重新传给 task_2，输出 'worker_2 done'，task_2 完成任务，从事件循环中退出；
10. 主任务输出 'awaited worker_2'，协程全任务结束，事件循环结束。

### 协程如何实现回调函数
> python3.7+中

- 对task对象调用add_done_callback()函数，即可绑定特定回调函数
- 回调函数接收一个fufure对象，可以通过fufure.result()来获取协程函数的返回值。
```python
import asyncio

async def crawl_page(url):
    print('crawling {}'.format(url))
    sleep_time = int(url.split('_')[-1])
    await asyncio.sleep(sleep_time)
    return 'OK {}'.format(url)

async def main(urls):
    tasks = [asyncio.create_task(crawl_page(url)) for url in urls]
    for task in tasks:
        task.add_done_callback(lambda future: print('result: ', future.result()))
    await asyncio.gather(*tasks)

%time asyncio.run(main(['url_1', 'url_2', 'url_3', 'url_4']))

########## 输出 ##########
crawling url_1
crawling url_2
crawling url_3
crawling url_4
result:  OK url_1
result:  OK url_2
result:  OK url_3
result:  OK url_4
Wall time: 4 s
```
## 总结：

- 协程和多线程的区别：
   - 协程是单线程
   - 协程是由用户决定（在哪些地方交出控制权，切换到下一个任务）
- 应对小并发需求：
   - 使用async/await 和create_task结合使用
- 写协程程序时需要注意：
   - 脑海里要清晰知道事件循环的概念，知道程序什么时候需要暂停，等待I/O，什么时候一并执行到底
   - 协程并没有利用多核特性，只是利用CPU等待磁盘或网络I/O的时候，把控制权交给其他任务
      - 如果一个任务不涉及到磁盘或网络I/O，协程就退化为一个单线程程序
      - 对于计算而言，协程并没有提高效率。
      - 协程更多的是应用在I/O场景中

