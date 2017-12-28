---
title: win10周年更新后程序各种卡死，进程无法结束怎么破？
date: 2016-11-04 15:57:00
categories: windows
tags: [windows]
---

最近THINKPAD T460P更新了WIN10周年版后程序各种卡死，运行一段时间，各种程序就开始崩溃，进程无法结束，最终只能强制关机。

这个BUG微软已经确认了，安装了SSD+HDD双硬盘的WIN10系统容易中招。

查看系统日志出现大片的警告：
事件ID:129，
来源：storahci，
常规：发出了对设备 \Device\RaidPort0 的重置。

故障原因大致是由于WIN10自带的IDE ATA/ATAPI控制器版本太旧，导致存储控制器不兼容，无法识别硬盘。

解决办法：安装intel最新的英特尔®快速存储技术 (英特尔® RST) RAID 驱动程序。已安装确认可行。

下载：https://downloadcenter.intel.com/zh-cn/download/26361/Intel-Rapid-Storage-Technology-Intel-RST-RAID-Driver?v=t