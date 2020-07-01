---
layout:     post               # 使用的布局（不需要改）
title:      docker安装的php没有php.ini?  # 标题 
subtitle:   但有php.ini-development 与 php.ini-production  #副标题
date:       2020-06-28         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - 网络安全
  
---

在使用习惯了php集成环境以后，发现需要自己配置环境之后就尴尬了。

虽然wamp ，phpstudy等一些集成php环境给我们带来了很大的方便，但同时也夺走了我们很多成长的机会。

最近在docker中安装好php容器之后，还需要配置扩展，就要去找php.ini ,结果到目录一看，很尴尬，

	cd /usr/local/etc/php

没有php.ini ,而是有这么两文件


php.ini-development （开发环境用）与

php.ini-production（生产环境用）。

而php还是会去加载php.ini作为配置文件的。我们更喜欢哪个建议，就把它备份，然后重命名为php.ini然后加入自己个性化的配置即可。

官方在这里   https://www.php.net/manual/zh/install.windows.legacy.index.php
![php.ini问题官方文档](https://s1.ax1x.com/2020/06/28/NgstiD.png)

所以我们在当前目录输入指令
	cp php.ini-development php.ini

复制文件并重命名为 php.ini ,然后就可以了