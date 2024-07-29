---
title: git配置
date: 2022-06-08 08:53:38
tags: 
  - Git
---
## 配置文件说明：

- /etc/gitconfig 文件：
  - 系统中对所有用户都普遍适用的配置。
  - 若使用 git config 时用 --system 选项，读写的就是这个文件。
- ~/.gitconfig 文件：
  - 用户目录下的配置文件只适用于该用户。
  - 若使用 git config 时用 --global 选项，读写的就是这个文件。
- .git/config
  - 当前项目的 Git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。
  - 每一个级别的配置都会覆盖上层的相同配置，所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。

## 配置示例

```bash
git config --local http.postBuffer 524288000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git config --global user.name "hl"
git config --global core.editor vim
git config --global merge.tool vimdiff
git config --list

```

## 配置密码：

```bash
#设置记住密码（默认15分钟）：
git config --global credential.helper cache

#永久
git config --global credential.helper store

#如果想自己设置时间，可以这样做：
git config credential.helper 'cache --timeout=3600'
```



## git设置代理

- 设置

```bash
git config --global http.proxy http://127.0.0.1:4780
git config --global https.proxy https://127.0.0.1:4780
```

- 取消设置

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```



## git 配置ssh

> centos系统

1. 配置帐户和密码

```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

2. 查看配置是否生效

```bash
git config --list
```

3. 在本地生成ssh key，生成公钥和私钥

```bash
ssh-keygen -t rsa -C "your_email@youremail.com"
```

4. 确认路径按1次回车，提示：Enter passphrase（输入密码），不用输密码再按2次回车即可。生成的密钥存放路径 /root/.ssh/id_rsa, id_rsa：私钥 ，id_rsa.pub：公钥

```bash
cat /root/.ssh/id_rsa.pub
```

5. gitlab上setting中的 New SSH Key添加一个新的ssh key(复制刚才生成的id_rsa.pub公钥)