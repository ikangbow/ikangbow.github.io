---
title: 阿里云web应用部署
date: 2019-02-24 09:19:46
tags: 运维
category: java
---
# 阿里云服务器

1. 阿里云服务器的注册及申请
2. 添加安全组规则。安全组在云端提供类似虚拟防火墙功能，用于设置单个或多个 ECS 实例的网络访问控制，它是重要的安全隔离手段。在创建 ECS 实例时，必须选择一个安全组。您还可以添加安全组规则，对该安全组下的所有 ECS 实例的出方向和入方向进行网络控制。 若没有配置安全组规则，直接在本地ping服务器，结果ping不通，ssh也连不上。

默认安全组中的默认规则仅设置针对ICMP协议、SSH 22端口、RDP 3389端口、HTTP 80端口和HTTPS 443端口的入方向规则。网络类型不同，安全且规则不同。

安全组常用配置 例如：redis端口6379/mongo数据库端口27017/mysql端口3306/

# 服务器环境部署
 
1. 使用shell工具上传 JDK；安装svn，安装maven；新建项目文件夹上传tomcat、mongo、redis安装包。
2. 配置环境变量
    
    java环境变量

	    JAVA_HOME=/usr/local/jdk1.8.0_162
	    export JRE_HOME=/usr/local/jdk1.8.0_162/jre
	    export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
	    export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$JAVA_HOME:$PATH:$MAVEN_HOME/bin
        export MAVEN_HOME=/usr/local/maven/apache-maven-3.5.4

    redis配置

	    找到redis.conf文件将bind 127.0.0.1注释，改为bind 0.0.0.0这样就可以支持远程连接
	    找到requirepass  设置redis登录密码

    mongo配置

	    找到mongo.conf配置auth先设置false
	    port=27017
	    logpath=/airthink/mongodb-linux-x86_64-3.6.3/mongod.log
	    pidfilepath=/airthink/mongodb-linux-x86_64-3.6.3/mongod.pid
	    logappend=true
	    fork=true
	    maxConns=3000
	    dbpath=/airthink/mongodb-linux-x86_64-3.6.3/data
	    auth=false

        ./mongodb-linux-x86_64-3.6.3/bin/mongo   进入mongo shell  创建超级管理员admin账户
		use admin
		db.createUser(
		  {
		    user: "admin",
		    pwd: "123456",
		    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
		  } )

	    现在启用auth 将配置文件中auth=false 改为 auth=true
	
	    use admin
	    db.auth('admin','123456');授权
	    
	    创建数据库
	    db.createUser({user:"用户名",pwd:"密码",roles:[{role:"readWrite",db:"项目名"}]})

# 部署web项目

在项目文件夹下使用svn命令
svn checkout svn://路径(目录或文件的全路径)

首次checkout出项目后，运行自动发布脚本，删除开发配置文件，替换生产配置文件，使用mvn打包，将war包复制到tomcat的webapp下，重启tomcat。

# 问题

Linux执行.sh文件，提示No such file or directory的问题的解决方法：

原因：在windows中写好shell脚本测试正常，但是上传到 Linux 上以脚本方式运行命令时提示No such file or directory错误，那么一般是文件格式是dos格式的缘故，改成unix 格式即可。一般有如下几种修改办法。

用vim打开该sh文件，输入：
:set ff 
回车，显示fileformat=dos，重新设置下文件格式：
:set ff=unix 
保存退出: 
:wq 
再执行

# 免费ssh证书申请及配置

    参考https://help.aliyun.com/knowledge_detail/95496.html?spm=a2c4g.11186623.2.11.53674c07M5nwXN


