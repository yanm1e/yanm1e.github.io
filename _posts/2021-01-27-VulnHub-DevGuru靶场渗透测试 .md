---
layout:  post    # 使用的布局（不需要改）
title:   VulnHub-DevGuru靶场渗透测试   # 标题 
subtitle:    #副标题
date:  2021-01-27  # 时间
author:  yanmie    # 作者
header-img: img/.jpg ##标签这篇文章标题背景图片
catalog: true      # 是否归档
tags:        
    - Vulnhub


---

靶机信息：

* **Name**: DevGuru: 1
* **Date release**: 7 Dec 2020
* **download:**  [https://www.vulnhub.com/entry/devguru-1,620/](https://www.vulnhub.com/entry/devguru-1,620/)
* **description**:  DevGuru is a fictional web development company hiring you for a pentest assessment. You have been tasked with finding vulnerabilities on their corporate website and obtaining root.

## 一、信息收集

####  1.1主机发现

```bash
nmap -sP 192.168.56.1/24
```

![image-20210127135544046](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127135544046.png)

#### 1.2端口探测

```bash
nmap -sT -Sv -T4 -p- 192.168.56.106

➜  ~ nmap -sT -sV -T4 -p- 192.168.56.106
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-27 14:15 CST
Nmap scan report for localhost (192.168.56.106)
Host is up (0.060s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  ssl/http Apache/2.4.29 (Ubuntu)
8585/tcp open  unknown

```

#### 1.3服务探测

22端口是ssh服务，

访问一下 80 http服务，

![image-20210127141818029](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127141818029.png)

信息收集一波，

october cms ，apache , php ,

有 git 泄露，

![image-20210127142157729](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127142157729.png)

## 二、漏洞利用

利用 [githack](https://github.com/BugScanTeam/GitHack.git) 下载源码，

```bash
python2 GitHack.py http://192.168.56.106/.git/
```

![image-20210127142832303](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127142832303.png)

目录下有`adminer.php` 

* adminer 是一个超轻量的MySQL管理工具，除了支持MySQL外，完整版本支持几乎所有的常见数据库。全部功能，只有一个PHP文件。几百K的体积，就有如此强大的功能。完全可以媲美phpMyAdmin.

那就找配置文件找数据库密码，成功找到，并登录

![image-20210127144116596](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127144116596.png)

发现账号密码，

```
$2y$10$hnhKQ8hTe9b3SoZgXhBuT.HG17VvEdBXe86hEq1qdIknkcM1rbxYi
```

但是这是 md5二次加密并加盐后的密码，并不能解密，但是既然我们已经控制了其数据库，那就可以直接修改数据了。改成

```
$2y$10$EJ64ugc3YEnGH2jaM06XCO68igbTx4LpkcfVPnzoJHRy8Wm8h0Hti
# 明文是 123456
# https://www.qumuban.com/7550.html
```

cms 后台

```
http://192.168.56.106/backend/backend/auth/signin
```

登录   frank/123456 ，成功登录。

找到一个上传头像的地方，白名单验证，无法利用。





8585未知，访问一下，是个 git 服务托管平台

![image-20210127140420691](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127140420691.png)

