---
title: algoplus期货量化（1）
date: 2022-01-14 21:38:26
tags: algoplus
category: algoplus
---

记录学习基于AlgoPlus的量化交易开发准备工作，原文参考[https://www.zhihu.com/column/AlgoPlus](https://www.zhihu.com/column/AlgoPlus)

## centos安装yum

	yum install -y git

	yum install -y vim

	yum -y install wget

	yum install -y bzip2

## 安装anocanda

    wget --no-check-certificate https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2021.11-Linux-x86_64.sh

	bash Anaconda3-2021.11-Linux-x86_64.sh -u

	vim /etc/profile
	
	export PATH=/root/anaconda3/bin:$PATH

	source /etc/profile

## anocanda使用

    conda create -n foralgo python=3.7

	conda env list

	source activate foralgo

## 添加镜像源

    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2


	#显示检索路径

	conda config --set show_channel_urls yes

	#显示镜像通道

	conda config --show channels
	
	#更新pip源

	pip install -i https://pypi.doubanio.com/simple pip -U --user

## 安装、启动mongo
	
    vim /etc/yum.repos.d/mongodb-org-4.2.repo

	编辑以下内容
	
	[mongodb-enterprise]
	name=MongoDB Enterprise Repository
	baseurl=https://repo.mongodb.com/yum/redhat/$releasever/mongodb-enterprise/4.2/$basearch/
	gpgcheck=1
	enabled=1
	gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
	
	使用yum进行安装
	
	yum install -y mongodb-org

	vim /etc/mongod.conf

	systemLog:
	  destination: file #日志输出方式。file/syslog,如果是file，需指定path，默认是输出到标准输出流中
	  path: /var/log/mongodb/mongod/log  #日志路径
	  logAppend: false #启动时，日志追加在已有日志文件内还是备份旧日志后，创建新文件记录日志, 默认false
	
	net:
	  port: 27017 #监听端口，默认27017
	  bindIp: 127.0.0.1 #绑定监听的ip，设置为127.0.0.1时，只会监听本机
	  maxIncomingConnections: 65536 #最大连接数，可接受的连接数还受限于操作系统配置的最大连接数
	  wireObjectCheck: true #校验客户端的请求，防止错误的或无效BSON插入,多层文档嵌套的对象会有轻微性能影响,默认true
	
	processManagement:
	  fork: true  # 后台运行
	
	security:
	  authorization: enabled  # enabled/disabled #开启客户端认证
	
	storage:
	  dbPath: /var/lib/mongodb  # 数据库地址
	  journal: 
	    enabled: true #启动journal,64位系统默认开启，32位默认关闭


	启动mongo
	
	mongod -f /etc/mongod.conf

## 安装algoplus

    pip install AlgoPlus -i http://mirrors.aliyun.com/pypi/simple/  --trusted-host mirrors.aliyun.com