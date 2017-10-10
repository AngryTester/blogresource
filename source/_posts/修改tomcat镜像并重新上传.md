---
title: 修改tomcat镜像并重新上传
date: 2017-02-20 17:58:14
tags: [docker,tomcat]
---

## 使用场景

最近遇到两个问题，需要对已有的镜像进行修改并重新上传。

### 修改tomcat默认设置

有个项目需要用到ojdbc的jar包，需要放到tomcat的lib里，默认的tomcat没有这个jar包，所以需要手动上传至lib然后重新打镜像。

### 修改应用参数

第二个场景略奇葩，有个项目需要做数据库迁移，但是项目的war包和源码都找不到了，唯一能拿到的地方就是现在正在运行的容器和镜像库里对应的镜像。

## 具体操作

官方已经提供了修改运行容器，然后从容器重新打镜像上传的方案。例如，你需要对官方的centos7镜像修改并重新打镜像上传。

### 第一步 进入需要修改的镜像

```
sudo docker run -it centos:7 /bin/sh
```

### 第二步 做相应修改，例如安装软件后退出

```
sudo yum install docker
```

安装完成之后用exit退出。

### 第三步 用docker commit从容器打包镜像

用`docker ps -a`找到第二步退出的目前已为exit状态的容器id。然后执行如下命令重新打镜像。

```
docker commit -m "add docker" -a "angrytester" 容器id centos:7-beta
```

之后用`docker images`就能在本地找到重新打好的镜像。如果希望上传到镜像库，则可使用`docker push centos:7-beta`

## 遇到的坑

### 容器启动一闪而过
如果参考以上操作修改tomcat镜像后重新上传启动容器，会发现tomcat容器无法启动（启动一下就默认退出），这个问题困扰了我整整一天。

原因是因为通过`/bin/sh`进入容器，会将镜像默认的入口覆盖，例如tomcat镜像默认入口是`catalina.sh run`，覆盖之后相当于启动容器时默认执行`/bin/sh`,非长时进程，因此容器启动一下就会自动跳出。

### 解决办法

我想到的有两个解决办法，一个是在启动时重新指定启动入口，如：

```
docker run tomcat:7 catalina.sh run
```

另外一个是重新定义一个Dockerfile，打镜像，在Dockerfile中指定容器入口。


----------

2.21更新

还有一种方法是启动容器之后，通过`docker exec`进入容器进行修改，然后再用`docker commit`打镜像。

这样做的好处是打出来的镜像入口不会被覆盖。