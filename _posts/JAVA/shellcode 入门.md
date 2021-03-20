# shellcode 入门

## 一、什么是shellcode

#### 1.1 简介

shellcode是一段用于利用软件漏洞而执行的代码，shellcode为16进制的机器码，因为经常让攻击者获得shell而得名。shellcode常常使用机器语言编写。 可在暂存器eip溢出后，塞入一段可让CPU执行的shellcode机器码，让电脑可以执行攻击者的任意指令。

* 独立运行，无需任何文件格式的包装
* 内存中运行，无需固定指定的宿主进程

#### 1.2 shellcode 基本特点

* 短小精悍
* 灵活多变

#### 1.3 shellcode 生成方法

* 编程语言编写
* shelllcode 生成器

## 二、环境

VS2019

win32控制台应用程序

#### 2.1 Resealse

 vs中的程序有debug和release两个版本。

Debug通常称为调试版本，通过一系列编译选项的配合，编译的结果通常包含调试信息，而且不做任何优化，以为开发 人员提供强大的应用程序调试能力。

Release通常称为发布版本，是为用户使用的，一般客户不允许在发布版本上进行调试。所以不保存调试信  息，同时，它往往进行了各种优化，以期达到代码最小和速度最优。为用户的使用提供便利。

我们选择 Release.

![image-20210207174819995](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207174819995.png)

关闭安全检查

![image-20210207175248826](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207175248826.png)



代码

```cpp
int main(){
	return 0;
}
```

编译生成 exe 文件，大小为 9 KB，

![image-20210207175405401](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207175405401.png)

我们写的代码是封装在 PE 文件中执行的，所以我们代码啥功能都没有但是要把PE文件该有的格式填充，所以这么大空间。

#### 2.2 去掉VS自动添加的函数

使用 IDA 反编译，左边有很多函数，都是 VS 为生成的 exe 自动添加的函数。

![image-20210207180039148](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207180039148.png)

在 VS 项目属性修改函数入口点

![image-20210207180220193](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207180220193.png)

修改代码：

```cpp
int StartMain(){
	return 0;
}
```

重新生成程序。

大小变为 2.5KB

![image-20210207180328633](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207180328633.png)

使用 IDA 打开，只有一个函数了，因为关闭了前面的缓冲区安全检查，所以只剩下一个函数了，如果前面没关掉会有两个函数。

![image-20210207180434954](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207180434954.png)

#### 2.3 设置项目工程兼容winXP

下载相关组件，

![image-20210207181206189](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207181206189.png)

![image-20210207181141244](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207181141244.png)

设置平台工具集：

![image-20210207181445130](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207181445130.png)

修改代码生成，运行库

* debug 版本为 MTD
* release 版本为 MT

![image-20210207181602267](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207181602267.png)

此时重新生成的 exe 文件即可兼容 winXP了。

#### 2.4 关闭生成清单

三个区段

代码段，只读数据段，资源段。

![image-20210207182409345](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207182409345.png)

这里 rsrc 是 VS 默认添加的清单数据。

关掉生成清单

![image-20210207182523402](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207182523402.png)

大小变为 1.5KB

![image-20210207182553972](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207182553972.png)

可以看到第三个代码段已经消失，

![image-20210207182658223](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207182658223.png)

#### 2.5 关闭调式信息

可以去除可读数据段。

VS2008 可去掉，vs2019测试没去掉。

![image-20210207182919406](https://gitee.com/luo_fan_1/yanmie-art/raw/master/img/image-20210207182919406.png)

