---
title: Selenium WebDriver+Grid+TestNG+Docker构建并行WebUI自动化测试框架
date: 2017-10-12 09:46:16
tags:  [Grid,并行自动化,TestNG]
---

## 前言

尽管当前业界各路言论都严重不看好WebUI层的自动化测试，但是不管是“质量金字塔”或是“721原则”，尽管优先级较低，UI层的自动化却仍有一席之地。UI层的测试最接近真实用户使用习惯，因此在一些特定场景下，仍发挥着不可替代的作用，最典型的场景就是冒烟测试。冒烟测试的意义在于快速发现质量缺陷，这类缺陷通常是流程性的较严重的功能问题，并且通常会严重影响用户使用。因此在设计冒烟测试用例时，通常会依据实际用户使用习惯，这时候，从前端页面驱动测试就很有必要了。

## 方案介绍

### 1.WebDriver和Grid

WebDriver已经是业界比较通用的UI自动化测试解决方案，相比Selenium1，抛弃了通过执行JS驱动浏览器的方式，而是通过直接调用浏览器的原生态接口来操作。由于得到浏览器厂商的支持，因此稳定性上面比Selenium1提升了不少。WebDriver是典型的“C/S”架构，启动时会现在本地起一个server绑定到指定端口，然后client端通过发送各种指令到server,server解析之后帮助我们达到操作浏览器的目的。也正是由于这种"C/S"的架构，client端支持各种语言的开发。

Grid是Selenium的另外一个产品，主要解决的是分布式测试运行的问题。Grid启动之后，会在指定的远程服务器上启动一个hub，然后所有的WebDriver命令都会通过这个hub转发至注册到这个hub的所有node上，从而达到分布执行的效果。

由于UI自动化测试需要驱动浏览器，浏览器自身的机制可能导致在同一台服务器上的多个浏览器操作互相影响，使用传统的WebDriver多线程并发操作会出现问题，因此需要结合Grid将不同用例分发至不同node运行，以此避免不同用例操作互相影响。

<!-- more -->
### 2.TestNG

TestNG是一个与JUnit类似的测试驱动框架。这里之所以选择TestNG是因为们要使用到它原生支持的并发策略。这也是它比JUnit强大的其中一方面。TestNG并发可以在多个级别,参考官网介绍：



- parallel="methods": TestNG will run all your test methods in separate threads. Dependent methods will also run in separate threads but they will respect the order that you specified.


- parallel="tests": TestNG will run all the methods in the same <test> tag in the same thread, but each <test> tag will be in a separate thread. This allows you to group all your classes that are not thread safe in the same <test> and guarantee they will all run in the same thread while taking advantage of TestNG using as many threads as possible to run your tests.


- parallel="classes": TestNG will run all the methods in the same class in the same thread, but each class will be run in a separate thread.


- parallel="instances": TestNG will run all the methods in the same instance in the same thread, but two methods on two different instances will be running in different threads.


只需要在TestNG的驱动文件testng.xml中加上`parallel="xx"`，并通`thread-count="x"`指定并发线程数，即可以实现并发执行。

### 3.Docker

由于需要多个node供并发运行，node之间的相互隔离主要是通过服务器之间的隔离，如果使用传统隔离方式，我们可能需要多台实体机或虚拟机，这需要耗费大量资源且不好管理，并且由于自动化测试运行环境的依赖众多，环境的初始化也是个大工程。Docker的出现正好解决了这个问题。

官方已经提供了hub和各种浏览器node的镜像：https://github.com/SeleniumHQ/docker-selenium

我们需要做的是在官方镜像的基础上做一些定制化，例如中文的支持,以chrome节点为例，Dockerfile参考如下：


```
FROM selenium/node-chrome-debug

MAINTAINER angrytester <thx_phila@yahoo.com>

USER root

VOLUME /etc/shm

RUN  apt-get update \
    && apt-get -y install ttf-wqy-microhei ttf-wqy-zenhei \
    && apt-get clean
    
USER seluser

```

注意官方的节点镜像分为带debug和不带debug的，区别在于带debug的镜像中自带启动一个vncserver，可通过vncviewer连接到容器查看浏览器运行情况，在重新制作镜像时要注意，必须将USER重新指定为seluser，否则vncserver会无法启动。启动的vncserver默认连接密码为`secret`。

### hub和node的启动

```
docker run -d --name hub -p 4444:4444 hub:1.0 
docker run -d --link hub:hub -p 5900:5900 node:1.0 
docker run -d --link hub:hub -p 5901:5900 node:1.0 
```

## 小结

在当前强调持续快速交付的情况下，快速反馈尤为重要，并行自动化测试实现用空间（服务器资源）换取时间，是持续交付过程中可考虑的实践之一。