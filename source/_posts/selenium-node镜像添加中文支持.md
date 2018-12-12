---
title: selenium node镜像添加中文支持
date: 2017-07-17 13:50:52
tags: [Docker]
---

## Dockerfile

    FROM selenium/node-chrome-debug

	MAINTAINER angrytester <thx_phila@yahoo.com>

	USER root

	RUN apt-get update \
	    && apt-get -y install ttf-wqy-microhei ttf-wqy-zenhei \
	    && apt-get clean
    
	USER seluser

<!-- more -->
## 需要注意的地方

> 最后的USER seluser必须加上,否则VncServer起不来。



