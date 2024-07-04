# Python环境及包管理工具选型

## 原生Python的安装方式
1. 源码安装（适用于：Linux、MacOS）
2. 图形界面安装（适用于：MacOS、Win）
3. Conda安装（适用于：Linux、MacOS、Win）

详细的介绍和安装阅读：[官方文档](https://www.python.org/)
## 多Python版本管理工具
> 一种方案就是对开发环境安装多个原生不同版本的Python。另一种方案就是下述介绍的conda管理工具。

### [Conda](https://docs.conda.io/en/latest/)：☆☆☆☆☆
> 多Python版本管理、多虚拟环境管理

**简述：**

- Conda是一个跨平台（Windows, macOS, and Linux）的开源的环境及包管理平台，官方称不仅限于管理Python版本和环境包的管理，还可以管理_ R, Ruby, Lua, Scala, Java, JavaScript, C/ C++, Fortran_等主流语言，但据笔者了解到，除了python开发者圈应用广泛外，其他语言的开发者几乎很难看到使用的。
- 特点是：可即时按需创建对应python版本的虚拟环境；Conda分为两个版本：[Anaconda](https://www.anaconda.com/)、[Miniconda](https://docs.conda.io/projects/miniconda/en/latest/);这两个工具的安装方式都是比较容易，且易于迁移；
- 相比较于后面的工具而言，占用空间比较大，相对比较重。

**版本说明：**

- [Anaconda](https://www.anaconda.com/):
   - 【重】大而全，除了安装 Conda 内核，还安装了完整的 Anaconda 的发行包，即内置集成了很多工具包，包括python的多个版本，常用机器学习等科学工具包
   - [官方下载](https://www.anaconda.com/download)，[清华源下载](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)；
- [Miniconda](https://docs.conda.io/projects/miniconda/en/latest/):
   - 【轻】小而美，只安装了 Conda 内核，即内置了多个python版本和必要的工具包。
   - [官方下载](https://docs.conda.io/projects/miniconda/en/latest/miniconda-other-installer-links.html)、[清华源下载](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)；

**选型说明：**

- 对于日常在一个环境有多Python版本的开发需求的，且不想在开发环境中安装多个原生Python,Anaconda和Miniconda都是一个很好的选择；
- 对于使用Python进行数据分析、机器学习及AI方向的开发者，Anaconda是一个很好的选择；
- 对于Python web开发来说，Miniconda是一个不错的选择，主要是因为适合web开发的环境包管理工具的选择是很多的。
> tip: 个人日常用的比较多，主要原因对应选型说明的前两点。

## 虚拟环境管理工具
### [Pipenv](https://github.com/pypa/pipenv)：☆☆☆
**简述：**

- 此工具是一个跨平台（Windows, macOS, and Linux）的开源的python虚拟环境管理工具，依赖于开发环境中原生安装的Python版本（目前官方仅支持Python3.7+）其主要特点有：
   - 使用Pipfile和Pipflile.lock文件来维护环境中的包版本
   - 支持pipenv CLI操作

**选型说明：**

- 个人安装官方文档实操过一遍，并未在生产环境中将其引入。
- 在实操过程中给人的体验还是不错的。
### [Poetry](https://github.com/python-poetry/poetry)：☆☆☆
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
### [PDM](https://github.com/pdm-project/pdm)：☆☆☆☆☆
**简述：**

- 此工具是一款跨平台包管理工具，要求python3.7+,主要的功能和上述的Poetry类似，但它摈弃了虚拟环境，直接将包文件夹放在项目根目录下，或放到统一的地方进行集中管理，另外其由一位国内的开发者在业余时间开发而成，主要特点有：
   - 支持最新PEP标准（兼容 [PEP 517](https://www.python.org/dev/peps/pep-0517) 的构建后端，用于构建发布包、[PEP 621](https://www.python.org/dev/peps/pep-0621) 元数据格式）
   - 像pnpm采用了去中心化安装缓存，节省磁盘空间
   - 用户脚本功能比较强大；
   - 使用pyproject.toml和lockfile管理包的版本。

**选型说明：**

- 在知道此工具后，逐渐放弃了poetry，虽然当前star数刚达到5k,但个人看好此工具的后续发展。
- 自定义脚本使用体验非常好。
### [venv](https://docs.python.org/zh-cn/3/library/venv.html#module-venv)：☆☆☆☆
**简述：**

- Python原生安装包中内置的虚拟环境，每个虚拟环境将拥有它们自己独立的安装在其 site,
- 使用起来比较简单

**选型说明：**

- 个人认为在使用docker构建项目是采用改工具进行包管理体验比较好，轻量简单易用。
### [Hatch](https://hatch.pypa.io/)：☆
**简述：**

- Hatch 也是一款虚拟环境管理工具，可以管理环境（它允许每个项目有多个环境，但不允许把它们放在项目目录中），并且可以管理包（但不支持 lockfile）。Hatch 也可以用来打包一个项目（用符合 PEP 621 标准的 pyproject.toml 文件）并上传到 PyPI。

**选型说明：**

- 仅知道此工具，从未实操体验过，是否推荐需要后续实操体验后才能决定。
### [Virtualenv](https://virtualenv.pypa.io/en/latest/)：☆☆
**简述：**

- 依赖于开发环境中的原生Python，仅能创建相同Python版本的不同虚拟环境，在早期的web开发中使用比较多。
- 跟详细的介绍请阅读[官方文档](https://virtualenv.pypa.io/en/latest/)

**选型说明：**

- 在刚从事Python相关学习和工作使用的比较多，近些年几乎不在使用，因为又上述更好的工具取代了它，不过有兴趣的小伙伴可以体验体验。
### [virtualenvwrapper](https://pypi.org/project/virtualenvwrapper)：☆☆
**简述：**

- 依赖于上述的Virtualenv,其本质是一个virtualenv的脚本工具，可以方便的创建和删除虚拟环境。

**选型说明：**

- 在个人早起使用virtualenv,基本都是搭载了virtualenvwrapper工具来进行相关开发工作。



