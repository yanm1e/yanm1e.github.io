---
layout:     post               # 使用的布局（不需要改）
title:      docker内安装php缺少的扩展mysql.so和mysqli.so 及安装vim  # 标题 
subtitle:      #副标题
date:       2020-06-28         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - 网络安全
  
---

## ***一、 docker容器安装vim***

在使用docker容器时，有时候里边没有安装vim，敲vim命令时提示说：`vim: command not found`，这个时候就需要安装vim，

可是当你敲`apt-get install vim`命令时，提示：

	Reading package lists... Done
	Building dependency tree       
	Reading state information... Done
	E: Unable to locate package vim

这时候需要敲：`apt-get update`，这个命令的作用是：同步 /etc/apt/sources.list 和 /etc/apt/sources.list.d 中列出的源的索引，这样才能获取到最新的软件包。

等更新完毕以后再敲命令：`apt-get install vim`命令即可。

但实际在使用过程中，运行 `apt-get update`，然后执行 `apt-get install -y vim`，下载地址由于是海外地址，下载速度异常慢而且可能中断更新流程，所以做下面配置：

	mv /etc/apt/sources.list /etc/apt/sources.list.bak
   	  echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >> /etc/apt/sources.list
  	  echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
 	   echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list
  	  echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list

之后在执行一遍，apt-get uodate。

## ***二、配置php.ini***

如找不到 php.ini ，可参考最近一篇文章(docker安装的php没有php.ini)。

![php.ini](https://s1.ax1x.com/2020/06/28/Ng6xIK.png)


vim 打开 php.ini 
![vim打开php.ini](https://s1.ax1x.com/2020/06/28/NgcCxH.png)

找到此处，可将mysql.dll 改为mysql.so ,去掉注释 。也可直接后面自行添加 。

	extension=mysql.so
	extension=mysqli.so

进入容器内docker安装扩展的目录：

![进入容器内docker安装扩展的目录](https://s1.ax1x.com/2020/06/28/Ngc8Zq.png)

因为我已经安装，所以出现了 mysql.so  mysqli.so

![mysql.so  mysqli.so](https://s1.ax1x.com/2020/06/28/NgcNJU.png)

使用命令安装需要的扩展

 	docker-php-ext-install mysql

 	docker-php-ext-install mysqli

安装成功，退出docker，重启容器，docker restart sqli_php

再次进入docker内，输入 php -m    即可看到安装的扩展 
