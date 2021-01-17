### About Release

* **Name**: y0usef: 1
* **Date release**: 10 Dec 2020
* downpload：[https://www.vulnhub.com/entry/y0usef-1,624/](https://www.vulnhub.com/entry)
* Get two flag
* Difficulty : easy

## 一、环境搭建

使用 `virtual Box` 搭建靶机。

win10 攻击机，使用 kali 作为辅助攻击机。

但是 我 kali 是在 vmware 上安装的，所以需要配置一下网络。
virtual box：
![图片.png](https://image.3001.net/images/20210115/1610687486_600123fe9a7b0be801114.png!small)
选用仅主机模式，在注意一下下边的 `界面名称`。

配置 VM ware
我在保留原有设置的基础上，在添加一块网卡 VMnet3，将其桥街到 virtualBOX 选用的那块网卡上，这样就可以在一个网络上了。
![图片.png](https://image.3001.net/images/20210115/1610687684_600124c4c08ff3201bee2.png!small)
配置完成，开始操作。

## 二、信息收集

![图片.png](https://image.3001.net/images/20210115/1610687866_6001257a9b3ff11328c24.png!small)

在这里先普及一下 nmap 主机发现的基础知识。

* -PN nmap进行其他扫描之前都会对目标进行一个ping扫描。如果目标对ping 扫描无反应将结束整个扫描过程。这种方法可以跳过那些没有响应的主机，从而节省大量时间，但如果目标在线只是采用某种手段屏蔽了ping 扫描，从而躲过我们的其他扫描操作，我们可以指定无论目标是否响应ping 扫描，都要将整个扫描过程完整的参数呈现出来；如 nmap -PN 192.168.56.1/24。
* -P\* 选项(用于选择 ping的类型)可以被结合使用。 `PS/PA/PU/PY[portlist]: TCP SYN/ACK, UDP or SCTP discovery to given ports`
* -PR 使用ARP协议发现主机。如： `nmap -PR 192.168.56.1/24`
* -sP 使用 ping 协议发现主机。如：`nmap -sP 192.168.56.1/24`
* -sS 半开放扫描，使用 TCP 协议发现主机. 如：`nmap -sS 192.168.56.1/24`
* -sT 全开放扫描，使用 TCP 协议发现主机。如：`nmap -sT 192.168.56.1/24`
* -sU 使用 UDP 协议发现主机。如：`nmap -sU 192.168.56.1/24`

我们使用命令 `nmap -sP -T5 192.168.56.1/24` 来进行 ping 主机发现。
![图片.png](https://image.3001.net/images/20210115/1610688691_600128b32e2b5f46ffadb.png!small)

#### 1\. 目标主机发现

先发现目标主机
然后进行端口扫描。
依旧使用 nmap 神器，再来看看看 nmap 端口扫描的基础知识：
nmap 端口扫描分成六个状态:

* open(开放的)
* closed(关闭的)
* filtered(被过滤的)
* unfiltered(未被过滤的)
* open\|filtered\(开放或者被过滤的\)
* closed\|filtered\(关闭或者被过滤的\)

#### 2\. 端口扫描

常见端口扫描选项：

* -sS TCP SYN 扫描，快速，不建立 TCP 连接，所以不易被发现。如 `nmap -sS 192.168.56.101`
* -sT TCP connect 扫描，较慢，三次握手建立 TCP 连接，容易被发现。如：`nmap -sT 192.168.56.101`
* -sA TCP ACK 扫描，用于发现防火墙规则，确定它们是有状态的还是无状态的，哪些端口是被过滤的。 如： `nmap -sA 192.168.56.101`
* -sU UDP 扫描，较快。如： `namp -sU 192.168.56.101`
* -p (只扫描指定的端口) 。 默认情况下，Nmap用指定的协议对端口1到1024以及nmap-services 文件中列出的更高的端口在扫描。如： `nmap -p 1-65535 192.168.56.101`
* -p- 从端口1扫描到65535。
* -T 设置延时，一般使用 `-T4`

在这里我们使用命令 `nmap -sY -T4 -p- 192.168.56.101`
![图片.png](https://image.3001.net/images/20210115/1610689911_60012d77226cbf99b5fdb.png!small)
显示所有端口都被过滤。。。
使用 TCP 进行扫描
![图片.png](https://image.3001.net/images/20210115/1610689948_60012d9c119f7cd565163.png!small)
发现 80 端口和 22 端口开放。在渗透测试过程中，单单经考单一的方法是不够的，需要多种方法结合起来才能展现最佳效果。
#### 3. 服务探测
这里我们直接先开始探测 靶机 80 端口。
这里也可以使用 `nmap` 进行服务版本探测
![图片.png](https://image.3001.net/images/20210115/1610690443_60012f8b18028a6a881eb.png!small)
可以看到SSH 版本以及 apache 版本。
访问网站 `http://192.168.56.101/`
![图片.png](https://image.3001.net/images/20210115/1610690504_60012fc8c8635ff94abc2.png!small)
php 站，（此插件为 Wappalyzer）
![图片.png](https://image.3001.net/images/20210115/1610690562_600130025f2cd393c4401.png!small)
访问发现一句话，此网站正在建设中，正在运行中。

#### 4. 目录扫描

![image-20210115140906003](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210115140906003.png)

发现有个目录比较特殊，访问一下，

![image-20210115141045878](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210115141045878.png)

确实很特殊，应为这个是代码限制的访问的。

ok,这应该是个管理目录，既然是代码限制，所以尝试 XFF 伪造 `127.0.0.1` ,

![image-20210115141213091](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210115141213091.png)

成功伪造。（此插件为 `X-Forwarded-For Header`）

## 三、操作

`We'll never share your username with anyone else.`

右键源代码也啥也没发现，

手动进行弱密码测试直接解开，

`admin/admin` ,省去了 burp 爆破的功夫。

进入后台，发现一处可以上传文件的操作

![image-20210115141459502](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210115141459502.png)

直接上传 php 马被限制，那就进行绕过。

修改`Content-Type` 即可绕过。 

![image-20210115141615387](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210115141615387.png)

并且返回了路径。

蚁剑上手。

![image-20210115141906389](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210115141906389.png)

想去 root 目录看看但是没权限。

在 `/home/` 目录下发现 `user.txt`

打开查看是：

```txt
c3NoIDogCnVzZXIgOiB5b3VzZWYgCnBhc3MgOiB5b3VzZWYxMjM=
```

burp base64解码得到 ssh 账号密码。

```txt
ssh : 
user : yousef 
pass : yousef123
```

正好靶机开放 22 端口，我们进行连接。

#### 提权

最简单的 前面加 `sudo`

![image-20210115142958283](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210115142958283.png)

还可以 `sudo su ` 输入当前用户`yousef` 密码即可成为root.

又得到一串 base64 字符串

```txt
You've got the root Congratulations any feedback content me twitter @y0usef_11
```

结束。

## 四、总结

主机发现 ---> 端口扫描 ---> 服务探测 ---> 目录扫描 ---> 伪造IP ---> 提权