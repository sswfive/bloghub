---
title: Jupyter系列工具的使用总结
date: 2022-01-02 21:25:10
tags: 
  - Python
---

# Jupyter系列工具的使用总结



## 一、Jupyter Notebook
### Jupyter概述
- 创始人（Fernando Pérez）所言：最初的梦想是做一个综合Ju(Julia)、Py(Python)和R三种科学运算语言的计算工具平台，所以将其命名为Ju-Py-te-R.发展到现在，Jupyter已经成为了一个几乎支持所有语言，能够把软件代码、计算输出、解释文档、多媒体资源整合在一起的多功能科学运算平台；

- Jupyter优点：

  - 整合所有的资源，提高开发效率
     - Jupyter 通过把所有和软件编写有关的资源全部放在一个地方
     - 当你打开一个 Jupyter Notebook 时，就已经可以看到相应的文档、图表、视频和相应的代码。这样，你就不需要切换窗口去找资料，只要看一个文件，就可以获得项目的所有信息。
  - 交互性编程体验
     - 为了解决在传统开发环境中，每一次实验都需要把所有的代码重新跑一遍，及其浪费时间；
     - 引入了Cell的概念，每次小实验可以只跑在一个小的Cell里，实现了所见即所得，这样强的互动性，让 Python 研究员可以专注于问题本身，不被繁杂的工具链所累，不用在命令行直接切换，所有科研工作都能在 Jupyter 上完成。
  - 零成本重现结果

- Jupyter解决的问题域：

  - 帮助用户解决繁琐的环境安装问题，给用户提供了云端的运行环境，致力提高用户真正生产力
    - 常见的场景是，我在论文里看到别人的方法效果很好，可是当我去重现时，却发现需要 pip 重新安装一堆依赖软件。这些准备工作可能会消耗你 80% 的时间，却并不是真正的生产力

  - 云端环境环境地址：	

    - Jupyter官方的[binder](https://mybinder.readthedocs.io/en/latest/index.html)平台
       - 当你用 Binder 打开一份 GitHub 上的 Jupyter Notebook 时，你不需要安装任何软件，直接在浏览器打开一份代码，就能在云端运行。

    - Google [Colab](https://colab.research.google.com/notebooks/welcome.ipynb)平台

### 环境安装：
使用云端环境：

- 第一个是 Jupyter 官方：[在线binder平台](https://mybinder.org/v2/gh/binder-examples/matplotlib-versions/mpl-v2.0/?filepath=matplotlib_versions_demo.ipynb)
- 第二个是 Google Research 提供的 Colab 环境，尤其适合机器学习的实践应用：[在线Colab平台](https://colab.research.google.com/notebooks/basic_features_overview.ipynb)

使用本地环境：

- 安装：[Jupyter安装](https://jupyter.org/install.html)
- 运行：[Jupyter运行](https://docs.jupyter.org/en/latest/running.html)
```bash
# install
pip install notebook

# run
jupyter notebook
```
### 使用总结

#### 支持两种模式：

- 编辑模式（Enter）
  - 命令模式下`回车Enter`或`鼠标双击`cell进入编辑模式
  - 可以**操作cell内文本**或代码，剪切／复制／粘贴移动等操作
- 命令模式（Esc）
  - 按`Esc`退出编辑，进入命令模式
  - 可以**操作cell单元本身**进行剪切／复制／粘贴／移动等操作

#### 常用快捷键

> notebook的文档格式是`.ipynb`
>
> cell行号前的 * ，表示代码正在运行

- 两种模式通用快捷键

  - **`Shift+Enter`，执行本单元代码，并跳转到下一单元**
  - **`Ctrl+Enter`，执行本单元代码，留在本单元**

- 命令模式

  ：按ESC进入

  - `Y`，cell切换到Code模式
  - `M`，cell切换到Markdown模式
  - `A`，在当前cell的上面添加cell
  - `B`，在当前cell的下面添加cell
  - `双击D`：删除当前cell
  - `Z`，回退
  - `L`，为当前cell加上行号 <!--
  - `Ctrl+Shift+P`，对话框输入命令直接运行
  - 快速跳转到首个cell，`Crtl+Home`
  - 快速跳转到最后一个cell，`Crtl+End` 

- 编辑模式

  ：按Enter进入

  - 多光标操作：`Ctrl键点击鼠标`（Mac:CMD+点击鼠标）
  - 回退：`Ctrl+Z`（Mac:CMD+Z）
  - 重做：`Ctrl+Y`（Mac:CMD+Y)
  - 补全代码：变量、方法后跟`Tab键`
  - 为一行或多行代码添加/取消注释：`Ctrl+/`（Mac:CMD+/）
  - 屏蔽自动输出信息：可在最后一条语句之后加一个分号

#### 插件安装

- 自动补全插件的安装

  

```bash
python -m pip install jupyter_contrib_nbextensions

jupyter contrib nbextension install --user --skip-running-check
```



## 二、JupyterLab

- [官方链接](https://docs.jupyter.org/en/latest/install/notebook-classic.html)

- jupyterlab是jupyter notebook的升级版

  ```bash
  # install
  pip install jupyterlab
  
  # run
  jupyter-lab
  ```



## 三、JupyterHub

- jupyterhub是juypter notebook的多用户版，内置jupyterlab。

### 安装部署概述

1. conda安装与配置
2. Nodejs安装与配置
3. Jupyterhub安装配置
4. 添加Linux系统用户登录
5. 启动运行
### 安装部署指南
> 基于python3.8环境安装

#### step1：Conda安装与配置
```bash
# step1: 下载安装包(有网：使用wget下载，无网时：下载离线安装包上传至安装服务器)
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2021.05-Linux-x86_64.sh

# step2: 按提醒输入安装指令
bash Anaconda3-2021.05-Linux-x86_64.sh

# step3:初始化conda
source ~/.bashrc
```
#### Step2： Nodejs安装与配置
> Note: 若当前服务器是Arch架构的，则建议使用[华为官方文档](https://support.huaweicloud.com/prtg-kunpengweb/nodejs_01_0001.html)给出的版本进行安装，否则会出现未知异常，

```bash
# step1: 下载
wget https://nodejs.org/dist/v16.15.1/node-v16.15.1-linux-x64.tar.xz

# step2: 解压
tar -xvf node-v16.15.1-linux-x64.tar.xz

# step3: 重命名
mv node-v16.15.1-linux-x64 nodejs

# step4: 创建软连接（或者添加环境变量）
ln -s /data/bluewhale/software/nodejs/bin/node  /usr/local/bin/node
ln -s /data/bluewhale/software/nodejs/bin/npm  /usr/local/bin/npm

# step5: 验证
(jupyterhub_py38env) [root@2-123 jupyterhub]# node -v
v16.15.1

# step6: 安装configurable-http-proxy
npm install -g configurable-http-proxy

# step7: 环境变量中如果配置了node的路径,这边可以忽略
c.JupyterHub.proxy_cmd = ['/opt/nodejs/bin/configurable-http-proxy',]
```
#### Step3: Jupyterhub安装配置
> Note：若在内网条件下安装，则需要在在有外网下构建好环境包，然后使用conda-pack进行打包迁移

- 安装
```bash
# step1: 创建虚拟环境
conda create -n jupyterhub_py38env python=3.8 -y

# step2:激活环境
conda activate jupyterhub_py38env

# step3: 安装jupyterhub及其依赖
pip install ipykernel notebook jupyterlab jupyterhub jupyterhub-dummyauthenticator mysql-connector -i https://pypi.tuna.tsinghua.edu.cn/simple

# step4: 安装kernal
conda install nb_conda_kernels ipykernel 

# 查看内核清单
jupyter kernelspec list

# step5：安装常用插件
jupyter labextension install @jupyterlab/hub-extension
```

- 生成配置
```bash
# 生成配置文件
jupyterhub --generate-config -f /etc/jupyterhub/jupyterhub_config.py

# 修改配置文件的权限，本质只授权可执行文件即可
chmod 777 jupyterhub_config.py
```

- 修改配置
```bash
c.JupyterHub.ip = '172.x.x.x'
c.JupyterHub.port = 30020
c.Spawner.ip = '172.x.x.x'
c.JupyterHub.base_url = '/jupyterhub'
c.PAMAuthenticator.encoding = 'utf8'
c.LocalAuthenticator.create_system_users = True
c.Authenticator.admin_users = {"demouser"}
c.Authenticator.whitelist = {'demouser'}
c.Spawner.notebook_dir = '/data/bluewhale/jupyterhubuser/' 
#c.JupyterHub.authenticator_class = 'remote_user_authenticator.RemoteUserAuthenticator'
```
#### Step4: 添加Linux系统用户

1. 在Linux Shell中执行
```bash
# 添加用户
useradd demouser

# 修改密码
passwd demouer
# 按照提示输入新密码（2次）
```

2. 修改jupyterhub配置文件
```bash
vim /etc/jupyterhub_config.py

#配置
c.Authenticator.whitelist = {"root", "demouser"}
```
#### Step5: 启动运行
```bash
# 前台启动（在虚拟环境中）
jupyterhub --config=/etc/jupyterhub/jupyterhub_config.py --no-ssl

# 后台启动
nohup jupyterhub --config=/etc/jupyterhub/jupyterhub_config.py --no-ssl > /dev/null 2>&1 &
```
#### Step6: 环境导出(非必须)
> 若需要将该jupyterhub环境包导入到其他环境，可执行下列操作

```bash
conda install -c conda-forge conda-pack

conda pack -n jupyterhub_py38env -o jupyterhub_py38env.tar.gz
```
### 案例：Arch系统下的详细安装流程
#### step1:安装conda
> [镜像链接](https://repo.anaconda.com/miniconda/)

```bash
cd /opt/
wget http://repo.continuum.io/miniconda/Miniconda3-latest-Linux-armv7l.sh
bash Miniconda3-latest-Linux-armv7l.sh

# 执行过程中留意：
When asked, change the install location: /home/pi/miniconda3  # 输入：/opt/miniconda
Do you wish the installer to prepend the Miniconda3 install location to PATH in your /root/.bashrc ? yes

# 激活conda
source /root/.bashrc

# 确认是否安装成功
conda
```
#### step2 创建虚拟环境
```bash
conda create -n jupyter python=3.7 -y

# 补充：
# 激活虚拟环境： 
conda activate jupyter
# 退出虚拟环境
conda deactivate 
```
#### step3: 安装nodejs
> 此处建议使用华为官方文档给出的版本进行安装：
> [https://support.huaweicloud.com/prtg-kunpengweb/nodejs_01_0001.html](https://support.huaweicloud.com/prtg-kunpengweb/nodejs_01_0001.html)

```bash
# 下载并安装
wget https://nodejs.org/dist/v10.16.0/node-v10.16.0-linux-arm64.tar.xz
tar -xvf node-v10.16.0-linux-arm64.tar.xz
ln -s /opt/nodejs/bin/node /usr/local/bin/node
ln -s /root/nodejs/bin/npm /usr/local/bin/npm

# 验证
./opt/nodejs/bin/node -v
```
#### step4: 安装configurable-http-proxy
```bash
npm install -g configurable-http-proxy # 安装代理工具
```
#### step5: 激活虚拟环境安装jupyterhub相关依赖包
```bash
source activate jupyter
pip install ipykernel notebook jupyterlab jupyterhub jupyterhub-dummyauthenticator mysql-connector
```
#### step6： 安装python开发依赖包
```bash
pip install -U scikit-learn pyspark 
```

- 备注：安装过程进行step4时可能出现的问题

      - 错误1：如果安装包下载速度太慢，且是非arch架构的系统环境，则可以将conda的下载源修改为国内的镜像源，设置方法如下
```bash
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --set show_channel_urls: yes
```

      - 错误二：
         - 报错信息如下：
```bash
Could not find a version that satisfies the requirement cffi>=1.0 (from versions: )
No matching distribution found for cffi>=1.0
You are using pip version 10.0.1, however version 21.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

```bash
 - 解决办法：升级pip版本python -m pip install --upgrade pip
```
#### setp7:生成配置文件并添加内容如下：
```bash
mkdir -p /data/jupyter && cd /data/jupyter

jupyterhub --generate-config

chmod 777 jupyterhub_config.py

vim jupyterhub_config.py
```
```bash
c.JupyterHub.base_url = '/jupyterhub'
c.JupyterHub.ip = "0.0.0.0"
c.Spawner.default_url = '/lab'
c.PAMAuthenticator.open_sessions = False
c.LocalAuthenticator.create_system_users = True
c.Authenticator.admin_users = {'hcdsj'}  # #管理员用户
c.JupyterHub.admin_access = True
c.JupyterHub.port = 8888
c.Spawner.cmd = ['jupyterhub-singleuser']
c.JupyterHub.proxy_cmd = ['/opt/nodejs/bin/configurable-http-proxy',]
c.PAMAuthenticator.encoding = 'utf8'
c.Authenticator.whitelist = {'hcdsj1','hcdsj2','hcdsj3'}  # 白名单
c.JupyterHub.statsd_prefix = 'jupyterhub'
c.Authenticator.delete_invalid_users = True

# 解决iframe权限问题
c.JupyterHub.statsd_prefix = 'jupyterhub'
#c.Spawner.cmd=['jupyterhub-singleuser']
c.Spawner.cmd=['jupyter-labhub']
c.JupyterHub.tornado_settings = {
     "headers": {
        "Content-Security-Policy": "frame-ancestors http://192.168.0.111:7080 http://192.168.0.111:9999/ http://192.168.0.111:9529 http://124.71.144.188:7080 http://124.71.144.188:9999/ http://124.71.144.188:9529; report-uri /api/security/csp-report"
    }
}

```
#### step8: 使用系统用户登录jupyterhub

- 添加用户并修改密码
```bash
# 查看用户
cat /etc/passwd

adduser hcdsj hcdsj1 hcdsj2 hcdsj3

useradd hcdsj -G test01  # 该用户在jupyterhub_config.py文件中添加

# 查看是否添加成功
cat /etc/group

# 注意使用下列命令修改
passwd hcdsj  # 此处密码设置为用户名(首字母大写)加123， 如：Hcdsj123
# 输出新密码即可
"""
(jupyter) [root@ecs-c589-0002 jupyter]# sudo passwd wss
更改用户 wss 的密码 。
新的 密码：
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
"""
```
#### step9 启动
> 在虚拟环境中执行下列命令

```bash
# 前台启动
jupyterhub -f /data/jupyter/jupyterhub_config.py

# 后台启动
nohup jupyterhub -f /data/jupyter/jupyterhub_config.py --no-ssl &
```
### 扩展（非必须）
#### 安装主题扩展
```bash
pip install jupyterthemes 

#列出主题
jt -l 

#应用monokai主题
jt -t monokai -T -N -altp -fs 13 -nfs 13 -tfs 13 -ofs 13
```
#### 安装sklearn环境
```bash
 pip install -U scikit-learn  -i https://pypi.tuna.tsinghua.edu.cn/simple
```
#### 安装scala kernel
```bash
# 下载
wget https://oss.sonatype.org/content/repositories/snapshots/com/github/alexarchambault/jupyter/jupyter-scala-cli_2.10.5/0.2.0-SNAPSHOT/jupyter-scala_2.10.5-0.2.0-SNAPSHOT.tar.xz

# 安装jupyter-scala
tar -xvf jupyter-scala_2.10.5-0.2.0-SNAPSHOT.tar.xz -C /usr/local/
bash /usr/local/jupyter-scala_2.10.5-0.2.0-SNAPSHOT/bin/jupyter-scala
```
#### 安装pyspark kernel
```bash
# 安装findspark，实现在jupyter中可以获取Spark Context环境
pip install pyspark  findspark -i https://pypi.tuna.tsinghua.edu.cn/simple

mkdir ~/.ipython/kernels/pyspark
vim ~/.ipython/kernels/pyspark/kernel.json

# kernel.json中写入
{
 "display_name": "pySpark",
 "language": "python",
 "argv": [
  "/usr/local/anacond/bin/python3",
  "-m",
  "IPython.kernel",
  "-f",
  "{connection_file}"
 ],
 "env": {
  "SPARK_HOME": "/usr/local/spark",
  "PYTHONPATH": "/usr/local/spark/python:/usr/local/spark/python/lib/py4j-0.10.4-src.zip",
  "PYTHONSTARTUP": "/usr/local/spark/python/pyspark/shell.py ",
  "PYSPARK_SUBMIT_ARGS": "pyspark-shell"
 }
}
```
#### 安装R kernel
```bash
# 使用conda创建一个R语言的环境，命名为R
conda create -n R r-essentials

# 激活环境
conda activate R

# 进入到R的Session中
R

# 依次执行以下命令安装R kernekl
> install.packages(c('pbdZMQ', 'repr', 'devtools')) 
> devtools::install_github('IRkernel/IRkernel') 
> IRkernel::installspec()

# 安装完成后会在目录/root/.local/share/jupyter/kernels/ir生成一份配置信息。手工拷贝到jupyterlab默认读取的位置/usr/local/share/jupyter/kernels
# 刷新页面就可以看到R的kernel了！

cp -r /root/.local/share/jupyter/kernels/ir /usr/local/share/jupyter/kernels
```
