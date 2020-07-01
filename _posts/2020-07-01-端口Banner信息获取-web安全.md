---
layout:     post                    # 使用的布局（不需要改）
title:     端口Banner信息获取-web安全  # 标题 
subtitle:      #副标题
date:       2020-07-01         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - 网络安全
  
---

## ***一、端口分类***
端口范围： 0 - 65535

TCP端口和UDP端口。由于TCP和UDP两个协议是独立的，因此各自的端口号也相互独立。比如TCP有235端口，UDP也有235端口，两者并不冲突。

端口分为：

1、周知端口

周知端口是众所周知的端口号，范围从0到1023，其中80端口分配给WWW服务，21端口分配给FTP,22端口分配给ssh等。我们在浏览器的地址栏中输入一个网址时不必指定端口号，是因为默认情况下时80端口，但如果服务绑定了其他端口，则需要冒号加上端口号进行访问。

2、动态端口

动态端口的范围是从49152到65535。之所以称为动态端口，是因为它一般不固定分配某种服务，而是动态分配。

3、注册端口

范围是从1024到49151，也就是剩下的端口，分配给用户进程或应用程序。这些进程主要是用户安装的程序。

## ***二、端口Banner获取 - Nmap***

使用Nmap扫描指定主机端口信息，并返回Banner信息.

	namp IP地址  -p 端口号(也可以是端口范围) --script banner

效果如下图：
![NoDCqg.png](https://s1.ax1x.com/2020/07/01/NoDCqg.png)

## ***三、端口Banner获取 - dmitry***

使用dmitry获取端口banner信息。

	dmitry -pb IP地址

效果如图：

![NoDVGq.png](https://s1.ax1x.com/2020/07/01/NoDVGq.png)

![NoDZR0.png](https://s1.ax1x.com/2020/07/01/NoDZR0.png)

## ***三、端口Banner获取 - netcat***

使用netcat获取Banner信息。

	nc -nv IP地址 端口号

效果如图：

![NoD1o9.png](https://s1.ax1x.com/2020/07/01/NoD1o9.png)

nc一般为单个端口获取

nmap 可以是单个也可以是范围

netcat 是范围获取。
