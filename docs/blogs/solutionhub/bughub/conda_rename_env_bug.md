---
title: Conda修改虚拟环境名导致异常
date: 2022-07-08 13:02:51
tags: 
  - Bug
---
## 事出有因

**由于不同的开发人员对conda环境名的的命名风格不统一，在交付部署时，为了风格统一，便于管理，于是就对其他服务的环境名进行的mv操作，导致环境包找不到**

## Bug现场

```
(project-service-py38env) [root@2-120 project-service]# bash run_server.sh
run_server.sh: /data/bluewhale/software/anaconda3/envs/project-service-py38env/bin/gunicorn: /data/bluewhale/software/anaconda3/envs/pms_py38env/bin/python: 坏的解释器: 没有那个文件或目录
```

```
# run_server.sh的内容
head run_server.sh

#!/bin/bash
gunicorn main:app -b 0.0.0.0:30010 -w 4 -k uvicorn.workers.UvicornH11Worker --daemon
```

## 迎刃而解

**根因分析：**

1. **执行gunicorn命令时，为什么还会使用原来环境名的python解释器？**
2. **于是找到gunicorn的可执行文件路径，打开看一下是不是哪里对python解释器的路径进行的硬编码，于是打开gunicorn文件,查看**

```
#!/data/bluewhale/software/anaconda3/envs/pms_py38env/bin/python
# -*- coding: utf-8 -*-
import re
import sys
from gunicorn.app.wsgiapp import run
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(run())
~
```

4. **果然第一行进行的python解释器路径的配置**
5. **经过查阅，发现conda没有提供这种修改文件名对环境名进行修改的操作，因为环境里有很多的脚本文件都配置了再最初创建环境名时设置的python解释器路径，于是乎想到了了一个命令，--clone**
6. **开始尝试,把之前通过mv命令重命名的文件名修改为原来的名字，然后在使用clone命令**

```
(pms_py38env) [root@2-120 project-service]# conda create -n project-service-py38env --clone pms_py38env
Source:      /data/bluewhale/software/anaconda3/envs/pms_py38env
Destination: /data/bluewhale/software/anaconda3/envs/project-service-py38env
Packages: 20
Files: 3367
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate project-service-py38env
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

7. **激活环境，重新运行，发现问题解决，再次打开gunicorn脚本文件查看，结果验证了第二步的猜想。**

**总结：**

1. **conda环境的重命名操作，需要使用conda create -n <new_env_name> --clone <old_env_name>**
2. **非必要情况下，不要轻易修改环境名，且不要想当然的认为修改环境名就是重命名操作。**


>  2022年07月08日13:04:56
