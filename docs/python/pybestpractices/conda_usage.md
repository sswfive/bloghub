---
title: Conda的实践应用
date: 2022-03-22 23:12:03
tags:
  - Python
---


## 实践1：Linux（Centos8）环境修改conda环境的安装目录
需求：

- 由于当前home目录磁盘空间不足，目前有没有扩容的计划，故需要将conda环境迁移到其他磁盘空间充足的目录下；

实施流程

1. 移动conda（anaconda）文件至新路径下
```bash
mv /home/workflow/conda  /data/conda
```

2. 修改conda的环境变量
- 修改 ~/.bashrc 中的conda环境变量
```bash
# 修改 ~/.bashrc 中的conda环境变量

vim root/.bashrc  或 vim ~/.bashrc

# 修改前：
## ----------------------------------------------
 __conda_setup="$('/home/workflow/conda/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
     eval "$__conda_setup"
 else
     if [ -f "/home/workflow/conda/etc/profile.d/conda.sh" ]; then
         . "/home/workflow/conda/etc/profile.d/conda.sh"
     else
         export PATH="/home/workflow/conda/bin:$PATH"
     fi
 fi
## ----------------------------------------------


# 修改后
## ----------------------------------------------
 __conda_setup="$('/data/conda/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
     eval "$__conda_setup"
 else
    if [ -f "/data/conda/etc/profile.d/conda.sh" ]; then
        . "/data/conda/etc/profile.d/conda.sh"
    else
        export PATH="/data/conda/bin:/usr/local/gcc/bin$PATH"
     fi
 fi
## ----------------------------------------------

# 保存退出，执行source
source ~/.bashrc 或 source root/.bashrc
```

- 修改conda
```bash
# 打开conda配置文件
vim /data/conda/bin/conda

# 修改conda文件，把conda的第一行的路径修改为：
## 修改前
#!/home/workflow/conda/bin/python

## 修改后
#!/data/conda/bin/python

# 测试是否修改成功(显示版本号即为修改成功)
conda --version
```

- 修改conda.sh
```bash
# 编辑conda.sh文件
vim /data/conda/etc/profile.d/conda.sh

## 修改前conda.sh 
_CONDA_EXE="/home/workflow/conda/bin/conda"
_CONDA_ROOT="/home/workflow/conda" 

## 修改后conda.sh 
_CONDA_EXE="/data/conda/bin/conda" 
_CONDA_ROOT="/data/conda"
```

- 修改activate和deactivate
```bash
# 修改activate
vim /data/conda/bin/activate

_CONDA_ROOT="/data/conda"


# 修改deactivate
vim /data/conda/bin/deactivate

_CONDA_ROOT="/data/conda"
```

- 修改虚拟环境中的pip
```bash
# 已存在的环境中
vim /data/conda/envs/automl/bin/pip

## 修改前：
/home/workflow/conda/envs/automl/bin/python

## 将此文件的第一行路径修改为：
data/conda/envs/automl/bin/python

# 测试是否成功
conda info -e
# logger>>>-------------------------------------------
# conda environments:
base                  *  /data/conda
automl                   /data/conda/envs/automl
cf                       /data/conda/envs/cf
demoenv                  /data/conda/envs/demoenv
dz2                      /data/conda/envs/dz2
graph_inductive          /data/conda/envs/graph_inductive
jupyter                  /data/conda/envs/jupyter
keras                    /data/conda/envs/keras
keras_spark3_rossmann     /data/conda/envs/keras_spark3_rossmann
only-keras               /data/conda/envs/only-keras
only-torch               /data/conda/envs/only-torch
torch                    /data/conda/envs/torch
zjp                      /data/conda/envs/zjp
```

## 实践2：将一个已存在的环境迁移到一个无外网（内网）环境中（Linux-centos7）的具体流程

- step1: 安装打包工具conda-pack
```bash
conda install -c conda-forge conda-pack
或
pip install conda-pack
```

- step2: 执行打包操作
```bash
# Pack environment my_env into my_env.tar.gz
conda pack -n my_env

# Pack environment my_env into out_name.tar.gz
conda pack -n my_env -o out_name.tar.gz

# Pack environment located at an explicit path into my_env.tar.gz
conda pack -p /explicit/path/to/my_env
```

- step3: 将打包好的环境包，迁移到预安装的系统中
```bash
# 查看到当前系统的conda默认安装路径（主要看base路径就好）
conda info -e

# 在base的同级目录创建一个文件夹（文件夹名即为环境名）
mkdir -p /data/software/anaconda/envs/demoenv
tar -xzf my_env.tar.gz -C /data/software/anaconda/envs/demoenv

# 查看是否成功
conda env list
```
## 实践3： conda在ARM环境中的安装
> **参考地址：**[https://bbs.huaweicloud.com/forum/thread-40622-1-1.html](https://bbs.huaweicloud.com/forum/thread-40622-1-1.html)

**在华为云上使用常规conda install xxx或 pip install xxx,会出现无法安装对应包**
**所以使用conda-forge解决无法install的问题**

- 举例conda命令安装numpy
1. 搜索华为云支持的对应版本
```bash
conda search numpy -c conda-forge
```

2. 得到华为云所有支持的numpy版本
![](https://jsd.cdn.zzko.cn/gh/sswfive/blog-pic@main/20230212/image.4cif76xn1k20.webp#id=wyODg&originHeight=487&originWidth=458&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
3. 选择想要安装的版本安装
```bash
conda install numpy=1.19.5 -c conda-forge
```
