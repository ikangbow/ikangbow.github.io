---
title: win10学linux
date: 2020-02-16 10:36:20
tags: 运维
category: java
---

# Linux环境

## win10家庭版自带虚拟机Hyper-V的设置

一般win10家庭版并不带Hyper-V虚拟机，解决方法：

新建Hyper-V.bat文件，内容为

    pushd "%~dp0"

    dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt

    for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"

    del hyper-v.txt

    Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL

右键-以管理员身份运行，这时就会自动的安装虚拟机功能。

## 下载linux镜像

下载linux镜像，运行Hyper-V管理器，并加载linux镜像