---
title: Superset提示`fetch data error`
date: 2018-12-24 08:35:54
tags: [Superset]
---

## 使用场景

为了应付各种汇报和考核需要，引入轻量级BI工具生成各个维护的视图，在工具选型阶段简单调研了`Metabase`,`Superset`以及`Knowage`(也就是Spago BI)，最后选择`Superset`是因为自带图表比`Metabase`酷炫且比`Knowage`轻量，足以满足当前需求。

## 问题

由于对数据库不够熟悉，可能是设计不足，数据量大了之后查询结果返回时间过长，导致在视图生成时提示`fetch data error`,查看后台提示发现是因为查询时间超出了`60s`的默认超时控制。

## 解决办法

尝试过通过`superset_config.py`启动时注入环境变量无效，因此查看镜像的`Dockerfile`，发现环境变量在其中定义，因此修改重新制作镜像解决，`Dockerfile`内容如下：

```

```