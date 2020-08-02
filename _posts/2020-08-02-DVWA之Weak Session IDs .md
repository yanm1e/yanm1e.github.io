---
layout:     post               # 使用的布局（不需要改）
title:      DVWA之Weak Session IDs    # 标题 
subtitle:    DVWA  #副标题
date:       2020-08-02       # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - DVWA
  
--- 

用户访问服务器的时候，一般服务器都会分配一个身份证 session id 给用户，用于标识。用户拿到 session id 后就会保存到 cookies 上，之后只要拿着 cookies 再访问服务器，服务器就知道你是谁了。
但是 session id 过于简单就会容易被人伪造。根本都不需要知道用户的密码就能访问，用户服务器的内容了。

## Low

```
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id'])) {
        $_SESSION['last_session_id'] = 0;
    }
    $_SESSION['last_session_id']++;
    $cookie_value = $_SESSION['last_session_id'];
    setcookie("dvwaSession", $cookie_value);
}
?> 
```

可以看到，最开始连接时，会将SessionID值设为0，然后随着每次交互递增。这个没啥实际意义，我觉得是在模拟一个有规律的Session值，可以循规律进行对其他用户身份的伪造。

这里也可以使用burp进行分析。

具体看[这里](https://t0data.gitbooks.io/burpsuite/content/chapter10.html)

## medium

```
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = time();
    setcookie("dvwaSession", $cookie_value);
}
?> 
```

这里变成了时间戳，有经验的一眼就看出来了。可使用burp分析。

## High

```
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id_high'])) {
        $_SESSION['last_session_id_high'] = 0;
    }
    $_SESSION['last_session_id_high']++;
    $cookie_value = md5($_SESSION['last_session_id_high']);
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], false, false);
}

?> 
```

跟low级别一样，就是md5加密了一下子。

## impossible

```
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = sha1(mt_rand() . time() . "Impossible");
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], true, true);
}
?> 
```

使用随机数+时间戳+固定字符串（"Impossible"）进行 sha1 运算，作为 session Id，完全就不能猜测到。