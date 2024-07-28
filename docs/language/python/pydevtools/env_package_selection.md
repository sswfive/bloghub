---
title: Python版本及包管理工具选型
date: 2023-05-01
tags:
  - Python
---
# Python版本及包管理工具选型


## Python多版本管理方案

在日常开发过程，可能会有多个不同Python版本的共存的需求，以下介绍几种关于python多版本共存的解决方案:

### 方案一：原生安装方法

- 从[Python官网](https://www.python.org/downloads/)下载需要安装的Python版本，在不同的目录（建议：目录名使用和版本对应的名称）中安装下载好的安装包，安装后，在环境变量中设置一个常用的Python版本作为默认版本，后续如果需要修改python版本，只需要修改环境变量中python解释器路径即可。
- 详细的介绍和安装阅读：[官方文档](https://www.python.org/)

### 方案二：使用 [conda](https://conda.io/projects/conda/en/latest/user-guide/install/index.html)

- 安装好conda后，通过创建不同python版本的虚拟环境进行多版本的管理，其中conda分为完整版（[Anaconda](https://www.anaconda.com/)）和精简版（[Miniconda](https://docs.conda.io/projects/miniconda/en/latest/)），根据需要选择合适的版本。
  - Anaconda:【重】大而全，除了安装 Conda 内核，还安装了完整的 Anaconda 的发行包，即内置集成了很多工具包，包括python的多个版本，常用机器学习等科学工具包；[官方下载](https://www.anaconda.com/download)，[清华源下载](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)；
  - Miniconda:【轻】小而美，只安装了 Conda 内核，即内置了多个python版本和必要的工具包；[官方下载](https://docs.conda.io/projects/miniconda/en/latest/miniconda-other-installer-links.html)、[清华源下载](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)；

> 选型建议：
>
> - 对于日常在一个环境有多Python版本的开发需求的，且不想在开发环境中安装多个原生Python,Anaconda和Miniconda都是一个很好的选择；
>
> - 对于使用Python进行数据分析、机器学习及AI方向的开发者，Anaconda是一个很好的选择；
>
> - 对于Python web开发来说，Miniconda是一个不错的选择，主要是因为适合web开发的环境包管理工具的选择是很多的。

### 方案三：使用 [pyenv](https://github.com/pyenv/pyenv)

- 安装pyenv后，通过`pyenv install 3.x.x` 的方式进行安装，通过`pyenv global 3.x.x`设置全局python版本方式，详细使用流程[参考文档](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md)
- 在UNIX/MacO系统中,使用pyenv, 在window系统中，使用[`pyenv-win`](https://github.com/pyenv-win/pyenv-win)

### 方案四：使用 [PDM](https://pdm-project.org/en/latest/usage/project/#install-python-interpreters-with-pdm)

- 安装pdm后，通过`pdm python install 3.x.x`方式进行安装，通过`pdm python install --list`查看可安装的版本清单
- 如果只需要python解释器，且已经安装了pdm工具，推荐使用此方式进行安装。

### 方案五：使用 [pythonz](https://github.com/saghul/pythonz)

- 安装好pythonz后，详细使用参考[官方文档](https://bettercallsaghul.com/pythonz/)
- 此工具对于有安装 CPython, Stackless, PyPy and Jython的需求时比较方便。

## 依赖包管理方案
### 方案一：[PDM](https://github.com/pdm-project/pdm)：☆☆☆☆☆

**简述：**

- 此工具是一款跨平台包管理工具，要求python3.7+,主要的功能和上述的Poetry类似，但它摈弃了虚拟环境，直接将包文件夹放在项目根目录下，或放到统一的地方进行集中管理，另外其由一位国内的开发者在业余时间开发而成，主要特点有：
  - 支持最新PEP标准（兼容 [PEP 517](https://www.python.org/dev/peps/pep-0517) 的构建后端，用于构建发布包、[PEP 621](https://www.python.org/dev/peps/pep-0621) 元数据格式）
  - 像pnpm采用了去中心化安装缓存，节省磁盘空间
  - 用户脚本功能比较强大；
  - 使用pyproject.toml和lockfile管理包的版本。

**选型说明：**

- 在知道此工具后，逐渐放弃了poetry，虽然当前star数刚达到5k,但个人看好此工具的后续发展。
- 自定义脚本使用体验非常好。

### 方法二：[Poetry](https://github.com/python-poetry/poetry)：☆☆☆☆

**简述：**

- 此工具是一个跨平台（Windows, macOS, and Linux）、开源的第包管理工具平台，本质还是需要创建虚拟环境进行包管理，官方描述：Python packaging and dependency management made easy；最新版本需要开发环境的python3.8+，其主要特点有：
  - 支持多种安装方式如：官方包安装、pipx安装、pip安装等
  - 取代了 setup.py、requirements.txt、setup.cfg等，仅通过pyproject.toml文件就能轻松机进行包的版本版本和迁移工作
  - 可以对包的安装场景进行分类管理如开发（静态检查相关包）、测试（pytest）、生产环境包的分组
  - 在配置文件中支持自定义Scripts,如：poetry run python xxx;

**选型说明：**

- 在生产环境中此工具被越来越多的项目引入使用，目前star超26k;
- 个人在近两年（PDM发现之前）的web开发中均由在使用，在体验上相当不错，缺点是如果不配置国内源，现在安装效率是极低的，且容易安装出错。
- 用好自定义脚本的功能可大大提升效率。

### 方案三：[Conda](https://docs.conda.io/en/latest/)：☆☆☆

- Conda是一个跨平台（Windows, macOS, and Linux）的开源的环境及包管理平台，官方称不仅限于管理Python版本和环境包的管理，还可以管理 *R, Ruby, Lua, Scala, Java, JavaScript, C/ C++, Fortran*等主流语言，但据笔者了解到，除了python开发者圈应用广泛外，其他语言的开发者几乎很难看到使用的。
  - 特点是：可即时按需创建对应python版本的虚拟环境；Conda分为两个版本：[Anaconda](https://www.anaconda.com/)、[Miniconda](https://docs.conda.io/projects/miniconda/en/latest/);这两个工具的安装方式都是比较容易，且易于迁移；
  - 相比较于后面的工具而言，占用空间比较大，相对比较重。
- 在conda环境中，既支持`conda add <package> `也支持 `pip install <package>`方式进行依赖包安装

### 方案四：[Pipenv](https://github.com/pypa/pipenv)：☆☆☆

**简述：**

- 此工具是一个跨平台（Windows, macOS, and Linux）的开源的python虚拟环境管理工具，依赖于开发环境中原生安装的Python版本（目前官方仅支持Python3.7+）其主要特点有：
   - 使用Pipfile和Pipflile.lock文件来维护环境中的包版本
   - 支持pipenv CLI操作

**选型说明：**

- 个人安装官方文档实操过一遍，并未在生产环境中将其引入。
- 在实操过程中给人的体验还是不错的。

### 方案五：[venv](https://docs.python.org/zh-cn/3/library/venv.html#module-venv)：☆☆☆
**简述：**

- Python原生安装包中内置的虚拟环境，每个虚拟环境将拥有它们自己独立的安装在其 site,
- 使用起来比较简单

**选型说明：**

- 个人认为在使用docker构建项目是采用改工具进行包管理体验比较好，轻量简单易用。

### 方案六：[Virtualenv](https://virtualenv.pypa.io/en/latest/)：☆☆

**简述：**

- 依赖于开发环境中的原生Python，仅能创建相同Python版本的不同虚拟环境，在早期的web开发中使用比较多。
- 跟详细的介绍请阅读[官方文档](https://virtualenv.pypa.io/en/latest/)

**选型说明：**

- 在刚从事Python相关学习和工作使用的比较多，近些年几乎不在使用，因为又上述更好的工具取代了它，不过有兴趣的小伙伴可以体验体验。

### 方案七：[virtualenvwrapper](https://pypi.org/project/virtualenvwrapper)：☆☆

**简述：**

- 依赖于上述的Virtualenv,其本质是一个virtualenv的脚本工具，可以方便的创建和删除虚拟环境。

**选型说明：**

- 在个人早起使用virtualenv,基本都是搭载了virtualenvwrapper工具来进行相关开发工作。

### 方案八：[Hatch](https://hatch.pypa.io/)：☆
**简述：**

- Hatch 也是一款虚拟环境管理工具，可以管理环境（它允许每个项目有多个环境，但不允许把它们放在项目目录中），并且可以管理包（但不支持 lockfile）。Hatch 也可以用来打包一个项目（用符合 PEP 621 标准的 pyproject.toml 文件）并上传到 PyPI。

**选型说明：**

- 仅知道此工具，从未实操体验过，是否推荐需要后续实操体验后才能决定。



### 方案九：[Rye](https://github.com/mitsuhiko/rye)

- 使用Rust开发，暂未研究，值得关注



方案十：[Pyflow](https://github.com/David-OConnor/pyflow) 

- An installation and dependency system for Python,此项目近两年未更新了。



## 常用工具安装步骤

### 【pip】安装及常用命令

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

### 【conda】安装及常用命令

#### Anaconda安装

- Linux/MacOS系统

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

- Win系统

> [安装包下载地址](https://www.anaconda.com/products/distribution)

- 选择自己的平台对应的安装包，双击安装包使用图形化界面进行安装，没有特殊的配置，一般使用默认项按照提示进行安装即可
- 由于笔者没有win系统，故没有场景进行安装验证，在此贴上一个【[安装教程](https://blog.51cto.com/u_13640625/3028327)】链接，仅供参考

#### Miniconda安装

> [安装包下载地址](https://docs.conda.io/en/latest/miniconda.html)

- 安装流程和配置与安装Anaconda类似，只是安装包不一样，再次不在赘述详细步骤

#### 常用命令

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

### 【pyenv】安装及常用命令

- [官网安装说明](https://github.com/pyenv/pyenv#installation)

#### 常用命令

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

#### 版本控制

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

#### virturalenv虚拟环境设置

```bash
# 使用插件 在plugins/pyenv-virtualenv中
pyenv virtualenv 3.8.13 py38env
```

