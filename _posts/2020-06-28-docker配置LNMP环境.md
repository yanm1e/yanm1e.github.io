---
layout:     post               # 使用的布局（不需要改）
title:      docker配置LNMP环境  # 标题 
subtitle:      #副标题
date:       2020-06-28         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - 网络安全
  
---

## ***一、docker部署mysql***

#### 1. 查看MySQL可用版本
也可进入docker hub 查看

	docker search mysql

#### 2. 拉取镜像
	docker pull mysql[:版本号]
默认为最新版本

#### 3. 查看本地镜像
	docker images
出现镜像则为已安装

#### 4. 运行容器
	docker run -id \
	 -p 3306:3306 \
	 --name=c_mysql \
	 -v /root/mysql/conf:/etc/mysql/conf.d \
	 -v /root/logs:/logs \
	 -v /root/data:/var/lib/mysql \
	 -e MYSQL_ROOT_PASSWORD=123456 \
	 mysql

其中 -v   为目录挂载，也可以不挂载

-p    端口映射，映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过 宿主机ip:3306 访问到 MySQL 的服务。

`MYSQL_ROOT_PASSWORD=123456` 设置 MySQL 服务 root 用户的密码。

#### 5. 安装成功
	docker ps

查看正在运行的容器

进入容器并登入数据库

	docker exec -it c_mysql /bin/bash
	mysql -uroot -p
	123456

## ***二、 docker安装php***
大概步骤相同。

#### 1. 启动php

	docker run -id --name =sqli_php -v ~/nginx/www:/www php:5.6-fpm

--name  : 将容器命名为 sqli_php。

-v ~/nginx/www:/www : 将主机中项目的目录 www 挂载到容器的/www

#### 2. 创建~/nginx/conf.d目录
	mkdir ~/nginx/conf/conf.d 
	(这条命令如果执行不成功，可分步创建文件夹)

在该目录下添加 ~/nginx/conf/conf.d/test.conf 文件，内容如下：

	server {
    	listen       80;
    	server_name  localhost;
	
    	location / {
    	    root   /usr/share/nginx/html;
    	    index  index.html index.htm index.php;
    	}

    	error_page   500 502 503 504  /50x.html;
    	location = /50x.html {
    	    root   /usr/share/nginx/html;
    	}

    	location ~ \.php$ {
    	    fastcgi_pass   php:9000;
    	    fastcgi_index  index.php;
    	    fastcgi_param  SCRIPT_FILENAME  /www/$fastcgi_script_name;
	        include        fastcgi_params;
    	}
	}


配置文件说明：

php:9000: 表示 php-fpm 服务的 URL。

/www/: 是 sqli_php 中 php 文件的存储路径，映射到本地的 ~/nginx/www 目录。

#### ***三、 docker安装nginx***

#### 1. 启动nginx
	docker run -id --name=sqli_nginx -p 80:80 \
    	-v ~/nginx/www:/usr/share/nginx/html:ro \
    	-v ~/nginx/conf/conf.d:/etc/nginx/conf.d:ro \
    	--link sqli_php:php \
    	nginx

--link sqli_php:php: 把 sqli_php 的网络并入 nginx，并通过修改 nginx 的 /etc/hosts，把域名 php 映射成 127.0.0.1，让 nginx 通过 php:9000 访问 php-fpm。