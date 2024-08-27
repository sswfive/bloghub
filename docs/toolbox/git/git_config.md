---
title: git实用配置
date: 2023-03-08 08:53:38
tags: 
  - Git
  - Tools
---
## 配置文件说明：



- **/etc/gitconfig 文件：**  
  - 系统中对所有用户都普遍适用的配置。
  - 若使用 git config 时用 --system 选项，读写的就是这个文件。
- **~/.gitconfig 文件：**
  - 用户目录下的配置文件只适用于该用户。
  - 若使用 git config 时用 --global 选项，读写的就是这个文件。
- **.git/config**
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



## git访问加速配置

#### 方式一：使用github Mirror 下载

> 直接在 GitHub 仓库前面拼接 Proxy 地址，不同的 Mirror 拼接方式可能有所不同

- [https://mirror.ghproxy.com](https://mirror.ghproxy.com)

```bash
$ git clone https://mirror.ghproxy.com/https://github.com/shaowenchen/scripts
```

- [https://github.com.cnpmjs.org](https://github.com.cnpmjs.org)

```bash
git clone https://github.com.cnpmjs.org/shaowenchen/scripts 
```

---

#### 方式二：通过gitee 导入github项目

> 可以参考文档: GitHub仓库快速导入Gitee及同步更新, 将 GitHub 仓库导入 Gitee。然后使用 Gitee 的地址拉取代码。

文档链接：[https://gitee.com/help/articles/4284](https://gitee.com/help/articles/4284)

---

#### 方式三：配置github host地址

打开 [https://www.ipaddress.com/](https://www.ipaddress.com/) 查询 github.com 的 IP 地址
编辑本地 /etc/hosts 文件，添加如下内容:

```bash
140.82.112.4 github.com
```

或者直接使用开源项目 GitHub520 获取最新的 IP 地址。

> 项目地址：[https://github.com/521xueweihan/GitHub520](https://github.com/521xueweihan/GitHub520)

接着就可以拉取代码了，但是速度并不会很快，因为 Github 用的是美国 IP。

---

#### 方式四：配置命令行代理

> 如果有可用的代理服务，那么在本地 Terminal 中配置代理即可。

```bash
# Proxy
function proxy_off(){
    unset http_proxy
    unset HTTP_PROXY
    unset https_proxy
    unset HTTPS_PROXY
    echo -e "已关闭代理"
}
function proxy_on(){
    export http_proxy="http://127.0.0.1:1087";
    export HTTP_PROXY="http://127.0.0.1:1087";
    export https_proxy="http://127.0.0.1:1087";
    export HTTPS_PROXY="http://127.0.0.1:1087";
    echo -e "已开启代理"
}
```



## git同时配置多个平台的SSH

> reference: [https://github.com/mr-dragon/day-day-up/blob/master/z-archive/document/configure-gitlab-github-gitee-development-environment.md](https://github.com/mr-dragon/day-day-up/blob/master/z-archive/document/configure-gitlab-github-gitee-development-environment.md)


### 前置准备：

- 确保 git 已经安装

### 1、清除 git 的全局设置

- 确认当前环境是否已经配置了 git 账号信息，若有则先清除

```bash
git config --global --list

git config --global --unset user.name "你的名字"
git config --global --unset user.email "你的邮箱"

git config --global --unset user.name "sswang"
git config --global --unset user.email "sswang@jshcbd.cn"

```

### 2、生成新的 SSH KEY

- Linux & MacOS  环境

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

- win

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

### 3、添加识别 SSH keys 新的私钥

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
>
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

### 4、在 github 、gitlab、gitee 网站添加 ssh

#### Github

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

#### Gitlab

- 注：此处的 gitlab 是公司的私有库

直达地址：[https://gitlab.com/profile/keys](https://gitlab.com/profile/keys)

1. 登录 Gitlab
2. 点击右上方的头像，点击 settings
3. 后续步骤如 Github

#### Gitee / 码云

直达地址：[https://gitee.com/profile/sshkeys](https://gitee.com/profile/sshkeys)

1. 登录 Gitee
2. 点击右上方的头像，点击 设置
3. 后续步骤如 Github

添加过程 码云 会提示你输入一次你的 Gitee 密码 ，确认后即添加完毕。

### 5、 测试是否连接成功

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

