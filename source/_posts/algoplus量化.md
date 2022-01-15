---
title: algoplus量化
date: 2022-01-14 21:38:26
tags: arctic
category: arctic
---

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

	conda activate foralgo

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
	pip install - i https://pypi.doubanio.com/simple pip -U --user