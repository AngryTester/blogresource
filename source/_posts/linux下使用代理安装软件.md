---
title: linux下使用代理安装软件
date: 2017-03-24 12:54:45
tags: [linux,yum,apt-get]
---

## yum

`vi /etc/yum.conf`

添加以下这行内容：http.proxy=http://127.0.0.1:8080

## apt-get

新建一个文件任意命名如

*conf*

文件内容如下：

	Acquire::http::proxy "http://127.0.0.1:8000/";
	Acquire::ftp::proxy "ftp://127.0.0.1:8000/";
	Acquire::https::proxy "https://127.0.0.1:8000/";
<!-- more -->
下载时使用：

`sudo apt-get -c conf`

