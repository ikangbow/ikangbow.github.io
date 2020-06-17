---
title: Git使用详解
date: 2020-06-17 17:23:22
tags: java
category: java
---
# Git的安装和使用

## 下载安装Git

    [https://git-scm.com/download/win](https://git-scm.com/download/win "下载Git")

![](1.png)

## 下载完成后双击安装

## 检验是否安装完成

鼠标右击如果看到有两个git单词则安装成功

![](2.png)

# Git基本工作流程

## Git的工作区域

![](3.png)

## 向仓库中添加文件流程

![](4.png)

# Git初始化及仓库创建和操作

## Git安装之后需要进行一些基本信息设置

1. 设置用户名：git  config -- global  user.name  '你再github上注册的用户名';
2. 设置用户邮箱：git  config -- global  user.email  '注册时候的邮箱';

注意：该配置会在github主页上显示谁提交了该文件

3. 配置ok之后，我们用如下命令来看看是否配置成功

　　git config --list

注意：git  config --global 参数，有了这个参数表示你这台机器上所有的git仓库都会使用这个配置，当然你也可以对某个仓库指定不同的用户名和邮箱

## 初始化一个新的git仓库


1. 创建文件夹

　　　　方法一：可以鼠标右击-》点击新建文件夹test1

　　　　方法二：使用git新建：$  mkdir test1

![](5.png)

2. 在文件内初始化git（创建git仓库）

　　　　方法一：直接输入 $ cd test1

　　　　方法一：点击test1文件下进去之后-》鼠标右击选择Git Bash Here->输入$ git int

![](6.png)

3. 向仓库中添加文件　　

　　方法一：用打开编辑器新建index.html文件

　　方法二：使用git命令。$  touch '文件名'，然后把文件通过$ git add '文件名'添加到暂存区，最后提交操作

![](7.png)

4. 修改仓库文件

　　方法一：用编辑器打开index.html进行修改

　　方法二：使用git命令。$  vi  '文件名'，然后在中间写内容，最后提交操作

![](8.png)

![](9.png)

5. 删除仓库文件

　　方法一：在编辑器中直接把要删除的文件删除掉

　　方法二：使用git删除：$ git rm '文件名'，然后提交操作

![](10.png)

# Git管理远程仓库

![](11.png)

# Git克隆操作

目的：将远程仓库（github上对应的项目）复制到本地

![](12.png)

## 代码：git clone 仓库地址

仓库地址由来如下：


## 克隆项目

![](13.png)

## 将本地仓库同步到git远程仓库中：git push

![](14.png)

# 注意

![](15.png)

解决：这是通过Git GUI进行提交时发生的错误，由 .git 文件夹中的文件被设为“只读”所致，将 .git 文件夹下的所有文件、文件夹及其子文件的只读属性去掉即可。
