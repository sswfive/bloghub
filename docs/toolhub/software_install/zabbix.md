---
title: Zabbix常用配置
date: 2021-08-09 23:26:08
tags:
  - Installation
---
### 验证zabbix-agent2的联通性
```bash
1.在服务端上通过命令，主动获取数据
 yum install zabbix-get -y  #yum安装zabbix采集服务
 
2.用命令监测服务端与客户端的连接
#获取ping
zabbix_get -s '17.16.1.190' -p 10050 -k 'agent.ping'
#获取主机名
zabbix_get -s '17.16.1.190' -p 10050 -k 'system.hostname'
```
### 解决图形字体乱码问题
```bash
1.安装字体
	yum -y install wqy-microhei-fonts

2.复制字体
\cp /usr/share/fonts/wqymicrohei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf

#失败了，使用下面博客的方法
 https://blog.csdn.net/sehn_/article/details/107455885
```
```bash
# 确认zabbix-agent2服务正确运行
systemctl is-active zabbix-agent2.service

systemctl restart zabbix-agent2

grep -Ev '^#|^$' /etc/zabbix/zabbix_agent2.conf   #过滤，去掉#和空格的行


1.服务端
tail -f /var/log/zabbix/zabbix_server.log

2.客户端
tail -f /var/log/zabbix/zabbix_agent2.log


zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

# 查看监控端口
netstat -lntup

```
