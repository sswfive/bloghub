---
title: git同时配置多个平台的SSH
date: 2024-04-14 22:42:58
tags:
  - Git
---
# 配置同时使用 Gitlab、Github、Gitee(码云) 共存的开发环境
> reference: [https://github.com/mr-dragon/day-day-up/blob/master/z-archive/document/configure-gitlab-github-gitee-development-environment.md](https://github.com/mr-dragon/day-day-up/blob/master/z-archive/document/configure-gitlab-github-gitee-development-environment.md)


## 前置准备：

- 确保 git 已经安装
## 1、清除 git 的全局设置

- 确认当前环境是否已经配置了 git 账号信息，若有则先清除
```bash
git config --global --list

git config --global --unset user.name "你的名字"
git config --global --unset user.email "你的邮箱"

git config --global --unset user.name "sswang"
git config --global --unset user.email "sswang@jshcbd.cn"

```
## 2、生成新的 SSH KEY
### Linux & MacOS  环境
```bash
# github ssh keys
# linux or mac
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "sswss@aliyun.com"

# gitee
# linux or mac
ssh-keygen -t rsa -f ~/.ssh/id_rsa.gitee -C "sswss@aliyun.com"

# gitlab 
# linux or mac
ssh-keygen -t rsa -f ~/.ssh/id_rsa.gitlab -C "sswang@jshcbd.cn"
```

- 完成后会在~/.ssh / 目录下生成以下文件
```bash
id_rsa.github
id_rsa.github.pub
id_rsa.gitlab
id_rsa.gitlab.pub
id_rsa.gitee
id_rsa.gitee.pub
```
### win
```bash
# github ssh keys
ssh-keygen -t rsa -f C:\Users\jhc142.JSHCBD\.ssh\id_rsa.github -C "sswss@aliyun.com"

# gitee
ssh-keygen -t rsa -f C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitee -C "sswss@aliyun.com"

# gitlab 
ssh-keygen -t rsa -f C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitlab -C "sswang@jshcbd.cn"
```

- 完成后会在C:\Users\jhc142.JSHCBD\.ssh目录下生成以下文件
```bash
id_rsa.github
id_rsa.github.pub
id_rsa.gitlab
id_rsa.gitlab.pub
id_rsa.gitee
id_rsa.gitee.pub
```

```bash
ssh-agent bash
ssh-add C:\Users\jhc142.JSHCBD\.ssh\id_rsa.github
ssh-add C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitlab
ssh-add C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitee
```

## 3、添加识别 SSH keys 新的私钥

- Linux & MacOS
```bash
ssh-agent bash
ssh-add ~/.ssh/id_rsa.github
ssh-add ~/.ssh/id_rsa.gitlab
ssh-add ~/.ssh/id_rsa.gitee
```

- win
   - **需要确认开启 ssh-agent**
> 开启方法：
> Fix Error **unable to start ssh-agent service, error: 1058(xxxx)**
> 1. win + R, Go To services.msc
> 2. Find And Check Is OpenSSH Authentication Agent Service Running

```bash
 ssh-agent bash
ssh-add C:\Users\jhc142.JSHCBD\.ssh\id_rsa.github
ssh-add C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitlab
ssh-add C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitee
```

在.ssh 目录下创建 config 文件

- Linux & MacOS
```bash
#Default gitHub user Self
Host github.com
    HostName github.com
    User git #默认就是git，可以不写
    IdentityFile ~/.ssh/id_rsa.github

#Add gitLab user 
    Host git@gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_rsa.gitlab

# gitee
Host gitee.com
    Port 22
    HostName gitee.com
    User git
    IdentityFile ~/.ssh/id_rsa.gitee

```

- win
```bash
#Default gitHub user Self
Host github.com
    HostName github.com
    User git #默认就是git，可以不写
    IdentityFile C:\Users\jhc142.JSHCBD\.ssh\id_rsa.github

#Add gitLab user 
    Host git@gitlab.com
    HostName gitlab.com
    User git
    IdentityFile C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitlab

# gitee
Host gitee.com
    Port 22
    HostName gitee.com
    User git
    IdentityFile C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitlab

```
```bash
#Default gitHub user Self
Host github.com
    HostName github.com
    User git #默认就是git，可以不写
    IdentityFile C:\Users\jhc142.JSHCBD\.ssh\id_rsa.github

#Add gitLab user 
    Host jshcbd
    HostName git.jshcbd.com.cn
    User git
    IdentityFile C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitlab

# gitee
Host gitee.com
    Port 22
    HostName gitee.com
    User git
    IdentityFile C:\Users\jhc142.JSHCBD\.ssh\id_rsa.gitlab
```

下面对上述配置文件中使用到的配置字段信息进行简单解释：

- Host 它涵盖了下面一个段的配置，我们可以通过他来替代将要连接的服务器地址。 这里可以使用任意字段或通配符。 当ssh的时候如果服务器地址能匹配上这里Host指定的值，则Host下面指定的HostName将被作为最终的服务器地址使用，并且将使用该Host字段下面配置的所有自定义配置来覆盖默认的/etc/ssh/ssh_config配置信息。
- Port 自定义的端口。默认为22，可不配置
- User 自定义的用户名，默认为git，可不配置
- HostName 真正连接的服务器地址
- PreferredAuthentications 指定优先使用哪种方式验证，支持密码和秘钥验证方式
- IdentityFile 指定本次连接使用的密钥文件

## 4、在 github 、gitlab、gitee 网站添加 ssh
### Github

- 可以用cat命令把 公钥 的内容打出来复制
```bash
# 进入.ssh 目录下
cat id_rsa.github
```

- 直达地址：[https://github.com/settings/keys](https://github.com/settings/keys)
- 过程如下：
   1. 登录 Github
   2. 点击右上方的头像，点击 settings
   3. 选择 SSH key
   4. 点击 Add SSH key
### Gitlab

- 注：此处的 gitlab 是公司的私有库

直达地址：[https://gitlab.com/profile/keys](https://gitlab.com/profile/keys)

1. 登录 Gitlab
2. 点击右上方的头像，点击 settings
3. 后续步骤如 Github
### Gitee / 码云
直达地址：[https://gitee.com/profile/sshkeys](https://gitee.com/profile/sshkeys)

1. 登录 Gitee
2. 点击右上方的头像，点击 设置
3. 后续步骤如 Github

添加过程 码云 会提示你输入一次你的 Gitee 密码 ，确认后即添加完毕。
## 5、 测试是否连接成功
由于每个托管商的仓库都有唯一的后缀，比如 Github 的是 [git@github.com](mailto:git@github.com):*。
所以可以这样测试： ssh -T [git@github.com](mailto:git@github.com)
而 gitlab 的可以这样测试： ssh -T [git@gitlab.corp.xyz.com](mailto:git@gitlab.corp.xyz.com) 如果能看到一些 Welcome 信息，说明就是 OK 的了

- ssh -T [git@github.com](mailto:git@github.com)
- ssh -T [git@gitlab.com](mailto:git@gitlab.com)
- ssh -T [git@gitee.com](mailto:git@gitee.com)
```bash
(base) PS C:\Users\jhc142.JSHCBD\.ssh> ssh -T git@git.jshcbd.com.cn
Welcome to GitLab, @sswang!
(base) PS C:\Users\jhc142.JSHCBD\.ssh> ssh -T git@gitee.com
Hi sswfive(@sswfive)! You've successfully authenticated, but GITEE.COM does not provide shell access.
(base) PS C:\Users\jhc142.JSHCBD\.ssh> ssh -T git@github.com
Hi sswfive! You've successfully authenticated, but GitHub does not provide shell access.
```
