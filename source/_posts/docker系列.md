---
title: docker系列
date: 2019-01-16 21:54:46
tags: docker
category: docker
---
# CentOS Docker 安装

Docker支持以下的CentOS版本：

- CentOS 7 (64-bit)
- CentOS 6.5 (64-bit) 或更高的版本

使用 yum 安装（CentOS 7下）

通过 uname -r 命令查看你当前的内核版本 

    [root@airthink ~]# uname -r 3.10.0-327.el7.x86_64
只需通过以下命令即可安装 Docker 软件：

    yum -y install docker-io

可使用以下命令，查看 Docker 是否安装成功

    docker version

    Client:
    Version:           18.09.1
    API version:       1.39
    Go version:        go1.10.6
    Git commit:        4c52b90
    Built:             Wed Jan  9 19:35:01 2019
    OS/Arch:           linux/amd64
    Experimental:      false

    Server: Docker Engine - Community
    Engine:
    Version:          18.09.1
     API version:      1.39 (minimum version 1.12)
     Go version:       go1.10.6
     Git commit:       4c52b90
    Built:            Wed Jan  9 19:06:30 2019
    OS/Arch:          linux/amd64
    Experimental:     false

若输出了 Docker 的版本号，则说明安装成功了，可通过以下命令启动 Docker 服务：

    service docker start

# 下载镜像
	docker search "镜像名"，搜索镜像，第一个字段为imagename（镜像名字），第四个字段为official，是否为官方镜像，个人也可以把自己制作的镜像放到docker hub上，找到自己想要下载的镜像名字就可以拉取了
	docker pull "镜像名"，下载镜像
	docker rmi "镜像名"，删除镜像
	运行一个新的container
	docker run [-i -t -d -p -P -c] [--name]:在容器内运行一个应用程序
	 -t :在新容器内指定一个伪终端或终端
	 -i:允许你对容器内的标准输入进行交互
	 -d：以进程方式运行容器，让容器在后台运行
	 -p：设置端口
	 -P：将容器内部使用的网络端口映射到我们使用的主机，就是让我们访问我们使用的主机就等同于访问到容器内部
	 -c：command,后面接命令
	 --name container name：指定容器名字
	
	docker ps -a  列出创建的所有容器
	docker ps -s  列出正在运行的容器
	docker start [container ID]
	docker exec -it [container ID] /bin/bash 即可进入一个已经在运行的容器
	exit 可退出容器
	docker stop [container ID] 停用容器
	Docker 1.13版本以后，可以使用 docker containers prune 命令，删除孤立的容器。
	docker container prune
	
	docker rmi -f image_ID//删除镜像
# Dockerfile部分命令
	首先，每个指令的前缀必须大写
	指令：FROM
	功能描述：设置基础镜像
	语法：FROM < image>[:< tag> | @< digest>]
	提示：镜像都是从一个基础镜像（操作系统或其他镜像）生成，可以在一个Dockerfile中添加多条FROM指令，一次生成多个镜像
	注意：如果忽略tag选项，会使用latest镜像
	
	指令：MAINTAINER
	功能描述：设置镜像作者
	语法：MAINTAINER < name>

	指令：RUN
	功能描述：
	语法：RUN < command>
	RUN [“executable”,”param1”,”param2”]
	提示：RUN指令会生成容器，在容器中执行脚本，容器使用当前镜像，脚本指令完成后，Docker Daemon会将该容器提交为一个中间镜像，供后面的指令使用
	补充：RUN指令第一种方式为shell方式，使用/bin/sh -c < command>运行脚本，可以在其中使用\将脚本分为多行
	RUN指令第二种方式为exec方式，镜像中没有/bin/sh或者要使用其他shell时使用该方式，其不会调用shell命令
	例子：RUN source $HOME/.bashrc;\
	echo $HOME
	
	RUN [“/bin/bash”,”-c”,”echo hello”]
	
	RUN [“sh”,”-c”,”echo”,”$HOME”] 使用第二种方式调用shell读取环境变量
	
	指令：CMD
	功能描述：设置容器的启动命令
	语法：CMD [“executable”,”param1”,”param2”]
	CMD [“param1”,”param2”]
	CMD < command>
	提示：CMD第一种、第三种方式和RUN类似，第二种方式为ENTRYPOINT参数方式，为entrypoint提供参数列表
	注意：Dockerfile中只能有一条CMD命令，如果写了多条则最后一条生效
	
	指令：LABEL
	功能描述：设置镜像的标签
	延伸：镜像标签可以通过docker inspect查看
	格式：LABEL < key>=< value> < key>=< value> …
	提示：不同标签之间通过空格隔开
	注意：每条指令都会生成一个镜像层，Docker中镜像最多只能有127层，如果超出Docker Daemon就会报错，如LABEL ..=.. <假装这里有个换行> LABEL ..=..合在一起用空格分隔就可以减少镜像层数量，同样，可以使用连接符\将脚本分为多行
	镜像会继承基础镜像中的标签，如果存在同名标签则会覆盖
	
	指令：EXPOSE
	功能描述：设置镜像暴露端口，记录容器启动时监听哪些端口
	语法：EXPOSE < port> < port> …
	延伸：镜像暴露端口可以通过docker inspect查看
	提示：容器启动时，Docker Daemon会扫描镜像中暴露的端口，如果加入-P参数，Docker Daemon会把镜像中所有暴露端口导出，并为每个暴露端口分配一个随机的主机端口（暴露端口是容器监听端口，主机端口为外部访问容器的端口）
	注意：EXPOSE只设置暴露端口并不导出端口，只有启动容器时使用-P/-p才导出端口，这个时候才能通过外部访问容器提供的服务
	
	指令：ENV
	功能描述：设置镜像中的环境变量
	语法：ENV < key>=< value>…|< key> < value>
	注意：环境变量在整个编译周期都有效，第一种方式可设置多个环境变量，第二种方式只设置一个环境变量
	提示：通过${变量名}或者 $变量名使用变量，使用方式${变量名}时可以用${变量名:-default} ${变量名:+cover}设定默认值或者覆盖值
	ENV设置的变量值在整个编译过程中总是保持不变的
	
	指令：ADD
	功能描述：复制文件到镜像中
	语法：ADD < src>… < dest>|[“< src>”,… “< dest>”]
	注意：当路径中有空格时，需要使用第二种方式
	当src为文件或目录时，Docker Daemon会从编译目录寻找这些文件或目录，而dest为镜像中的绝对路径或者相对于WORKDIR的路径
	提示：src为目录时，复制目录中所有内容，包括文件系统的元数据，但不包括目录本身
	src为压缩文件，并且压缩方式为gzip,bzip2或xz时，指令会将其解压为目录
	如果src为文件，则复制文件和元数据
	如果dest不存在，指令会自动创建dest和缺失的上级目录
	
	指令：COPY
	功能描述：复制文件到镜像中
	语法：COPY < src>… < dest>|[“< src>”,… “< dest>”]
	提示：指令逻辑和ADD十分相似，同样Docker Daemon会从编译目录寻找文件或目录，dest为镜像中的绝对路径或者相对于WORKDIR的路径
	
	指令：ENTRYPOINT
	功能描述：设置容器的入口程序
	语法：ENTRYPOINT [“executable”,”param1”,”param2”]
	ENTRYPOINT command param1 param2（shell方式）
	提示：入口程序是容器启动时执行的程序，docker run中最后的命令将作为参数传递给入口程序
	入口程序有两种格式：exec、shell，其中shell使用/bin/sh -c运行入口程序，此时入口程序不能接收信号量
	当Dockerfile有多条ENTRYPOINT时只有最后的ENTRYPOINT指令生效
	如果使用脚本作为入口程序，需要保证脚本的最后一个程序能够接收信号量，可以在脚本最后使用exec或gosu启动传入脚本的命令
	注意：通过shell方式启动入口程序时，会忽略CMD指令和docker run中的参数
	为了保证容器能够接受docker stop发送的信号量，需要通过exec启动程序；如果没有加入exec命令，则在启动容器时容器会出现两个进程，并且使用docker stop命令容器无法正常退出（无法接受SIGTERM信号），超时后docker stop发送SIGKILL，强制停止容器
	
	指令：VOLUME
	功能描述：设置容器的挂载点
	语法：VOLUME [“/data”]
	VOLUME /data1 /data2
	提示：启动容器时，Docker Daemon会新建挂载点，并用镜像中的数据初始化挂载点，可以将主机目录或数据卷容器挂载到这些挂载点
	
	指令：USER
	功能描述：设置RUN CMD ENTRYPOINT的用户名或UID
	语法：USER < name>
	
	指令：WORKDIR
	功能描述：设置RUN CMD ENTRYPOINT ADD COPY指令的工作目录
	语法：WORKDIR < Path>
	提示：如果工作目录不存在，则Docker Daemon会自动创建
	Dockerfile中多个地方都可以调用WORKDIR，如果后面跟的是相对位置，则会跟在上条WORKDIR指定路径后（如WORKDIR /A   WORKDIR B   WORKDIR C，最终路径为/A/B/C）
	
	指令：ARG
	功能描述：设置编译变量
	语法：ARG < name>[=< defaultValue>]
	注意：ARG从定义它的地方开始生效而不是调用的地方，在ARG之前调用编译变量总为空，在编译镜像时，可以通过docker build –build-arg < var>=< value>设置变量，如果var没有通过ARG定义则Daemon会报错
	可以使用ENV或ARG设置RUN使用的变量，如果同名则ENV定义的值会覆盖ARG定义的值，与ENV不同，ARG的变量值在编译过程中是可变的，会对比使用编译缓存造成影响（ARG值不同则编译过程也不同）
	例子：ARG CONT_IMAG_VER <换行> RUN echo $CONT_IMG_VER
	ARG CONT_IMAG_VER <换行> RUN echo hello
	当编译时给ARG变量赋值hello，则两个Dockerfile可以使用相同的中间镜像，如果不为hello，则不能使用同一个中间镜
示例：Dockerfile以阿里中间件大赛给的debian-jdk8镜像为例，Dockerfile文件如下：

	FROM debian:stretch
	ARG DEBIAN_FRONTEND=noninteractive
	ARG JAVA_VERSION=8
	ARG JAVA_UPDATE=172
	ARG JAVA_BUILD=11
	ARG JAVA_PACKAGE=jdk
	ARG JAVA_HASH=a58eab1ec242421181065cdc37240b08
	
	ENV LANG C.UTF-8
	ENV JAVA_HOME=/opt/jdk
	ENV PATH=${PATH}:${JAVA_HOME}/bin
	
	RUN set -ex \
	 && apt-get update \
	 && apt-get -y install ca-certificates wget unzip \
	 && wget -q --header "Cookie: oraclelicense=accept-securebackup-cookie" \
	     -O /tmp/java.tar.gz \
	     http://download.oracle.com/otn-pub/java/jdk/${JAVA_VERSION}u${JAVA_UPDATE}-b${JAVA_BUILD}/${JAVA_HASH}/${JAVA_PACKAGE}-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64.tar.gz \
	 && CHECKSUM=$(wget -q -O - https://www.oracle.com/webfolder/s/digest/${JAVA_VERSION}u${JAVA_UPDATE}checksum.html | grep -E "${JAVA_PACKAGE}-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64\.tar\.gz" | grep -Eo '(sha256: )[^<]+' | cut -d: -f2 | xargs) \
	 && echo "${CHECKSUM}  /tmp/java.tar.gz" > /tmp/java.tar.gz.sha256 \
	 && sha256sum -c /tmp/java.tar.gz.sha256 \
	 && mkdir ${JAVA_HOME} \
	 && tar -xzf /tmp/java.tar.gz -C ${JAVA_HOME} --strip-components=1 \
	 && wget -q --header "Cookie: oraclelicense=accept-securebackup-cookie;" \
	     -O /tmp/jce_policy.zip \
	     http://download.oracle.com/otn-pub/java/jce/${JAVA_VERSION}/jce_policy-${JAVA_VERSION}.zip \
	 && unzip -jo -d ${JAVA_HOME}/jre/lib/security /tmp/jce_policy.zip \
	 && rm -rf ${JAVA_HOME}/jar/lib/security/README.txt \
	   /var/lib/apt/lists/* \
	   /tmp/* \
	   /root/.wget-hsts
体验一下如何用Dockerfile打包一个镜像，新建一个空目录，假设就是~/debian-jdk8吧，cd进这个目录，新建一个Dockerfile，然后把上面的内容copy进去，然后执行下面的命令：
	docker build -t debian-jdk8:v1.0 .
其中-t debian-jdk8:v1.0表示打包的镜像名为debian-jdk，tag为v1.0（前面说过，tag是可以任意命名的，不一定要是这种格式），注意命令的最后有一个.，这个表示打包的上下文（其实就是Dockerfile所在目录）是在当前目录，然后目录下的Dockerfile就会被编译执行。
执行完毕后运行docker images就会发现多了一个debian-jdk8镜像。
下面来解释一下Dockerfile的结构，那些字母全部大写的每行第一个单词都是Dockerfile的指令，可以看出这个Dockefile中包括的指令有FROM、ARG、ENV、RUN
FROM：FROM debian:stretch表示以debian:stretch作为基础镜像进行构建
RUN	可以看出RUN后面跟的其实就是一些shell命令，通过&&将这些脚本连接在了一行执行，这么做的原因是为了减少镜像的层数，每多一行RUN都会给镜像增加一层，所以这里选择将所有命令联结在一起执行以减少层数
ARG	特地将这个指令放在RUN之后讲解，这个指令可以进行一些宏定义，比如我定义ENV JAVAHOME=/opt/jdk，之后RUN后面的shell命令中的${JAVAHOME}都会被/opt/jdk代替
ENV	可以看出这个指令的作用是在shell中设置一些环境变量（其实就是export）

# Docker部署web程序
为了搭建 Java Web 运行环境，我们需要安装 JDK 、数据库、redis与 Tomcat，首先要再宿主机上传JDK 、数据库、redis与 Tomcat压缩包，以/mnt/software/为例。下面的过程均在容器内部进行。我们不妨选择/opt/目录作为安装目录，首先需要通过cd /opt/命令进入该目录。

首先，解压程序包

    tar -zxf /mnt/software/jdk-7u67-linux-x64.tar.gz
    tar -zxf /mnt/software/apache-tomcat-7.0.55.tar.gz
    tar -zxf /mnt/software/redis
    tar -zxf /mnt/software/mongo

设置环境变量

 首先，编辑.bashrc文件

    vi ~/.bashrc

然后，在该文件末尾添加如下配置：

    export JAVA_HOME=/opt/jdk
    export PATH=$PATH:$JAVA_HOME

最后，需要使用source命令，让环境变量生效：

    source ~/.bashrc

编写运行脚本

    vi /root/run.sh
然后，编辑脚本内容如下：

    source ~/.bashrc
    sh /opt/tomcat/bin/catalina.sh run

注意：这里必须先加载环境变量，然后使用 Tomcat 的运行脚本来启动 Tomcat 服务。

最后，为运行脚本添加执行权限：

    chmod u+x /root/run.sh

# 退出容器

当以上步骤全部完成后，可使用exit命令，退出容器。

随后，可使用如下命令查看正在运行的容器：

    docker ps//未输出
    docker ps -a//输出

输出如下内容：

    CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS                      PORTS               NAMES
    57c312bbaad1        docker.cn/docker/centos:centos6   "/bin/bash"         27 minutes ago      Exited (0) 19 seconds ago                       naughty_goldstine

记住以上CONTAINER ID（容器 ID），随后我们将通过该容器，创建一个可运行 Java Web 的镜像。

# 创建 Java Web 镜像

使用以下命令，根据某个“容器 ID”来创建一个新的“镜像”：

    docker commit 57c312bbaad1 ikangbow/javaweb:0.1

该容器的 ID 是“57c312bbaad1”，所创建的镜像名是“ikangbow/javaweb:0.1”，随后可使用镜像来启动 Java Web 容器。

# 启动 Java Web 容器

首先使用docker images命令，查看当前所有的镜像：

    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    ikangbow/javaweb    0.1                 95b5f6b09ca6        6 hours ago         1.51GB
    centos              latest              1e1148e4cc2c        5 weeks ago         202MB


可见，此时已经看到了最新创建的镜像“ikangbow/javaweb:0.1”，其镜像 ID 是“95b5f6b09ca6”。正如上面所描述的那样，我们可以通过“镜像名”或“镜像 ID”来启动容器，与上次启动容器不同的是，我们现在不再进入容器的命令行，而是直接启动容器内部的 Tomcat 服务。此时，需要使用以下命令：

    docker run -d -p 58080:8080 --name javaweb ikangbow/javaweb:0.1 /root/run.sh

稍作解释：
- -d：表示以“守护模式”执行/root/run.sh脚本，此时 Tomcat 控制台不会出现在输出终端上。
- -p：表示宿主机与容器的端口映射，此时将容器内部的 8080 端口映射为宿主机的 58080 端口，这样就向外界暴露了 58080 端口，可通过 Docker 网桥来访问容器内部的 8080 端口了。
- --name：表示容器名称，用一个有意义的名称命名即可。

关于 Docker 网桥的内容，需要补充说明一下。实际上 Docker 在宿主机与容器之间，搭建了一座网络通信的桥梁，我们可通过宿主机 IP 地址与端口号来映射容器内部的 IP 地址与端口号，

在一系列参数后面的是“镜像名”或“镜像 ID”，怎么方便就怎么来。最后是“初始命令”，它是上面编写的运行脚本，里面封装了加载环境变量并启动 Tomcat 服务的命令。

当运行以上命令后，会立即输出一长串“容器 ID”，我们可通过docker ps命令来查看当前正在运行的容器。

    CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS                     NAMES
    82f47923f926        ikangbow/javaweb:0.1   "/root/run.sh"      4 seconds ago       Up 3 seconds        0.0.0.0:58080->8080/tcp   javaweb

此处实测有问题，需要继续研究。

附常用命令：


1. 删除容器实例

寻找已经停止运行的container
    
    docker ps -a  
    docker rm 容器id 删除实例
    docker ps -a 查看实例已经删除
    docker rm $(docker ps -a -q)  删除所有container

2.删除镜像 

停止所有的container，这样才能够删除其中的images：

    docker stop $(docker ps -a -q)
    docker rmi <image id>
    docker images 查看所有镜像
    docker rmi 镜像id 删除镜像
    docker images 查看镜像 发现已经删除 

# 实际应用

    docker stop application//停止应用
    docker rm application//删除镜像
    docker run -i -t -d -p 88:8080 -v /badou/badou/badou1.war:/usr/local/tomcat/webapps/badou.war -v /badou/conf/apiclient_cert.p12:/usr/local/tomcat/apiclient_cert.p12 -v /badou/badou/index.html:/usr/local/tomcat/webapps/ROOT/index.html --name=application andylaun/tomcat:v5

    -v  挂载文件目录
    
