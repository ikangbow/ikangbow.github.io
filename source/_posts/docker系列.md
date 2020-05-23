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

    [root@runoob ~]# uname -r 3.10.0-327.el7.x86_64
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
使用以下命令查看本地所有的镜像：

    docker images

首先，访问 Docker 中文网，在首页中搜索名为“centos”的镜像，在搜索的结果中，有一个“官方镜像”，它就是我们所需的。

然后，进入 CentOS 官方镜像页面，在“Pull this repository”输入框中，有一段命令，把它复制下来，在自己的命令行上运行该命令，随后将立即下载该镜像。

最后，使用以下命令查看本地所有的镜像：

    REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    centos                   latest             1e1148e4cc2c        5 weeks ago         202MB

如果看到以上输出，说明您可以使用“centos”这个镜像了，或将其称为仓库（Repository），该镜像有一个名为“latest ”的标签（Tag），此外还有一个名为“1e1148e4cc2c ”的镜像 ID（可能您所看到的镜像 ID 与此处的不一致，那是正常现象，因为这个数字是随机生成的）。此外，我们可以看到该镜像只有 202MB MB，非常小巧，而不像虚拟机的镜像文件那样庞大。

# 启动容器

容器是在镜像的基础上来运行的，一旦容器启动了，我们就可以登录到容器中，安装自己所需的软件或应用程序。

只需使用以下命令即可启动容器：

    docker run -i -t -v /root/software/:/mnt/software/ 1e1148e4cc2c /bin/bash

docker run <相关参数> <镜像 ID> <初始命令>解析

- -i：表示以“交互模式”运行容器
- -t：表示容器启动后会进入其命令行
- -v：表示需要将本地哪个目录挂载到容器中，格式：-v <宿主机目录>:<容器目录>
假设我们的所有安装程序都放在了宿主机的/root/software/目录下，现在需要将其挂载到容器的/mnt/software/目录下。

需要说明的是，不一定要使用“镜像 ID”，也可以使用“仓库名:标签名”，例如：centos: latest。

初始命令表示一旦容器启动，需要运行的命令，此时使用“/bin/bash”，表示什么也不做，只需进入命令行即可。

# 安装相关软件
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
    
