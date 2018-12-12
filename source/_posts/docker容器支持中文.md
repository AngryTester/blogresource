---
title: docker容器支持中文
date: 2017-04-06 11:28:26
tags: [docker,中文]
---

## 问题

在基底镜像centos:7基础上制作的镜像所起的容器中运行java程序，打出日志中文部分均显示为问号（?）

## 解决方法（网络收集）

在镜像Dockerfile中加上以下内容：


> RUN yum -y install kde-l10n-Chinese && yum -y reinstall glibc-common && localedef -c -f UTF-8 -i zh_CN zh_CN.utf8

> ENV LC_ALL zh_CN.utf8


<!-- more -->