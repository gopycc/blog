---
title: .NET 4.5+项目迁移.NET Core的问题记录
date: 2016-09-23 14:45:00
categories: dotnet
tags: [netcore]
---

这几天试着把目前的开发框架迁移到新的.net core平台，中间遇到的问题在这里简单记录一下。

迁移过程遇到的最大的问题IOC容器。我目前使用的IOC容器Castle Windsor还没有.net core版本的实现，虽然core本身提供有注入功能，但我想在代码上尽量保持与.NET Framework的兼容，最后还是选择使用第三方容器Autofac，不过在容器上层做了隔离，也就是可以随时替换掉IOC。
关于第三方容器接管.net core的注入实现，官方文档有介绍，也可以参考autofac的实现
https://github.com/autofac/Autofac.Extensions.DependencyInjection

第二个问题是.net core没有App Domain，不能像以前那样方便的加载项目程序集，替换方法是手工扫描BIN目录，再通过AssemblyLoadContext加载到内存。不过遇到一个问题是在加载入口项目DLL的时候，会提示无法加载，目前我是通过Assembly.GetEntryAssembly()后简单粗暴的排除掉。哪位朋友知道有更好的方法，劳烦告知一下。

> 其他方式:
> var assemblies = DependencyContext.Default.RuntimeLibraries;//需要引用程序集
> ...

第三个问题是很多Type的反射接口不存在了，可以通过Type.GetTypeInfo()获取，不过要额外引用扩展库。

第四个问题是在发布到服务器IIS之后，出现下面的异常：

```
HTTP Error 502.5 - Process Failure

Common causes of this issue:
The application process failed to start.
The application process started but then stopped.
The application process started but failed to listen on the configured port.
```

WINDOWS系统日志：

```
Failed to start process with commandline '"dotnet" .\Portal.dll', ErrorCode = '0x80070002'.
```

首先说一下，在部署IIS的时候，需要在windows server上安装文件DotNetCore.1.0.1-WindowsHosting.exe，它会在IIS上添加一个aspnetcore module，托管net core的运行。新建站点后将应用程序池修改为无托管模式即可。就在这个地方我遇到上边的错误，页面一直提示502无法打开，端口和编译平台都没有问题。
网上找了很多方法试过后都没有效果，最后感觉还是hosting的问题，而且我在安排hosting文件的时候的确报过一次错，后来是用右键管理员运行安装完成的，然后到iis模块下果真没有找到aspnetcore module。
卸载后重装又出现了第一次的问题，安装失败。最后在stackoverflow上看到一句话

```
Make sure when you install the DotNetCore WindowsHosting you have access to the internet because the installer download the VS 2015 resist x64 as dependency.

VS 2015 resist x64 - http://download.microsoft.com/download/8/c/b/8cb4af84-165e-4b36-978d-e867e07fc707/vc_redist.x64.exe
```

果断下载安装后再运行WindowsHosting就没再报错了，这里还需要重启一下服务器。

https://aka.ms/dotnetcore_windowshosting_1_1_0

手指在键盘上飞快的敲下那一串域名，点击Enter，那一刻的感觉好极了@@