---
title: Docker实用技巧
date: 2022-04-01 23:41:59
tags: 
  - Docker
---
## | 批量load镜像
> 一键load文件夹里的说用镜像

### 方式一：
```bash
 ls -F /data/package_3th/*.tar | awk '{cmd="docker load -i "$0;print(cmd);system(cmd)}'
 # /data/package_3th 为镜像的文件夹
```
### 方式二：
```python
import os
import sys

if __name__ == "__main__":
    workdir = '/data/package_3th'
    os.chdir(workdir)
    files = os.listdir(workdir)
    for filename in files:
        print(filename)
        os.system('docker load -i %s'%filename)
```

---

## | runlike:通过容器打印出容器的启动命令
> 在runlike传递一个容器名称，就会直接打印出该容器的运行命令。`runlike`使用起来非常方便，可以直接通过`pip`安装，也可以通过容器方式免安装使用:

```bash
# pip
$ pip install runlike

# by docker
$ alias runlike="docker run --rm -v /var/run/docker.sock:/var/run/docker.sock assaflavie/runlike"
```
## | whaler: 通过镜像导出dockerfile
> 通过 **whaler**[2] 来快速的导出. 这里我们依旧不安装，通过容器化的方式使用dfimage命令，便于使用，我们将该命令写成命令别名：

```bash
# alias export docker image to dockerfile
$ alias whaler="docker run -t --rm -v /var/run/docker.sock:/var/run/docker.sock:ro pegleg/whaler"
```

