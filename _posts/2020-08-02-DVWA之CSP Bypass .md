---
layout:     post               # 使用的布局（不需要改）
title:      DVWA之CSP Bypass    # 标题 
subtitle:    DVWA  #副标题
date:       2020-08-02       # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - DVWA
  
--- 

[CSP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)就是浏览器的安全策略，如果 标签，或者是服务器中返回 HTTP 头中有 `Content-Security-Policy` 标签 ，浏览器会根据标签里面的内容，判断哪些资源可以加载或执行.

内容安全策略（CSP）是一种声明机制，允许Web开发者在其应用程序上指定多个安全限制，由支持的用户代理（浏览器）来负责强制执行。CSP旨在“作为开发人员可以使用的工具，以各种方式保护其应用程序，减轻内容注入漏洞的风险和减少它们的应用程序执行的特权”。

## lOW

```
<?php

$headerCSP = "Content-Security-Policy: script-src 'self' https://pastebin.com  example.com code.jquery.com https://ssl.google-analytics.com ;"; // allows js from self, pastebin.com, jquery and google analytics.

header($headerCSP);

# https://pastebin.com/raw/R570EE00

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    <script src='" . $_POST['include'] . "'></script>
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>You can include scripts from external sources, examine the Content Security Policy and enter a URL to include here:</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';
```

浏览器F12也可以看到允许加载js代码的网站

	"Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";

![at4c9A.png](https://s1.ax1x.com/2020/08/02/at4c9A.png)

在这里`https://pastebin.com/`创建一个

![at4IAg.png](https://s1.ax1x.com/2020/08/02/at4IAg.png)

将生成的连接输入`https://pastebin.com/raw/F3i5EgnP`

即可触发xss弹框。

在pastebin上保存的js代码被执行了。那就是因为pastebin网站是被信任的。攻击者可以把恶意代码保存在收信任的网站上，然后把链接发送给用户点击，实现注入。

## Medium

```
<?php

$headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";

header($headerCSP);

// Disable XSS protections so that inline alert boxes will work
header ("X-XSS-Protection: 0");

# <script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert(1)</script>

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>Whatever you enter here gets dropped directly into the page, see if you can get an alert box to pop up.</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';
```


http头信息中的script-src的合法来源发生了变化，说明如下

    unsafe-inline，允许使用内联资源，如内联<script>元素，javascript:URL，内联事件处理程序（如onclick）和内联< style>元素。必须包括单引号。
    nonce-source，仅允许特定的内联脚本块，nonce=“TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA”

可以直接输入以下代码
	<script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert(/xss/)</script>

## High

```
<?php
$headerCSP = "Content-Security-Policy: script-src 'self';";

header($headerCSP);

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>The page makes a call to ' . DVWA_WEB_PAGE_TO_ROOT . '/vulnerabilities/csp/source/jsonp.php to load some code. Modify that page to run your own code.</p>
    <p>1+2+3+4+5=<span id="answer"></span></p>
    <input type="button" id="solve" value="Solve the sum" />
</form>

<script src="source/high.js"></script>
';

```

这个级别已经没有输入框了, 不过题目已经给了足够多的提示. 首先先看一下 CSP 头, 发现只有 script-src 'self';, 看来只允许本界面加载的 javascript 执行. 然后研究了一下这个点击显示答案的逻辑(逻辑在 source/high.js里), 大致如下: 点击按钮 -> js 生成一个 script 标签(src 指向 source/jsonp.php?callback=solveNum), 并把它加入到 DOM 中 -> js 中定义了一个 solveNum 的函数 -> 因此 script 标签会把远程加载的 solveSum({"answer":"15"}) 当作 js 代码执行, 而这个形式正好就是调用了 solveSum 函数, 然后这个函数就会在界面适当的位置写入答案.

一般是没办法修改服务器上的 jsonp.php 文件的，但是在查看服务端源码的时，可以看到下面代码:

```
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
" . $_POST['include'] . "
";
}
```

这里还是会接收 include 参数, 这可以作为一个注入点,可以构造 Payload: 

post传参 `include=<script src="source/jsonp.php?callback=alert('cdd');"></script>`.

## Impossible

```
<?php

$headerCSP = "Content-Security-Policy: script-src 'self';";

header($headerCSP);

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>Unlike the high level, this does a JSONP call but does not use a callback, instead it hardcodes the function to call.</p><p>The CSP settings only allow external JavaScript on the local server and no inline code.</p>
    <p>1+2+3+4+5=<span id="answer"></span></p>
    <input type="button" id="solve" value="Solve the sum" />
</form>

<script src="source/impossible.js"></script>
';
```

该级别主要还是修复了 callback 参数可被控制问题，无法进行攻击。