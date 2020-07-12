---
layout:     post               # 使用的布局（不需要改）
title:      upload-labs中    # 标题 
subtitle:       #副标题
date:       2020-07-11         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - upload-labs
  
---

## Pass-08

**windows特性**:

上传 shell.php(%80-99)

NTFS ADS特性

* 上传 shell.php::$DATA
* 上传 test.jpg..
* 上传 1.php<> 之类的后缀名，和 ② 类似。
* 上传 1.php:.jpg

1.php:.jpg 这类文件在上传后，在Windows中会被保存为一个 空的 1.php 文件，然后可
以再上传一个 1.php<> 或者 1.php<< 或者 其他Windows不允许的后缀，就会覆盖 前边那个空的
1.php

**windows系统文件命令规则**

上传不符合windows文件命名规则的文件名

　　test.asp.

　　test.asp(空格)

　　test.php:1.jpg

　　test.php::$DATA

　　shell.php::$DATA…….

会被windows系统自动去掉不符合规则符号后面的内容。

简单讲就是在php+windows的情况下：如果文件名+"::$DATA"会把::$DATA之后的数据当成文件流处理,不会检测后缀名.且保持"::$DATA"之前的文件名。

以往关卡`file_ext = str_ireplace('::$DATA', '', $file_ext);`//去除字符串::$DATA

        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = trim($file_ext); //首尾去空

这里源代码并没有替换掉`::$DATA`,依旧在windows环境中上传`ma8.php`即可。

## Pass-09

	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])) {
	    if (file_exists(UPLOAD_PATH)) {
	        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
	        $file_name = trim($_FILES['upload_file']['name']);
	        $file_name = deldot($file_name);//删除文件名末尾的点
	        $file_ext = strrchr($file_name, '.');
	        $file_ext = strtolower($file_ext); //转换为小写
	        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
	        $file_ext = trim($file_ext); //首尾去空
        
	        if (!in_array($file_ext, $deny_ext)) {
	            $temp_file = $_FILES['upload_file']['tmp_name'];
	            $img_path = UPLOAD_PATH.'/'.$file_name;
	            if (move_uploaded_file($temp_file, 	$img_path)) {
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

在windows环境中：

这关我们可以看到集合了前面所有的过滤，但是我们依旧可以使用`ma9.php. .`，点空格点绕过，

也就是说，如果从第三关到第九关，如果目标服务器是windows系统的话，均可用点空格点绕过。

此种方法的执行过程已经具体在[此篇文章](https://yanmie-art.github.io/2020/07/07/upload-labs%E9%9D%B6%E5%9C%BA%E4%B8%8A/)Pass-04关卡讲解。

此关还可以利用`ma9.php.x.x.x.xx`进行绕过。

ma9.php.x.x.x.xx

apache:多后缀解析漏洞

在Apache 2.0.x <= 2.0.59，Apache 2.2.x <= 2.2.17，Apache 2.2.2 <= 2.2.8中Apache 解析文件的规则是从右到左开始判断解析,如果后缀名为不可识别文件解析,就再往左判断。
如1.php.abc，因apache2不识别.abc后缀，所以向前解析php

经测试及聂总知道，并不是只有apache有这个解析漏洞........

## Pass-10


	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])) {
	    if (file_exists(UPLOAD_PATH)) {
	        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");
	
	        $file_name = trim($_FILES['upload_file']['name']);
	        $file_name = str_ireplace($deny_ext,"", $file_name);
	        $temp_file = $_FILES['upload_file']['tmp_name'];
	        $img_path = UPLOAD_PATH.'/'.$file_name;        
	        if (move_uploaded_file($temp_file, $img_path)) 	{
	            $is_upload = true;
	        } else {
	            $msg = '上传出错！';
	        }
	    } else {
	        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
	    }
	}

`str_ireplace()` 函数替换字符串中的一些字符（不区分大小写）。

果真过滤。

![UlgHyQ.png](https://s1.ax1x.com/2020/07/11/UlgHyQ.png)

可直接双写绕过。`ma.pphphp`

![UlgxYV.png](https://s1.ax1x.com/2020/07/11/UlgxYV.png)

## Pass-11

	$is_upload = false;
	$msg = null;
	if(isset($_POST['submit'])){
    	$ext_arr = array('jpg','png','gif');
    	$file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    	if(in_array($file_ext,$ext_arr)){
    	    $temp_file = $_FILES['upload_file']['tmp_name'];
    	    $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

    	    if(move_uploaded_file($temp_file,$img_path)){
    	        $is_upload = true;
    	    } else {
    	        $msg = '上传出错！';
    	    }
    	} else{
    	    $msg = "只允许上传.jpg|.png|.gif类型文件！";
    	}
	}


`substr(string,start,length) `函数返回字符串的一部分。

注释：如果 start 参数是负数且 length 小于或等于 start，则 length 为 0。

`strrpos()` 函数查找字符串在另一字符串中最后一次出现的位置。

`move_uploaded_file()` 函数将上传的文件移动到新位置。

若成功，则返回 true，否则返回 false。

可以随便输个后缀，上传失败，可以断定是白名单上传，以前的都是黑名单。

![UlgxYV.png](https://s1.ax1x.com/2020/07/11/UlgxYV.png)

但是我们还有一个新发现就是这里的`$img_path`也就是路径是可控的。

`$_GET['sava_path']`和`随机时间函数`进行拼接，拼接成文件存储路径。这里构造文件存储路径利用了_GET传入，导致服务器最终存储的文件名可控。故可以利用这个点进行绕过。

这里利用的是00截断。即move_uploaded_file函数的底层实现类似于C语言，遇到0x00会截断.

**截断条件：**

1、php版本小于5.3.4

2、php.ini的magic_quotes_gpc为OFF状态

事实上0x00，%00，/00这三类阶截断都是属于同种原理，只是表示不同而已


在url中%00表示ascll码中的0 ，而ascii中0作为特殊字符保留，表示字符串结束，所以当url中出现%00时就会认为读取已结束

如在1.php文件名改为1.php%00.jpg会被解析为1.php，这样就能绕过后缀限制，上传shell

那么我们在这里构造参数`?save_path=../upload/1122.php%00`

![Ul5ZtA.png](https://s1.ax1x.com/2020/07/11/Ul5ZtA.png)

访问`1122.php`即可。

![Ul5K6f.png](https://s1.ax1x.com/2020/07/11/Ul5K6f.png)	

如果00截断条件没设置好，上传会出错的。

![Ul5BBF.png](https://s1.ax1x.com/2020/07/11/Ul5BBF.png)

apache环境和nginx环境都可以。

## Pass-12

	$is_upload = false;
	$msg = null;
	if(isset($_POST['submit'])){
	    $ext_arr = array('jpg','png','gif');
	    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
	    if(in_array($file_ext,$ext_arr)){
	        $temp_file = $_FILES['upload_file']['tmp_name'];
	        $img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;
	
	        if(move_uploaded_file($temp_file,$img_path)){
	            $is_upload = true;
	        } else {
	            $msg = "上传失败";
	        }
	    } else {
	        $msg = "只允许上传.jpg|.png|.gif类型文件！";
	    }
	}


这关拼接的参数save_path是通过POST方式传递的，同样抓包修改save_path，但是因为`POST不像GET能URL解码%00`，所以我们需要在二进制中修改,修改对应位置为00

通常我们会用Burp抓包后，在文件名插入一个空格，然后再HEX中找到空格对应的16进制编码“20”，把它改成00（即16进制ASCII码00，对应十进制的0），就可以插入空字符了。PS:这里的空格纯粹只是一个标记符号，便于我们找到位置，其实这里是什么字符都无所谓，只不过空格比较有特异性，方便在HEX中查找位置

这里我用windows中的演示。(我这里windows13关相当于linux中的12关....)

![U1kGsf.png](https://s1.ax1x.com/2020/07/11/U1kGsf.png)

空格hex编码之后为20，所以把20改为00，上传。这里是遇到0x00截断，

![U1At6x.png](https://s1.ax1x.com/2020/07/11/U1At6x.png)

访问ma12.php，成功。


## Pass-13

![U1AfHS.png](https://s1.ax1x.com/2020/07/11/U1AfHS.png)

这里有一个文件包含。

![U1EPjx.png](https://s1.ax1x.com/2020/07/11/U1EPjx.png)

这里简单的改后缀显示`文件未知，上传失败。`

![U1E4r6.png](https://s1.ax1x.com/2020/07/11/U1E4r6.png)

猜测这里检测了文件开头内容或者文件MIME类型。


这里我们构造gif文件头，发现可以绕过检测，成功上传。(说明只检测了文件后缀和文件内容)

![U1ZEfH.png](https://s1.ax1x.com/2020/07/11/U1ZEfH.png)

访问，成功。

![U1ZY1s.png](https://s1.ax1x.com/2020/07/11/U1ZY1s.png)

**上传png图片**：

图片吗制作`copy 1.png/b + ma.php/a ma.png`,生成的ma.png即为图片马。

上传成功。

![U1mXtS.png](https://s1.ax1x.com/2020/07/11/U1mXtS.png)

进行文件包含。

![U1nZp4.png](https://s1.ax1x.com/2020/07/11/U1nZp4.png)

**jpg文件上传**

方法和上面一样。

![U1unPS.png](https://s1.ax1x.com/2020/07/11/U1unPS.png)

看看源代码把。

	function getReailFileType($filename){
    	$file = fopen($filename, "rb");
    	$bin = fread($file, 2); //只读2字节
    	fclose($file);
    	$strInfo = @unpack("C2chars", $bin);    
    	$typeCode = intval($strInfo['chars1'].$strInfo['chars2']);    
    	$fileType = '';    
    	switch($typeCode){      
    	    case 255216:            
    	        $fileType = 'jpg';
    	        break;
    	    case 13780:            
    	        $fileType = 'png';
    	        break;        
    	    case 7173:            
    	        $fileType = 'gif';
    	        break;
    	    default:            
    	        $fileType = 'unknown';
    	    }    
    	    return $fileType;
	}

	$is_upload = false;
	$msg = null;
	if(isset($_POST['submit'])){
	    $temp_file = $_FILES['upload_file']['tmp_name'];
	    $file_type = getReailFileType($temp_file);

	    if($file_type == 'unknown'){
	        $msg = "文件未知，上传失败！";
	    }else{
	        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$file_type;
        	if(move_uploaded_file($temp_file,$img_path)){
            	$is_upload = true;
        	} else {
            	$msg = "上传出错！";
        	}
    	}
	}

解读代码：

`getReailFileType`函数的基本意思是以二进制可读方式打开上传图片，检测前两个字节，关闭图片，

`unpack()` 函数从二进制字符串对数据进行解包。

之后利用`unpack()`函数解包。

`intval()` 函数用于获取变量的整数值。

`intval()` 函数通过使用指定的进制 base 转换（默认是十进制），返回变量 var 的 integer 数值。 intval() 不能用于 object，否则会产生 E_NOTICE 错误并返回 1。

最后switch分支进行选择。

ps:

$_FILES这个变量用与上传的文件的参数设置，是一个多维数组

数组的用法就是 $_FILES['key']['key2'];

$_FILES['upfile']是你表单上传的文件信息数组，upfile是文件上传字段，在上传时由服务器根据上传字段设定。

$_FILES['upfile']包含了以下内容:

* $_FILES['upfile']['name'] 客户端文件的原名称。
* $_FILES['upfile']['type'] 文件的 MIME 类型，需要浏览器提供该信息的支持，例如"image/gif"。
* $_FILES['upfile']['size'] 已上传文件的大小，单位为字节。
* $_FILES['upfile']['tmp_name'] 文件被上传后在服务端储存的临时文件名。
* $_FILES['upfile']['error'] 和该文件上传相关的错误代码。


## Pass-14

	function isImage($filename){
    	$types = '.jpeg|.png|.gif';
    	if(file_exists($filename)){
    	    $info = getimagesize($filename);
    	    $ext = image_type_to_extension($info[2]);
    	    if(stripos($types,$ext)){
    	        return $ext;
    	    }else{
    	        return false;
    	    }
    	}else{
    	    return false;
    	}
	}

	$is_upload = false;
	$msg = null;
	if(isset($_POST['submit'])){
	    $temp_file = $_FILES['upload_file']['tmp_name'];
	    $res = isImage($temp_file);
	    if(!$res){
	        $msg = "文件未知，上传失败！";
	    }else{
	        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").$res;
	        if(move_uploaded_file($temp_file,$img_path)){
	            $is_upload = true;
	        } else {
	            $msg = "上传出错！";
	        }
	    }
	}


`getimagesize()`  函数将测定任何 GIF，JPG，PNG，SWF，SWC，PSD，TIFF，BMP，IFF，JP2，JPX，JB2，JPC，XBM 或 WBMP 图像文件的大小并返回图像的尺寸以及文件类型和一个可以用于普通 HTML 文件中 IMG 标记中的 height/width 文本字符串。

如果不能访问 filename 指定的图像或者其不是有效的图像，getimagesize() 将返回 FALSE 并产生一条 E_WARNING 级的错误。 

返回一个具有四个单元的数组。索引 0 包含图像宽度的像素值，索引 1 包含图像高度的像素值。索引 2 是图像类型的标记：1 = GIF，2 = JPG，3 = PNG，4 = SWF，5 = PSD，6 = BMP，7 = TIFF(intel byte order)，8 = TIFF(motorola byte order)，9 = JPC，10 = JP2，11 = JPX，12 = JB2，13 = SWC，14 = IFF，15 = WBMP，16 = XBM。这些标记与 PHP 4.3.0 新加的 IMAGETYPE 常量对应。索引 3 是文本字符串，内容为“height="yyy" width="xxx"”，可直接用于 IMG 标记。 

`image_type_to_extension()` — 根据指定的图像类型返回对应的后缀名。

`stripos()` — 查找字符串首次出现的位置（不区分大小写）


绕过方法和上边一样的。用指令`copy 1.png/b + ma.php/a ma.png`生成图片马。

![U11i7R.png](https://s1.ax1x.com/2020/07/11/U11i7R.png)