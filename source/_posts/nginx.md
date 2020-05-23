---
title: nginx
date: 2020-02-13 17:05:42
tags: nginx
category: nginx
---
# nginx介绍

Nginx("engine x")是一款是由俄罗斯的程序设计师Igor Sysoev所开发高性能的 Web和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 代理服务器。

在高连接并发的情况下，Nginx是Apache服务器不错的替代品。

# Nginx 安装

## 下载nginx

    下载地址：http://nginx.org/download/nginx-1.6.2.tar.gz
    [root@bogon src]# cd /usr/local/src/
    [root@bogon src]# wget http://nginx.org/download/nginx-1.6.2.tar.gz

## 解压安装包

    [root@bogon src]# tar zxvf nginx-1.6.2.tar.gz

## 编译安装

    [root@bogon nginx-1.6.2]# ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35
    [root@bogon nginx-1.6.2]# make
    [root@bogon nginx-1.6.2]# make install

## 查看nginx版本

    [root@bogon nginx-1.6.2]# /usr/local/webserver/nginx/sbin/nginx -v

# Nginx 配置

## 创建 Nginx 运行使用的用户 www

    [root@bogon conf]# /usr/sbin/groupadd www 
    [root@bogon conf]# /usr/sbin/useradd -g www www

## 配置nginx.conf ，将/usr/local/webserver/nginx/conf/nginx.conf替换为以下内容

    [root@bogon conf]#  cat /usr/local/webserver/nginx/conf/nginx.conf

    user www www;
    worker_processes 2; #设置值和CPU核心数一致
    error_log /usr/local/webserver/nginx/logs/nginx_error.log crit; #日志位置和日志级别
    pid /usr/local/webserver/nginx/nginx.pid;
    #Specifies the value for maximum file descriptors that can be opened by this process.
    worker_rlimit_nofile 65535;
	events
	{
	  use epoll;
	  worker_connections 65535;
	}
	http
	{
	  include mime.types;
	  default_type application/octet-stream;
	  log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
	               '$status $body_bytes_sent "$http_referer" '
	               '"$http_user_agent" $http_x_forwarded_for';
	  
	#charset gb2312;
	     
	  server_names_hash_bucket_size 128;
	  client_header_buffer_size 32k;
	  large_client_header_buffers 4 32k;
	  client_max_body_size 8m;
	     
	  sendfile on;
	  tcp_nopush on;
	  keepalive_timeout 60;
	  tcp_nodelay on;
	  fastcgi_connect_timeout 300;
	  fastcgi_send_timeout 300;
	  fastcgi_read_timeout 300;
	  fastcgi_buffer_size 64k;
	  fastcgi_buffers 4 64k;
	  fastcgi_busy_buffers_size 128k;
	  fastcgi_temp_file_write_size 128k;
	  gzip on; 
	  gzip_min_length 1k;
	  gzip_buffers 4 16k;
	  gzip_http_version 1.0;
	  gzip_comp_level 2;
	  gzip_types text/plain application/x-javascript text/css application/xml;
	  gzip_vary on;
	 
	  #limit_zone crawler $binary_remote_addr 10m;
	 #下面是server虚拟主机的配置
	 server
	  {
	    listen 80;#监听端口
	    server_name localhost;#域名
	    index index.html index.htm index.php;
	    root /usr/local/webserver/nginx/html;#站点目录
	      location ~ .*\.(php|php5)?$
	    {
	      #fastcgi_pass unix:/tmp/php-cgi.sock;
	      fastcgi_pass 127.0.0.1:9000;
	      fastcgi_index index.php;
	      include fastcgi.conf;
	    }
	    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|ico)$
	    {
	      expires 30d;
	  # access_log off;
	    }
	    location ~ .*\.(js|css)?$
	    {
	      expires 15d;
	   # access_log off;
	    }
	    access_log off;
	  }
	
	}

## 检查配置文件nginx.conf的正确性命令

    [root@bogon conf]# /usr/local/webserver/nginx/sbin/nginx -t

# 启动 Nginx

    [root@bogon conf]# /usr/local/webserver/nginx/sbin/nginx

# Nginx 其他命令

    /usr/local/webserver/nginx/sbin/nginx -s reload            # 重新载入配置文件
    /usr/local/webserver/nginx/sbin/nginx -s reopen            # 重启 Nginx
    /usr/local/webserver/nginx/sbin/nginx -s stop              # 停止 Nginx

# 详细配置


	#user  nobody; #配置用户或者组，默认为nobody nobody。
	worker_processes  auto; #number | auto;默认为1，最好配置成auto,自动匹配进程数

	 #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
	#error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;
	
	#pid        logs/nginx.pid; #指定nginx进程运行文件存放地址
	
	
	events {
	    worker_connections  1024; #最大连接数，默认为512
	}
	
	
	http {
	    include       mime.types; #文件扩展名与文件类型映射表
	    default_type  application/octet-stream;  #默认文件类型，默认为text/plain
	
	    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	    #                  '$status $body_bytes_sent "$http_referer" '
	    #                  '"$http_user_agent" "$http_x_forwarded_for"';
	
	    #access_log  logs/access.log  main;
	
	    sendfile        on; #允许sendfile方式传输文件，默认为off
	    #tcp_nopush     on;
	
	    #keepalive_timeout  0;
	    keepalive_timeout  65; #连接超时时间，默认为75s
	
	    #gzip  on; #启用Gizp压缩
	
	    #include /usr/local/nginx/conf/vhost/*;
	
	    server {
	            listen       80;
	            server_name  #监听地址;
	
	            #charset koi8-r;
	
	            #access_log  logs/host.access.log  main;
	
	            location / { #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
	               proxy_pass http://mysvr;  #请求转向mysvr 定义的服务器列表
	               #设置禁止浏览器缓存，每次都从服务器请求
	               add_header Cache-Control no-cache;
	               add_header Cache-Control private;#表明响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）,可以缓存响应内容。响应只作为私有的缓存，不能在用户间共享。如果要求HTTP认证，响应会自动设置为private。
		        deny 127.0.0.1;  #拒绝的ip
	               allow 172.18.5.54; #允许的ip
	            }
	
	            location /material/ {
	               proxy_pass http://mysvr;  #请求转向mysvr 定义的服务器列表
	               #设置禁止浏览器缓存，每次都从服务器请求
	               add_header Cache-Control no-cache;
	               add_header Cache-Control private;#表明响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）,可以缓存响应内容。响应只作为私有的缓存，不能在用户间共享。如果要求HTTP认证，响应会自动设置为private。
	            }
	     }
	
	     server {
	            listen       80;
	            server_name  #监听地址;
	
	            #charset koi8-r;
	
	            #access_log  logs/host.access.log  main;
	
	            location / { #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
	               proxy_pass http://mysvr;  #请求转向mysvr 定义的服务器列表
	               #设置禁止浏览器缓存，每次都从服务器请求
	               add_header Cache-Control no-cache;
	               add_header Cache-Control private;#表明响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）,可以缓存响应内容。响应只作为私有的缓存，不能在用户间共享。如果要求HTTP认证，响应会自动设置为private。
		        deny 127.0.0.1;  #拒绝的ip
	               allow 172.18.5.54; #允许的ip
	            }
	
	            location /material/ {
	               proxy_pass http://mysvr;  #请求转向mysvr 定义的服务器列表
	               #设置禁止浏览器缓存，每次都从服务器请求
	               add_header Cache-Control no-cache;
	               add_header Cache-Control private;#表明响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）,可以缓存响应内容。响应只作为私有的缓存，不能在用户间共享。如果要求HTTP认证，响应会自动设置为private。
	            }
	     }
	
	
	    #server {
	    #    listen       80;
	    #    server_name  xxx.com;
	    #
	    #    #charset koi8-r;
	    #
	    #    #access_log  logs/host.access.log  main;
	    #
	    #    location / {
	    #        root   html;
	    #        index  index.html index.htm;
	    #    }
	    #
	    #    #error_page  404              /404.html;
	    #
	    #    # redirect server error pages to the static page /50x.html
	    #    #
	    #    error_page   500 502 503 504  /50x.html;
	    #    location = /50x.html {
	    #        root   html;
	    #    }
	    #
	    #    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
	    #    #
	    #    #location ~ \.php$ {
	    #    #    proxy_pass   http://127.0.0.1;
	    #    #}
	    #
	    #    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
	    #    #
	    #    #location ~ \.php$ {
	    #    #    root           html;
	    #    #    fastcgi_pass   127.0.0.1:9000;
	    #    #    fastcgi_index  index.php;
	    #    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
	    #    #    include        fastcgi_params;
	    #    #}
	    #
	    #    # deny access to .htaccess files, if Apache's document root
	    #    # concurs with nginx's one
	    #    #
	    #    #location ~ /\.ht {
	    #    #    deny  all;
	    #    #}
	    #}
	
	
	    # another virtual host using mix of IP-, name-, and port-based configuration
	    #
	    #server {
	    #    listen       8000;
	    #    listen       somename:8080;
	    #    server_name  somename  alias  another.alias;
	
	    #    location / {
	    #        root   html;
	    #        index  index.html index.htm;
	    #    }
	    #}
	
	
	    # HTTPS server
	    #
	    #server {
	    #    listen       443 ssl;
	    #    server_name  localhost;
	
	    #    ssl_certificate      cert.pem;
	    #    ssl_certificate_key  cert.key;
	
	    #    ssl_session_cache    shared:SSL:1m;
	    #    ssl_session_timeout  5m;
	
	    #    ssl_ciphers  HIGH:!aNULL:!MD5;
	    #    ssl_prefer_server_ciphers  on;
	
	    #    location / {
	    #        root   html;
	    #        index  index.html index.htm;
	    #    }
	    #}
	
	}

# NGINX之反向代理与负载均衡

反向代理：

![](nginx01.png)

从图中，我们可以知道，对于浏览器来说，他会发一个http://www.a.com/uri请求到Nginx服务器，对于他来说，他认为数据就是从http://www.a.com/uri域中返回的，事实上，当http://www.a.com/uri到达Nginx服务器后，Nginx服务器会将其转发给http://www.b.com/uri,从http://www.b.com/uri域中取得数据并将其返回给浏览器，这个步骤浏览器是不知道的，也就是说，浏览器并不知道http://www.b.com/uri该域的存在，同理，http://www.b.com/uri所在的域（图中的Tomcat）也并不知道浏览器的存在，他也只对Nginx负责。Nginx的这么一个过程便称为反向代理。

那么，Nginx服务器是如何实现这一步的呢，事实上也很简单，只需要在location中做一下简单的配置即可，命令大概如下图所示：（配置完命令记得reload重新加载才能生效）

![](nginx02.png)

重点在于location处，这样的配置代表的是，所有来自浏览器的请求，在Nginx收到之后，都会代理到http://192.168.1.62:8080所在的地方

比如，我浏览器上发起http://192.168.1.61/a/index.html；Nginx收到之后，将会发出http:// 192.168.1.62:8080/a/index.html这么一个请求到所连接的服务器上，如上图的Tomcat。

接下来我们做这样一个假设，假如后端连接着几台。几十台服务器呢，这个时候Nginx也是做同样的代理吗，答案是肯定的。图示如下：那么，在这么多台服务器上，Nginx的转发又是基于怎样的策略呢？这个时候就涉及在负载均衡了，说白了就是，应该怎样的分发，才能做到资源的最大限度的利用？

![](nginx03.png)

负载均衡策略

	（
	
	我们这里假设三台服务器的IP地址分别为
	
	http:// 192.168.1.62:8080
	
	http:// 192.168.1.63:8080
	
	http:// 192.168.1.64:8080
	
	）

![](nginx04.png)

这里我们把后台所有的服务器放入upstream中，并在代理中进行引用。

![](nginx05.png)

![](nginx06.png)

![](nginx07.png)

其他的配置
备份与停机状态：
server 192.168.1.64 backup;//备份，不参与转发，只有当所有服务器都挂掉时才参与转发；

server 192.168.1.65 down;//临时停机维护，不参与任何转发，是关闭状态，

down存在的意义在于，有时我们需要对服务器做临时停机更新维护，假如我们直接关闭服务器的话，那么对于Nginx来说，他还是会把请求发到该服务器上的，因为他并不知道服务器已关，而设置down后，Nginx则不会再发到该服务器上了，避免造成无用的请求浪费。

max_fails:        达到指定次数后认为服务器挂掉

 fail_timeout：挂掉多久后再次测试是否已经挂掉

配置命令

server 192.168.1.66 max_fails=2 fail_timeout=60s;
