---
title: Hyper-v虚拟机文件VHDX与VHD的格式转换
date: 2016-10-19 10:46:00
categories: CI
tags: [hyper-v,CI]
---

今天遇到一个坑，我在本机(windows 10)上创建的CentOS虚拟机作为docker的宿主机，部署了gitlab等容器，准备迁移到服务器上的时候，发现始终无法导入，提示必须通过Hyper-v导出的方式。

实际并不是那么回事。

因为导入不行，我就想着新建个空的虚拟机，然后把现有虚拟文件附加进去总可以吧。

结果我错了，因为这个时候我才发现，原来Windows Server 2008不支持VHDX文件，我一下子懵逼了0_0，总不能让我再从头再来一遍吧。。。

好吧，但我相信程序猿不会如此为难程序猿的，果然在MSDN上发现了MS的虚拟文件转换命令，Windows PowerShell下执行：

```powershell
Convert-VHD –Path c:\test\child1vhdx.vhdx –DestinationPath c:\test\child1vhd.vhd
```

附上MSDN：https://technet.microsoft.com/library/hh848454.aspx