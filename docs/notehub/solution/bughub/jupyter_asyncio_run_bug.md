---
title: Jupyter运行asyncio run报错
date: 2022-08-30 08:58:28
tags: 
  - Bug
---

# Jupyter运行asyncio run报错

## 事出有因

- 在jupyter中运行下了代码
```python
import asyncio

async def fake_page(url):
    print(f"faking:{url}")
    sleep_time = int(url.split('_')[-1])
    await asyncio.sleep(sleep_time)
    print(f"OK：{url}")
    
async def main(urls):
    for url in urls:
        await fake_page(url)


%time asyncio.run(main(['url1', 'url2', 'url3', 'url4']))  # %time是jupyter的语法
```
## Bug现场
```python
RuntimeError: asyncio.run() cannot be called from a running event loop
```

## 迎刃而解
### 根因分析：
> 参考了下列链接的内容：
> - [https://stackoverflow.com/questions/55409641/asyncio-run-cannot-be-called-from-a-running-event-loop](https://stackoverflow.com/questions/55409641/asyncio-run-cannot-be-called-from-a-running-event-loop)
> - [https://github.com/jupyter/notebook/issues/3397](https://github.com/jupyter/notebook/issues/3397)

- 初步结论是jupyter依赖tornado，默认启动了event loop

### 实践求证：

- 在jupyter中输入
```python
import asyncio

asyncio.get_event_loop()

# >>>
<_UnixSelectorEventLoop running=True closed=False debug=False>
```

- 在terminal中输入ipython，然后在输入下列内容
```python
In [1]: import asyncio

In [2]: asyncio.get_event_loop()
Out[2]: <_UnixSelectorEventLoop running=False closed=False debug=False>
```

结论：确实jupyter默认启动了event loop

### 解决方式

- 在jupyter中在输入并运行下列内容
```python
!pip install nest_asyncio  # 安装

import nest_asyncio  
nest_asyncio.apply()
```

- 再运行开头那段程序，报错解决
