---
title: Nginx部署
date: 2023-07-20 22:07:19
tags:
  - SoftwareInstall
---
### 部署说明：
| 事项 | 详情 |
| --- | --- |
| 部署节点 | 192.168.21.71 |
| 安装包位置 | /data/software/nginx-1.22.1.tar.gz |
| 安装目录 | /data/workspace/nginx |
| 配置文件目录 | /data/workspace/nginx/conf/nginx.conf； /data/workspace/nginx/conf/subconf/frontend-wl.conf |

### 部署指南

   - 依赖库安装（安装前检查当前服务器是否有下列库，若有跳过此步骤）
```bash
sudo yum -y install gcc gcc-c++   # 安装 gcc环境,nginx 编译时依赖 gcc 环境 
sudo yum -y install pcre pcre-devel  # 安装 pcre,让nginx支持重写功能
sudo yum -y install zlib zlib-devel # 安装 zlib,zlib 库提供了很多压缩和解压缩的方式，nginx 使用 zlib 对 http 包内容进行 gzip 压缩 
sudo yum -y install openssl openssl-devel # 安装 openssl,安全套接字层密码库，用于通信加密

# 上述四条命令可合并执行
sudo yum -y install gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

   - Nginx安装
      - 暂时未配置开机启动，若服务器重启后需要手动启动nginx
```bash
# 下载安装包
wget http://nginx.org/download/nginx-1.22.1.tar.gz

# step1:进入Nginx安装包路径，执行下述命令进行解压
tar -xzvf nginx-1.22.1.tar.gz -C /data/workspace

mv nginx-1.21.6 nginx 

# step2:执行编译
./configure --prefix=/data/workspace/nginx
make && make install

# step4:打开环境变量文件
vim /etc/profile

# step5: 添加nginx环境变量，并保存退出
export NGINX_HOME= /data/workspace/nginx  ###为nginx安装目录
export PATH=$PATH:$NGINX_HOME/sbin

# step6:重载环境变量文件
source /etc/profile

#step7：启动nginx
nginx

#step8： 查看是否正常启动（此处使用查看nginx进程号的方式）
ps axu | grep nginx
```
### 配置开机启动
```bash
vim /usr/lib/systemd/system/nginx.service

[Unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/data/workspace/nginx/sbin/nginx
ExecReload=/data/workspace/nginx/sbin/nginx -s reload
ExecStop=/data/workspace/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
## 常用命令
```bash
# 设置开机启动
systemctl enable nginx
# 查看状态
systemctl status nginx
# 启动服务
systemctl start nginx
# 停止
systemctl stop nginx
```
