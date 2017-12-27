---
title: CentOS网络配置
date: 2016-10-19 10:25:00
categories: centos
tags: [centos,network]
---

修改配置文件
```
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
静态网络配置
```
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.0.109
PREFIX=24 //子网掩码:255.255.255.0
GATEWAY=192.168.0.1
DNS1=192.168.0.1
BROADCAST=192.168.0.255
DEVICE=eth0
```
然后ESC退出编辑模式

保存退出
```
:wq!
```
或放弃修改
```
:q!
```
重启网络服务
```
service network restart
```
查看IP地址
```
ip addr
```