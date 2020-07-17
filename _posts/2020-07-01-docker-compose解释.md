---
layout:     post               # 使用的布局（不需要改）
title:      docker-compose 启动容器  # 标题 
subtitle:       #副标题
date:       2020-07-16         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - 运维
  
---

在做ssrf-lab的时候用到了这个命令。在这里做个小笔记。

## docker-compose 概念

docker-compose 是一个用来把 docker 自动化的东西。

有了 docker-compose 你可以把所有繁复的 docker 操作全都一条命令，自动化的完成。

* Compose 简介

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。


Compose 使用的三个步骤：

* 使用 Dockerfile 定义应用程序的环境。

* 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。

* 最后，执行 docker-compose up 命令来启动并运行整个应用程序。



## 安装

	sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
	chmod +x /usr/local/bin/docker-compose

## Docker Compose 常用命令与配置


* ps：列出所有运行容器

	docker-compose ps

* logs：查看服务日志输出

	docker-compose logs

* port：打印绑定的公共端口，下面命令可以输出 eureka 服务 8761 端口所绑定的公共端口

	docker-compose port eureka 8761

* build：构建或者重新构建服务

	docker-compose build

* start：启动指定服务已存在的容器

	docker-compose start eureka

* stop：停止已运行的服务的容器

	docker-compose stop eureka

* rm：删除指定服务的容器

	docker-compose rm eureka

* up：构建、启动容器

	docker-compose up

* 如果你想在后台执行该服务可以加上 -d 参数：

	docker-compose up -d

* kill：通过发送 SIGKILL 信号来停止指定服务的容器

	docker-compose kill eureka

* pull：下载服务镜像
* scale：设置指定服务运气容器的个数，以 service=num 形式指定

	docker-compose scale user=3 movie=3

* run：在一个服务上执行一个命令

	docker-compose run web bash

## yml 配置指令参考

请看[这里](https://www.runoob.com/docker/docker-compose.html)
