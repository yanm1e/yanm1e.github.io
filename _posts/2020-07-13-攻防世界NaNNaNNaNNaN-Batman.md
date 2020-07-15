---
layout:     post               # 使用的布局（不需要改）
title:      NaNNaNNaNNaN-Batman     # 标题 
subtitle:    攻防世界   #副标题
date:       2020-07-13         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---

下载附件，

```
<script>_='function $(){e=getEleById("c").value;length==16^be0f23233ace98aa$c7be9){tfls_aie}na_h0lnrg{e_0iit\'_ns=[t,n,r,i];for(o=0;o<13;++o){	[0]);.splice(0,1)}}}	\'<input id="c">< onclick=$()>Ok</>\');delete _var ","docu.)match(/"];/)!=null=["	write(s[o%4]buttonif(e.ment';for(Y in $='	')with(_.split($[Y]))_=join(pop());eval(_)</script>
```

发现是乱码。

`eval()` 函数可计算某个字符串，并执行其中的的 JavaScript 代码。

这里我们直接将`eval`改成`alert`,是js代码弹出。

将js代码放到控制台，使它弹出。

![UYHiSU.png](https://s1.ax1x.com/2020/07/13/UYHiSU.png)

```
function $()
{
var e=document.getElementById("c").value;
if(e.length==16)
if(e.match(/^be0f23/)!=null)
if(e.match(/233ac/)!=null)
if(e.match(/e98aa$/)!=null)
if(e.match(/c7be9/)!=null)
{
var t=["fl","s_a","i","e}"];
var n=["a","_h0l","n"];
var r=["g{","e","_0"];
var i=["it'","_","n"];
var s=[t,n,r,i];
for(var o=0;o<13;++o)
{
document.write(s[o%4][0]);
s[o%4].splice(0,1)
}
}
}
document.write('<input id="c"><button onclick=$()>Ok</button>');
delete _
```

虽然js没咋学，但能看懂逻辑就完事了。

一堆`if()`条件之后，出来正菜，咋也不用管那些if语句，直接就当他为真。

`t`,`n`,`r`,`i`为一元数组。

`s`为二元数组。

`splice()` 方法可删除从 index 处开始的零个或多个元素，

然后直接`for`循环，读一个字符串删一个，这样保证二元数组即使为0,也可以都读到数组中的字符，因为读一个删一个，导致后边的数组下标就会变为0，被读到。

按照他的方式组装字符串。
s[0][0],s[1][0],s[2][0],s[3][0],s[0][0],a[1][0]..........


flag{it's_a_h0le_in_0ne}

成功拿到flag。