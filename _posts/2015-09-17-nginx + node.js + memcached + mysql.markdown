---
layout: post
title: "nginx + node.js + memcached + mysql"
date: 2015-09-17 14:00:39
author: "YorkYan"
categories: nginx
---

1. 安装nginx
	1. 安装 nginx
	
			yum insatll nginx
	
	2. 开机启动 nginx
	
			systemctl enable nginx
	
	3. 启动 nginx
	
			systemctl start nginx
	
	4. 停止 nginx
	
		立即退出：
	
			nginx -s stop
	
		任务完成后退出：
	
			nginx -s quit
			或者通过 pid 退出：
			nginx -s QUIT pid
	
	5. 重新启动 nginx
	
			nginx -s reload
	
	6. 设置防火墙接受 http https （永久打开 --permanent）
	
			firewall-cmd --permanent --zone=public --add-service=http 
			firewall-cmd --permanent --zone=public --add-service=https
	
	7. 内网，外网 ip 访问正常。

2. 安装 node.js + npm + express

	1. 安装 node.js + npm + express：

			yum install nodejs
			yum install npm
			nmp install express -gd(-g 是代表全局安装(安装至 /user/lib 下)，d是同时安装依赖关系)
	
			安完后如果命令行中没有 express 命令，执行下面这条安装指令，一般都是要安装的,express 4.x版本都需要安装这个命令行工具
			nmp install express-generator -gd

	2. 使用 express 创建一个 nodejs 项目并启动：

			express myserver
			cd myserver && npm install && npm start &

3. nginx 反向代理至 myserver

	默认情况下我们尽可能不要更改 /etc/nginx/nginx.conf 以及 /etc/nginx/nginx.conf.default 文件，通过查看文件 nginx.conf 我们知道默认会包含 conf.d 目录下面的配置文件(*.conf)，而且监听了默认 http 请求的 80 端口，因此我们先注释掉其中的 server 部分，并且在 conf.d 目录下面建立自己的配置文件 myserver.conf，并写入如下内容：

		upstream myserver{
		        server 127.0.0.1:3000;
		}
		
		server {
		    listen       80 default_server;
		    listen       [::]:80 default_server;
		
		    location / {
		        proxy_pass http://myserver;
		    }
		}

	当我们通过浏览器访问时方式 502 错误，通过 nginx 的错误日志 /var/log/nginx/error.log 可以看到：

		(13: Permission denied) while connecting to upstream...

	通过查证知道这是 selinux 引起的，首先我们查看它的配置情况：

		getsebool -a

	可以看到其中的 `httpd_can_network_connect` 是关闭的，因此我们打开它(永久打开则加参数 -P)

		setsebool httpd_can_network_connect on -P

	至此我们就能成的访问了。
