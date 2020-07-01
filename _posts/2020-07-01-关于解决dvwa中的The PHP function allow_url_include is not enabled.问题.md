---
layout:     post                    # 使用的布局（不需要改）
title:      关于解决dvwa中的The PHP function allow_url_include is not enabled.问题  # 标题 
subtitle:      #副标题
date:       2020-07-01         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - DVWA
  
---

在Windows系统中搭建的dvwa练习平台（dvwa+phpstudy），练习File Inclusion的时候，却出现了问题 

![出现了问题 ](https://s1.ax1x.com/2020/07/01/NowVG6.png)

解决方法： 找到php.ini  修改配置文件，搜索 allow_url_include，修改off为ON 

![解决方法](https://s1.ax1x.com/2020/07/01/NowZRK.png)

然后重启phpstudy 

![重启phpstudy ](https://s1.ax1x.com/2020/07/01/NowexO.png)