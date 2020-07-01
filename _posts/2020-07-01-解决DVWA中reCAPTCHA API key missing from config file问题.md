---
layout:     post                    # 使用的布局（不需要改）
title:     解决DVWA中reCAPTCHA API key missing from config file问题  # 标题 
subtitle:      #副标题
date:       2020-07-01         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - DVWA
  
---

在DVWA中测试Insecure CAPTCHA不安全的验证码关卡时，出现以下报错：

![NorW1x.png](https://s1.ax1x.com/2020/07/01/NorW1x.png)

出现该问题是因为使用reCAPTCHA没有申请密钥，因此需要手动填入密钥，打开如图中提示的配置文件，（这里以我的目录，具体看自己安装目录）

找到 C:\phpstudy_pro\WWW\DVWA-master\config\config.inc.php  中的相关配置，

	$_DVWA[ 'recaptcha_public_key' ]  = '';
	$_DVWA[ 'recaptcha_private_key' ] = '';


![Nor5nO.png](https://s1.ax1x.com/2020/07/01/Nor5nO.png)

改为：

	$_DVWA[ 'recaptcha_public_key' ]  = '6LdK7xITAAzzAAJQTfL7fu6I-0aPl8KHHieAT_yJg';
	$_DVWA[ 'recaptcha_private_key' ] = '6LdK7xITAzzAAL_uw9YXVUOPoIHPZLfw2K1n5NVQ';

之后，即可刷新DVWA靶场，发现错误消失。

但是在解决DVWA中`reCAPTCHA API key missing from config file`问题之后，又出现了新的问题，

	需要网站所有者处理的错误：
	网站密钥的网域无效

因为此关卡需要加载谷歌，所以需要科学上网才能看到验证，

![Nor7AH.png](https://s1.ax1x.com/2020/07/01/Nor7AH.png)

因为这个密钥是要绑定域名的，但是上此密钥不知道绑定的啥域名，所以我网上又找了一个密钥，且知道绑定的域名为http://www.dvwa.com. (当然大家也可以自行去  https://www.google.com/recaptcha/admin 申请密钥，自定义绑定网址。)
那么教程就开始了。

## ***一、配置密钥***

打开\DVWA-master\config\config.inc.php ，配置密钥

	$_DVWA[ 'recaptcha_public_key' ]  = '6LeGsK8UAAAAAE3vT3pAytiffWq7Gcz_pwgIBp9r';
	$_DVWA[ 'recaptcha_private_key' ] = '6LeGsK8UAAAAAFP6ZrlXp9jsvpxT9yaSrJ0-AutK';

## ***二、配置host***
因为上面的密钥是和域名（http://www.dvwa.com）绑定在一起的,所以需要在 host 文件中将服务器 ip 映射到该域名。

**windows 配置方法 ：**

打开路劲 C:\windows\System32\drivers\etc ， 编辑host文件，（此文件为系统文件所以需要修改权限才能修改文件，下一篇文章我将会做出相应教程）

在 host 文件最后添加：

	服务器ip www.dvwa.com

温馨提示：服务器ip 可通过cmd命令 ipconfig 查看，

可以在 cmd 中输入ipconfig /flushdns 进行 dns解析 刷新。

**Linux配置方法：**

输入命令进入文件进行修改即可

sudo vim /etc/hosts

## ***三、结果***
访问  http://www.dvwa.com/DVWA-master/DVWA-master/vulnerabilities/captcha/ 

即可看到验证。

![NosSHg.png](https://s1.ax1x.com/2020/07/01/NosSHg.png)