---
title: docker的启动参数配置
date: 2018-05-17 08:57:16
tags: [Docker]
---

以centos系统为例，启动参数涉及的文件主要为：`/etc/docker/daemon.json`和`/etc/systemd/system/docker.service.d/http-proxy.conf`.

`daemon.json`涉及完整参数可参考官方文档，介绍几个核心配置（自己用的比较多的）：

- "registry-mirrors":["http://aad0405c.m.daocloud.io"]-镜像加速地址，若有多个，可用逗号分割

- "bip":"172.172.172.1/24"-bridge ip区间，即docker容器默认会取172.172.172.1/24这个区间的ip，由于默认区间段和代理有冲突，所以需要自定义

- "insecure-registries":["http://aad0405c.m.daocloud.io"]-镜像仓库地址，若有多个，可用逗号分割

`http-proxy.conf`为代理设置，参见内容参考如下：
<!-- more -->
```
[Service]
Environment="HTTP_PROXY=http://172.17.1.1:8080" "HTTPS_PROXY=http://172.17.1.1:8080" "NO_PROXY=172.26.x.x,172.26.x.x"
```
修改参数以后，执行以下命令生效。

```
systemctl daemon-reload
systemctl restart docker
```

