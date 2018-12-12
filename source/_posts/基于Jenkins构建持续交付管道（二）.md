---
title: 基于Jenkins构建持续交付管道（二）
date: 2017-02-27 16:16:33
tags: [Jenkins,Pipeline,持续交付]
---

## 前言

本系统旨在从零基础开始讲解如何基于Jenkins搭建持续交付管道，因此本系列前面几篇会比较基础。本章结合实例讲解Jenkins的基础用法。

## 安装插件

上一章中提到了在第一次启动过程中选择插件安装，那之后的使用过程中如果需要用到其他插件呢？

具体操作是：进入`系统管理>管理插件`，点击“可选插件标签”，可看到所有可用插件，可通过右上角输入框进行过滤：

例如需要安装maven插件，可在输入框中输入"maven",可看到可用插件被过滤到只剩几个。选择Maven Integration plugin，点击直接安装。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%BA%8C%EF%BC%89/4.png)

<!-- more -->
>部分插件可能需要重启服务才能生效。


## 工具设置

之前第一章中已经说到，Jenkins主要依赖服务器上的插件完成构建任务，所以在安装完插件之后，还需要在Jenkins中进行设置，将服务器上的相关工具做关联。

具体操作是：进入`系统管理>Global Tool Configuration`，可看到下图：

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%BA%8C%EF%BC%89/1.png)

### 新增JDK

点击“新增JDK”按钮，取消“自动安装”选项，输入别名（可自定义），JAVA_HOME输入服务器上的JDK地址。如图：

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%BA%8C%EF%BC%89/2.png)

可点击“新增JDK”增加更多其他版本JDK（例如一个服务器上存在多个JDK的情况）。

### 新增MAVEN

同新增JDK，如图：

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%BA%8C%EF%BC%89/3.png)

>如果安装了其他插件如Git，也参考以上步骤进行设置。

## 邮件设置

在Job构建结束后可能需要通过邮件通知到构建负责人结果，所以需要进行邮件设置。Jenkins提供了邮件服务，但需要进行邮件服务器的相关配置。邮件服务依赖邮件相关插件，推荐Email Extension Plugin。安装完插件后，还需要进行相关配置。

具体操作是：进入`系统管理>系统设置`，可看到下图：

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%BA%8C%EF%BC%89/5.png)

点击高级，勾选“Use SMTP Authentication”，配置SMTP服务器以及用户名密码，即使用Jenkins的邮件服务了。

>注意，配置完邮件服务器相关信息后，需要配置系统管理员邮件地址，格式为：address not configured yet <nobody@nowhere>，例如Jenkins <12345678@qq.com>,其中12345678@qq.com必须为之前配置的邮件服务器对应的邮箱地址。

## 新建一个maven项目

在安装了MAVEN插件之后，新建Job选项就会出现新建一个Maven项目。

![](https://raw.githubusercontent.com/AngryTester/blog/master/%E5%9F%BA%E4%BA%8EJenkins%E6%9E%84%E5%BB%BA%E6%8C%81%E7%BB%AD%E4%BA%A4%E4%BB%98%E7%AE%A1%E9%81%93%EF%BC%88%E4%BA%8C%EF%BC%89/6.png)

>注意，在Maven项目的Goals and options中，不需要输入mvn，例如执行单元测试，只需要输入clean test。

## 配置邮件通知

在安装了相关插件并进行了相关配置之后，可以在Job中使用邮件通知。

具体操作是在`Job设置>构建后操作`，选择“Editable Email Notification”。

- Project Recipient List：如果不修改，默认是读取系统设置中的收件人列表，也可修改为aa@xx.com,bb@xx.com

- Default Subject与Default Content：同上。

- Attachments：如果需要在邮件中附上附件，可在这个选项中配置。默认取相对路径。例如在工作空间子目录aa下有一个1.txt，则这里配置aa/1.txt

- advanced setting-Triggers：设置触发条件和发送对象。












