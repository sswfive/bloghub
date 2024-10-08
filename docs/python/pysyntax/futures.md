---
title: python中的Futures
date: 2019-05-26 16:52:33
tags: 
  - Python
---

# python中的Futures

## 并发并行
### 概念
并发（Concurrency）：

- 在某个特定的时刻，它只允许有一个操作发生，只不过线程/任务之间会相互切换，知道完成
   - 线程：threading
      - 对于threading,操作系统知道每个线程的所有信息，因此它会做主在适当的时候做线程切换。
      - 特点：
         - 代码容易书写，程序员不需要做任何切换处理的操作。
         - 操作系统切换线程的操作，有可能会出现race condition
   - 任务：asyncio
      - 切换任务的操作由程序员决定
      - 特点：
         - 主程序想要切换任务，必须得到此任务可以被切换的通知。
         - 可以避免race condition

并行（Parallelism）：

- 同一时刻，同时发生多进程：multi-processing

### 并发与并行的对比

- 并发：通常用于I/O操作频繁的场景，比如要从网站下载多个文件，I/O操作时间可能会比CPU运行处理时间长得多。
- 并行：更多的是应用于CPU heavy的场景，比如MapReduce中的并行计算，为了加快运行速度，一般会多用几台机器，多个处理器完成。


## 并发编程Futures

- 概念
   - Futures模块，位于concurrent.futures和asyncio中，它们都表示带有延迟的操作。
   - futures会将处于等待状态的操作包裹起来放到队列中，这些操作的状态随时可以查询，对于结果或者异常，也能够在操作完成后被获取
- 提示：
   - 通常不需要去考虑创建futures，这些futures底层已经帮我们处理好，
   - 日常工作中，只需要怎么去schedule这些futures的执行就好
- futures重点
   - Executor类：
      - 执行executor。submit(func)时，会执行func()函数，并返回创建好的future实例，以便后续查询调用
   - done()方法：
      - 表示相对应的操作是否完成，
         - True:完成
         - False:没有完成
      - done()是non-blocking的，会立即返回结果
   - add_done_callback(fn)
      - 表示futures完成后，相对应的参数函数fn,会被通知并执行调用。
   - result()
      - 表示当future完成后，返回其对应的结果或异常，
   - as_completed(fs)
      - 针对给定的future迭代器fs,在其完成后，返回完成后的迭代器

```python
def download_all(sites):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        to_do = []
        for site in sites:
            future = executor.submit(download_one, site)
            to_do.append(future)
        
        for future in concurrent.futures.as_completed(to_do):  # future顺序不固定，取决于系统的调度和每个future的执行时间
            future.result()

```
## 注意：

- python中的之所以同一时刻只允许一个线程运行，其实是由全局解释器锁的存在（GIL）
   - 为了解决race condition而引入的
- 对于I/O操作而言，当其被block的时候，全局解释器锁便被释放，使其他线程继续执行。

---

- 多线程：
```python
def download_one(url):
    resp = requests.get(url)
    print(f"Read {len(resp.content)} from {url}")

def download_all(sites):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        executor.map(download_one, sites)
```

- 多进程（并发）
```python
def download_one(url):
    resp = requests.get(url)
    print(f"Read {len(resp.content)} from {url}")

def download_all(sites):
    with with futures.ProcessPoolExecutor() as executor::
        executor.map(download_one, sites)
```

- executor.map()和map()函数类似，对每一个元素，并发调用download_one()
- 此处开了5个线程
   - 线程不是越多越好，线程的创建、维护、删除也有开销

