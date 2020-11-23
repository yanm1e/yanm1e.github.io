---
 layout:  post    # 使用的布局（不需要改）
 title:  从0开始学web之php特性  # 标题 
 subtitle:  ctfshow   #副标题
 date:  2020-11-22  # 时间
 author:  yanmie    # 作者
 header-img: img/.jpg ##标签这篇文章标题背景图片
 catalog: true      # 是否归档
 tags:        
   - CTF

---


## web89~数组绕过preg_match

```php
<?php

include("flag.php");
highlight_file(__FILE__);

if(isset($_GET['num'])){
    $num = $_GET['num'];
    if(preg_match("/[0-9]/", $num)){
        die("no no no!");
    }
    if(intval($num)){
        echo $flag;
    }
} 
```

* preg_match — 执行匹配正则表达式

> 返回值：
>
> **preg_match()**返回 `pattern` 的匹配次数。  它的值将是0次（不匹配）或1次，因为**preg_match()**在第一次匹配后  将会停止搜索。[preg_match_all()](https://www.php.net/manual/zh/function.preg-match-all.php)不同于此，它会一直搜索`subject`  直到到达结尾。   如果发生错误**preg_match()**返回 **`FALSE`**。  

数组绕过

当匹配数组是返回false绕过。

```php
?num[]=1
```

## web90~intval函数

```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==="4476"){
        die("no no no!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }else{
        echo intval($num,0);
    }
}
```

* **intval()** 函数用于获取变量的整数值。

  **intval()** 函数通过使用指定的进制 base 转换（默认是十进制），返回变量 var 的 integer 数值。 intval() 不能用于 object，否则会产生 E_NOTICE 错误并返回 1。

> ```
> int intval ( mixed $var [, int $base = 10 ] )
> ```
>
> 参数说明：
>
> - $var：要转换成 integer 的数量值。
> - $base：转化所使用的进制。
>
> 如果 base 是 0，通过检测 var 的格式来决定使用的进制：
>
> - 如果字符串包括了 "0x" (或 "0X") 的前缀，使用 16 进制 (hex)；否则，
> - 如果字符串以 "0" 开始，使用 8 进制(octal)；否则，
> - 将使用 10 进制 (decimal)。

传参一个数既要使其不强等于`4476` ,又要使得`intaval`函数处理后的结果强等于`4476`,结合上述函数解释

构造

```
?num=4476a
```

## web91~/m之%0a绕过

```php
<?php

show_source(__FILE__);
include('flag.php');
$a=$_GET['cmd'];
if(preg_match('/^php$/im', $a)){
    if(preg_match('/^php$/i', $a)){
        echo 'hacker';
    }
    else{
        echo $flag;
    }
}
else{
    echo 'nonononono';
}

Notice: Undefined index: cmd in /var/www/html/index.php on line 15
nonononono
```



[![D3vAXQ.md.png](https://s3.ax1x.com/2020/11/22/D3vAXQ.md.png)](https://imgchr.com/i/D3vAXQ)

题目要求，传参 cmd.

但是第一个正则匹配要求多行匹配`php`,但是另一个要求有去掉修饰符`m`要求不匹配`php` .那么就应该能想到是截断。

第一个匹配多行，第二个只匹配单行，那么我们可以构造`php%0ap` 进行匹配，当多行匹配的时候前后都是`p` ,

还可以这里有有两个条件，第一个需要是php，第二个又不可以php，不过有个差距就是m模式，/m代表匹配多行数据，第一个if有匹配到第二行的php,而第二个if匹配不到为假。

CVE-2017-15715

https://blog.csdn.net/qq_46091464/article/details/108278486



## web92~intval八十六进制科学计数法绕过

```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(intval($num,0)==4476){
        echo $flag;
    }else{
        echo intval($num,0);
    }
}
```

同样考点是`intval` ，但这道题不同于web90,而是弱比较，所以说`4476a`等肯定不行的。

这里用到十六进制`0x117c` 或者八进制`010574`

https://www.runoob.com/php/php-intval-function.html

也可以科学计数法`4476e2` ,在第一个if会计数比较，在`intval`函数中会被看做字符串。

## web93~intval八进制绕过

```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(preg_match("/[a-z]/i", $num)){
        die("no no no!");
    }
    if(intval($num,0)==4476){
        echo $flag;
    }else{
        echo intval($num,0);
    }
} 
```

过滤了字母，但我们可以使用八进制`010574`

## web94~八进制，小数点绕过

```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==="4476"){
        die("no no no!");
    }
    if(preg_match("/[a-z]/i", $num)){
        die("no no no!");
    }
    if(!strpos($num, "0")){
        die("no no no!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }
} 
```

* `strpos`查找 "php" 在字符串中第一次出现的位置：
* strpos() 函数对大小写敏感。
* 返回字符串在另一字符串中第一次出现的位置，如果没有找到字符串则返回 FALSE。

比上一关增加条件，strpos函数限制了传参第一位不能为0，如果为0，就die.

但是如果找不到的话又会die.

仔细观察这里时强等于。

我们可以在八进制前加一个空格 
```
?num=  010574
```

或者用小数点

```
?num=4476.0
```

或者再加个 + 

```
?num=+4476.0
```

## web95~空格加八进制绕intval

```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(preg_match("/[a-z]|\./i", $num)){
        die("no no no!!");
    }
    if(!strpos($num, "0")){
        die("no no no!!!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }
} 
```

又增加过滤了`.`

构造`空格+八进制`

```
?num=  010574
```

也可以

```
?num=+010574
?num=%2b010574
```



## web96~绝对路径相对路径

```php
<?php

highlight_file(__FILE__);

if(isset($_GET['u'])){
    if($_GET['u']=='flag.php'){
        die("no no no");
    }else{
        highlight_file($_GET['u']);
    }


}
```

可以看到不能直接等于`flag.php`,

但是我们可以构造路劲让其显示

```php
?u=/var/www/html/flag.php
?u=./flag.php
```

## web97~md5数组绕过

```php
<?php

include("flag.php");
highlight_file(__FILE__);
if (isset($_POST['a']) and isset($_POST['b'])) {
if ($_POST['a'] != $_POST['b'])
if (md5($_POST['a']) === md5($_POST['b']))
echo $flag;
else
print 'Wrong.';
}
?>

```

这里是强比较。

弱比较的话可以百度有好多md5加密后是0e开头的，弱比较 0=0

**强比较**

如果传入`md5` 函数的不是字符串而是数组，那么就会返回`null`, null=null绕过。

构造

```php
a[]=1&b[]=2
```

还有md5强碰撞

https://blog.csdn.net/EC_Carrot/article/details/109525162

https://www.cnblogs.com/kuaile1314/p/11968108.html

## web98

```php

Notice: Undefined index: flag in /var/www/html/index.php on line 15

Notice: Undefined index: flag in /var/www/html/index.php on line 16

Notice: Undefined index: HTTP_FLAG in /var/www/html/index.php on line 17
<?php
    
include("flag.php");
$_GET?$_GET=&$_POST:'flag';
$_GET['flag']=='flag'?$_GET=&$_COOKIE:'flag';
$_GET['flag']=='flag'?$_GET=&$_SERVER:'flag';
highlight_file($_GET['HTTP_FLAG']=='flag'?$flag:__FILE__);

?>
```

