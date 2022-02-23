---
title: "Windows 10 安装 WSL2"
date: 2020-08-03T16:51:40+08:00
draft: false
toc: true
images:
tags: 
  - WSL2
---

###   前言

有一些服务想要部署，但是没有那么多的测试环境。用 VirtualBox 的话占用的资源也不少。想起之前看过 WSL 的介绍，说启动一个Linux只需要几十M的内存！话不多说，搞起！

###   启用 WSL

以管理员身份打开 PowerShell ：

``` powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

```

###  更新到 WSL 2（可选）

1. 升级 Windows
Windows 10  Version 2004 以上的版本才能安装 WSL 2。可以运行 winver 查看自己的版本。

	下载 [更新工具](https://go.microsoft.com/fwlink/?LinkId=691209) 进行更新

2. 启用“虚拟机平台”可选组件
``` powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
3. 升级 WSL2 Linux 内核

   要使用 WSL 2 必须先下载 WSL 2 Linux 内核更新包安装即可。[官方教程](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-kernel)   [ x64安装包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)  [arm64安装包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_arm64.msi) 

4. 将 WSL 2 设置为默认版本

``` powershell
wsl --set-default-version 2	
```
### 安装 Linux
打开 Windows Store 搜索 Ubuntu 下载安装即可