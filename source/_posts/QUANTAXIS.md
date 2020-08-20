---
title: QUANTAXIS
date: 2020-08-20 21:47:00
tags: 投资
category: 投资
---

# 环境准备

## 安装cmder

1. 官网下载地址
	
	`http://cmder.net/`

	下载好解压包可直接使用

2. 环境变量配置

	在系统属性里面配置环境变量，将Cmder.exe所在文件路径添加至path里

![](1.webp)

win+R,输入cmder,确定，即可运行cmder

3. 配置右键快捷启动

	    // 设置任意地方鼠标右键启动Cmder
		Cmder.exe /REGISTER ALL

![](2.webp)

4. 快捷键

    Tab       自动路径补全
	Ctrl+T    建立新页签
	Ctrl+W    关闭页签
	Ctrl+Tab  切换页签
	Alt+F4    关闭所有页签
	Alt+Shift+1 开启cmd.exe
	Alt+Shift+2 开启powershell.exe
	Alt+Shift+3 开启powershell.exe (系统管理员权限)
	Ctrl+1      快速切换到第1个页签
	Ctrl+n      快速切换到第n个页签( n值无上限)
	Alt + enter 切换到全屏状态
	Ctr+r       历史命令搜索
	Tab         自动路径补全
	Ctrl+T      建立新页签
	Ctrl+W      关闭页签
	Ctrl+Tab    切换页签
	Alt+F4      关闭所有页签
	Alt+Shift+1 开启cmd.exe
	Alt+Shift+2 开启powershell.exe
	Alt+Shift+3 开启powershell.exe (系统管理员权限)
	Ctrl+1      快速切换到第1个页签
	Ctrl+n      快速切换到第n个页签( n值无上限)
	Alt + enter 切换到全屏状态
	Ctr+r       历史命令搜索
	Win+Alt+P   开启工具选项视窗

5. 中文乱码问题

将下面的4行命令添加到cmder/config/aliases文件末尾。

    l=ls --show-control-chars 
	la=ls -aF --show-control-chars 
	ll=ls -alF --show-control-chars 
	ls=ls --show-control-chars -F

## 安装docker桌面版

### 下载

    https://www.docker.com/

下载Docker Desktop

### 安装可能遇到的问题

    Installation failed:one pre-requisite is not fullfilled

提示我们系统版本低，解决办法，伪装成专业版系统。用管理员权限开启运行[cmd]命令开启命令行，输入如下指令

    REG ADD "HKEY_LOCAL_MACHINE\software\Microsoft\Windows NT\CurrentVersion" /v EditionId /T REG_EXPAND_SZ /d Professional /F

再次安装docker可以成功

## 下载QUANTAXIS的docker-compose.yaml文件

    `https://github.com/QUANTAXIS/QUANTAXIS`

如果你是股票方向的 ==>  选择 qa-service 下的docker-compose.yaml

如果你是期货方向的 ==> 选择 qa-service-future 下的docker-compose.yaml

你可以理解 docker的构成类似搭积木的模式,  你需要这个功能的积木, 就选择他放在你的docker-compose.yaml里面

期货方向的yaml 比股票多一个  QACTPBEE的docker-container  [这是用于分发期货的tick行情所需的 股票则无需此积木]

可通过git拉取全部代码到本地，从本地拷贝出需要的dockerfile文件

## docker部署quantaxis

1. 选取一个空间较大的盘，最好不放c盘，新建quantaxis文件夹，将docker-compose.yaml拷贝到quantaxis文件夹
2. cd到quantaxis文件夹，运行如下命令

		docker-compose up -d

意思在后台启动这个docker环境，如果需要控制台打印输出，则把-d去掉

3. 在无报错的情况下，打开浏览器输入localhost:81即可看到

![](3.jpg)

在上方随意点击栏目，都可进入登陆界面，默认密码是quantaxis



