---
 layout:   post        # 使用的布局（不需要改）
 title:   [RoarCTF 2019]Easy Calc   # 标题 
 subtitle:   #副标题
 date:    2020-10-01    # 时间
 author:   yanmie       # 作者
 header-img: img/.jpg  ##标签这篇文章标题背景图片
 catalog: true            # 是否归档
 tags:                
    - CTF

--- 


打开环境，是一个计算机

![0MTodI.png](https://s1.ax1x.com/2020/10/01/0MTodI.png)


右键源代码：

```
<!--I've set up WAF to ensure security.-->
<script>
    $('#calc').submit(function(){
        $.ajax({
            url:"calc.php?num="+encodeURIComponent($("#content").val()),
            type:'GET',
            success:function(data){
                $("#result").html(`<div class="alert alert-success">
            <strong>答案:</strong>${data}
            </div>`);
            },
            error:function(){
                alert("这啥?算不来!");
            }
        })
        return false;
    })
</script>
```

这里

	calc.php?num=encodeURIComponent($("#content").val())

`("#content").val()`
获取id为content的HTML标签元素的值,是JQuery

$("#content")
同 `document.getElementById(“content”)`;

$("#content").val()
同 `document.getElementById(“content”).value`;

访问 `calc.php` ,出现了源码，

```
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?> 
```

这里正则匹配，过滤。

但是当传参 `字母` 或 `数字+数字`的时候会 `Forbidden` 。

被waf 拦截了。

绕过方法

## PHP字符串解析特性

构造payload: `calc.php? num=a` 成功输出字母 a .(num前有空格)

* 加入 waf 不允许 num 变量传递字母，可以在 num 前加个空格，这样 waf 就找不到 num 这个变量了，现在的变量 num 前面多了一个空格。但在 php 解析的时候，会先把空格去掉，这样我们的代码还能正常运行，还上传了非法字符。
* 如果字母被过滤，可以用 `char()` 函数转ascii 再拼接。
* php需要将所有参数转换为有效的变量名，因此再解析字符时，他会做两件事： 1. 删除空白符 。 2.将某些字符转换为下划线(包括空格) 【当waf不让你过的时候，php却可以让你过】





参考： https://www.freebuf.com/articles/web/213359.html

https://www.cnblogs.com/chrysanthemum/p/11757363.html

https://blog.csdn.net/a3320315/article/details/102937797