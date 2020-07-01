---
layout:     post                    # 使用的布局（不需要改）
title:     DVWA中SQL注入"Illegal mix of collations for operation 'UNION'"解决方法  # 标题 
subtitle:      #副标题
date:       2020-07-01         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - DVWA
  
---

在DVWA练习的时候，出现了 `Illegal mix of collations for operation 'UNION' `。

SQL注入时的语句是

`1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #`

结果就出现了问题。

![NorM1P.png](https://s1.ax1x.com/2020/07/01/NorM1P.png)

这里的事情了语句 1后边的一撇是英文状态下的， 

之后我尝试着改为 中文状态下的一瞥，发现却成功了。

之后我百度了一些方法说的都很模糊。

在这里给出一种方法。（这边成功了哦！）

更换MySQL 版本，或者 把MySQL 恢复到默认设置，之后重启 phpstudy 继续尝试一下。