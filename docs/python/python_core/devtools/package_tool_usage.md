# Python常用包管理工具安装及使用


## 常用国内源
```bash
 # 清华大学 
 https://pypi.tuna.tsinghua.edu.cn/simple/ 
 
 # 阿里云 
 http://mirrors.aliyun.com/pypi/simple/ 

 # 中国科技大学 
 https://pypi.mirrors.ustc.edu.cn/simple/ 

 # 豆瓣(douban) 
 http://pypi.douban.com/simple/ 
```
## 【pip】安装及使用
### 常用命令

- 在python3以后，安装原生python后，默认会安装pip。故此处安装略
```bash
# 常用命令

# 安装 pkgname1和 pkgname2
pip install  <pkgname1>  <pkgname2>

# 查看已经安装的包
pip list

# 查看当前源中能够安装的包及版本
pip search keyword 或者 pypi

# 安装帮助
pip help install

# 查看pip版本
pip -V

# 将已安装的包及版本输出到requirements.txt文件中
pip freeze > requirement.txt

# 安装requirements.txt中全部包（除#注释外）
pip install -r requirement.txt

# 更新pip版本
pip install -U pip  
```
### pip国内源配置
#### 临时指定国内源安装

- 在python3.6后，官方推荐使用 `python -m pip`的方式进行安装相关包x
```bash
python -m pip install -i https://mirrors.aliyun.com/pypi/simple <pkgname>
# pip install -i https://mirrors.aliyun.com/pypi/simple <pkgname>

python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple <pkgname>

python -m pip install -i https://mirrors.ustc.edu.cn/pypi/web/simple <pkgname>
```
#### 永久设置国内源安装

- 下列以清华源设置为例，其他源设置类似
```bash
mkdir -p ~/.pip/
tee > ~/.pip/pip.conf <<EOF
{源名称下面的配置}
EOF

----

# 清华源
[global]
timeout = 6000 
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install] 
trusted-host = pypi.tuna.tsinghua.edu.cn
```
#### 命令行设置默认源
```bash
pip install pip -U -i https://pypi.tuna.tsinghua.edu.cn/simple
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

---

## 【conda】安装及使用
### 安装Anaconda
#### Linux/MacOS系统
> [清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/?C=M&O=D)：

```bash
# step1: 执行安装
bash Anaconda3-2021.11-Linux-x86_64.sh

-----------------------------------------------------------------------------
logger:

Do you accept the license terms? [yes|no]
[no] >>>
Please answer 'yes' or 'no':'
>>> yes

Anaconda3 will now be installed into this location:
/root/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/root/anaconda3] >>> /data/bluewhale/software/anaconda3
PREFIX=/data/bluewhale/

installation finished.
Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
[no] >>> yes

--------------------------------------------------------------------

# step2:配置环境变量
vim /etc/profile

export PATH=$PATH:/data/bluewhale/software/anaconda3/bin

source /etc/profile

conda init
```
#### Win系统
> [安装包下载地址](https://www.anaconda.com/products/distribution)

- 选择自己的平台对应的安装包，双击安装包使用图形化界面进行安装，没有特殊的配置，一般使用默认项按照提示进行安装即可
- 由于笔者没有win系统，故没有场景进行安装验证，在此贴上一个【[安装教程](https://blog.51cto.com/u_13640625/3028327)】链接，仅供参考
### 安装Miniconda
> [安装包下载地址](https://docs.conda.io/en/latest/miniconda.html)

- 安装流程和配置与安装Anaconda类似，只是安装包不一样，再次不在赘述详细步骤
### 常用命令

- 创建、进入、退出、删除
```bash
# 创建环境
conda create -n <env_name> python=3.8

# 查看所有环境
conda env list 
conda info -e

# 激活虚拟环境
conda activate <env_name>

# 退出虚拟环境
conda deactivate 

# 删除虚拟环境
conda remove -n <env_name> --all
```

- 查看、安装、卸载依赖
```bash
# 查看版本
conda -V

# 查看当前环境所有的依赖包(已激活的环境的时)
conda list

# 查看当前环境所有的依赖包（未激活环境时）
conda list -n <env_name>

# 查看指定依赖包
conda list | grep <pkg_name>

# 安装依赖包
conda install <pkg_name>

# 在指定环境中安装依赖包
conda install -n <env_name> <pkg_name>
#或
pip install numpy -i https://pypi.douban.com/simple  # 临时使用国内源进行安装
```

- 升级相关命令
```bash
# 升级conda
conda update conda

# 升级python
conda update python

# 升级指定依赖包
conda update <pkg_name>
```

- 分享、导出、迁移
```bash
# 导出当前环境已安装包列表到yaml文件中
conda env export > environment.yaml

# 从文件中创建环境
conda env create -f environment.yaml

# 克隆虚拟环境
conda create -n <env_name1> --clone <env_name>
```
### conda国内源配置
```bash
# 中科大源
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/

# 阿里源
conda config --add channels https://mirrors.aliyun.com/pypi/simple/

# 豆瓣源
conda config --add channels http://pypi.douban.com/simple/

# 其他配置
## 显示检索路径，每次安装包时会将包源路径显示出来
conda config --set show_channel_urls yes
conda config --set always_yes True

## 显示所有镜像通道路径命令
conda config --show channels

# 恢复默认源
conda config --remove-key channels
```

## 【pyenv】安装及使用
[官网](https://github.com/pyenv/pyenv)
### 安装
[官网安装说明](https://github.com/pyenv/pyenv#installation)
### 常用命令
```bash
pyenv help install

# 列出所有可用版本
pyenv install --list

# 在线安装指定版本
pyenv install 3.8.13
pyenv versions

# 使用缓存方式安装
## 在~/.pyenv 目录下新建cache目录，放入下载好的待下载安装版本的文件
## 不确定要哪一个文件时，把下载的三个文件都放进去
pyenv install 3.8.13 -v
```
### 版本控制
```bash
# 显示当前的python版本
pyenv version 

# 显示所有可用的python版本，和当前版本
pyenv versions 

# global全局设置
pyenv global 3.8.13

# 提示：如果是root用户安装，不建议使用global，否则影响太大

# shell会话
pyenv shell 3.8.13

# local本地设置
## 使用pyenv local 设置
pyenv local 3.8.13
```
### virturalenv虚拟环境设置
```bash
# 使用插件 在plugins/pyenv-virtualenv中
pyenv virtualenv 3.8.13 py38env
```
