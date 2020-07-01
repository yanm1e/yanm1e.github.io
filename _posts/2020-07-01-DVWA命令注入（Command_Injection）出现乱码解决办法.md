---
layout:     post                    # 使用的布局（不需要改）
title:      DVWA命令注入(Command_Injection)出现乱码解决办法  # 标题 
subtitle:      #副标题
date:       2020-07-01         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - DVWA
  
---

 在做DVWA攻防练习时发现，注入命令返回的信息竟然是乱码，猜测可能是因为DVWA是在windows下部署的原因，底层系统是中文GBK编码所致，所以在网上查阅相关资料解决办法如下

* 输入测试   127.0.0.1  时，发现出现乱码情况 

![出现乱码情况 ](https://s1.ax1x.com/2020/07/01/NodHbj.png)

* 在 .../DVWA/dvwa/includes目录下，有个dvwaPage.inc.php文件，直接搜索charset(在277行)，将utf-8修改为GBK 

![乱码情况解决方法](https://s1.ax1x.com/2020/07/01/NodTKg.png)

* 然后重新输入注入的命令 127.0.0.1 后，显示不再是乱码。 

![重新输入注入的命令](https://s1.ax1x.com/2020/07/01/NodjP0.png)

