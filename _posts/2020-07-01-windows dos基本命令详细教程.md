---
layout:     post                    # 使用的布局（不需要改）
title:     windows dos基本命令详细教程  # 标题 
subtitle:      #副标题
date:       2020-07-01         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - 网络安全
  
---

DOS命令，计算机术语，是指DOS操作系统的命令，是一种面向磁盘的操作命令，主要包括目录操作类命令、磁盘操作类命令、文件操作类命令和其它命令。

（温馨提示：以下dos命令建议在虚拟机中练习。）

## ***一、如何操作dos命令***

开始 => 运行 => 输入cmd => 回车，

或者直接打开exe软件  C:\windows\system32\cmd.exe

## ***二、基本命令***

#### 1、 color命令，
改变cmd窗口背景及字体颜色。可以输入 color ? 查看帮助，

例如 命令 color 0f  可将cmd窗口背景设置为黑色，将字体颜色设置为亮白色。

#### 2、cls 清屏命令

#### 3、目录相关命令

dir  浏览当前文件夹内容。（带<dir>标识的为文件夹，否则为文件）

dir 指定路径

dir d:\

dir d:\ctf

dir /a    浏览所有内容，包括隐藏内容

c:   d:     f:    切换盘符命令    

cd ..      退出当前目录，进入到上一级目录，但是如果到了某个盘根目录，就退不出了。

cd 文件夹名     进入文件夹

tab键    可补全路径

cd\       直接退到根目录

md 文件夹 [文件夹 文件夹 ...]    新建文件夹，可新建多个

rd 文件夹 [文件夹 文件夹 ...]     删除文件夹，可删除多个

md a\a\a\a\a          新建连续子文件夹

rd a /s       删除非空文件夹，但是会出现提示“是否确认”

rd a /s/q      无提示删除非空文件夹

cd ..\..\b        相对路径进入上一级的上一级的b目录

cd \a\a       绝对路径进入根目录的a目录的下一级a目录

rd \ /s/q     无提示删除本盘符所有文件（虽然出现拒绝访问回显，但是已经删除成功）

#### 4、文件相关命令

创建文件

echo 字符串 >[路径\]文件名.扩展名

ps: > 和 >> 都可以将命令的输出内容输入到某文件中（只输入正确回显），若文件不存在则创建文件。

      > 为覆盖，会将原有内容覆盖重新写入新内容 
      >> 为追加，会换行输入新内容
      该命令实际为   1>        1>>    通常把1省略

例如：echo a 1>>a.txt     将a输入到a.txt

          dir 1>>a.txt        将dir 命令的回显输入到 a.txt
         echo aaa 2>>a.txt    则此命令会在cmd窗口回显 aaa ,而不会将 aaa 输入 a.txt
         rd \ /s/q >>c:\itfd.txt (或者在根目录执行  rd \ /s/q >>c:\itfd.txt) 此命令结果是：cmd窗口内出现回显（ \SYSTEM~1 - 拒绝访问。）[其实回显是报错了，但是命令执行成功了]，同时在c盘创建文件 itfd.txt ,但是内容并没有写入。
         rd \ /s/q 2>>c:\itfd.txt    此命令执行后，cmd窗口不会出现回显，但是内容( \SYSTEM~1 - 拒绝访问。)已经写入itfd.txt并且已经命令执行成功。
       rd \ /s/q 2>>nul    将错误回显丢掉，同时不创建文件，命令执行也成功了。
       rd \ /s/q >nul 2>>nul       不管是正确回显还是错误回显，此命令会执行成功，且丢掉回显。

ps:   >     >>    1>  2>>  为重定向符号。

copy con itfd.txt         复制屏幕内容到itfd.txt. 按回车之后，输入自己想向 itfd.txt中输入的内容，按ctrl+z 回车结束输入。
读取文件

type itfd.txt         将 itfd.txt 中的内容显示到cmd窗口。

type itfd.txt | more       之后按空格翻页显示，按回车逐行显示

dir c:\windows | more

删除文件

del itfd.txt

ps：命令行下删除的文件不会放到回收站，会直接删除。

批量删除文件

利用 * 号通配符。

del *.txt             删除当前目录先所有 txt文件

del *.* /q           无提示删除所有文件，（但不删除目录）【 /q 
为无提示】

del *.* /s/q         无提示删除所有文件包括目录里的文件，只剩下空目录。(此命令会输出回显:删除文件- .... )【 /s逐级删除】

del *.* /s/q >nul 2>nul    无提示无回显删除所有文件

#### 5、复制粘贴剪切操作

复制文件：copy [路径\]源文件全名 目标路径[\新文件全名]

移动文件：move [路径\]源文件全名 目标路径[\新文件全名]

在根目录中创建a目录和b目录，在a目录中创建 a.txt,

复制文件

在a目录下执行      copy a.txt \b\b.txt           将a.txt复制
到 b目录并且重命名为 b.txt

在根目录下执行     copy \a\a.txt \b\b.txt       将a目录下a.txt复制到 b目录并且重命名为 b.txt

copy \a\a.txt \b\             将a目录下a.txt复制到 b目录

在a目录下执行     copy \b\b.txt .                  将b目录下的b.txt复制到本目录

copy \b\b.txt .\c.txt           将b目录下的b.txt复制到本目录并且命名为c.txt

移动文件

将上述命令换成 move 即可。

#### 6、重命名

ren  旧名  新名          (rename)

ren a.txt b.txt         将a.txt 重命名为 b.txt

#### 7、隐藏目录

修改文件或者文件夹隐藏属性

attrib +h 文件全名/文件夹名                        隐藏文件或者文件夹

attrib -h 文件全名/文件夹名                         取消隐藏文件或者文件夹

dir            此命令此时已经看不到被隐藏的文件

dir /a        可以看到被隐藏的文件，（当然也可以在图形化界面文件夹选项设置，让隐藏文件夹展现）同时还有一个系统隐藏文件，上边 rd \ /s/q 报错就是因为一个隐藏的系统文件不让删除，所以报错。

attrib +s +h a       提升为被系统保护的文件，但是此时rd \ /s/q   仍可以删除它。

attrib -s -h a        

![NoyZRI.png](https://s1.ax1x.com/2020/07/01/NoyZRI.png)

#### 8、定时关机或重启

shutdown -s -t 秒                                 定时关机

shutdown -s -f -t 秒                               定时强制关机

shutdown -r -t 秒                                   定时重启，同时加上 -f 为强制重启

shutdown -a                                          取消定时

shutdown -l                                           注销，与 logoff 命令相同

shutdown -s -f -t 秒 -c "你要关机了！"      提示信息
更多命令可 输入 shutdown 回车查看

#### 9、快速生成一个空文件

fsutil file createnew c:\windows\system.ini 409600000        快速在c:\windows目录下生成409600000字节的文件，（目录和大小都可以自己修改）内容是空的但是文件大小不是空的。

attrib +h +s system.ini              将其隐藏，会发现此时磁盘中无缘无故少了好多空间。

#### 10、修改关联

assoc .txt=exefile               此时发现txt文件此时双击已经打不开了，（报错不是可执行程序），但是右键用记事本打开还是可以的。

assoc .txt=txtfile               双击又可以用记事本打开了。
那么如果我们 执行  assoc .exe=txtfile     之后用win+R 打开cmd 发现是以记事本打开的。

之后去C:\Windows\System32  找到cmd.exe ,想着可以右键打开方式以exe 打开，结果很尴尬，没有打开方式这个选项。那岂不是凉凉了。嘿嘿。我这就给出几种修复方法。

第一、重装

第二、有快照的话恢复快照.........

第三、电脑开机后，按F8进入，选择带命令行的安全模式，之后等待cmd窗口出现，输入
assoc .exe=exefile 回车即可，输入 shutdown -r -t 0    重启即可。

![NoynQP.png](https://s1.ax1x.com/2020/07/01/NoynQP.png)