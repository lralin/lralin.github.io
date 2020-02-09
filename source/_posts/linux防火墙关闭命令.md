---
title: 防火墙关闭命令
date: 2020-08-25 17:55:25
updated: 2019-08-25 17:55:25
tags: 
- shell
- linux

---

1. 查看防火状态
```
systemctl status firewalld

service  iptables status
```
2. 暂时关闭防火墙
```
systemctl stop firewalld

service  iptables stop
```
3. 永久关闭防火墙
```
systemctl disable firewalld

chkconfig iptables off
```
4. 重启防火墙
```
systemctl enable firewalld

service iptables restart  
```
5. 永久关闭防火墙
```
chkconfig iptables on
```