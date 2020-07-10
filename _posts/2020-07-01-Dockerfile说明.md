---
layout:     post                    # 使用的布局（不需要改）
title:      Dockerfile镜像原理  # 标题 
subtitle:      #副标题
date:       2020-07-01         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - 运维
  
---

Linux文件系统由bootfs和rootfs两部分组成

 bootfs：包含bootloader（引导加载程序）和 kernel（内核）

 rootfs： root文件系统，包含的就是典型 Linux 系统中的/dev，/proc，/bin，/etc等标准目录和文件

不同的linux发行版，bootfs基本一样，而rootfs不同，如ubuntu，centos等

## 一、Docker 镜像原理

Docker镜像是由特殊的文件系统叠加而成

最底端是 bootfs，并使用宿主机的bootfs 

第二层是 root文件系统rootfs,称为base image

然后再往上可以叠加其他的镜像文件

统一文件系统（Union File System）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。 

一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像成为基础镜像。

当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器

## 二、把容器转为镜像


	docker commit 容器id 镜像名称:版本号
	docker save -o 压缩文件名称 镜像名称:版本号
	docker load –i 压缩文件名称

## 三、Dockerfile 概念


Dockerfile 是一个文本文件

包含了一条条的指令

每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像

对于开发人员：可以为开发团队提供一个完全一致的开发环境

对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了

对于运维人员：在部署时，可以实现应用的无缝移植

Dochub网址：https://hub.docker.com 

可从dockerhub看到编写代码。


| 关键字      | 作用                     | 备注                                                         |
| ----------- | ------------------------ | ------------------------------------------------------------ |
| FROM        | 指定父镜像               | 指定dockerfile基于那个image构建                              |
| MAINTAINER  | 作者信息                 | 用来标明这个dockerfile谁写的                                 |
| LABEL       | 标签                     | 用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看 |
| RUN         | 执行命令                 | 执行一段命令 默认是/bin/sh 格式: RUN command 或者 RUN ["command" , "param1","param2"] |
| CMD         | 容器启动命令             | 提供启动容器时候的默认命令 和ENTRYPOINT配合使用.格式 CMD command param1 param2 或者 CMD ["command" , "param1","param2"] |
| ENTRYPOINT  | 入口                     | 一般在制作一些执行就关闭的容器中会使用                       |
| COPY        | 复制文件                 | build的时候复制文件到image中                                 |
| ADD         | 添加文件                 | build的时候添加文件到image中 不仅仅局限于当前build上下文 可以来源于远程服务 |
| ENV         | 环境变量                 | 指定build时候的环境变量 可以在启动的容器的时候 通过-e覆盖 格式ENV name=value |
| ARG         | 构建参数                 | 构建参数 只在构建的时候使用的参数 如果有ENV 那么ENV的相同名字的值始终覆盖arg的参数 |
| VOLUME      | 定义外部可以挂载的数据卷 | 指定build的image那些目录可以启动的时候挂载到文件系统中 启动容器的时候使用 -v 绑定 格式 VOLUME ["目录"] |
| EXPOSE      | 暴露端口                 | 定义容器运行的时候监听的端口 启动容器的使用-p来绑定暴露端口 格式: EXPOSE 8080 或者 EXPOSE 8080/udp |
| WORKDIR     | 工作目录                 | 指定容器内部的工作目录 如果没有创建则自动创建 如果指定/ 使用的是绝对地址 如果不是/开头那么是在上一条workdir的路径的相对路径 |
| USER        | 指定执行用户             | 指定build或者启动的时候 用户 在RUN CMD ENTRYPONT执行的时候的用户 |
| HEALTHCHECK | 健康检查                 | 指定监测当前容器的健康监测的命令 基本上没用 因为很多时候 应用本身有健康监测机制 |
| ONBUILD     | 触发器                   | 当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后 会执行 ONBUILD的命令 但是不影响当前镜像 用处也不怎么大 |
| STOPSIGNAL  | 发送信号量到宿主机       | 该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出。       |
| SHELL       | 指定执行脚本的shell      | 指定RUN CMD ENTRYPOINT 执行命令的时候 使用的shell            |


## 四、dockerfile案例

自定义centos7镜像。要求：
1. 默认登录路径为 /usr
2. 可以使用vim

实现：

	定义父镜像：FROM centos:7
	定义作者信息：MAINTAINER  itheima <itheima@itcast.cn>
	执行安装vim命令： RUN yum install -y vim
	定义默认的工作目录：WORKDIR /usr
	定义容器启动执行的命令：CMD /bin/bash
	通过dockerfile构建镜像：docker bulid –f dockerfile文件路径 –t 镜像名称:版本





