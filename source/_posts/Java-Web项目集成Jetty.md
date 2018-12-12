---
title: Java Web项目集成Jetty
date: 2018-07-03 14:02:34
tags: [Jetty]
---

## 坑

最近花好多时间在[inproctester](https://github.com/aharin/inproctester)这个项目上,暂时还是没有很大的进展，挫败感强烈。虽然进展不多，但过程中还是接触了一些网上貌似不太好查到的新东西，记录一下不枉费这段时间的心血。

## Jetty是什么

可以内嵌到应用里的中间件，作用与`tomcat`或`jboss`类似，优点是不需要安装，可直接通过代码内嵌进应用，然后直接和直接应用程序一样执行。

## 配置过程中可能存在的问题

- Jetty对EL表达式的支持不好，导致页面报500无法访问，主要是针对EL表达式中的三目运算符，冒号前后的选项必须空格。

- Jetty运行应用时，通过`servletContext.getRealPath("/")`相比其他中间件获取到的路径会少一个`/`，导致某些资源访问可能存在问题。

<!-- more -->
- 出现JSP页面无法编译的报错，需加入如下依赖：

```xml
<dependency>
    <groupId>org.mortbay.jetty</groupId>
    <artifactId>jsp-2.1-glassfish</artifactId>
    <version>9.1.1.B60.25.p2</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```

- 其他的网上都能查到，我靠，花这么多时间就整出这么一点东西！！！



