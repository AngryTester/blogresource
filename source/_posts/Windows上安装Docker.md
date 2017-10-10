---
title: Windows上安装Docker
date: 2017-05-05 02:43:05
tags: [Docker On Windows]
---

## 官方已有文档，why折腾

公司内网，与default配置有区别。

## Windows机器配置

> 操作系统:64位Windows 7专业版

> 内存：4GB

## 安装Docker Toolbox

> 下载：ftp://ftp.cqrd.com/upload/DockerToolbox.exe

> 傻瓜式一步一步安装。

> 注意事项：安装路径尽量别有中文。

## Docker启动配置

> 将安装路径下的boot2docker.iso文件复制到C:\Users\你的用户\.docker\machine\cache目录下。另外修改桌面上多出来的Docker Quickstart Terminal，右键属性-目标，修改前面的bash.exe为你之前安装的Git目录下的bash.exe。

## 使用Docker-Machine创建含Docker运行环境的虚拟机

参考如下脚本创建：

```
docker-machine create -d virtualbox ^
--virtualbox-host-dns-resolver ^
--engine-env HTTP_PROXY=http://XXX.XX.XX.XX:XXXX ^
--engine-env HTTPS_PROXY=http://XXX.XX.XX.XX:XXXX ^
--engine-env NO_PROXY=xx.com,.xx.com ^
--engine-opt bip=172.172.172.1/24 ^
--engine-opt dns=xxx.xxx.xx.xx  ^
--engine-registry-mirror https://xxx.xxx.xx ^
--engine-insecure-registry xx.com ^
--virtualbox-disk-size "20000" ^
default
```

> 你也可以将以上脚本保存成bat文件，双击运行。

## 使用连接工具连接虚拟机

> 前面的步骤实现了创建一个含Docker运行环境的虚拟机，你也可以通过virtualbox管理界面看到多了一个叫default的虚拟机。默认生成的defalt的ip为`192.168.99.100`,通过SSH连接工具你可以连接这个虚拟机，默认用户名密码为`docker/tcuser`。


## 关闭虚拟机和启动虚拟机

> 虚拟机的第一次启动会比较慢，之后每次启动都会比较快，和咱们平时创建虚拟机是一样的。

### 关闭虚拟机

> 运行如下bat命令：

```
docker-machine kill default
```

### 启动虚拟机

> 方法一：

> 双击桌面上的Docker Quickstart Terminal。

> 方法二：

> 运行如下bat命令：

```
docker-machine start default
```

