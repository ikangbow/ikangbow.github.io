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




