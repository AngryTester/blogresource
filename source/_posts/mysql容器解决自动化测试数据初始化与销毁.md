---
title: mysql容器解决自动化测试数据初始化与销毁
date: 2019-05-20 08:54:38
tags: [docker]
---

## 使用场景

在自动化测试过程中可能遇到无法通过被测系统自身提供的功能对自动化测试产生的数据进行销毁的情况，通过直接执行DDL销毁数据在数据关系比较复杂的情况下可能带来未知异常，因此可以考虑使用容器作为数据源，通过容器的起停来达成测试数据自动初始化和销毁的目的。

## 镜像制作流程

- 启动mysql容器并将数据目录映射至宿主机

```
docker run --name mysqldock -e MYSQL_ROOT_PASSWORD=123456 -d -p 3306:3066 -v /data:/var/lib/mysql mysql
```
<!-- more -->
- 进入容器允许mysql外部连接

```
docker exec -it mysqldocker /bin/bash
mysql -uroot -p123456
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION; 
```
- 使用数据库连接工具连接上一步启动的数据库容器新建数据库并初始化数据

- 制作Dockerfile并制作镜像

```
FROM mysql:latest

COPY data /var/lib/mysql
```

> `data`文件夹为上面映射至宿主机的数据文件目录

## 注意事项

mysql容器启动时会自动将数据文件主目录映射到宿主机上，`docker commit`无法将默认挂载的文件或目录打到镜像中，因此如果操作数据库添加数据之后直接使用`docker commit`制作镜像，添加的数据将不复存在,需要使用先将数据文件挂载到宿主机上，然后通过`Dockerfile`复制进去。

