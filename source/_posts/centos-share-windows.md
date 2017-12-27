---
title: CentOS访问Windows共享文件夹
date: 2016-10-21 15:11:00
categories: centos
tags: [centos]
---

**1 在地址栏中输入下面内容:**

smb://Windows IP/Share folder name，smb为Server Message Block协议的简称，是一种IBM协议，运行在TCP/IP协议之上。

从Windows 95开始，Microsoft Windows都提供了Server和Client的SMB协议支持，Microsoft为Internet提供了SMB开源版本，及CIFS(Common Internet File System)，通用文件系统。

**2 将Windows的共享文件夹挂载到本地**

在终端中输入命令 
> cd /
>
> mkdir myShare
>
> mount -t cifs -o username="administrator",password="123456" //192.168.1.1/ShareFolder /myShare

注意命令行中的空格和逗号，空密码也可以。

此命令就是将192.168.1.1上的共享文件夹ShareFolder 挂载到本地的/mnt/MyShare文件夹，执行完，就可在MyShare里看到ShareFolder里的内容。

删除挂载用命令：
> umount /myShare

**3 开机自动挂载**
> vi /etc/fstab
> 
> //192.168.1.1/ShareFolder    /myShare    cifs    defaults,username=administrator,password=123456    0 2

保存退出

**4 创建软链接**
> ln -s /myShare/source /data/dist