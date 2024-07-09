---
title: Jupyterhub离线部署踩坑总结
date: 2022-01-02 22:31:51
tags: 
  - Bug
---

## 事出有因

客户内网服务器部署，只有特定端口8888，只有特定pip安装源

## 迎刃而解：

1. **安装python**
2. **创建虚拟环境virtualenv,最好将安装virtualenv的文件夹的权限调整711以上，否则多用户会用权限问题（此处踩坑较长，除了重新部署以外，修改文件夹的对应可执行权限应该是工作量最小的做法。）**
3. **激活虚拟环境，source /或使用workon**
4. **安装pip依赖包**
5. **下载node源码，进行离线安装**
6. **下载configurable-http-proxy，最佳实践是：在有网的环境中，新建一个空目录，在此目录下执行：npm install configurable-http-proxy ,安装完成以后，将该文件夹的node_modules的内容全部移动服务中的node/lib/node_module下，即可，此处踩坑实践较长**
7. **生成配置并修改配置**

```
c.JupyterHub.base_url = '/jupyterhub'
c.JupyterHub.ip = "10.19.10.2"
c.Spawner.default_url = '/lab'
c.PAMAuthenticator.open_sessions = False
c.LocalAuthenticator.create_system_users = True
c.Authenticator.admin_users = {'zhongtai'}  # #管理员用户
c.JupyterHub.admin_access = True
c.JupyterHub.port = 8888
c.Spawner.cmd = ['jupyterhub-singleuser']

c.PAMAuthenticator.encoding = 'utf8'
c.Authenticator.whitelist = {'hcjupyter','zhongtai'}  # 白名单
c.JupyterHub.statsd_prefix = 'jupyterhub'
#c.LocalProcessSpawner.shell_cmd = ["bash", "-l", "-c"]
c.JupyterHub.proxy_cmd = ['/usr/local/node/bin/configurable-http-proxy',]
c.Authenticator.delete_invalid_users = True


/opt/nodejs/bin/configurable-http-proxy
```

8. **启动命令**

```
# 在虚拟环境中执行
# 前台启动
jupyterhub -f /etc/jupyterhub/config.py

# 后台启动
nohup jupyterhub -f /etc/jupyterhub/config.py --no-ssl & 
```
