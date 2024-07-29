---
title: Go在不同平台的安装及配置指南
date: 2023-02-08 22:05:18
tags: 
  - Go
---
## 前置说明：
### 当前现状

- 当前Go最新稳定版：go1.19（2022年08月11日）
- 官网：[https://go.dev/](https://go.dev/)
- github地址：[https://github.com/golang/go](https://github.com/golang/go)
- 当前Star：103k
### 下载站点

- Go官方下载站点：[https://go.dev/dl/](https://go.dev/dl/)
- 国内镜像下载站点：
   - [https://golang.google.cn/dl/](https://golang.google.cn/dl/)
   - [https://gomirrors.org/](https://gomirrors.org/)
   - [https://studygolang.com/dl](https://studygolang.com/dl)

### CDN加速代理站点
七牛：Goproxy 中国 [https://goproxy.cn](https://goproxy.cn)
阿里： mirrors.aliyun.com/goproxy/
官方： < 全球 CDN 加速 [https://goproxy.io/>](https://goproxy.io/>)
其他：jfrog 维护 [https://gocenter.io](https://gocenter.io)

## 安装一个版本
### Linux安装
#### 安装方案

1. 从发行版仓库安装（不建议，一般版本较老）
2. 下载二进制包安装（推荐，本手册使用此安装方式）
3. 源码编译安装
#### 安装流程
```bash
# step1: 下载并解压Go Linux安装包
wget -c https://golang.google.cn/dl/go1.19.linux-amd64.tar.gz 

# step2: 将安装包解压到安装目录中
tar -C /usr/local -xvf go1.19.linux-amd64.tar.gz 

# step3:配置环境变量
# vim $HOME/.profile
export PATH=$PATH:/usr/local/go/bin

source ~/.profile

# step4: 验证, 输出版本信息表示安装成功
go version
```
### Mac安装
#### 安装方案

1. 在镜像站点 下载安装包进行图像化界面方式进行安装
2. 在terminal 上下载安装包,使用和linux相同方式进行安装
#### 安装流程

- 方式一：brew安装
```bash
brew info go 
brew install go
go version
```

- 方式二：下载安装包安装
```bash
# step1 下载安装包
wget -c https://golang.google.cn/dl/go1.19.darwin-amd64.pkg

# step2 双击安装包，进行图形化界面安装

# step3 配置环境变量
# vim ~/.zshrc
export PATH=$PATH:/usr/local/go/bin

source ~/.zshrc

# step4: 验证, 输出版本信息表示安装成功
go version

```

### Windows安装
#### 安装方案

1. 下载安装包进行图像化界面安装方式
#### 安装流程

1. 下载

![image](https://jsd.cdn.zzko.cn/gh/sswfive/blog-pic@main/20230208/image.5r4085izwfk0.webp)

2. 使用默认安装即可
> 只需一路点击“继续（next）”就可完成 Go 程序的安装了，安装程序默认会把 Go 安装在 C:\Program Files\Go 下面，当然也可以自己定制你的安装目录。

3. 配置环境变量
> 除了会将 Go 安装到你的系统中之外，Go 安装程序还会自动为你设置好 Go 使用所需的环境变量，包括在用户环境变量中增加 GOPATH，它的值默认为 C:\Users[用户名]\go，在系统变量中也会为 Path 变量增加一个值：C:\Program Files\Go\bin，这样我们就可以在任意路径下使用 Go 了。

4. 验证
```bash
go version
```

## 卸载
```bash
# 删除/usr/local/go 目录

# 删除PATH环境变量 /usr/local/go/bin

# 若通过安装包安装的，直接删除/etc/paths.d/go 即可
```
## 安装多个Go版本
### 适用场景：

- 一个版本用于学习或本地开发，另外一个版本用于生产构建。
- 学习一般可以采用最新版。
- 业务生产开发一般用于最新版本的前一个稳定版
### 安装方案

1. 重新设置PATH环境变量
2. go get 命令
3. go get 命令安装费稳定版本
### 安装流程
方案一安装流程

- 只需要将不同版本的 Go 安装在不同路径下，然后将它们的 Go 二进制文件的所在路径加入到 PATH 环境变量中就可以了
```bash
# 以Linux下 安装1.15.13版本为例

# step1 将指定版本的go安装到go1.15.13目录下
$mkdir /usr/local/go1.15.13
$wget -c https://golang.google.cn/dl/go1.15.13.linux-amd64.tar.gz
$tar -C /usr/local/go1.15.13 -xzf go1.15.13.linux-amd64.tar.gz

# step2 配置PATH，将原来的PATH变量替换为新的安装目录路径。

export PATH=$PATH:/usr/local/go/bin
# 改为
export PATH=$PATH:/usr/local/go1.15.13/go/bin

source ~/.profile

# step3 验证
go version
```

- 对于后续需要在两个版本之间切换，只需要重新按照上述步骤修改PATH变量即可。

方案二安装流程
> 此方法的前提是，系统已经安装了一个go版本了

```bash
# 以Linux下 安装1.15.13版本为例

# step1 将 $ HOME/go/bin 加入到 PATH 环境变量中并生效，即便这个目录当前不存在也没关系：
export PATH=$PATH:/usr/local/go/bin:~/go/bin

# step2 执行下面这个命令安装 Go 1.15.13 版本的下载器：
go get golang.org/dl/go1.15.13
# 这个命令会将名为 Go 1.15.13 的可执行文件安装到 $HOME/go/bin 这个目录下

# step3 再执行下列命令进行安装
go1.15.13 download
>>>log:
Downloaded   0.0% (    16384 / 121120420 bytes) ...
Downloaded   1.8% (  2129904 / 121120420 bytes) ...
Downloaded  84.9% (102792432 / 121120420 bytes) ...
Downloaded 100.0% (121120420 / 121120420 bytes)
Unpacking /root/sdk/go1.15.13/go1.15.13.linux-amd64.tar.gz ...
Success. You may now run 'go1.15.13'

# step4 验证
go1.15.13 version
>>>log:
go version go1.15.13 linux/amd64
# 提示：如果需此版本进行工作，需要带上go版号再加上相关命令进行执行，如：
go1.15.13 env GOROOT
>>>log:
/root/sdk/go1.15.13
```
方案三安装流程
> 非稳定版本：
> - beta 版本
> - 最新的 tip 版本，

```bash
go get golang.org/dl/go1.19beta1@latest
go1.19beta1@latest download

```
## 配置Go
### 常用配置项说明

![image](https://jsd.cdn.zzko.cn/gh/sswfive/blog-pic@main/20230208/image.29jlxdbyam7.webp)


- Linux环境下
```bash
go env

# 设置代理
go env -w GOPROXY=https://goproxy.cn,direct

# 启用 Go Modules 功能
go env -w GO111MODULE=on

# 清除模块缓存
go clean --modcache
```

- Windows环境下
```bash
# 启用 Go Modules 功能
$env:GO111MODULE="on"

# 配置 GOPROXY 环境变量，以下三选一

# 1. 七牛 CDN
$env:GOPROXY="https://goproxy.cn,direct"

# 2. 阿里云
$env:GOPROXY="https://mirrors.aliyun.com/goproxy/,direct"

# 3. 官方
$env:GOPROXY="https://goproxy.io,direct"
```

- 测试
```bash
# 测试
go time go get golang.org/x/tour
```

---

### 低于1.14版本的配置

-  一定要将代码新建到gopath目录之下的src下 
-  需要设置 GO111MODULE=off 
```shell
go env  # 查看go的环境变量
go env -w GO111MODULE=off
```

---

### 高于1.14版本的配置

- 确认GO111MODULE=on 是否设置
- 使用go modules的方式，在任意目录下新建项目（不一定非要放在gopath/src 或 goroot/src下），都可以运行代码

Note: 如果之前的项目没有使用go modules的方式长创建项目，当下需要改为此模式，则执行下列操作即可：
```
1. 在项目的根路径下新建go.mod 文件2. 在go.mod文件中写入module "PackageTestgo 1.16
```

- 总结 
   - 能用go modules就用modules ,不用去考虑以前的开发模式**
   - 即使使用了以前的开发模式，也可以自动设置为现在的modules模式

---

### 推荐插件安装

- go fmt
- Goimports
