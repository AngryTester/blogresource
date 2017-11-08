---
title: 查询DNS解析域名对应IP
date: 2017-11-08 08:52:32
tags: [IP，DNS]
---


挺简单一命令，公司搭建了内部DNS服务，经常需要查域名对应IP,使用以下命令即可：

```shell
nslookup test.com xxx.xx.xx.xxx(DNS地址)
```