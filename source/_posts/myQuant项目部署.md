---
title: myQuant项目部署
date: 2021-12-18 10:37:29
tags: 投资
category: 投资
---

# 基于Backtrader的量化投资项目

## 基于centos的docker镜像

    docker pull centos:7

## centos安装yum

    yum install yum-utils
	
	yum install -y git
	
	yum install -y vim
	
	yum -y install wget

## 在新安装的Centos中安装python3.7 解决pip和yum问题

	yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
	
	wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz

## python环境创建


	tar xvf Python-3.7.0.tar.xz

	mv Python-3.7.0 python3

	cd python3

	./configure --prefix=/usr/local/python3

	make && make install

	检查python3.7的编译器：

	/usr/local/python3/bin/python3.7

	建立Python3和pip3的软链

	ln -s /usr/local/python3/bin/python3 /usr/bin/python3

	并将/usr/local/python3/bin加入PATH

	（1）vim /etc/profile
	
	（2）按“I”，然后贴上下面内容：
	
		# vim ~/.bash_profile
		
		# .bash_profile
		
		# Get the aliases and functions
		
		if [ -f ~/.bashrc ]; then
		
		. ~/.bashrc
		
		fi
		
		# User specific environment and startup programs
		
		PATH=$PATH:$HOME/bin:/usr/local/python3/bin
		
		export PATH

## 安装mongo4

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

	systemctl start

## 安装crontab

	yum install vixie-cron
	
	yum install crontabs

	service crond status
	
	启动rsyslog&crond服务

	systemctl start rsyslog
	systemctl start crond
	tail -f /var/log/cron

	更改时区

	tzselect

	crontab -l 查看所有定时任务

## 将当前容器创建为镜像（id）

	docker commit -a "ikangbow" 730661ccf053 ikangbow/myquant:1.0.1

## 推送到dockerhub

	docker push ikangbow/myquant:1.0.1

## docker运行

	docker run -e TZ="Asia/Shanghai" --privileged -it -d --name myquant ikangbow/myquant:1.0.3