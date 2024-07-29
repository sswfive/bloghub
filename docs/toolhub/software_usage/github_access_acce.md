---
title: github访问加速配置
date: 2022-06-08 21:23:38
tags: 
  - Git
---

## 方式一：使用github Mirror 下载
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

## 方式二：通过gitee 导入github项目
> 可以参考文档: GitHub仓库快速导入Gitee及同步更新, 将 GitHub 仓库导入 Gitee。然后使用 Gitee 的地址拉取代码。

文档链接：[https://gitee.com/help/articles/4284](https://gitee.com/help/articles/4284)

---

## 方式三：配置github host地址
打开 [https://www.ipaddress.com/](https://www.ipaddress.com/) 查询 github.com 的 IP 地址
编辑本地 /etc/hosts 文件，添加如下内容:
```bash
140.82.112.4 github.com
```
或者直接使用开源项目 GitHub520 获取最新的 IP 地址。
> 项目地址：[https://github.com/521xueweihan/GitHub520](https://github.com/521xueweihan/GitHub520)

接着就可以拉取代码了，但是速度并不会很快，因为 Github 用的是美国 IP。

---

## 方式四：配置命令行代理
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

