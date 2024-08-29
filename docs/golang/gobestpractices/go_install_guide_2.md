---
title: Go环境安装和配置
date: 2022-01-27 20:33:16
tags: 
  - Go
---

## 1.下载地址：

Go官网下载地址：https://golang.org/dl/

Go官方镜像站（推荐）：https://golang.google.cn/dl/



## 2.安装



###　linux环境

```shell
# 下载tar包
# 使用 wget 或在官方下载后上传至服务器
wget https://golang.org/dl/go1.16.2.linux-amd64.tar.gz

# 解压至/usr/local/目录下
tar -C /usr/local -xzf go1.16.2.linux-amd64.tar.gz

# 配置环境变量
vim ~/.bashrc
  
export GOROOT=/root/go
export GOPATH=/root/projects/go
export PATH=$PATH:$GOROOT/bin:$GPPATH/bin

# 编辑保存并退出vim后，执行：
source ~/.bashrc

# 查看go的版本
go version 
```

---



### Mac环境

- 方式一：brew安装

```shell
brew info go 
brew install go
go version
```



- 方式二：下载安装包安装

```shell
#　一般默认安装在：users/local/go

# 配置环境变量vim .zshrc 或其他配置文件（.bash_profile）
export GOROOT=/usr/local/go
export GOPATH=$HOME/workspace/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin/
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct

# 重载配置文件
source ~/.zshrc
```

---



### Window环境

- 下载安装包
- 运行安装包文件
- 配置环境变量PATH
- 重启命令窗口
- go version 测试安装结果

---

## 3:拓展补充

###  环境变量中的重点名词

```shell
GOROOT：Go语言安装根目录的路径，也就是GO语言的安装路径。
GOPATH：若干工作区目录的路径。是我们自己定义的工作空间。
GOBIN：GO程序生成的可执行文件（executable file）的路径。
```

>  Go1.14版本之后，都推荐使用`go mod`模式来管理依赖环境了，也不再强制我们把代码必须写在`GOPATH`下面的src目录了，你可以在你电脑的任意位置编写go代码。（网上有些教程适用于1.11版本之前。）

---



### go的基本命令

```shell
go build main.go 
  编译源码文件和当前目录的代码包
  编译命令源码文件后，创建一个可执行文件
  -o: output 指定编译输出的名称，代替默认的包名。
  -i: install 安装作为目标的依赖关系的包(用于增量编译提速)。
  
go run main.go 可以直接运行 Go 程序。
  用于运行命令源码文件
  只能接受一个命令源码文件以及若干个库源码文件作为文件参数
  -a 强制编译相关代码
  -p [n] 并行编译
  -v 展示被编译的代码包名称
  -work 不删除创建的临时目录  
  
go install
  编译和安装代码包或源码文件
  安装代码包会在工作区/pkg/平台目录生成归档文件
  安装命令源码文件会在 bin目录下生成可执行文件
  
go get
  从远程代码仓库拉取代码包,拉取到 GOPATH 的 src 目录中
  -u 更新已存在的代码包
  用导入路径编译程序时，go build 命令会使用 GOPATH 的设置，在磁盘上搜索这个包。事实上， 这个导入路径代表一个 URL，指向 GitHub 上的代码库。如果路径包含 URL，可以使用 Go 工具链从 DVCS 获取包，并把包的源代码保存在 GOPATH 指向的路径里与 URL 匹配的目录里。这个获取过程 使用 go get 命令完成。go get 将获取任意指定的 URL 的包，或者一个已经导入的包所依赖的其 他包。由于go get的这种递归特性，这个命令会扫描某个包的源码树，获取能找到的所有依赖包。
```

---



### go的目录说明

- 工作区

```shell
src目录:
	存放源码文件,以代码包为组织形式

pkg目录:
	存放源码文件以及安装后的结果文件(归档文件),名称以 .a结尾
	隐藏环境变量 $GOOS _$GOARCH,如 linux_amd64
		GOOS 操作系统
		GOARCH 计算架构

bin目录:
	存放go的可执行文件
	仅仅在没有设置goBin时生效 
```

- 项目目录

```shell
cmd：命令行功能集合。
configs：项目的配置文件集合。
deployments：构建Docker镜像等文件集合。
docs：文档集合。
initializers：初始化文件集合。
logs：存储项目日志文件集合。
pkg：项目辅助功能，比如中间件、插件等。
scripts：脚本文件集合。
src：项目的核心文件，核心的处理环节。
tools：项目辅助工具。
vendor：第三方库文件集合。
main.go：项目函数入口。
Makefile：项目构建命令集合。
```

---



### 开发环境配置和切换

> 导包顺序：先查找gopath/src/这个目录下的包是否有，再找goroot/src/这个目录下的包是否有

#### go 版本低于1.14的配置

- 一定要将代码新建到gopath目录之下的src下

- 需要设置 GO111MODULE=off 

```shell
go env  # 查看go的环境变量
go env -w GO111MODULE=off 
```

---

#### go 版本高于1.14放入配置

- 确认GO111MODULE=on 是否设置
- 使用go modules的方式，在任意目录下新建项目（不一定非要放在gopath/src 或 goroot/src下），都可以运行代码

Note: 如果之前的项目没有使用go modules的方式长创建项目，当下需要改为此模式，则执行下列操作即可：

```
1. 在项目的根路径下新建go.mod 文件
2. 在go.mod文件中写入

module "PackageTest
go 1.16
```

- 总结
    - 能用go modules就用modules ,不用去考虑以前的开发模式**
    - 即使使用了以前的开发模式，也可以自动设置为现在的modules模式

---

### 设置代理加速

```shell
go env -w GOPROXY=https://goproxy.io,direct
go env -w GO111MODULE=on
```

---



### 推荐插件安装

- go fmt
- Goimports



---

