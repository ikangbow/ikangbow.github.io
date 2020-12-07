---
title: compose
date: 2020-12-07 21:19:37
tags: compose
category: compose
---

# 如何写docker-compose.yml，Docker compose file 参考文档

## Compose 简介

Compose 文件是一个YAML文件，用于定义services、netword和volumes。 Compose 文件的默认路径为./docker-compose.yaml(后缀为.yml和.yaml都可以)。

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

Compose 使用的三个步骤：

使用 Dockerfile 定义应用程序的环境。

使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。

最后，执行 docker-compose up 命令来启动并运行整个应用程序。



## 安装 Docker Compose

    sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
	sudo chmod +x /usr/local/bin/docker-compose

	docker-compose -v

## 工程、服务、容器

Docker Compose 将所管理的容器分为三层，分别是工程（project）、服务（service）、容器（container）

Docker Compose 运行目录下的所有文件（docker-compose.yml）组成一个工程,一个工程包含多个服务，每个服务中定义了容器运行的镜像、参数、依赖，一个服务可包括多个容器实例

Docker Compose默认的模板文件是docker-compose.yml，其中定义的每个服务都必须通过image指令指定镜像或使用Dockerfile的build指令进行自动构建，其它大部分指令跟docker run中选项类似。

如果使用Dockerfile的build指令，则在Dockerfile中设置的选项如CMD、EXPOSE、VOLUME、ENV等将会自动被获取，无需在docker-comopse.yml文件中再次设置。

如果使用image指定为镜像名称或镜像ID时镜像在本地不存在，Compose将会尝试去拉取这个镜像。

典型的docker-comopse.yml文件格式

	version: '2'
	services:
	  web:
	    image: dockercloud/hello-world
	    ports:
	      - 8080
	    networks:
	      - front-tier
	      - back-tier
	 
	  redis:
	    image: redis
	    links:
	      - web
	    networks:
	      - back-tier
	 
	  lb:
	    image: dockercloud/haproxy
	    ports:
	      - 80:80
	    links:
	      - web
	    networks:
	      - front-tier
	      - back-tier
	    volumes:
	      - /var/run/docker.sock:/var/run/docker.sock 
	 
	networks:
	  front-tier:
	    driver: bridge
	  back-tier:
	driver: bridge

## docker-compose.yml
version：指定 docker-compose.yml 文件的写法格式
services：多个容器集合
build：配置构建时，Compose 会利用它自动构建镜像，该值可以是一个路径，也可以是一个对象，用于指定 Dockerfile 参数

Compose文件是一个定义服务services、网络networks和卷volumes的YAML文件，默认路径是./docker-compose.yml，可使用.yml或.yaml作为文件扩展名。

服务services定义包含应用于为该服务启动的每个容器的配置，类似传递命令行参数一样docker container create。同样，网络networks和卷volumes的定义类似于docker network create和docker volume create。正如docker container create在Dockerfile指定选项，如CMD、EXPOSE、VOLUME、ENV，在默认情况下，不需要在docker-compose.yml配置中再次指定。可以使用Bash类${VARIABLE}语法在配置值中使用环境变量。

标准配置文件应该包含version、services、networks三部分，其中最关键的是services和networks两个部分。

### images
images用来指定服务的镜像名称或镜像ID，如果镜像在本地不存在，compose将会尝试去拉取这个镜像。

	services:
	  web:
	    image:redis

### volumes
volumes指令用于设置数据卷挂载路径，数据卷挂载路径可以是一个目录或一个已经存在的数据卷容器，可以设置宿主机路径HOST:CONTAINER或加上访问模式HOST:CONTAINER:ro。使用ro表示对于容器来说数据卷是只读的，这样可以有效地保护宿主机的文件系统。

	volumes:
	  # 指定一个容器内的路径，Docker会自动创建一个数据卷。
	  - /var/lib/mysql
	  # 使用绝对路径挂载数据卷
	  - /opt/data:/var/lib/mysql
	  # 以compose配置文件所在目录为根的相对路径作为数据卷挂载到容器
	  - ./cache:/tmp/cache
	  # 使用用户的相对路径
	  - ~/configs:/etc/configs:ro

### networks

networks指令用于设置指定网络

	services:
	  some-service:
	    networks:
	      - some-network
## 多服务

	version: '2'
	
	services:
	
	 service-eureka: 
	   image: java
	   volumes:
	     - /Users/objcat/jar/service-eureka.jar:/usr/local/service-eureka.jar
	   ports:
	     - 8081:8081
	   command:
	     - /bin/sh
	     - -c
	     - |
	       echo 192.168.1.126 servicehost >> /etc/hosts
	       java -jar /usr/local/service-eureka.jar
	
	 service-a: 
	   image: java
	   volumes:
	     - /Users/objcat/jar/service-a.jar:/usr/local/service-a.jar
	   ports:
	     - 8082:8082
	   command:
	     - /bin/sh
	     - -c
	     - |
	       echo 192.168.1.126 servicehost >> /etc/hosts
	       java -jar /usr/local/service-a.jar
	
	 service-b: 
	   image: java
	   volumes:
	     - /Users/objcat/jar/service-b.jar:/usr/local/service-b.jar
	   ports:
	     - 8083:8083
	   command:
	     - /bin/sh
	     - -c
	     - |
	       echo 192.168.1.126 servicehost >> /etc/hosts
	       java -jar /usr/local/service-b.jar
